# 🍺 Assistente de Vendas Heineken

Assistente com IA que atua como **co-piloto do representante comercial** de uma distribuidora de bebidas durante visitas a pontos de venda (PDVs). O foco é vender mais Heineken argumentando por **margem e giro** — não por desconto — e respeitando dados reais de estoque, preço e histórico vindos do ERP/CRM.

> **Princípio central:** vender Heineken é vender lucro por garrafa e demanda de público, não "comprar mais caixa".

---

## 📂 Estrutura do projeto

| Documento | Descrição |
|---|---|
| [`prompts/prompt-simples.md`](prompts/prompt-simples.md) | Prompt base do assistente, com entrada manual feita pelo representante. |
| [`prompts/prompt-integrado.md`](prompts/prompt-integrado.md) | Versão de produção, que recebe dados do ERP/CRM via bloco `<contexto_pdv>`. |
| [`docs/spec-tecnica-contexto-pdv.md`](docs/spec-tecnica-contexto-pdv.md) | Especificação técnica de como o backend monta e injeta o `<contexto_pdv>`. |
| [`docs/teste-bar-do-babu.md`](docs/teste-bar-do-babu.md) | Teste real do prompt integrado — caso de Recuperação de cliente. |

---

## 🧭 Por onde começar

1. **Entender o conceito** → leia este README.
2. **Ver o assistente funcionando** → [Teste Bar do Babu](docs/teste-bar-do-babu.md).
3. **Implementar** → [Prompt integrado](prompts/prompt-integrado.md) + [Spec técnica](docs/spec-tecnica-contexto-pdv.md).

---

## ✅ Princípios de design

- **Margem e giro acima de desconto** — o argumento que convence o dono do PDV.
- **Anti-alucinação em dados duros** — preço, estoque, desconto e disponibilidade vêm do sistema; campo vazio vira "confirmar", nunca um palpite.
- **Desenhado para o fluxo de campo** — respostas curtas, voz, e uso offline.
- **Alinhado à indústria** — mix obrigatório, loja perfeita e ativações Heineken.
- **Venda responsável de álcool.**

---

## 📌 Status

Prova de conceito. Próximos passos sugeridos: integração real do `<contexto_pdv>` ao ERP/CRM e definição das métricas de sucesso do piloto (volume e mix de Heineken, taxa de reativação, cobertura da carteira).
