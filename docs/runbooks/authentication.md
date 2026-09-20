# Runbook: falha de autenticação

Use quando login falha ou aparecem 401/403 em massa.

## 1. Identifique o serviço

Antes de olhar claims, descubra **qual mecanismo de autenticação a rota usa**.

| Serviço | Mecanismo |
| --- | --- |
| Spring API | JWT Keycloak via JWKS + audience |
| Auth público | credenciais humanas / sem Bearer |
| Auth interno | service JWT Keycloak |
| AI Server | Bearer estático ou JWT HS256 local |
| Knowledge MCP | Bearer estático |
| Telemetry | Bearer estático |
| GitHub Manager | cookie de sessão |
| Auto Review | HMAC de webhook |

Veja [Compatibilidade de tokens](../apis/token-compatibility.md).

## 2. Login first-party não emite token

Fluxo:

```text
Client → Auth Service → Keycloak → User Storage/Auth → PostgreSQL
```

Cheque:

1. `GET /health`;
2. `GET /ready`;
3. PostgreSQL;
4. Redis/rate limiter;
5. Keycloak issuer/token endpoint;
6. client `ms-auth-service-broker`;
7. broker secret;
8. User Storage.

### 429

Respeite `Retry-After`.

Não desative o limiter para “testar”.

## 3. Token existe, Spring responde 401

Cheque o JWT:

- assinatura;
- `iss`;
- `exp`;
- `aud` contém `ms-spring-api`;
- JWKS acessível;
- uma role reconhecida existe.

O token retornado por `/v1/auth/token` deve vir do broker com audience do Spring.

Se o token foi emitido antes de uma mudança de client scope/audience, gere um token novo.

## 4. Spring responde 403

Autenticação passou. O problema agora é autorização.

Cheque:

- role;
- `database_id`/lookup local;
- enterprise do funcionário;
- farm do produtor;
- ownership do recurso.

Exemplo:

> um FARM_OWNER autenticado não ganha direito de editar outra fazenda apenas porque conhece o ID.

## 5. Auth interno responde 401

Cheque o service token de `keycloak-user-storage`:

- issuer;
- audience `ms-auth-service-internal`;
- client autorizado;
- expiração.

## 6. Auth interno responde 403

Na rota interna de credencial, 403 significa **credencial humana inválida**.

O 401 é reservado para a autenticação do serviço.

## 7. AI Server

### 401

Cheque:

- `AUTH_BEARER_TOKEN`; ou
- `AUTH_JWT_SECRET` + issuer/audience.

O AI Server não usa automaticamente o JWKS Keycloak.

### 403

Com `AUTH_REQUIRE_USER_JWT=true`:

- token compartilhado não basta para dados pessoais;
- `user_id` do request precisa coincidir com a identidade autenticada.

## 8. Knowledge MCP

O verifier atual aceita apenas igualdade com `MCP_AUTH_TOKEN`.

Se o AI Server estiver configurado somente com `MCP_JWT_SECRET`, as tools podem desaparecer/falhar porque o MCP não valida esse JWT.

Modo interoperável atual:

```text
AI MCP_ACCESS_TOKEN == MCP MCP_AUTH_TOKEN
```

## 9. Telemetry

Rotas de negócio usam `API_BEARER_TOKEN`.

- token ausente/errado: 401;
- token não configurado no servidor: 503.

Não use o access token Keycloak do Spring esperando compatibilidade.

## 10. GitHub Manager

Rotas de negócio dependem do cookie `session`.

Se login funciona mas API retorna 401:

- confirme armazenamento/envio do cookie;
- Secure em HTTPS;
- expiração;
- `SESSION_SECRET` consistente após restart/redeploy.

## 11. Evidências úteis

Guarde sem secrets:

- timestamp;
- serviço/rota;
- status;
- request ID;
- issuer esperado;
- audiences do token;
- client ID;
- role;
- versão/commit.

Nunca copie o token inteiro.

## 12. Depois do incidente

- reproduza em teste;
- corrija readiness/alerta quando aplicável;
- atualize docs;
- remova workaround temporário.
