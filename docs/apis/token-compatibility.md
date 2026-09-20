# Compatibilidade de tokens e credenciais

Nem todo `Authorization: Bearer ...` no Ouros representa o mesmo tipo de credencial.

Esta matriz mostra **o que é aceito por quem** no estado atual.

## Matriz

| Credencial | Origem | Spring | Auth público | Auth interno | AI Server | Knowledge MCP | Telemetry |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| JWT Keycloak do `/v1/auth/token` | Auth broker / Keycloak | ✅ | n/a | ❌ | ❌* | ❌ | ❌ |
| service JWT `keycloak-user-storage` | Keycloak Client Credentials | ❌ | n/a | ✅ | ❌ | ❌ | ❌ |
| `AUTH_BEARER_TOKEN` | AI Server config | ❌ | n/a | ❌ | ✅ | ❌ | ❌ |
| JWT HS256 do AI Server | secret local do AI | ❌ | n/a | ❌ | ✅ | ❌ | ❌ |
| `MCP_AUTH_TOKEN` | Knowledge MCP config | ❌ | n/a | ❌ | ❌ | ✅ | ❌ |
| `API_BEARER_TOKEN` | Telemetry config | ❌ | n/a | ❌ | ❌ | ❌ | ✅ |
| cookie `session` | GitHub Manager | ❌ | n/a | ❌ | ❌ | ❌ | ❌ |

* O AI Server poderia aceitar um JWT externo **somente se** ele fosse assinado em HS256 com o `AUTH_JWT_SECRET` local e satisfizesse issuer/audience configurados. O JWT Keycloak padrão usa o modelo de chaves/JWKS do Keycloak e não é automaticamente compatível.

## 1. JWT Keycloak do usuário

Obtido por:

```http
POST /v1/auth/token
```

Caminho:

```text
ms-auth-service
  ↓
ms-auth-service-broker
  ↓
Keycloak
  ↓ aud=ms-spring-api
access_token
```

Uso correto:

```http
Authorization: Bearer <access_token>
```

em `ms-spring-api`.

O Spring valida:

- assinatura JWKS;
- issuer;
- expiração;
- audience;
- role.

## 2. Service JWT do User Storage

O `keycloak-user-storage` usa Client Credentials e recebe token destinado a:

```text
aud=ms-auth-service-internal
```

Esse token é para:

```text
/internal/v1/**
```

do Auth Service.

Não é token de usuário e não deve ser entregue a web/mobile.

## 3. Credencial do AI Server

### Bearer compartilhado

```text
AUTH_BEARER_TOKEN
```

Autentica o cliente, mas não carrega identidade individual.

Quando `AUTH_REQUIRE_USER_JWT=true`, esse modo não serve para operações personalizadas.

### JWT HS256

```text
AUTH_JWT_SECRET
AUTH_JWT_ISSUER
AUTH_JWT_AUDIENCE
```

É um contrato próprio do AI Server.

Não usa automaticamente o JWKS do Keycloak.

## 4. Credencial MCP

Knowledge MCP atual:

```text
MCP_AUTH_TOKEN
```

O verifier faz comparação exata.

Compatibilidade recomendada com AI Server:

```text
AI Server MCP_ACCESS_TOKEN
        =
Knowledge MCP MCP_AUTH_TOKEN
```

### JWT MCP ainda não interoperável

O AI Server sabe gerar JWT curto via `MCP_JWT_SECRET`.

O Knowledge MCP **não sabe validar esse JWT hoje**.

Resultado se ativar apenas esse caminho:

```text
AI gera JWT
   ↓
MCP compara com string estática
   ↓
401 / tools indisponíveis
```

## 5. Telemetry

Rotas `/v1/dashboards/**` usam:

```text
API_BEARER_TOKEN
```

É token estático.

O resource server já existe no Keycloak, mas a aplicação ainda não foi migrada para validar JWT.

## 6. GitHub Manager

Não usa Bearer nas rotas de criação.

Fluxo:

```text
POST /auth/login
  ↓
Set-Cookie: session=...
  ↓
cookie assinado
  ↓
/templates
/repositories/**
```

`/metrics` é exceção e usa `METRICS_TOKEN`.

## 7. Auto Review

Webhook não usa Bearer.

Usa:

```http
X-Hub-Signature-256: sha256=<hmac>
```

com o webhook secret do GitHub App.

## Diagnóstico rápido de 401

Pergunte nessa ordem:

1. qual serviço estou chamando?
2. qual **tipo** de credencial ele aceita?
3. o token foi emitido para esse audience/resource?
4. o ambiente usa o mesmo secret/config?
5. o token expirou?
6. o usuário possui role/ownership depois de autenticar?

O fato de duas credenciais aparecerem no mesmo header `Authorization: Bearer` não torna elas intercambiáveis.
