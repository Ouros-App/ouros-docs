# Compatibilidade de tokens e credenciais

Nem todo `Authorization: Bearer ...` no Ouros representa o mesmo contexto de autenticação. O ecossistema principal usa JWT RS256 emitido pelo Keycloak, mas cada resource server valida a própria audience e pode aplicar autorização adicional.

## Matriz atual

| Credencial | Origem | Spring | AI Server | Knowledge MCP | Telemetry | Auth interno |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| JWT `ouros-mobile` | Authorization Code + PKCE | ✅ | ✅ | ✅* | ✅** | ❌ |
| JWT do broker legado | `ms-auth-service-broker` | ✅ | ✅ | ✅ | ✅** | ❌ |
| JWT `ms-ai-server-debug` | Direct Grant interno | ❌ | ✅ | ✅ | ❌ | ❌ |
| service JWT `keycloak-user-storage` | Client Credentials | ❌ | ❌ | ❌ | ❌ | ✅ |

\* O Android não chama o Knowledge MCP diretamente. A audience existe porque o AI Server encaminha o mesmo JWT ao MCP quando o Midas usa tools.

\** O Telemetry autentica o JWT, mas as rotas atuais de dashboard exigem também a realm role `admin`. Usuários autenticados sem essa role recebem `403`.

GitHub Manager e Auto Review usam mecanismos separados, respectivamente cookie de sessão e assinatura HMAC de webhook.

## JWT mobile

O mobile usa:

```text
client_id     = ouros-mobile
flow          = Authorization Code + PKCE S256
redirect_uri  = com.ourosapp.ourosandroidapp:/oauth2redirect
scopes        = openid ouros-identity
```

O access token possui as audiences:

```text
ms-spring-api
ms-ai-server
ms-telemetry-dashboard-service
ms-mcp-server-ouros-knowledge
```

As três primeiras são APIs mobile-facing. A quarta permite delegação interna AI Server → Knowledge MCP sem um segundo login.

O refresh token é enviado somente ao endpoint de token do Keycloak.

## Validação por resource server

### Spring API

Valida assinatura/JWKS, issuer, timestamps e:

```text
aud contains ms-spring-api
```

Depois aplica role e ownership de negócio.

### AI Server

Valida JWT RS256 do Keycloak e:

```text
aud contains ms-ai-server
```

A identidade autenticada é propagada para o grafo e o access token validado pode ser encaminhado ao Knowledge MCP.

### Knowledge MCP

Valida JWT RS256 do Keycloak e:

```text
aud contains ms-mcp-server-ouros-knowledge
```

Não usa token estático como contrato atual.

### Telemetry Dashboard

Valida JWT RS256 do Keycloak e:

```text
aud contains ms-telemetry-dashboard-service
realm_access.roles contains admin
```

Audience válida não concede role.

## Service JWT do User Storage

O client `keycloak-user-storage` usa Client Credentials e recebe token com:

```text
aud=ms-auth-service-internal
```

Esse token é exclusivo das rotas `/internal/v1/**` do Auth Service. Não representa usuário e nunca deve ir para web/mobile.

## Debug do AI Server

O client `ms-ai-server-debug` é uma exceção operacional:

```text
Direct Access Grant
aud=ms-ai-server
aud=ms-mcp-server-ouros-knowledge
```

Ele existe apenas para o console `/debug` e não é contrato do aplicativo Android.

## Broker legado

`ms-auth-service-broker` continua disponível durante a transição. Ele pode emitir JWTs Keycloak para consumers first-party legados, mas novos clientes mobile não devem chamar `POST /v1/auth/token`.

O caminho novo é sempre Browser Flow + Authorization Code + PKCE.

## Diagnóstico rápido de 401/403

Cheque nesta ordem:

1. qual serviço recebeu o request;
2. `iss` do token;
3. `exp` e `iat`;
4. se o `aud` contém a audience do serviço;
5. se a assinatura resolve contra o JWKS do realm;
6. para `403`, role e ownership exigidos;
7. se o token foi emitido antes de uma mudança recente de client scopes, gere/renove outro.

Não use `403` como gatilho para relogar. `403` significa que a autenticação passou e a autorização negou a operação.
