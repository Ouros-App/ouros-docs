# Exemplos de API

Exemplos usam placeholders e não contêm credenciais reais.

## Variáveis shell

```bash
export AUTH_URL="https://ms-auth-service.discloud.app"
export KEYCLOAK_ISSUER="https://ouros-keycloak.discloud.app/realms/ouros"
export AI_URL="https://ms-ai-server.discloud.app"
export SPRING_URL="<spring-api-url>"
export TELEMETRY_URL="<telemetry-url>"
export ACCESS_TOKEN="<access-token>"
```

## Keycloak discovery

```bash
curl -fsS "$KEYCLOAK_ISSUER/.well-known/openid-configuration"
```

JWKS:

```bash
curl -fsS "$KEYCLOAK_ISSUER/protocol/openid-connect/certs"
```

## Login first-party

```bash
curl -sS "$AUTH_URL/v1/auth/token" \
  -H 'content-type: application/json' \
  --data-binary '{
    "email": "usuario@example.com",
    "password": "<senha>"
  }'
```

Resposta esperada contém access token emitido pelo Keycloak.

Nunca coloque senha real em histórico de shell compartilhado ou documentação.

## Verificar credencial

```bash
curl -sS "$AUTH_URL/v1/auth/credentials/verify" \
  -H 'content-type: application/json' \
  --data-binary '{
    "email": "usuario@example.com",
    "password": "<senha>",
    "account_type": "farm_owner"
  }'
```

## Readiness do Auth

```bash
curl -i "$AUTH_URL/ready"
```

Interpretação:

- 200: dependências prontas;
- 503: processo vivo, mas dependência/config não pronta.

## Chat Midas

```bash
curl -sS "$AI_URL/v1/chat" \
  -H "authorization: Bearer $ACCESS_TOKEN" \
  -H 'content-type: application/json' \
  --data-binary '{
    "user_id": "42",
    "message": "Como está meu consumo de energia?",
    "thread_id": "exemplo-thread-001"
  }'
```

Resposta contém:

- `thread_id`;
- `message`;
- `agents`;
- `tools`.

## Histórico do Midas

```bash
curl -sS "$AI_URL/v1/chat/exemplo-thread-001/history" \
  -H "authorization: Bearer $ACCESS_TOKEN"
```

A thread precisa pertencer ao usuário autenticado.

## Métricas do AI Server

```bash
curl -sS "$AI_URL/metrics" \
  -H "authorization: Bearer $ACCESS_TOKEN"
```

## Spring API: health

```bash
curl -i "$SPRING_URL/health"
```

## Spring API: recurso autenticado

Exemplo genérico de leitura:

```bash
curl -sS "$SPRING_URL/farms" \
  -H "authorization: Bearer $ACCESS_TOKEN"
```

!!! note
    O Spring API ainda pode usar JWT legado durante a migração. Use o token esperado pelo ambiente específico.

## Telemetry: readiness

```bash
curl -i "$TELEMETRY_URL/ready"
```

## Telemetry: listar dashboards

```bash
curl -sS "$TELEMETRY_URL/v1/dashboards" \
  -H "authorization: Bearer $ACCESS_TOKEN"
```

No serviço atual, o Bearer pode ser token estático de API, não necessariamente JWT Keycloak.

## Telemetry: listar charts

```bash
curl -sS "$TELEMETRY_URL/v1/dashboards/<dashboard-id>/charts" \
  -H "authorization: Bearer $ACCESS_TOKEN"
```

## Telemetry: PNG

```bash
curl -fsS "$TELEMETRY_URL/v1/dashboards/<dashboard-id>/charts/<chart-id>/png" \
  -H "authorization: Bearer $ACCESS_TOKEN" \
  -o chart.png
```

## Telemetry: request ID

```bash
curl -i "$TELEMETRY_URL/health" \
  -H 'X-Request-ID: debug-123'
```

Use o mesmo ID ao procurar logs.

## Knowledge MCP: health

Base pública observada como exemplo:

```bash
export MCP_URL="https://ms-midas-mcp.discloud.app"
curl -i "$MCP_URL/health"
```

O endpoint MCP em si usa protocolo MCP Streamable HTTP e autenticação específica; não trate uma chamada `curl` simples como substituto do cliente MCP.

## GitHub Manager local

Login:

```bash
curl -i http://localhost:8000/auth/login \
  -H 'content-type: application/json' \
  --data-binary '{
    "username": "admin",
    "password": "<senha>"
  }' \
  -c cookies.txt
```

Listar templates com sessão:

```bash
curl -sS http://localhost:8000/templates -b cookies.txt
```

## Dicas de diagnóstico

Adicionar verbosidade:

```bash
curl -v ...
```

Ver apenas headers:

```bash
curl -I ...
```

Preservar status sem falhar o shell:

```bash
curl -sS -o response.json -w '%{http_code}\n' ...
```

## Segurança nos exemplos

Nunca cole em issue/PR:

- access token real;
- refresh token;
- senha;
- API key;
- cookie de sessão;
- private key.
