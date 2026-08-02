# Dicionário de dados — engenharia reversa

> Campos, tipos, coerções e defaults reversos do código (modelos,
> CSV/JSON semeados, contratos). Fonte de verdade quando não há schema
> declarado formalmente — ex.: CSV lido por posição de coluna.

## Entidade: <nome, ex. "Restaurante">

**Fontes observadas**: `<arquivos>` (ex.: `server/model.js`,
`server/data/restaurants.csv`, `specs/.../contracts/order-api.yaml`)

| Campo | Tipo observado | Origem/derivação | Default | Obrigatório? | Evidência |
|---|---|---|---|---|---|
| `id` | string | derivado de `name` via `idFromName` (minúsculas, sem não-alfanuméricos), ou fornecido explicitamente | — | sim (derivado) | `server/model.js:1-3, 35` |
| `name` | string | entrada direta | — | sim (única validação de servidor) | `server/model.js:53-56` |
| ... | ... | ... | ... | ... | ... |

## Schema implícito por posição (quando aplicável)

Para dados lidos por índice de array (não por nome de campo), documente a
ordem exata — é o contrato real, mesmo sem estar declarado:

| Posição | Coluna no arquivo | Campo na entidade | Evidência |
|---|---|---|---|
| 0 | <cabeçalho do CSV> | <campo> | `<arquivo>:<linha>` |
| 1 | ... | ... | ... |

## Divergências de tipo/coerção observadas

- <ex.: "price convertido sempre; rating só quando string — ver REG-xxx"> (`<arquivo>:<linha>`)
