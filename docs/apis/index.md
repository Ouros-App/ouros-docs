# APIs do Ouros

Esta seção é a referência de integração entre clientes e serviços. O objetivo é responder quatro perguntas sem exigir leitura do código:

1. **qual serviço chamar**;
2. **como autenticar**;
3. **qual contrato enviar/receber**;
4. **quais erros e regras de autorização esperar**.

## Estado atual

| Serviço | Interface | Responsabilidade | Autenticação atual |
| --- | --- | --- | --- |
| `ms-spring-api` | REST/OpenAPI | domínio transacional | **JWT Keycloak** com issuer + JWKS + audience `ms-spring-api` |
| `ms-auth-service` | REST | login, credenciais e bridge de identidade | rotas públicas + JWT de serviço nas rotas internas |
| `ouros-keycloak` | OIDC/OAuth2 | tokens, roles, audiences e federação | protocolos Keycloak |
| `ms-ai-server` | REST | chat Midas e histórico | JWT Keycloak, audience `ms-ai-server` |
| Knowledge MCP | MCP Streamable HTTP | conhecimento, contexto e importação | JWT Keycloak, audience `ms-mcp-server-ouros-knowledge` |
| Telemetry | REST | dashboards Databricks e renderização | JWT Keycloak, audience própria + role `admin` |
| GitHub Manager | REST + UI | criação/padronização de repos | cookie de sessão assinado |
| Auto Review | webhook HTTP | revisão automática de PRs | assinatura HMAC do GitHub |

!!! important "Spring já migrou para Keycloak"
    O `ms-spring-api` atual não possui mais endpoints de login local. Ele é um OAuth2 Resource Server e valida tokens Keycloak por JWKS, issuer e audience.

## Fluxo Android

```mermaid
sequenceDiagram
    participant U as Usuário
    participant M as Android
    participant K as Keycloak
    participant S as APIs

    M->>K: Authorization Code + PKCE S256
    K->>U: Browser Flow (senha + OTP quando habilitado)
    U->>K: autenticação
    K-->>M: authorization code
    M->>K: code + code_verifier
    K-->>M: access + refresh + id token
    M->>S: Authorization: Bearer <access_token>
```

O client `ouros-mobile` é público, não possui client secret e recebe um access token multi-audience aceito por Spring, AI Server e Telemetry. A audience adicional do Knowledge MCP existe para a delegação interna feita pelo Midas.

## Fluxo legado first-party

```text
cliente legado
  → POST /v1/auth/token no ms-auth-service
  → ms-auth-service-broker
  → Keycloak
  → JWT first-party
```

Esse broker permanece por compatibilidade durante o rollout. O APK novo não deve chamar `POST /v1/auth/token`.

## Qual interface usar?

### Login no Android

Use o **Keycloak** diretamente via Browser Flow, Authorization Code + PKCE S256 com `client_id=ouros-mobile`.

O app não envia senha ao token endpoint e não possui client secret. Veja [Autenticação no Android](../guides/mobile-authentication.md).

### Login legado

```http
POST /v1/auth/token
```

Use o **Auth Service** apenas para consumidores first-party ainda compatíveis com o broker legado.

### CRUD de fazenda, empresa, lotes, água, energia e perfis

Use o **Spring API**.

### Chat Midas

```http
POST /v1/chat
```

Use o **AI Server**. Clientes comuns não devem reproduzir o roteamento de tools MCP diretamente.

### Conhecimento e dados para agentes

Use o **Knowledge MCP** por um consumidor MCP autorizado, normalmente o AI Server.

### Dashboard Databricks

Use o **Telemetry Dashboard Service**.

### Criar/configurar repositório da org

Use o **GitHub Manager**. É uma API interna privilegiada, protegida por sessão.

### Auto review

O **Auto Review** não é uma API de cliente convencional. Ele recebe webhooks do GitHub em `POST /webhooks/github`.

## Fonte de verdade dos contratos

| Tipo | Fonte preferida |
| --- | --- |
| shape HTTP exato | OpenAPI/Swagger da versão implantada |
| autorização/ownership | service layer + esta documentação |
| OIDC/JWT | IaC do `ouros-keycloak` |
| tools MCP | `app/mcp_server.py` + referência Midas/MCP |
| mudanças de banco | repos de database/migrations |

A wiki explica contexto e integração. Quando OpenAPI e texto divergirem, confira o commit/deploy e corrija a documentação.

## Convenções importantes

Não existe um único formato universal de erro entre todos os serviços:

- Spring usa `ProblemDetail`;
- FastAPI normalmente usa `{"detail": ...}`;
- validação Pydantic usa 422;
- Auth rate limit usa 429 + `Retry-After`.

Veja [Convenções de API](conventions.md).

## Referências

- [Spring API](spring-api-reference.md)
- [Auth Service](auth-service-reference.md)
- [Midas e MCP](ai-mcp-reference.md)
- [Telemetry](telemetry-reference.md)
- [Keycloak clients e audiences](keycloak-clients.md)
- [Autenticação e identidade](authentication.md)
- [Exemplos executáveis](examples.md)
