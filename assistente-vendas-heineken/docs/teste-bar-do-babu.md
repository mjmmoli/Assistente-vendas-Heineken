# 🍺 Teste do Assistente de Vendas Heineken

> **Distribuidora de Bebidas · Prova de Conceito**
> Simulação de visita a PDV usando o prompt integrado ao ERP/CRM — **caso de Recuperação de cliente**.

Este documento registra um teste da versão integrada do assistente. Foi escolhido de propósito um caso de **Recuperação** (cliente que parou de comprar Heineken), com uma **observação ao vivo do representante** que conflita com o dado do sistema, para validar dois comportamentos críticos:

- o disparo automático do **modo Recuperação**;
- a **priorização do que o representante observa na hora** sobre o dado do sistema.

---

## 📑 Índice

- [1. O que o sistema injeta](#1-o-que-o-sistema-injeta-no-assistente)
- [2. Resposta do assistente](#2-resposta-do-assistente)
- [3. Leitura dos resultados](#3-leitura-dos-resultados)

---

## 1. O que o sistema injeta no assistente

Bloco `<contexto_pdv>` preenchido automaticamente pelo ERP/CRM no início da visita:

```yaml
<contexto_pdv>
  pdv_nome:            Bar do Babu
  pdv_tipo:            bar
  perfil_publico:      adulto 30-45, classe média, happy hour pós-trabalho
  fluxo:               quinta a sábado, 18h-23h

  historico_pedidos:   set/25: Heineken LN 10cx + popular lata 40cx
                       out/25: Heineken LN 6cx + popular lata 42cx
                       nov/25: popular lata 45cx (sem Heineken)
                       dez/25 a mai/26: só popular lata
  mix_atual:           marca popular lata (volume estável)
  mix_heineken_atual:  nenhum (parou em out/25)
  tendencia_volume:    Heineken inativo há ~7 meses; volume geral estável

  espaco_refrigeracao: 1 cooler de marca Heineken (instalado, em uso)
  ativos_trade:        cooler Heineken, sem chopeira, sem taças

  preco_tabela:        indisponível
  desconto_permitido:  até 5% (autorizado p/ reativação)
  promocao_vigente:    combo reativação LN - 1cx bônus a cada 10cx (ativo)
  pendencia_financeira: em dia

  metas_industria:     mix obrigatório inclui 1 SKU premium; loja perfeita exige cooler abastecido
  objecao_recente:     out/25 - "long neck encalhou, saiu menos que eu esperava"
</contexto_pdv>
```

> ⚠️ **Observação ao vivo do representante:**
> "Cheguei aqui e o cooler Heineken tá ligado, mas tá abastecido com a lata popular dele. Não tem nenhuma Heineken dentro."

---

## 2. Resposta do assistente

### A) Briefing da visita

O Bar do Babu é um bar de happy hour adulto (30–45) que **comprava Heineken long neck regularmente até out/25 e parou há ~7 meses** — hoje só gira a popular em lata, com volume estável. Ele **tem um cooler Heineken instalado**, mas (observação de agora) está abastecido com a lata popular. O combo de reativação está ativo e ele está em dia financeiramente. Oportunidade clara de recuperação, com um alerta de execução grave: o ativo da marca está sendo usado contra ela.

### B) Diagnóstico de oportunidade

**`Recuperação`**

- Ele já provou o produto e tinha giro (16cx em set–out) → a demanda existiu, não é PDV frio.
- Parou por uma percepção registrada: *"long neck encalhou"* (out/25). É objeção de **giro percebido**, não de preço.
- **Alerta de execução (prioridade):** cooler Heineken abastecido com produto concorrente fere a "loja perfeita" e mata a visibilidade da marca — provavelmente parte da causa do "encalhe".
- **Alerta de meta:** mix obrigatório premium não está sendo cumprido hoje.
- Financeiro em dia e desconto de reativação (5%) + combo bônus disponíveis como ferramentas.

### C) Lacunas a confirmar na visita

- [ ] Por que ele parou de fato — foi só o "encalhe" de out, ou teve preço de revenda, validade, ou algo mais?
- [ ] A que preço ele revendia a long neck quando girava — e se acha que estava certo.
- [ ] Topa voltar a abastecer o cooler com Heineken (uso correto do ativo) se a gente ajustar o giro?

> _Preço de tabela veio `indisponível` — não cito valor de compra; confirmo no sistema antes de fechar número._

### D) Oferta principal recomendada

| Item | Detalhe |
|---|---|
| **O que oferecer** | Retorno enxuto de Heineken long neck, menor que o último pedido que encalhou (ex.: começar com 6cx, não 10), usando o combo de reativação (bônus a cada 10cx) e rearrumando o cooler para a long neck ficar visível e gelada. |
| **Por que faz sentido** | Ele já teve giro de long neck; o problema não foi demanda, foi volume alto demais + execução (cooler ocupado pela popular). Voltar pequeno, visível e gelado resolve o "encalhe" que ele lembra. |
| **Como apresentar em 1 frase** | *"Vamos voltar com a Heineken na medida certa e no cooler certo — menos caixa, mais visível, pra ela girar em vez de encalhar."* |

### E) Oferta complementar (cross-sell)

- **Taças Heineken** → ele não tem; elevam a percepção no happy hour e ajudam a justificar o preço de revenda da long neck.
- **Material de PDV / abastecimento correto do cooler** → não é venda, é execução: cooler abastecido com Heineken puxa a demanda que hoje está invisível.
- _Não oferece cooler (ele já tem) nem chopeira (perfil de happy hour com lata/LN não pede chopp agora)._

### F) Ancoragem e argumentação

1. **Margem por garrafa** — cada long neck deixa mais lucro por unidade que a lata popular; com o combo de bônus, a conta melhora ainda mais. Foco no lucro por garrafa, não no preço de compra.
2. **Giro × ticket** — o público de happy hour 30–45 é exatamente quem topa pagar pela long neck; menos volume, ticket maior, ocupando o cooler que já está ali parado para a marca.

> 🛠️ **Ferramentas disponíveis:** desconto de reativação até 5% e combo bônus — apoio ao "voltar com segurança", não leilão de preço.

### G) Mensagens prontas

**Abertura**
> "Babu, reparei que sua Heineken parou em outubro. Não vim te empurrar caixa — vim entender por que encalhou e voltar de um jeito que ela gire de verdade."

**Contorno da objeção (encalhe)**
> "Encalhou porque veio caixa demais e o cooler tava cheio da lata. Vamos fazer o contrário: menos caixa, Heineken visível e gelada no cooler dela, e com o bônus que tá rolando agora. Se não girar como combinamos, a gente reavalia."

**Follow-up (WhatsApp)**
> "Fala, Babu! Como combinei, te montei o retorno enxuto da Heineken com o bônus de reativação. Passo amanhã pra deixar o cooler arrumado certinho. Fechado?"

### H) Registro pós-visita (CRM)

- [ ] Causa real da parada em out/25 (encalhe × preço × execução).
- [ ] Pedido fechado de reativação (qtd de long neck + se entrou no combo bônus).
- [ ] Combinado sobre abastecimento do cooler com Heineken (execução loja perfeita).
- [ ] Retorno sugerido: ~3 semanas (validar giro do volume reduzido antes de subir).

> _"Confirma os pontos da seção C e me diz como ele reagiu, que eu refino a oferta na hora."_

---

## 3. Leitura dos resultados

O teste validou os comportamentos que importavam:

| ✅ Comportamento | Resultado |
|---|---|
| **Modo Recuperação** | Disparou sozinho ao ler `tendencia_volume = inativo` e reconstruiu a história do cliente pelo histórico de pedidos, em vez de tratá-lo como PDV novo. |
| **Conflito de dados** | O sistema dizia que ele tinha cooler (verdade), mas a observação ao vivo revelou que estava abastecido com a concorrente — o assistente priorizou o que o representante viu e fez disso o insight central. |
| **Anti-alucinação** | `preco_tabela` veio `indisponível` e nenhum valor foi inventado, mas o desconto e a promoção (preenchidos) foram usados normalmente. |
| **Ativos e metas** | Não ofereceu cooler a quem já tem e considerou a meta de indústria (mix premium + cooler abastecido). |

> 💡 **Aprendizado para a integração:** a frase "parou em outubro" só foi possível porque o histórico veio com **datas por pedido**. Garanta que o ERP entregue os últimos 3–6 pedidos com data, e não um total agregado — é esse detalhe que faz o representante parecer que conhece o cliente de verdade.
