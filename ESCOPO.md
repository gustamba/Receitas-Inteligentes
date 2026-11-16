# Escopo funcional do projeto

A aplicação terá três funcionalidades principais: cadastrar receitas, controlar os ingredientes que você possui e descobrir o que consegue cozinhar.

1. Cadastro de receitas
- Nome, descrição e tempo de preparo.
- Lista de ingredientes e respectivas quantidades.
- Modo de preparo dividido em etapas ordenadas.
- Categoria, como almoço, sobremesa ou lanche.
- Edição, consulta e exclusão de receitas.

2. Minha geladeira e meu armário
- Cadastro de ingredientes disponíveis.
- Quantidade e unidade de medida.
- Local de armazenamento: geladeira, freezer ou armário.
- Atualização e remoção de ingredientes.
- Possibilidade de indicar ingredientes que estão acabando.

3. O que posso cozinhar?
- Listar receitas que você consegue preparar.
- Mostrar receitas que precisam de poucos ingredientes adicionais.
- Informar os ingredientes que estão faltando.
- Exibir o percentual de ingredientes disponíveis.
- Abrir a receita e consultar o passo a passo.

# Tecnologias e arquitetura

## Frontend — Angular + TypeScript
Telas, formulários, navegação e comunicação HTTP

## API REST — Spring Boot
Controllers, validações, regras de negócio e serviços

## Persistência — Spring Data JPA
Entidades, repositórios e consultas ao banco

## PostgreSQL
Receitas, ingredientes, quantidades e etapas

A comunicação será feita por HTTP usando JSON. O Angular não acessará o banco de dados diretamente: todas as operações passarão pela API Java.

# Modelo de entidades

## 1. RECIPE

| Campo | Descrição |
|---|---|
| `id` | Identificador da receita |
| `name` | Nome da receita |
| `description` | Descrição da receita |
| `preparation_time` | Tempo de preparo |
| `category` | Categoria da receita |
| `created_at` | Data de criação |

**Relacionamento:** Uma receita possui várias etapas e vários ingredientes.

## 2. RECIPE_STEP

| Campo | Descrição |
|---|---|
| `id` | Identificador da etapa |
| `recipe_id` | Referência à receita |
| `step_number` | Número da etapa |
| `instruction` | Instrução de preparo |

## 3. RECIPE_INGREDIENT

| Campo | Descrição |
|---|---|
| `recipe_id` | Referência à receita |
| `ingredient_id` | Referência ao ingrediente |
| `quantity` | Quantidade utilizada |
| `unit` | Unidade de medida |

**Relacionamento:** Ingrediente compartilhado entre receitas e estoque.

## 4. INGREDIENT

| Campo | Descrição |
|---|---|
| `id` | Identificador do ingrediente |
| `name` | Nome do ingrediente |
| `default_unit` | Unidade padrão |

## 5. PANTRY_ITEM

| Campo | Descrição |
|---|---|
| `id` | Identificador do item do estoque |
| `ingredient_id` | Referência ao ingrediente |
| `quantity` | Quantidade disponível |
| `unit` | Unidade de medida |
| `location` | Local onde o item está armazenado |

---

## Relacionamentos entre entidades

| Relacionamento | Descrição |
|---|---|
| `RECIPE` → `RECIPE_STEP` | Uma receita possui várias etapas. |
| `RECIPE` → `RECIPE_INGREDIENT` | Uma receita possui vários ingredientes. |
| `INGREDIENT` → `RECIPE_INGREDIENT` | Um ingrediente pode ser compartilhado entre várias receitas. |
| `INGREDIENT` → `PANTRY_ITEM` | Um ingrediente pode estar relacionado ao estoque doméstico. |

A estrutura acima representa o relacionamento lógico. Na implementação, `RECIPE_INGREDIENT` e `PANTRY_ITEM` terão suas próprias chaves primárias, e as referências serão protegidas por chaves estrangeiras.


# Como funcionará a inteligência das receitas?

Vamos imaginar que você tenha estes ingredientes em casa:

## SEU ESTOQUE
`Arroz`, `Ovos`, `Tomate`, `Cebola`, `Sal`

## RECEITAS ENCONTRADAS

### Ovos mexidos com tomate
`Disponível`
Todos os ingredientes necessários estão disponíveis, considerando as quantidades exigidas.


### Arroz com frango
`Falta ingrediente`
Falta frango. O sistema mostra o que precisa ser comprado para preparar a receita.

# Algoritmo de recomendação — primeira versão

Para cada receita, o backend deverá:

1. Buscar os ingredientes exigidos pela receita.
2. Buscar os ingredientes disponíveis no estoque.
3. Comparar os ingredientes e suas respectivas quantidades.
4. Identificar os ingredientes totalmente disponíveis e os que estão faltando.
5. Calcular o percentual de ingredientes disponíveis.
6. Classificar a receita como disponível ou incompleta.
7. Retornar os resultados ordenados por prioridade.

### Regra inicial de cálculo

O percentual de ingredientes disponíveis será calculado pela seguinte fórmula:

$$
Percentual (%) = (Ingredientes disponíveis / Total de ingredientes exigidos) * 100
$$


# Organização das pastas e arquivos

```text
receitas-inteligentes/
├── frontend/
├── backend/
├── database/
│   └── init.sql
├── docs/
│   ├── arquitetura.md
│   └── modelo-dados.md
├── docker-compose.yml
├── .gitignore
└── README.md
```

## Backend — Java + Spring Boot

```text
backend/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/br/com/receitas/
    │   │   ├── ReceitasApplication.java
    │   │   ├── config/
    │   │   ├── controller/
    │   │   │   ├── RecipeController.java
    │   │   │   ├── IngredientController.java
    │   │   │   ├── PantryController.java
    │   │   │   └── RecommendationController.java
    │   │   ├── service/
    │   │   │   ├── RecipeService.java
    │   │   │   ├── IngredientService.java
    │   │   │   ├── PantryService.java
    │   │   │   └── RecipeRecommendationService.java
    │   │   ├── repository/
    │   │   ├── entity/
    │   │   ├── dto/
    │   │   ├── mapper/
    │   │   └── exception/
    │   └── resources/
    │       ├── application.yml
    │       └── db/migration/
    └── test/
        └── java/br/com/receitas/
```

### Responsabilidade de cada camada:

| Camada | O que faz |
|---|---|
|`controller`|Recebe requisições HTTP e retorna respostas|
|`service`|Implementa as regras de negócio|
|`repository`|Consulta e persiste dados no PostgreSQL|
|`entity`|Representa as entidades persistidas|
|`dto`|Define os dados recebidos e enviados pela API|
|`mapper`|Converte entidades em DTOs e vice-versa|
|`exception`|Trata erros de negócio e respostas de erro|
|`config`|Configurações da aplicação|

## Frontend — Angular

```text
frontend/
├── package.json
├── angular.json
└── src/
    ├── app/
    │   ├── core/
    │   │   ├── services/
    │   │   ├── interceptors/
    │   │   └── models/
    │   ├── shared/
    │   │   ├── components/
    │   │   └── pipes/
    │   ├── features/
    │   │   ├── recipes/
    │   │   │   ├── pages/
    │   │   │   ├── components/
    │   │   │   └── services/
    │   │   ├── ingredients/
    │   │   ├── pantry/
    │   │   └── recommendations/
    │   ├── app.routes.ts
    │   └── app.config.ts
    ├── environments/
    └── styles.css
```
No Angular, cada funcionalidade terá suas próprias telas, componentes e serviços. Por exemplo:
- recipes/pages/recipe-list — lista de receitas.
- recipes/pages/recipe-form — cadastro e edição.
- pantry/pages/pantry-list — estoque doméstico.
- recommendations/pages/recommendation-list — sugestões de pratos.
- core/services/api — configurações ou recursos compartilhados de acesso à API, caso sejam necessários.

# Dependências necessárias
## Backend: Spring Initializr
Acesse Spring Initializr e configure:

|Tipo|Descrição|
|---|---|
|Project|Maven|
|Language|Java|
|Packaging|Jar|
|Group|br.com.receitas|
|Artifact|receitas-api|
|Java|21 ou 25 LTS|

Adicione estas dependências:
|Dependência|Finalidade|
|---|---|
|Spring Web|Criar a API REST|
|Spring Data JPA|Persistência e consultas|
|PostgreSQL Driver|Conectar ao PostgreSQL|
|Validation|Validar dados de entrada|
|Spring Boot DevTools|Facilitar o desenvolvimento local|
|Flyway Migration|Versionar o esquema do banco|
|Spring Boot Actuator|Monitoramento básico da aplicação|

Para os testes, utilize JUnit 5 e Mockito, normalmente já disponíveis por meio do Spring Boot Starter Test. O suporte ao Testcontainers com PostgreSQL é uma boa evolução para testar com um banco real em contêiner.

O Flyway merece atenção: mantenha as alterações estruturais do banco em arquivos versionados, como V1__create_initial_schema.sql, dentro de src/main/resources/db/migration/. Isso ajuda a reproduzir o banco em qualquer ambiente.

### Frontend: Angular

Instale o Node.js em uma versão LTS compatível com a versão do Angular escolhida e crie o projeto pelo CLI oficial.

```bash
npm install -g @angular/cli

ng new receitas-frontend --routing --style=css

cd receitas-frontend

ng serve
```

Durante a criação, escolha a configuração de SSR somente se você decidir que precisa de renderização no servidor; para o MVP, uma SPA tradicional é suficiente.

Dependências recomendadas:
- Angular CLI: criação e manutenção do projeto.
- Angular HttpClient: comunicação com o backend.
- Angular Reactive Forms: formulários com validação.
- Angular Router: navegação entre páginas.
- Angular Material: biblioteca opcional de componentes visuais.
- Vitest ou a configuração de testes oferecida pela versão escolhida: testes unitários do frontend.
Angular Material não é obrigatório. Você pode começar com CSS e componentes simples e adicioná-lo quando quiser padronizar formulários, diálogos, tabelas e navegação.

Links oficiais para consulta:
- Documentação do Angular
- Documentação do Spring Boot
- Documentação do PostgreSQL
- Documentação do Flyway

# Endpoints da API REST
Antes de desenvolver as telas, vale definir o contrato entre o Angular e o Spring Boot. Isso deixa claro quais operações existirão e quais dados cada uma deverá receber e devolver.

Use /api/v1 como prefixo para facilitar a evolução futura da API.
|Método|Endpoint|Objetivo|
|---|---|---|
|GET|/api/v1/recipes|Listar receitas|
|GET|/api/v1/recipes/{id}|Consultar uma receita|
|POST|/api/v1/recipes|Cadastrar uma receita|
|PUT|/api/v1/recipes/{id}|Atualizar uma receita|
|DELETE|/api/v1/recipes/{id}|Excluir uma receita|
|GET|/api/v1/ingredients|Listar ingredientes|
|POST|/api/v1/ingredients|Cadastrar um ingrediente no catálogo|
|GET|/api/v1/pantry|Consultar o estoque doméstico|
|POST|/api/v1/pantry|Adicionar ingrediente ao estoque|
|PUT|/api/v1/pantry/{id}|Atualizar quantidade ou localização|
|DELETE|/api/v1/pantry/{id}|Remover ingrediente do estoque|
|GET|/api/v1/recommendations|Consultar receitas recomendadas|

O endpoint de recomendações poderá aceitar filtros opcionais no futuro, como:

`GET /api/v1/recommendations?maxMissingIngredients=2`

Isso permitiria retornar receitas que precisam de, no máximo, dois ingredientes adicionais.
Exemplo: cadastrar uma receita
Requisição `POST /api/v1/recipes:`
```JSON
{
  "name": "Omelete com tomate",
  "description": "Omelete simples para o café da manhã",
  "preparationTimeMinutes": 15,
  "category": "BREAKFAST",
  "ingredients": [
    {
      "ingredientId": 1,
      "quantity": 2,
      "unit": "UNIT"
    },
    {
      "ingredientId": 2,
      "quantity": 100,
      "unit": "GRAM"
    }
  ],
  "steps": [
    {
      "stepNumber": 1,
      "instruction": "Pique o tomate."
    },
    {
      "stepNumber": 2,
      "instruction": "Bata os ovos e misture os ingredientes."
    },
    {
      "stepNumber": 3,
      "instruction": "Cozinhe em uma frigideira até atingir o ponto desejado."
    }
  ]
}
```

Os IDs acima são ilustrativos: os ingredientes precisam existir no catálogo antes de serem associados à receita.

Observe que a receita é enviada em um único documento JSON. O backend pode salvar a receita, os ingredientes associados e as etapas em uma transação, garantindo que o cadastro não fique parcialmente gravado em caso de erro.

# Fluxo completo de dados
Vamos usar a funcionalidade de recomendações como exemplo, pois ela é o principal diferencial do sistema.

1. Usuário abre “O que posso cozinhar?”
Angular solicita os dados de recomendações

2. RecommendationController
Recebe a requisição HTTP

3. RecipeRecommendationService
Aplica as regras e calcula os ingredientes faltantes

4. Repositories + PostgreSQL
Consultam receitas, ingredientes e estoque

5. DTO de resposta
Retorna disponibilidade, percentual e itens faltantes

6. Angular exibe os resultados
Cards de receitas com filtros e detalhes


A regra mais importante desse fluxo é que o backend continua sendo responsável pela decisão. O Angular apenas apresenta o resultado, sem duplicar o algoritmo.
