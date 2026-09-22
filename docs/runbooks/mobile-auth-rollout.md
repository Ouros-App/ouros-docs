# Runbook: rollout da autenticação mobile

Este runbook é para quem opera backend/infra. A pessoa que implementa o Android deve usar apenas o guia [Autenticação no Android](../guides/mobile-authentication.md).

## Estado desejado

O Android autentica uma vez no Keycloak com Authorization Code + PKCE S256 e recebe um access token para:

```text
ms-spring-api
ms-ai-server
ms-telemetry-dashboard-service
ms-ai-server-mcp-exchange
```

A última audience não é um endpoint mobile. Ela permite que o AI Server use o token como `subject_token` em Standard Token Exchange v2.

O Knowledge MCP aceita somente o token delegado:

```text
aud=ms-mcp-server-ouros-knowledge
azp=ms-ai-server-mcp-exchange
```

## Ordem de rollout

A ordem importa porque o client confidencial de exchange nasce no Keycloak.

### 1. Reconciliar Keycloak

Publique primeiro o `ouros-keycloak` com:

- `ouros-mobile`;
- `ms-ai-server-mcp-exchange`;
- Standard Token Exchange v2 habilitado no client de exchange;
- audiences/scopes do contrato mobile.

Confirme no Admin Console:

```text
Clients
→ ms-ai-server-mcp-exchange
→ Credentials
```

O client deve ser confidencial e possuir client secret.

### 2. Bootstrap do secret no AI Server

Copie o secret do client `ms-ai-server-mcp-exchange` para o ambiente de produção do AI Server:

```text
Infisical
environment: prod
path: /ms-ai-server

MCP_KEYCLOAK_TOKEN_EXCHANGE_CLIENT_SECRET=<valor atual do client>
```

Não coloque esse valor em Git, docs, Android ou logs.

As demais configurações têm defaults versionados:

```text
MCP_KEYCLOAK_TOKEN_EXCHANGE_URL=https://ouros-keycloak.discloud.app/realms/ouros/protocol/openid-connect/token
MCP_KEYCLOAK_TOKEN_EXCHANGE_CLIENT_ID=ms-ai-server-mcp-exchange
MCP_KEYCLOAK_TOKEN_EXCHANGE_AUDIENCE=ms-mcp-server-ouros-knowledge
MCP_KEYCLOAK_TOKEN_EXCHANGE_TIMEOUT_SECONDS=8
```

### 3. Publicar AI Server

Publique o `ms-ai-server` com token exchange habilitado.

Critério de segurança:

- falha de exchange não pode fazer fallback para o JWT original;
- tools MCP ficam indisponíveis quando a troca falha;
- nenhum client secret aparece em logs.

### 4. Validar delegação antes de endurecer o MCP

Use uma sessão real ou o console de debug do AI Server e execute uma pergunta que use Knowledge MCP.

Procure nas traces/logs apenas eventos sanitizados:

```text
mcp.token_exchange_failed
mcp.tools_loaded
mcp.tools_load_failed
```

Sucesso significa que as tools são carregadas sem erro de exchange.

Não registre access token, refresh token ou client secret durante o diagnóstico.

### 5. Publicar Knowledge MCP

Depois que o AI Server já estiver fazendo exchange, publique o Knowledge MCP com:

```text
MCP_JWT_ISSUER=https://ouros-keycloak.discloud.app/realms/ouros
MCP_JWT_AUDIENCE=ms-mcp-server-ouros-knowledge
MCP_JWT_AUTHORIZED_PARTY=ms-ai-server-mcp-exchange
```

A partir daqui, um JWT emitido diretamente para `ouros-mobile` deve ser rejeitado mesmo que seja um JWT Keycloak válido.

## Teste mobile E2E

Depois do client mobile reconciliado e de SMTP/OTP habilitados:

```bash
python3 scripts/test-mobile-auth.py
```

O smoke test valida:

1. Browser Flow;
2. PKCE S256;
3. OTP dentro do login;
4. assinatura RS256/JWKS;
5. `azp=ouros-mobile`;
6. audiences do contrato mobile;
7. refresh token;
8. novo access token após refresh.

Para diagnóstico explícito de APIs:

```bash
python3 scripts/test-mobile-auth.py --output ./mobile-auth-tokens.json
```

Apague o arquivo ao terminar.

## Verificações finais

O rollout está completo quando:

- `ouros-mobile` existe e não possui client secret;
- Direct Access Grant está desligado no mobile;
- PKCE S256 está obrigatório;
- o token mobile não contém `ms-mcp-server-ouros-knowledge`;
- o token mobile contém `ms-ai-server-mcp-exchange`;
- AI Server consegue obter token delegado;
- token delegado contém `aud=ms-mcp-server-ouros-knowledge`;
- token delegado contém `azp=ms-ai-server-mcp-exchange`;
- Knowledge MCP rejeita token com `azp=ouros-mobile`;
- Spring e AI Server aceitam o mesmo access token mobile;
- Telemetry autentica o mesmo token e continua aplicando a role `admin`;
- refresh funciona sem novo OTP enquanto a sessão puder ser renovada.

## Rollback

Se a delegação ao MCP falhar após o deploy:

1. não reduza a validação do MCP;
2. não adicione a audience do MCP ao `ouros-mobile`;
3. não encaminhe o JWT mobile diretamente;
4. reverta temporariamente a versão do AI Server/MCP conforme necessário;
5. corrija client secret, audience ou configuração do exchange;
6. refaça o teste antes de reativar o caminho.

O boundary de segurança é parte do contrato, não uma otimização.
