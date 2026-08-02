---
name: engenharia-reversa-foodme
description: Engenharia reversa das regras de negócio e fluxos do FoodMe (AngularJS 1.x no cliente + Express/Node.js no servidor) direto do código-fonte — rastreia a jornada ponta a ponta (view → controller → serviço → rota Express → modelo → armazenamento), documenta cada regra com evidência em arquivo:linha, e cruza cada achado com as specs e contratos já aprovados (trilho onp-spec-driven) para classificá-lo como coberto, não coberto ou divergente. Escrita é restrita a docs/engenharia-reversa/ — NUNCA altera contrato de API, spec aprovada, constituição ou código de produção. Use quando o pedido for "entender como funciona X", "documentar regra de negócio", "mapear o fluxo de Y", "o que esse código faz de verdade", "achar regra escondida no código", ou antes de especificar uma feature sobre código legado sem spec. NÃO use para escrever specs novas (isso é papel de onp-spec-driven) nem para alterar comportamento do sistema.
license: MIT
metadata:
  version: 1.0.0
  agent: claude
---

# engenharia-reversa-foodme — entender o sistema como ele realmente é

O FoodMe usa o [onp-spec-driven](../onp-spec-driven/SKILL.md) como trilho de
spec-driven development: a especificação (`.spec/features/<feature>/spec.md`)
é escrita **antes** do código, e auditada mecanicamente contra ele. Esta
skill cobre o caso oposto: código que **já existe** — parte dele sem spec
nenhuma, parte com spec que pode ter ficado para trás — e cuja regra de
negócio só está escrita, hoje, dentro do próprio código-fonte.

```
CÓDIGO EXISTENTE  →  [engenharia reversa]  →  catálogo de regras (REG-xxx)
                            │                          │
                            │                          └─→ alimenta onp-spec-driven
                            └─→ nunca decide sozinha; quem decide é o usuário
```

Esta skill é **diagnóstica e somente-leitura sobre o sistema**: ela ilumina
o que o código faz, cruza com o que já foi decidido por escrito, e entrega
o material bruto para quem for especificar. Ela não especifica, não
implementa, não conserta.

## Contrato de execução — inegociável

1. **Escrita restrita a `docs/engenharia-reversa/`.** Nunca edite `app/`,
   `server/`, `.spec/features/*/spec.md`, `.spec/features/*/tasks.md` de
   features já aprovadas, nem `.spec/constituicao.md`. Esses arquivos são
   **lidos** como premissa, nunca escritos por esta skill.
2. **Toda regra catalogada precisa de evidência**: caminho de arquivo +
   linha + trecho literal do código. Sem evidência, a regra não existe —
   não infira comportamento a partir do nome de uma função ou do que
   "provavelmente" o sistema faz.
3. **Toda regra é cruzada contra specs e contratos existentes antes de ser
   classificada.** Pular esse cruzamento é achado incompleto — a mesma regra
   catalogada duas vezes (uma no código, outra já na spec) é ruído, não
   descoberta.
4. **Achado que diverge de uma spec ou contrato aprovado vira DIVERGÊNCIA
   (DIV-xxx) e é escalado ao usuário.** Esta skill nunca decide se o código
   está errado ou se a spec está desatualizada — quem decide é o dono do
   produto.
5. **Comportamento ambíguo (pode ser bug, pode ser regra deliberada) é
   registrado com as duas leituras** e marcado
   `[CONFIRMAR COM O DONO DO PRODUTO]` — nunca escolha uma interpretação
   sozinha.
6. **Mecânica técnica não é regra de negócio.** Data binding do Angular,
   `$watch`, boilerplate de HTTP, formatação de código — isso é
   implementação. Só entra no catálogo o que carrega semântica de domínio
   (uma validação, um cálculo, uma restrição, um efeito colateral que o
   negócio se importaria em saber que existe).

## Vocabulário — códigos próprios, sem colidir com o SDD

| Código | Significado |
|---|---|
| REG-xxx | **regra de negócio descoberta** no código, com evidência |
| FLX-xxx | **fluxo mapeado** ponta a ponta (uma jornada de usuário pela pilha) |
| DIV-xxx | **divergência** entre o código e uma spec/contrato já aprovado |

São códigos de **descoberta**, não de especificação — não confunda com
US/AC/T/ASM/Q/P do onp-spec-driven. Uma REG-xxx só vira critério de aceite
de verdade quando o usuário decidir levá-la para uma spec formal.

## Onde as regras se escondem nesta stack

Mapa completo, camada por camada, com exemplos reais do repositório:
[mapa-da-stack.md](references/mapa-da-stack.md). Carregue antes de começar
a rastrear um fluxo — ele evita procurar regra de negócio em lugar errado
(ex.: dentro de uma diretiva que só faz data binding).

## Passo a passo

### 0. Escopo
Escolha **um fluxo por vez** (ex.: "fazer pedido", "listar restaurantes",
"cadastrar cliente/endereço de entrega"). Se o usuário não delimitou, use
AskUserQuestion com as jornadas visíveis no `app/js/app.js` (rotas) como
opções. Engenharia reversa de um sistema inteiro de uma vez produz um
catálogo raso; um fluxo por vez produz evidência de verdade.

### 1. Carregar premissas antes de tocar no código
Leia, nesta ordem: `onp-spec licoes list` (lições já aprendidas em features
anteriores), a spec da feature relevante em `.spec/features/<feature>/
spec.md` (se já existir) e a constituição do projeto
(`.spec/constituicao.md`). Isso evita "descobrir" algo que já está escrito
— e dá o vocabulário certo (US-xxx, AC-xxx) para citar quando uma regra já
estiver coberta.

### 2. Rastrear a jornada ponta a ponta
Siga a pilha na ordem em que a requisição realmente viaja:

```
view (app/views/*.html)
  → controller (app/js/controllers/*.js)
    → serviço/factory (app/js/services/*.js)
      → $http / $resource
        → rota Express (server/index.js)
          → modelo (server/model.js)
            → armazenamento (server/storage.js ou persistência futura)
          ← resposta JSON
      ← promise resolvida
    ← atualização de $scope
  ← re-render
```

Desenhe um diagrama de sequência Mermaid por fluxo e salve em
`docs/engenharia-reversa/mapas/<fluxo>.md`, com uma frase de narrativa por
etapa. Modelo pronto: [mapa-de-fluxo.md](templates/mapa-de-fluxo.md).

### 3. Extrair regras camada por camada
Use a tabela de [mapa-da-stack.md](references/mapa-da-stack.md) para saber
onde procurar em cada camada. Cada regra encontrada vira uma entrada
`REG-xxx` com:
- **Regra** — em linguagem de negócio, uma frase, sem jargão de código.
- **Evidência** — `arquivo:linha` + trecho literal (2-6 linhas).
- **Categoria** — validação · cálculo · autorização · ciclo-de-vida ·
  formatação/apresentação · efeito colateral.

Metodologia completa e gramática dos códigos:
[metodo.md](references/metodo.md).

### 4. Cruzar com o que já está aprovado
Para cada `REG-xxx`, procure se uma spec já aprovada (`.spec/features/*/
spec.md`) ou a constituição (`.spec/constituicao.md`) já cobre aquilo.
Classifique:
- **COBERTA** — cite a `US-xxx`/`AC-xxx`/princípio correspondente.
- **NÃO COBERTA** — comportamento real sem registro formal (o caso comum em
  código legado).
- **DIVERGENTE** — código e spec aprovada dizem coisas diferentes. Vira
  `DIV-xxx` em `divergencias.md`, sem veredito.

### 5. Registrar
Grave no catálogo (`docs/engenharia-reversa/regras-de-negocio.md`) e, se a
regra envolver dado persistido/trafegado, no dicionário de dados
(`docs/engenharia-reversa/dicionario-de-dados.md`). Estrutura completa de
saída: ver seção abaixo.

### 6. Fechar com resumo e próximo passo (sem executar por conta própria)
Resuma: quantas regras (`REG-xxx`), quantas cobertas/não cobertas/
divergentes, e quantos fluxos mapeados (`FLX-xxx`). Sugira o próximo passo
sem tomá-lo:
- Regras não cobertas → material para `onp-spec new <feature>` (viram
  candidatas a US-xxx/AC-xxx).
- Divergências → precisam de decisão do dono do produto antes de qualquer
  mudança de código ou de spec.

Como isso se encaixa no restante do fluxo de SDD do projeto (e os limites
do que esta skill pode tocar): [integracao-sdd.md](references/integracao-sdd.md).

## Estrutura de saída

```
docs/engenharia-reversa/
├── regras-de-negocio.md     # catálogo REG-xxx — tabela: código, regra,
│                             # evidência, categoria, status (coberta/
│                             # não coberta/divergente)
├── mapas/
│   └── <fluxo>.md            # FLX-xxx — diagrama Mermaid + narrativa
├── dicionario-de-dados.md    # campos, tipos, coerções e defaults
│                             # reversos de model.js / CSV / JSON / contratos
└── divergencias.md           # DIV-xxx — código vs. spec/contrato aprovado,
                              # sem veredito, pendente de decisão
```

Modelos prontos para cada arquivo em [templates/](templates/).

## Regra de ouro

Se a regra não tem `arquivo:linha`, ela não existe. Se o achado conflita
com uma spec ou contrato já aprovado, ele vira uma pergunta para o dono do
produto — nunca uma edição feita por esta skill, nem no código, nem na
spec, nem no contrato.
