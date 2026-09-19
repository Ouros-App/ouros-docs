# ms-ouros-front-web

**Stack atual:** React 19, TypeScript, Vite 7, React Router, Tailwind CSS, DOMPurify.

**Repo:** [Ouros-App/ms-ouros-front-web](https://github.com/Ouros-App/ms-ouros-front-web)

## Estado real

!!! warning "Ainda é essencialmente o template"
    O código atual permanece muito próximo de `ms-webfront-template`. O próprio `package.json` ainda usa o nome `ms-webfront-template` e o README aponta badges para o template.

Não trate este repo como frontend integrado ao backend de produção ainda.

## Rotas atuais

| Rota | Proteção | Componente |
| --- | --- | --- |
| `/` | pública | Home |
| `/login` | pública | Login |
| `/products` | `PrivateRoute` | Products |
| `/products/:id` | `PrivateRoute` | ProductDetail |
| `*` | pública | NotFound |

Essas rotas ainda são genéricas do scaffold, não representam o domínio final do Ouros.

## Autenticação atual

`AuthContext` mantém estado apenas no React por `useReducer`.

A tela de login atual:

1. valida formato do email e presença da senha;
2. **não chama o backend**;
3. injeta um usuário demo;
4. usa token literal de demonstração;
5. redireciona para `/`.

Portanto:

!!! danger
    O login atual é placeholder de UI. Ele não deve ser interpretado como autenticação real.

O caminho de integração esperado deve usar o contrato de identidade do Ouros, preferencialmente Keycloak/BFF conforme a arquitetura adotada, e nunca armazenar refresh token em localStorage.

## Cliente HTTP

`apiClient.ts`:

- usa `VITE_API_URL`;
- envia `Content-Type: application/json`;
- lança erro genérico para status não-2xx;
- não injeta `Authorization`;
- não faz refresh;
- não trata 401/403 de forma central;
- não implementa retry/correlation ID.

Default atual:

```text
http://localhost:8000/api
```

## Services atuais

### `userService`

Contratos genéricos:

- `GET /users/me`;
- `POST /auth/login`.

Essas rotas não correspondem diretamente ao contrato atual do `ms-auth-service`.

### `productService`

Contratos demo:

- `GET /products`;
- `GET /products/{id}`.

“Products” não é recurso central implementado no backend Ouros atual. Trate como scaffold.

## Estrutura

```text
src/
├── components/
│   ├── common/
│   ├── complex/
│   └── layout/
├── context/
├── hooks/
├── pages/
├── reducers/
├── routes/
├── services/
├── types/
└── utils/
```

## Configuração

`.env.example`/Vite usa:

- `APP_NAME`;
- `VITE_APP_TITLE`;
- `VITE_API_URL`;
- `VITE_STORAGE_VERSION`;
- `PORT`.

## Build

```bash
npm install
npm run dev
npm run lint
npm run build
npm run preview
```

Docker:

- build em Node 22 Alpine;
- publicação estática em Nginx 1.27 Alpine.

## O que falta para virar frontend Ouros

No estado atual, os principais blocos são:

1. substituir login demo pelo fluxo de autenticação real;
2. definir estratégia segura de sessão/token;
3. trocar services genéricos por contratos do domínio;
4. consumir Spring API/Auth/AI/telemetry conforme papéis;
5. implementar autorização por role;
6. modelar páginas de fazenda, consumo, lotes, metas e Midas;
7. adicionar testes de componente/integração;
8. revisar offline/cache se o requisito web demandar;
9. alinhar nome/metadados do pacote e README.

## Relações

- autenticação: `ms-auth-service` / `ouros-keycloak`;
- domínio: `ms-spring-api`;
- Midas: `ms-ai-server`;
- dashboards: `ms-telemetry-dashboard-service`.
