# Integração com o onp-spec-driven

O FoodMe usa o [onp-spec-driven](../../onp-spec-driven/SKILL.md) como
trilho de spec-driven development: `Especificar → Projetar → Tarefas →
Plano → Executar → Auditar → Aprender`, com specs em
`.spec/features/<feature>/spec.md` auditadas mecanicamente contra o
código.

Esta skill (`engenharia-reversa-foodme`) **não substitui nem compete** com
o onp-spec-driven — ela produz o **insumo bruto** que o onp-spec-driven
consome quando o ponto de partida é código legado em vez de uma ideia
nova. A engenharia reversa não escreve história de usuário, não escreve
critério de aceite, não fecha feature — isso é sempre trabalho do
onp-spec-driven, guiado pelo usuário.

## O que esta skill entrega para o onp-spec-driven

- **Regra `REG-xxx` classificada como NÃO COBERTA** → material bruto para
  a fase Especificar. Leve o texto da regra e a evidência para
  `onp-spec new <feature>` e escreva a história (US-xxx) e o critério de
  aceite (AC-xxx) a partir dela.
- **Regra `REG-xxx` classificada como COBERTA** → não gera trabalho novo;
  confirma que a spec já existente reflete o comportamento real.
- **Regra `REG-xxx` classificada como COBERTA (pendente de
  implementação)** → confirma um gap que a spec já sabia que existia;
  vira insumo para a fase Executar da feature correspondente, não para
  uma feature nova.
- **Divergência `DIV-xxx`** → nunca resolvida por esta skill. Quando o
  usuário decidir o desfecho, ela normalmente vira uma **suposição**
  (`ASM-xxx`) confirmada/invalidada ou uma **pergunta em aberto**
  (`Q-xxx`) respondida na spec da feature que trata do assunto. Esta
  skill só levanta a divergência com as duas leituras lado a lado — quem
  edita a spec e decide o desfecho é a fase Especificar do
  onp-spec-driven, com o usuário.
- **Fluxo `FLX-xxx` mapeado** → referência viva de como o sistema
  funciona hoje; útil como contexto ao escrever `spec.md` ou ao revisar
  um `plano-execucao.md`.

## Limites — o que esta skill nunca escreve

- `app/**`, `server/**` — código de produção. Esta skill não refatora,
  não corrige bugs encontrados, não implementa nada.
- `.spec/features/*/spec.md`, `.spec/features/*/tasks.md` — pertencem ao
  onp-spec-driven; só ele, guiado pelo usuário, edita.
- `.spec/constituicao.md` — princípios inegociáveis do projeto. Uma regra
  que pareça violá-los é `DIV-xxx` de prioridade alta, nunca uma sugestão
  de alterar o princípio.

A única área de escrita desta skill é `docs/engenharia-reversa/`.

## Quando NÃO usar esta skill

- Para escrever uma spec nova do zero (sem código legado envolvido) — use
  `onp-spec-driven` diretamente.
- Para implementar algo — nenhum dos artefatos desta skill é executável;
  eles são leitura para humano e para a fase Especificar do
  onp-spec-driven.
- Para revisar qualidade/estilo de código — isso é escopo de revisão de
  código, não de engenharia reversa de regra de negócio.
