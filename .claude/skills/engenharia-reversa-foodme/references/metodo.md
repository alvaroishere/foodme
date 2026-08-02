# Método — gramática dos códigos, evidência e classificação

## Gramática dos códigos

- `REG-xxx` — regra de negócio descoberta. Numeração contínua, única em
  todo `docs/engenharia-reversa/regras-de-negocio.md` (não reinicia por
  fluxo).
- `FLX-xxx` — fluxo/jornada mapeado ponta a ponta. Um por arquivo em
  `docs/engenharia-reversa/mapas/`.
- `DIV-xxx` — divergência entre código e spec/contrato aprovado.
  Numeração contínua em `docs/engenharia-reversa/divergencias.md`.

Três dígitos mínimo (`REG-001`, não `REG-1`). Nunca reutilize um código já
usado, mesmo que a regra correspondente seja descartada depois — marque
`[descartada: <motivo>]` em vez de reaproveitar o número.

## Regra de evidência — sem arquivo:linha, a regra não existe

Cada `REG-xxx` precisa de:
1. **Caminho relativo ao repositório** (`app/js/services/cart.js`), nunca
   caminho absoluto do disco.
2. **Linha ou faixa de linhas** (`16-30`).
3. **Trecho literal** — copie do código, não parafraseie o trecho (a
   paráfrase vai na descrição da regra, não na evidência).

Se a regra foi inferida de mais de um lugar (ex.: view + controller
repetindo a mesma validação), liste todas as evidências na mesma entrada.
Não crie uma `REG-xxx` por arquivo quando é a mesma regra de negócio.

## Classificação — sempre as três opções, nunca pular o cruzamento

Para cada `REG-xxx`, procure explicitamente em:
- `.spec/features/*/spec.md` (seções de histórias e critérios de aceite)
- `.spec/constituicao.md` (princípios `P-xxx`)

E classifique:

| Status | Quando usar | O que registrar |
|---|---|---|
| **COBERTA** | Uma spec já aprovada descreve exatamente esse comportamento | Cite a `US-xxx`/`AC-xxx`/princípio correspondente |
| **COBERTA (pendente de implementação)** | Uma spec aprovada descreve o comportamento **desejado**, mas o código ainda faz outra coisa (ex.: um endpoint que hoje não persiste nada, mas uma spec aprovada já define que deveria) | Cite a spec e descreva o gap — isso NÃO é `DIV-xxx`, porque não há conflito de decisão, só trabalho pendente |
| **NÃO COBERTA** | Comportamento real, sem nenhuma spec tratando do assunto | Marque como candidata a virar história (`onp-spec new <feature>`) |
| **DIVERGENTE** | Código e spec aprovada **descrevem comportamentos diferentes e incompatíveis** para a mesma situação | Vira `DIV-xxx` em `divergencias.md`, com as duas versões lado a lado, sem veredito |

A diferença entre "COBERTA (pendente de implementação)" e "DIVERGENTE"
importa: a primeira é uma spec aprovada esperando ser codificada (não é
uma decisão em aberto); a segunda é um conflito real de entendimento que
precisa de decisão humana.

## Comportamento ambíguo — bug ou regra deliberada?

Quando o código faz algo que parece inconsistente (nomes de variável
sugerem uma intenção, mas a lógica faz outra coisa — como o exemplo de
precedência de operador em `model.js` descrito em
[mapa-da-stack.md](mapa-da-stack.md)):

1. Registre a leitura literal do código (o que ele faz de fato).
2. Registre a intenção provável (o que o nome/contexto sugere que deveria
   fazer).
3. Marque `[CONFIRMAR COM O DONO DO PRODUTO]` — nunca decida sozinho qual
   das duas é "a regra certa", e nunca corrija o código para alinhar com a
   leitura que parece mais razoável.

## O que NÃO vira `REG-xxx`

- Mecânica de framework sem semântica de domínio (bind de evento DOM,
  `$watch` de sincronismo puro, roteamento técnico).
- Código morto ou comentado (`// logger.info(...)` em `server/index.js`) —
  mencione como observação lateral no relatório, se relevante, mas não
  como regra ativa.
- Estilo de código, nomenclatura, formatação — isso é para uma revisão de
  código, não para engenharia reversa de negócio.

## Tamanho do escopo por rodada

Um fluxo (`FLX-xxx`) por vez. Se ao rastrear a jornada você perceber que
ela se ramifica em mais de ~2 sub-fluxos independentes (ex.: "checkout"
puxando também "cálculo de frete" como sistema à parte), pare, feche o
fluxo atual, e trate o sub-fluxo como um novo `FLX-xxx` em rodada
separada. Catálogo raso de um sistema inteiro vale menos que catálogo
fundo de um fluxo.
