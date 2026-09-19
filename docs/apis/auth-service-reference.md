# Auth Service: referência de API

Contrato atual de `ms-auth-service`.

## Base pública

Produção referenciada no ecossistema:

```text
https://ms-auth-service.discloud.app
```

## Meta

### GET `/`

Disponibilidade básica.

### GET `/health`

Liveness.

Resposta:

```json
{"status":"ok"}
```

### GET `/ready`

Valida PostgreSQL e Redis quando configurado.

Sucesso:

```json
{"status":"ready"}
```

Falha de dependência: **503**.

## POST `/v1/auth/credentials/verify`

Verifica credencial sem emitir token.

Request:

```json
{
  "email": "usuario@example.com",
  "password": "<senha>",
  "account_type": "farm_owner"
}
```

`account_type` é opcional.

Tipos:

```text
farm_owner
company_employee
admin
```

Regras:

- email 3..255;
- password é tratado como `SecretStr`;
- password passa por limite de tamanho no validator;
- email é normalizado/validado pelo schema.

Sucesso:

```json
{
  "authenticated": true,
  "identity": {
    "id": 42,
    "email": "usuario@example.com",
    "account_type": "farm_owner",
    "realm_role": "farm_owner",
    "name": "Nome",
    "farm_id": 7,
    "enterprise_id": null,
    "first_access": false
  }
}
```

Campos contextuais podem ser nulos.

Sem `account_type`, o serviço busca candidatos suportados. Identidade ambígua pode resultar em **409**.

Credencial inválida é tratada como **401** no contrato público.

Rate limit pode gerar **429**.

## POST `/v1/auth/token`

Login first-party que devolve tokens emitidos pelo Keycloak.

Request:

```json
{
  "email": "usuario@example.com",
  "password": "<senha>"
}
```

Response:

```json
{
  "access_token": "<access-token>",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "<refresh-token>",
  "token_type": "Bearer",
  "scope": "openid ouros-identity"
}
```

Os tempos acima são apenas forma de exemplo. Use os valores reais retornados pelo Keycloak.

Regras do schema:

- `access_token`: não vazio;
- `expires_in > 0`;
- `refresh_expires_in >= 0` quando presente;
- `token_type`: literalmente `Bearer`.

## Rotas internas

Prefixo:

```text
/internal/v1
```

Não aparecem no schema OpenAPI público.

Todas exigem service JWT válido do fluxo Keycloak User Storage.

### GET `/internal/v1/identities/by-email?email=...`

Lookup por email.

Query:

- email 3..255.

Não encontrado: **404**.

### GET `/internal/v1/identities/{account_type}/{database_id}`

Lookup por identidade federada.

Não encontrado: **404**.

### POST `/internal/v1/credentials/verify`

Validação de password para o provider.

Senha humana inválida: **403**.

Falha de autenticação do próprio service token: **401**.

Essa distinção é intencional.

## Identidade retornada

```text
id
email
account_type
realm_role
name?
farm_id?
enterprise_id?
first_access?
```

O campo `id` é o ID legado de banco, não o `sub` Keycloak.

## Rate limit

Defaults observados:

```text
IP burst:        3 / 10 s
IP:              5 / min
IP:             20 / 15 min
email:           5 / 15 min
```

Redis torna o estado compartilhado entre réplicas.

## Segurança para clientes

- nunca logar request de login;
- nunca persistir password;
- não mostrar refresh token em console;
- tratar 429 respeitando `Retry-After`;
- não usar `credentials/verify` como substituto de sessão;
- usar o access token Keycloak para resource servers que já migraram.

## Separação de erros

| Status | Interpretação típica |
| --- | --- |
| 401 | autenticação pública inválida ou service token inválido |
| 403 | credencial humana inválida em rota interna/autorização |
| 404 | identidade interna não encontrada |
| 409 | identidade pública ambígua |
| 429 | rate limit |
| 503 | readiness/dependência indisponível |
