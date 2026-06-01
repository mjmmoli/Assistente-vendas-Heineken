# Especificação Técnica — Montagem e Injeção do `<contexto_pdv>`

**Assistente de Vendas Heineken — Camada de Integração ERP/CRM**

Este documento descreve como o backend deve montar, preencher e injetar o bloco `<contexto_pdv>` que alimenta o assistente no início de cada visita. Destina-se ao time de desenvolvimento.

---

## 1. Visão geral do fluxo

O assistente é um LLM sem memória entre chamadas. Todo o conhecimento sobre o PDV precisa ser **montado pelo backend a cada visita** e injetado no prompt. O fluxo é:

1. Representante abre o cliente na rota (app de força de vendas).
2. Backend recebe `pdv_id` + `representante_id`.
3. Backend consulta as fontes (ERP, CRM, trade, financeiro), monta o bloco `<contexto_pdv>`, aplica as regras de disponibilidade e mascaramento.
4. Backend injeta o bloco no prompt do assistente, junto com qualquer observação ao vivo digitada/falada pelo representante.
5. Assistente responde nas seções A–H.

Regra de ouro da integração: **o backend é responsável por nunca enviar dado errado.** Campo que não pôde ser validado vai como `indisponível`, nunca com palpite. O prompt já instrui o assistente a tratar `indisponível` como "confirmar na visita".

---

## 2. Mapeamento campo a campo

Para cada campo: a fonte sugerida, a regra de preenchimento e quando marcar `indisponível`.

### Identificação e perfil

| Campo | Fonte | Regra | Marcar `indisponível` quando |
|---|---|---|---|
| `pdv_nome` | Cadastro de clientes (ERP) | Razão/nome fantasia | Nunca (campo obrigatório de cadastro) |
| `pdv_tipo` | Cadastro / classificação comercial | Enum: bar, restaurante, mercado, conveniência, adega, eventos | Tipo não classificado → `indisponível` |
| `perfil_publico` | CRM / segmentação de trade | Texto curto da segmentação | Sem segmentação registrada |
| `fluxo` | CRM / notas de visita | Dias/horários de pico | Sem registro |

### Histórico e mix (núcleo da personalização)

| Campo | Fonte | Regra | Marcar `indisponível` quando |
|---|---|---|---|
| `historico_pedidos` | ERP — pedidos faturados | **Últimos 3–6 pedidos com data, produto, formato e volume.** Não agregar; manter a linha do tempo | Sem histórico (cliente novo) → enviar `cliente novo, sem histórico` |
| `mix_atual` | ERP — itens faturados recorrentes | Marcas/formatos comprados nos últimos ~90 dias | Sem compras no período |
| `mix_heineken_atual` | ERP — filtro SKU Heineken | O que de Heineken ele compra hoje, ou `nenhum` | — (sempre derivável; default `nenhum`) |
| `tendencia_volume` | Cálculo sobre histórico | Regra abaixo (seção 3) | — (sempre calculável) |

> **Importante:** o teste mostrou que a frase "parou em outubro" só foi possível porque o histórico veio com **datas por pedido**. Não substituir por um total agregado — a linha do tempo é o que gera personalização.

### Refrigeração e ativos de trade

| Campo | Fonte | Regra | Marcar `indisponível` quando |
|---|---|---|---|
| `espaco_refrigeracao` | Trade / última visita | Capacidade e estado conhecido | Sem registro de execução |
| `ativos_trade` | Sistema de comodato/trade | Lista de ativos instalados (cooler, chopeira, taças, material) | Sem registro → `indisponível` (não assumir que não tem) |

> O estado *real* do ativo (ex.: "cooler abastecido com concorrente") raramente está no sistema — vem da observação ao vivo do representante. O backend envia o que sabe; o prompt já prioriza a observação ao vivo em caso de conflito.

### Comercial e financeiro (DADOS SENSÍVEIS — ver seção 5)

| Campo | Fonte | Regra | Marcar `indisponível` quando |
|---|---|---|---|
| `preco_tabela` | Motor de preços | **Só a faixa/política vigente e válida para este PDV** | Tabela não resolvida para o PDV/data → `indisponível` (NUNCA chutar) |
| `desconto_permitido` | Política comercial / alçada | Limite autorizado **para este representante e este PDV** | Sem regra resolvida → `0%` explícito, não `indisponível` |
| `promocao_vigente` | Trade / campanhas | Campanha ativa hoje, com condição | Nenhuma ativa → `nenhuma` |
| `pendencia_financeira` | Financeiro/contas a receber | `em dia` / `pendência de R$X` / `bloqueado` | Status não retornado → tratar conservador: `verificar antes de faturar` |

### Indústria e relacionamento

| Campo | Fonte | Regra | Marcar `indisponível` quando |
|---|---|---|---|
| `metas_industria` | Trade / diretrizes Heineken | Mix obrigatório, regras de loja perfeita, ativações vigentes | Sem diretrizes carregadas |
| `objecao_recente` | CRM — última visita | Última objeção registrada (texto + data) | Sem registro |

---

## 3. Regras de cálculo derivado

**`tendencia_volume`** — calcular, não armazenar manualmente:

- `crescendo`: volume médio dos últimos 2 meses > média dos 3 meses anteriores.
- `estável`: variação dentro de ±15%.
- `caindo`: queda > 15% sustentada.
- `inativo há X dias/meses`: sem pedido do SKU/categoria relevante no período. **Calcular separadamente para Heineken e para volume geral**, porque o PDV pode estar ativo no geral e inativo em Heineken (foi o caso do teste — disparou o modo Recuperação correto).

Recomendação: enviar as duas leituras, ex.: `Heineken inativo há ~7 meses; volume geral estável`.

---

## 4. Regras de disponibilidade ("indisponível" vs. valor padrão)

Distinção crítica para não confundir o assistente:

- **`indisponível`** = "o sistema não conseguiu resolver este dado; representante deve confirmar". O assistente pergunta na seção C.
- **Valor padrão explícito** (`nenhum`, `0%`, `em dia`) = "o sistema sabe e a resposta é essa". O assistente NÃO pergunta.

Erro a evitar: mandar `desconto_permitido: indisponível` quando na verdade é zero. Isso faria o assistente sugerir confirmar uma alçada que não existe. Para alçada não resolvida, o seguro é `0%`.

---

## 5. Tratamento de dados sensíveis (preço, desconto, financeiro)

`preco_tabela`, `desconto_permitido`, `promocao_vigente` e `pendencia_financeira` são sensíveis comercialmente. Cuidados:

1. **Resolução por contexto autorizado.** O bloco deve ser montado server-side a partir de `pdv_id` + `representante_id` autenticados. Nunca aceitar esses valores vindos do cliente (app) e nunca expor a tabela inteira — apenas o recorte válido para aquele PDV/representante/data.
2. **Alçada de desconto sempre derivada da política, nunca livre.** Mesmo que o assistente "sugira" desconto, ele só pode operar dentro de `desconto_permitido`. O fechamento real do pedido (no ERP) deve revalidar a alçada server-side — o assistente não é autoridade de preço.
3. **Não persistir o prompt montado com esses dados em logs sem necessidade.** Se for logar para depuração, mascarar preço/desconto. Considerar marcar a chamada como sem retenção quando o provedor de LLM permitir.
4. **Bloqueio financeiro tem precedência.** Se `pendencia_financeira = bloqueado`, o backend pode (a) ainda assim montar o contexto, mas o prompt já orienta a tratar a pendência antes de empurrar volume; e (b) idealmente impedir o faturamento no ERP independentemente do que o assistente disser.
5. **Validade temporal.** Preço e promoção têm data de vigência — resolver sempre para a data da visita, não cachear além da validade (ver seção 6).

---

## 6. Frescor, cache e modo offline

- **Frescor.** `pendencia_financeira`, `preco_tabela` e `promocao_vigente` devem refletir o dia da visita. Histórico de pedidos e perfil podem tolerar cache de algumas horas.
- **Offline.** Venda externa enfrenta sinal ruim em muitos PDVs. Estratégia recomendada: pré-carregar o `<contexto_pdv>` de toda a rota do dia na sincronização da manhã, com `timestamp` de montagem. Exibir ao representante a data/hora do contexto para ele saber se está olhando algo fresco.
- **Conflito de versão.** Se houver dado mais novo no servidor que o cacheado (ex.: pendência criada hoje), priorizar o servidor quando houver conexão; offline, sinalizar que o financeiro pode estar desatualizado.

---

## 7. Formato de injeção e observação ao vivo

- Injetar o bloco `<contexto_pdv>` no prompt exatamente com os nomes de campo da especificação do assistente (eles são o contrato entre backend e prompt — não renomear sem atualizar o prompt).
- Anexar separadamente a observação ao vivo do representante, claramente rotulada, ex.: `[Observação ao vivo do representante]: ...`. O prompt já instrui a priorizar essa observação sobre o sistema em caso de conflito.
- Campos vazios: enviar `indisponível` ou o valor padrão — **não omitir a linha**, para o assistente saber que o campo existe e foi avaliado.

---

## 8. Checklist de validação antes de produção

- [ ] Histórico vem com datas por pedido (não agregado).
- [ ] `tendencia_volume` calcula Heineken e geral separadamente.
- [ ] `indisponível` x valor padrão aplicados conforme seção 4.
- [ ] Preço/desconto resolvidos server-side por PDV/representante autenticado; alçada revalidada no fechamento.
- [ ] Promoção e preço resolvidos para a data da visita.
- [ ] Bloqueio financeiro impede faturamento no ERP independentemente do assistente.
- [ ] Logs mascaram dados sensíveis.
- [ ] Contexto da rota pré-carregado para uso offline, com timestamp visível.
- [ ] Nomes de campo idênticos aos do prompt do assistente.
