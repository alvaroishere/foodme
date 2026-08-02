# Onde as regras se escondem nesta stack

O FoodMe é uma aplicação AngularJS 1.x (cliente) + Express/Node.js
(servidor), com dados semeados em CSV/JSON e evoluindo para persistência em
PostgreSQL (spec `002-persistencia-postgres`). Cada camada tem um "sotaque"
próprio de onde a regra de negócio costuma morar — e onde costuma ser só
mecânica técnica, que não entra no catálogo.

## Views — `app/views/*.html`

O que procurar: `ng-show` / `ng-hide` / `ng-if` condicionais (regra de
visibilidade), `ng-required` / `ng-pattern` / `ng-minlength` (regra de
validação de formulário), texto condicional (`{{ ... }}` dentro de blocos
`ng-if`). É comum a MESMA regra aparecer duplicada aqui e no controller —
cite as duas evidências na mesma entrada `REG-xxx` quando isso acontecer,
não crie duas regras.

## Controllers — `app/js/controllers/*.js`

Orquestração de tela e guardas de fluxo. Raramente cálculo; quase sempre
"quando isto pode acontecer".

- `CheckoutController.js:13` — `if ($scope.submitting) return;` é uma
  regra real: **impede envio duplicado do pedido** enquanto a requisição
  anterior ainda não voltou. Sem essa linha, um clique duplo faria dois
  pedidos.
- `RestaurantsController.js:6-8` — se o cliente não tem endereço
  cadastrado, redireciona para `/customer`. Regra de negócio: **não é
  possível listar restaurantes sem endereço de entrega definido**.
- `RestaurantsController.js:19-52` — filtro (cuisine/price/rating) e
  ordenação da lista de restaurantes são regras de apresentação com efeito
  de negócio direto (o que o cliente vê e em que ordem).

## Serviços/factories — `app/js/services/*.js`

Onde mora a regra de negócio mais "dura" do cliente — estado e cálculo,
não só orquestração de tela.

- `cart.js:16-30` (`self.add`) — **um carrinho só pode ter itens de um
  restaurante por vez**; tentar misturar dispara um alerta
  (`"Não é possível misturar itens de restaurantes diferentes."`). Ao
  adicionar um item já presente, a regra é **incrementar quantidade**, não
  duplicar linha.
- `cart.js:34-42` (`self.remove`) — remover o último item **reseta o
  restaurante do carrinho** (`self.restaurant = {}`), permitindo trocar de
  restaurante depois de esvaziar.
- `cart.js:45-49` (`self.total`) — total é `Σ preço × quantidade`; note
  que preço é coagido com `Number(...)` aqui, mas não em `model.js`
  (comparar com a divergência de coerção descrita abaixo).
- `cart.js:75` — comentário explícito `// don't keep CC info in
  localStorage`: **dados de pagamento nunca são persistidos no
  localStorage do navegador**, só em memória de sessão (`self.payment =
  {}`, não passa por `createPersistentProperty`). É regra de negócio com
  peso de segurança/privacidade — cruzar sempre com o que uma spec aprovada
  (`.spec/features/*/spec.md`) disser sobre retenção de dados de pagamento,
  se já houver uma.
- `customer.js` — nome/endereço do cliente persistem em `localStorage`
  entre sessões (regra de conveniência: não precisa recadastrar a cada
  visita).

## Diretivas — `app/js/directives/*.js`

Quase sempre mecânica de UI (data binding customizado), **não** regra de
negócio. Exceções, quando a diretiva encapsula uma constraint de domínio:

- `fmRating.js` — o rating máximo default é 5 (`scope.max || 5`,
  linha 14); isso é regra de apresentação, mas se um AC em alguma spec
  falar de "avaliação de 1 a 5 estrelas", esta é a evidência de onde esse
  limite é aplicado.
- `fmCheckboxList.js`, `fmDeliverTo.js` — puro data binding (`ngModel`
  custom), sem semântica de domínio. Não catalogar como `REG-xxx`.

## Filtros — `app/js/filters/*.js`

Regras de apresentação/formatação — parecem triviais mas codificam de
fato uma escala de negócio:

- `dollars.js` — mapeia faixa de preço `1..5` para `$`..`$$$$$`. A
  **escala de preço do domínio é 1 a 5**, não um valor monetário direto —
  isso importa para qualquer spec que fale de "preço do restaurante".
- `stars.js` — mesmo padrão para avaliação (`1..5` → `★`..`★★★★★`).

## Rotas Express — `server/index.js`

Este é o **contrato real observável da API** — leia para documentar o que
o sistema faz hoje, nunca para alterar. Se notar que o comportamento
diverge de uma spec aprovada (`.spec/features/*/spec.md`), isso é uma
`DIV-xxx`, não um convite a "corrigir" a rota.

- `API_URL_ORDER` (`POST /api/order`, linhas 63-67) — hoje **não persiste
  nada**: devolve `{ orderId: Date.now() }` e descarta o corpo da
  requisição. Se já existir uma spec aprovada tratando de persistência de
  pedidos, isso é ótimo exemplo de regra **COBERTA (pendente de
  implementação)**, não "não coberta" — o gap já está documentado, só
  falta codificar.
- `PUT /api/restaurant/:id` (linhas 77-93) — regra de upsert: se o
  restaurante existe, atualiza; se não existe, **cria com o id da URL**.
  Isso é `PUT`-como-upsert, não é REST puro — vale registrar como regra
  explícita, não assumir que é convenção óbvia.
- `POST` e `PUT` sobre `/api/restaurant` (linhas 51-61, 86-92) validam via
  `restaurant.validate(errors)` — a única regra de validação hoje é "nome é
  obrigatório" (ver `model.js`).
- Handler de `SIGINT` (linhas 116-123) — ao encerrar o processo, o
  servidor **grava o storage inteiro de volta no `DATA_FILE` JSON**. Sem
  isso, tudo em `MemoryStorage` se perde. Regra de negócio implícita:
  "os dados de restaurante sobrevivem a um shutdown limpo, mas não a um
  crash" — relevante para qualquer spec sobre durabilidade.

## Modelo — `server/model.js`

Onde vivem validação, normalização e derivação de identificadores.

- `idFromName` (linhas 1-3) — o **id de um restaurante é derivado do
  nome** (minúsculas, sem caracteres não-alfanuméricos), a menos que um id
  seja explicitamente fornecido. Regra de negócio com risco: dois nomes
  que normalizem para a mesma string colidem — não há checagem de
  duplicidade em nenhuma camada observada. Vale registrar como `REG-xxx`
  **e** como candidato a `DIV-xxx`/pergunta em aberto se alguma spec
  assumir id único garantido.
- `parseDays` (linhas 19-23) — dias da semana no CSV usam abreviação de
  duas letras em inglês (`Su,Mo,Tu,We,Th,Fr,Sa`) mapeadas para índice
  numérico `0..6`. É a definição formal (e a única) de "dia da semana" no
  sistema — qualquer spec que fale de "dias de funcionamento" depende
  desse mapeamento.
- `Restaurant.prototype.update` (linhas 38-50) — **atenção, comportamento
  ambíguo**: a linha
  `if (key === 'price' || key === 'rating' && isString(data[key]))`
  tem precedência de operador `&&` mais forte que `||`. Na prática isso
  equivale a `key === 'price' || (key === 'rating' && isString(data[key]))`:
  **todo campo `price` é sempre convertido com `parseInt`, mesmo se já for
  number; `rating` só é convertido quando chega como string**. Regra de
  negócio esperada (provável, a julgar pelo nome das variáveis): "price e
  rating são sempre inteiros". Implementação real: inconsistente entre os
  dois campos. Catalogar como `REG-xxx` com a nota
  `[possível bug de precedência, não regra deliberada —
  CONFIRMAR COM O DONO DO PRODUTO]` — não presumir qual dos dois
  comportamentos é o "correto".
- `Restaurant.prototype.validate` (linhas 52-58) — única regra de
  validação do lado servidor hoje: **nome é campo obrigatório**. Preço,
  rating, horário, dias — nenhum é validado no servidor.
- `Restaurant.fromArray` / `MenuItem.fromArray` (linhas 60-73, 81-86) —
  o CSV é lido **por posição de coluna**, não por cabeçalho nomeado. Isso é
  um contrato implícito nunca declarado formalmente: a ordem das colunas
  em `server/data/restaurants.csv` e `menus.csv` É o schema. Documentar
  isso explicitamente no dicionário de dados — é o tipo de regra que
  ninguém escreve numa spec porque "está óbvio no código", até quebrar.

## Armazenamento — `server/storage.js`

`MemoryStorage` é um array em memória, sem índice, sem persistência
própria — a persistência entre reinícios vem inteiramente do par
leitura-no-boot / escrita-no-`SIGINT` em `server/index.js`. Isso é a razão
de existir das specs `001-persistencia-pedidos` e
`002-persistencia-postgres`: qualquer regra sobre "os dados sobrevivem a X"
tem que ser lida em conjunto com essas duas specs, nunca só a partir de
`storage.js` isolado.

## Dados semeados — `server/data/*.csv`, `*.json`

- `restaurants.csv` — cabeçalho nomeado, mas a leitura real (`fromArray`)
  ignora o cabeçalho e usa posição. Ordem real:
  nome, id, cozinha, abre, fecha, dias, preço (1-5), rating (1-5),
  localização, descrição.
- `menus.csv` — cabeçalho `Cuisine,Item Name,Price`, lido por
  `MenuItem.fromArray` como `[cozinha (ignorado no MenuItem), nome, preço]`
  — confirme no código quem realmente consome a coluna "Cuisine" antes de
  presumir que ela é usada; pode ser dado morto.
- `restaurants.json` — formato "carregado direto" no boot do servidor
  (`fs.readFile(DATA_FILE, ...)` em `index.js`), é o snapshot vivo que o
  `SIGINT` reescreve. Difere do CSV, que é a semente original.

## Specs e contratos existentes — sempre a premissa, nunca o alvo

- `.spec/features/<feature>/spec.md` (quando existir) — decisões já
  aprovadas pelo trilho onp-spec-driven.
- `.spec/constituicao.md` — princípios inegociáveis do projeto.

Leia essas fontes **antes** de rastrear o código (passo 1 do `SKILL.md`).
Elas mudam a classificação de toda regra encontrada.
