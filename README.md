# mark85-cypress-api

Projeto de automação de testes de API com Cypress para validação dos fluxos de usuários, sessões e tarefas, incluindo suporte a limpeza de massa no MongoDB e validação de mensageria via AMQP.

## Tecnologias e plugins

- Node.js + npm
- Cypress (`^13.3.0`)
- `cypress-plugin-api` (requisições HTTP em testes de API)
- MongoDB Driver (`mongodb`)
- `dotenv` (carregamento de variáveis de ambiente)
- Allure (`@shelex/cypress-allure-plugin` + `allure-commandline`)

## Estrutura do projeto

```text
.
|-- cypress
|   |-- e2e
|   |   |-- sessions.cy.js
|   |   |-- users.cy.js
|   |   `-- tasks
|   |       |-- delete.cy.js
|   |       |-- get.cy.js
|   |       |-- post.cy.js
|   |       `-- put.cy.js
|   |-- fixtures
|   |   |-- users.json
|   |   `-- tasks
|   |       |-- delete.json
|   |       |-- get.json
|   |       |-- post.json
|   |       `-- put.json
|   `-- support
|       |-- e2e.js
|       |-- mongo.js
|       `-- commands
|           |-- amqp.js
|           `-- services.js
|-- cypress.config.js
|-- package.json
`-- .env
```

## Pre-requisitos

- Node.js instalado
- npm instalado
- API alvo disponível e acessível via `BASE_URL`
- Acesso ao MongoDB utilizado pelo projeto
- Acesso ao endpoint AMQP (CloudAMQP API)

## Variáveis de ambiente

Defina as variáveis no arquivo `.env`:

```env
BASE_URL=http://localhost:3333
MONGO_URI=mongodb+srv://<usuario>:<senha>@<cluster>/<database>?retryWrites=true&w=majority
AMQP_HOST=https://<host-rabbitmq>/api/queues/<vhost>
AMQP_QUEUE=tasks
AMQP_TOKEN=Basic <token-base64>
```

## Instalação e execução

Instalar dependências:

```bash
npm install
```

Executar toda a suíte em modo headless:

```bash
npm test
```

Comando equivalente definido no projeto:

```bash
npx cypress run
```

## Comandos customizados Cypress

Os comandos abaixo ficam em `cypress/support/commands/services.js` e `cypress/support/commands/amqp.js`:

- `postUser(user)`
- `postSession(user)`
- `postTask(task, token)`
- `getTasks(token)`
- `getUniqueTask(taskId, token)`
- `deleteTask(taskId, token)`
- `putTaskDone(taskId, token)`
- `purgeMessages()`
- `getMessageQueue()`

## Tasks de banco (Node Events)

Registradas em `cypress.config.js` com conexão em `cypress/support/mongo.js`:

- `removeUser(email)`: remove usuários por e-mail
- `removeTask(taskName, emailUser)`: remove tarefa por nome vinculada ao usuário
- `removeTasksLike(key)`: remove tarefas por padrão de nome

Essas tasks são usadas para preparar massa de teste antes dos cenários.

## Cobertura atual de testes

### Usuários

Arquivo: `cypress/e2e/users.cy.js`

- `POST /users` com sucesso (`201`)
- validação de e-mail duplicado (`409`)
- validações de campos obrigatórios (`400`): `name`, `email`, `password`

### Sessões

Arquivo: `cypress/e2e/sessions.cy.js`

- `POST /sessions` com credenciais válidas (`200`)
- senha inválida (`401`)
- e-mail não encontrado (`401`)

### Tarefas

Arquivos em `cypress/e2e/tasks/`:

- `post.cy.js`
- `get.cy.js`
- `put.cy.js`
- `delete.cy.js`

Cenários cobertos:

- `POST /tasks`
  - criação de tarefa (`201`)
  - validação de duplicidade (`409`)
  - validação de mensagem em fila (AMQP)
- `GET /tasks`
  - listagem de tarefas do usuário autenticado (`200`)
- `GET /tasks/:id`
  - busca por tarefa específica (`200`)
  - tarefa não encontrada (`404`)
- `PUT /tasks/:id/done`
  - atualização para concluída (`204`) e validação de estado
  - tarefa não encontrada (`404`)
- `DELETE /tasks/:id`
  - remoção de tarefa (`204`)
  - remoção de tarefa inexistente (`404`)

## Dados de teste (fixtures)

- `cypress/fixtures/users.json`
- `cypress/fixtures/tasks/post.json`
- `cypress/fixtures/tasks/get.json`
- `cypress/fixtures/tasks/put.json`
- `cypress/fixtures/tasks/delete.json`

As fixtures armazenam perfis de usuário e massas de tarefas utilizadas pelas suítes.

## Relatórios

O projeto possui integração com Allure:

- plugin carregado em `cypress/support/e2e.js`
- writer registrado em `cypress.config.js`
- resultados gerados em `allure-results/`


