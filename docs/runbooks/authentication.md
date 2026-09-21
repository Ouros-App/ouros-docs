# Runbook: falha de autenticação

Use quando login, refresh ou chamadas autenticadas começam a falhar.

## Mapa atual

| Componente | Mecanismo |
| --- | --- |
| Android | Authorization Code + PKCE S256 + Browser Flow |
| Spring API | JWT Keycloak, audience `ms-spring-api` |
| AI Server | JWT Keycloak, audience `ms-ai-server` |
| Knowledge MCP | JWT Keycloak, audience `ms-mcp-server-ouros-knowledge` |
| Telemetry | JWT Keycloak, audience própria + role `admin` |
| Auth interno | service JWT `keycloak-user-storage` |
| AI Debug | Direct Grant isolado em `ms-ai-server-debug` |
| GitHub Manager | cookie de sessão |
| Auto Review | HMAC de webhook |

Veja [Compatibilidade de tokens](../apis/token-compatibility.md).

## Mobile: login não conclui

Fluxo esperado:

```text
Android
  → Keycloak Browser Flow
  → senha validada via User Storage
  → OTP por e-mail
  → authorization code
  → code + PKCE verifier
  → access + refresh + id token
```

Cheque:

1. client `ouros-mobile` existe;
2. Standard Flow está ativo;
3. PKCE S256 está obrigatório;
4. redirect URI usado é exatamente o cadastrado;
5. SMTP do realm está configurado;
6. `OUROS_EMAIL_OTP_ENABLED=true`;
7. User Storage consegue consultar o Auth Service;
8. usuário possui e-mail válido;
9. logs do Keycloak mostram o evento de login.

O app não deve enviar senha diretamente ao endpoint de token.

## OTP não chega

Cheque no Keycloak:

```text
OUROS_SMTP_HOST
OUROS_SMTP_PORT
OUROS_SMTP_AUTH
OUROS_SMTP_STARTTLS
OUROS_SMTP_SSL
OUROS_SMTP_USER
OUROS_SMTP_PASSWORD
OUROS_SMTP_FROM
OUROS_EMAIL_OTP_ENABLED
OUROS_EMAIL_OTP_HMAC_SECRET
```

O reconciliador recusa habilitar OTP sem SMTP.

Valores padrão do desafio:

- expiração: 300 s;
- máximo: 5 tentativas;
- cooldown de reenvio: 30 s.

Nunca registre o OTP em logs.

## Token existe, API responde 401

Cheque:

- assinatura RS256;
- `iss=https://ouros-keycloak.discloud.app/realms/ouros`;
- expiração;
- JWKS acessível;
- audience esperada.

Audiences:

```text
Spring     → ms-spring-api
AI Server  → ms-ai-server
MCP        → ms-mcp-server-ouros-knowledge
Telemetry  → ms-telemetry-dashboard-service
```

Se uma configuração de client/audience acabou de mudar, renove o token.

## API responde 403

A autenticação passou. Não faça novo login automaticamente.

Cheque:

- realm role;
- `account_type`;
- `database_id`;
- ownership do recurso;
- escopo de fazenda/empresa.

No Telemetry, as rotas de dashboard atualmente exigem `admin`. Um usuário mobile comum pode estar autenticado corretamente e receber 403.

## Midas autentica no AI Server, mas tools somem/falham

O AI Server encaminha o mesmo access token ao Knowledge MCP.

Cheque se o token contém simultaneamente:

```text
ms-ai-server
ms-mcp-server-ouros-knowledge
```

Depois confirme issuer/JWKS no MCP.

## Refresh falha

O refresh token conversa somente com o Keycloak.

Cheque:

- `client_id=ouros-mobile`;
- refresh token ainda válido;
- sessão não revogada;
- client ainda habilitado.

Se o refresh não puder mais renovar a sessão, o mobile deve limpar tokens locais e iniciar um novo Browser Flow.

## Auth interno falha

O User Storage usa:

```text
client=keycloak-user-storage
aud=ms-auth-service-internal
```

Um 401 nas rotas internas aponta para autenticação service-to-service. Credencial humana inválida é tratada separadamente pelo bridge de identidade.

## Debug do AI Server falha

O console `/debug` usa `ms-ai-server-debug`, não `ouros-mobile`.

Cheque:

- `DEBUG_UI_ENABLED=true`;
- client ID;
- client secret atual;
- Direct Access Grant;
- token endpoint;
- audience do AI Server e do Knowledge MCP.

Não use esse fluxo no Android.

## Evidências úteis

Guarde sem secrets:

- timestamp;
- serviço/rota;
- status;
- request ID;
- client ID;
- issuer;
- audiences;
- role/account type;
- versão/commit.

Nunca copie access token, refresh token, client secret, senha ou OTP para logs/tickets.
