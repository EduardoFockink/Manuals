# Swagger

**Data de criação:** 2026-03-29
**Descrição:** Referência prática sobre Swagger/OpenAPI — estrutura de specs, integração com Elixir/Phoenix e Node.js/Next.js, e como realizar testes completos de APIs.

---

## Sumário

1. [O que é Swagger / OpenAPI](#1-o-que-é-swagger--openapi)
2. [Estrutura de uma especificação OpenAPI](#2-estrutura-de-uma-especificação-openapi)
3. [Integração com Elixir/Phoenix](#3-integração-com-elixirphoenix)
4. [Integração com Node.js/Next.js](#4-integração-com-nodejsnextjs)
5. [Swagger UI — navegando a interface](#5-swagger-ui--navegando-a-interface)
6. [Testando APIs com Swagger](#6-testando-apis-com-swagger)

---

## Corpo do documento

### 1. O que é Swagger / OpenAPI

**Para que serve:**
Documentar APIs REST de forma padronizada e legível por máquinas. A spec gerada alimenta ferramentas como Swagger UI (interface interativa), geradores de cliente, e suítes de teste.

**Como funciona:**
OpenAPI é o padrão (especificação); Swagger é o ecossistema de ferramentas em torno dele. Você descreve sua API num arquivo YAML ou JSON seguindo o schema OpenAPI 3.x. Esse arquivo pode ser gerado manualmente, via anotações no código, ou automaticamente pelo framework.

**Principais formas de uso:**
- **Spec-first** — escreve o YAML antes do código; o contrato define a implementação
- **Code-first** — anota o código existente; a spec é gerada a partir das anotações
- **Auto-gerada pelo framework** — o próprio framework (ex: `open_api_spex`) reflete as rotas e gera a spec

---

### 2. Estrutura de uma especificação OpenAPI

**Para que serve:**
Descrever todos os endpoints da API: rotas, métodos HTTP, parâmetros, corpo da requisição, respostas e schemas de dados.

**Como funciona:**
O arquivo segue uma hierarquia fixa. Os schemas em `components` são reutilizados via `$ref` para evitar repetição.

```yaml
openapi: "3.0.3"

info:
  title: Minha API
  version: "1.0.0"

paths:
  /users/{id}:
    get:
      summary: Busca usuário por ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        "200":
          description: Usuário encontrado
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "404":
          description: Não encontrado

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

**Principais parâmetros:**
- `openapi` — versão da especificação (ex: `"3.0.3"`)
- `info.title` / `info.version` — metadados da API
- `paths` — mapa de rotas; cada rota contém os métodos HTTP como chaves
- `parameters[].in` — onde o parâmetro está: `path`, `query`, `header`, `cookie`
- `requestBody.content` — corpo da requisição com media type (ex: `application/json`)
- `responses` — mapa de status HTTP com descrição e schema de retorno
- `components.schemas` — schemas reutilizáveis referenciados via `$ref`
- `components.securitySchemes` — define autenticação (Bearer, API Key, OAuth2)

---

### 3. Integração com Elixir/Phoenix

**Para que serve:**
Gerar e servir automaticamente a spec OpenAPI a partir das rotas e schemas do Phoenix, sem manter um arquivo YAML separado.

**Como funciona:**
A lib `open_api_spex` reflete as rotas Phoenix e usa módulos Elixir como schemas. Cada controller action recebe uma anotação `@doc` com a spec da operação. A spec final é gerada em tempo de compilação e servida via plug.

**Principais formas de uso:**

**1. Instalação**
```elixir
# mix.exs
{:open_api_spex, "~> 3.18"}
```

**2. Definir o spec base**
```elixir
defmodule MyAppWeb.ApiSpec do
  alias OpenApiSpex.{Info, OpenApi, Paths, Server}
  @behaviour OpenApiSpex.OpenApi

  def spec do
    %OpenApi{
      info: %Info{title: "Minha API", version: "1.0.0"},
      servers: [%Server{url: "http://localhost:4000"}],
      paths: Paths.from_router(MyAppWeb.Router)
    }
    |> OpenApiSpex.resolve_schema_modules()
  end
end
```

**3. Adicionar o plug no router**
```elixir
# router.ex
pipeline :api do
  plug OpenApiSpex.Plug.PutApiSpec, module: MyAppWeb.ApiSpec
end

scope "/api" do
  pipe_through :api
  get "/openapi", OpenApiSpex.Plug.RenderSpec, []
end

# Swagger UI em /swaggerui
scope "/" do
  get "/swaggerui", OpenApiSpex.Plug.SwaggerUI,
    path: "/api/openapi"
end
```

**4. Anotar um controller**
```elixir
defmodule MyAppWeb.UserController do
  use MyAppWeb, :controller
  use OpenApiSpex.ControllerSpecs

  operation :show,
    summary: "Busca usuário",
    parameters: [
      id: [in: :path, type: :integer, required: true]
    ],
    responses: [
      ok: {"Usuário", "application/json", MyApp.Schemas.UserResponse},
      not_found: {"Não encontrado", "application/json", nil}
    ]

  def show(conn, %{"id" => id}) do
    # ...
  end
end
```

**5. Definir um schema**
```elixir
defmodule MyApp.Schemas.UserResponse do
  require OpenApiSpex
  alias OpenApiSpex.Schema

  OpenApiSpex.schema(%{
    title: "UserResponse",
    type: :object,
    properties: %{
      id: %Schema{type: :integer},
      name: %Schema{type: :string},
      email: %Schema{type: :string}
    },
    required: [:id, :name, :email]
  })
end
```

**6. Validação automática do request**
```elixir
# Adiciona o plug no controller para validar params/body
plug OpenApiSpex.Plug.CastAndValidate, json_render_error_v2: true
```

---

### 4. Integração com Node.js/Next.js

**Para que serve:**
Documentar e servir a spec OpenAPI em projetos Node com Express ou rotas de API do Next.js, sem sair do JavaScript.

**Como funciona:**
`swagger-jsdoc` extrai anotações JSDoc dos arquivos de rota e monta o objeto OpenAPI. `swagger-ui-express` (para Express) ou um endpoint customizado (para Next.js) serve a interface Swagger UI.

**Principais formas de uso:**

**1. Instalação**
```bash
npm install swagger-jsdoc swagger-ui-express
# ou para Next.js sem express:
npm install swagger-jsdoc next-swagger-doc
```

**2. Configuração base (Express)**
```js
// swagger.js
const swaggerJsdoc = require('swagger-jsdoc')

const options = {
  definition: {
    openapi: '3.0.3',
    info: { title: 'Minha API', version: '1.0.0' },
  },
  apis: ['./routes/*.js'],  // arquivos com anotações JSDoc
}

module.exports = swaggerJsdoc(options)
```

```js
// app.js
const swaggerUi = require('swagger-ui-express')
const swaggerSpec = require('./swagger')

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec))
```

**3. Anotar uma rota (Express)**
```js
/**
 * @openapi
 * /users/{id}:
 *   get:
 *     summary: Busca usuário por ID
 *     parameters:
 *       - name: id
 *         in: path
 *         required: true
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: Usuário encontrado
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       404:
 *         description: Não encontrado
 */
router.get('/users/:id', async (req, res) => {
  // ...
})
```

**4. Integração com Next.js (App Router)**
```js
// lib/swagger.js
import { createSwaggerSpec } from 'next-swagger-doc'

export function getApiDocs() {
  return createSwaggerSpec({
    apiFolder: 'app/api',  // pasta com as rotas
    definition: {
      openapi: '3.0.3',
      info: { title: 'Minha API', version: '1.0.0' },
    },
  })
}
```

```js
// app/api/docs/route.js
import { getApiDocs } from '@/lib/swagger'

export async function GET() {
  return Response.json(getApiDocs())
}
```

```js
// app/docs/page.jsx — serve o Swagger UI
'use client'
import SwaggerUI from 'swagger-ui-react'
import 'swagger-ui-react/swagger-ui.css'

export default function DocsPage() {
  return <SwaggerUI url="/api/docs" />
}
```

**5. Anotar uma rota Next.js**
```js
// app/api/users/[id]/route.js

/**
 * @openapi
 * /api/users/{id}:
 *   get:
 *     summary: Busca usuário
 *     parameters:
 *       - name: id
 *         in: path
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: OK
 */
export async function GET(request, { params }) {
  // ...
}
```

---

### 5. Swagger UI — navegando a interface

**Para que serve:**
Explorar visualmente os endpoints da API, ler a documentação e disparar requisições diretamente pelo browser sem precisar de curl ou Postman.

**Como funciona:**
O Swagger UI carrega a spec OpenAPI (YAML ou JSON) e renderiza cada endpoint como um painel expansível. Você preenche os parâmetros nos campos gerados automaticamente e executa a chamada. A resposta é exibida com status, headers e body.

**Principais formas de uso:**
- **Expandir um endpoint** — clica no método HTTP + rota para abrir o painel
- **"Try it out"** — habilita os campos de entrada para preenchimento manual
- **Preencher parâmetros** — campos de `path`, `query`, `header` e `body` são exibidos separadamente
- **"Execute"** — dispara a requisição real contra o servidor configurado em `servers`
- **Autorizar** — botão "Authorize" no topo aplica token/API key a todas as requisições da sessão
- **Filtrar** — campo de busca no topo filtra endpoints por nome ou tag
- **Modelos** — seção "Schemas" no final lista todos os tipos definidos em `components/schemas`
- **Curl gerado** — após executar, o UI exibe o comando `curl` equivalente à requisição

---

### 6. Testando APIs com Swagger

**Para que serve:**
Validar o comportamento real dos endpoints usando a spec como contrato: verificar se respostas batem com os schemas declarados, testar casos de erro, e garantir que a documentação reflete a implementação.

**Como funciona:**
O Swagger UI serve para testes exploratórios manuais diretamente no browser. Para testes automatizados, ferramentas como `schemathesis` (contract testing) ou `jest` + `supertest` consomem a spec e validam as respostas da API contra os schemas declarados.

**Principais formas de uso:**

**Teste manual no Swagger UI**
1. Abra o Swagger UI (`/swaggerui`, `/api-docs`, ou `/docs`)
2. Clique no endpoint desejado → "Try it out"
3. Preencha os parâmetros obrigatórios (marcados com `*`)
4. Para endpoints com body, edite o JSON no campo "Request body"
5. Clique "Execute" e verifique:
   - **Status code** bate com o esperado (200, 201, 404, etc.)
   - **Response body** segue a estrutura do schema documentado
   - **Headers** contêm os campos esperados (ex: `Content-Type`, `Location`)
6. Para endpoints autenticados, clique "Authorize" antes e informe o token Bearer ou API key

**Testando autenticação**
```
1. Clique em "Authorize" (cadeado no topo)
2. Em BearerAuth: informe o token sem o prefixo "Bearer" (o UI adiciona automaticamente)
3. Confirme com "Authorize" → "Close"
4. Todos os endpoints da sessão passarão o header Authorization automaticamente
```

**Teste de contrato automatizado com Schemathesis (Python)**
```bash
# Instalar
pip install schemathesis

# Rodar contra a spec servida pelo servidor local
schemathesis run http://localhost:4000/api/openapi --checks all

# Rodar contra um arquivo local
schemathesis run ./openapi.yaml --base-url http://localhost:4000

# Flags úteis
--checks all              # valida status codes, schemas de resposta, e headers
--hypothesis-max-examples 100  # número de casos gerados por endpoint
--validate-schema true    # valida a própria spec antes de testar
--workers 4               # paraleliza os testes
```

**Teste de contrato com Jest + Supertest (Node.js)**
```js
// __tests__/api.test.js
const request = require('supertest')
const Ajv = require('ajv')
const app = require('../app')
const spec = require('../swagger-output.json')

const ajv = new Ajv()

test('GET /users/:id responde com schema correto', async () => {
  const res = await request(app).get('/users/1')
  expect(res.status).toBe(200)

  const schema = spec.components.schemas.User
  const valid = ajv.validate(schema, res.body)
  expect(valid).toBe(true)
})
```

**Teste de contrato com ExUnit + open_api_spex (Elixir/Phoenix)**
```elixir
# test/my_app_web/controllers/user_controller_test.exs
defmodule MyAppWeb.UserControllerTest do
  use MyAppWeb.ConnCase
  alias OpenApiSpex.TestAssertions

  test "GET /api/users/:id retorna schema válido", %{conn: conn} do
    conn = get(conn, ~p"/api/users/1")
    assert json_response(conn, 200)
    # Valida a resposta contra o schema declarado na spec
    TestAssertions.assert_schema(json_response(conn, 200), "UserResponse", MyAppWeb.ApiSpec.spec())
  end
end
```

**Checklist de testes completos por endpoint:**
- [ ] Caso feliz (200/201): body segue o schema, headers corretos
- [ ] Parâmetros inválidos: retorna 400/422 com mensagem de erro
- [ ] Recurso inexistente: retorna 404
- [ ] Sem autenticação: retorna 401
- [ ] Sem permissão: retorna 403
- [ ] Body malformado (JSON inválido): retorna 400
- [ ] Campos obrigatórios ausentes: retorna 422 com indicação do campo
