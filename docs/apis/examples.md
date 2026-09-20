# Exemplos de API

Exemplos seguros com placeholders.

## Ambiente

```bash
export AUTH_URL="https://ms-auth-service.discloud.app"
export KEYCLOAK_ISSUER="https://ouros-keycloak.discloud.app/realms/ouros"
export SPRING_URL="<spring-api-url>"
export AI_URL="<ai-server-url>"
export TELEMETRY_URL="<telemetry-url>"
```

## Descobrir OIDC

```bash
curl -fsS "$KEYCLOAK_ISSUER/.well-known/openid-configuration" | jq
```

## Login → Spring API

Obter token:

```bash
ACCESS_TOKEN="$(
  curl -fsS "$AUTH_URL/v1/auth/token"     -H 'content-type: application/json'     --data-binary '{"email":"usuario@example.com","password":"<senha>"}' |
  jq -r .access_token
)"
```

Chamar Spring:

```bash
curl -fsS "$SPRING_URL/farms"   -H "Authorization: Bearer $ACCESS_TOKEN" | jq
```

O broker atual possui audience `ms-spring-api`.

## Criar registro de energia

```bash
curl -fsS "$SPRING_URL/energy-registries"   -X POST   -H "Authorization: Bearer $ACCESS_TOKEN"   -H 'content-type: application/json'   --data-binary '{
    "registration_date": "2026-09-19",
    "energy_consumption": 450.75,
    "id_farm": 1
  }' | jq
```

Para `farm_owner`, `id_farm` pode ser omitido e o backend usa a fazenda vinculada.

## Auth readiness

```bash
curl -i "$AUTH_URL/ready"
```

## Verificar credencial sem emitir token

```bash
curl -fsS "$AUTH_URL/v1/auth/credentials/verify"   -H 'content-type: application/json'   --data-binary '{
    "email":"usuario@example.com",
    "password":"<senha>",
    "account_type":"farm_owner"
  }' | jq
```

## AI Server

O token depende da configuração do próprio AI Server e **não deve ser assumido como o mesmo token Keycloak do Spring**.

```bash
export AI_TOKEN="<token-aceito-pelo-ai-server>"

curl -fsS "$AI_URL/v1/chat"   -H "Authorization: Bearer $AI_TOKEN"   -H 'content-type: application/json'   --data-binary '{
    "user_id":"42",
    "message":"Como está meu consumo de energia?"
  }' | jq
```

## Histórico Midas

```bash
curl -fsS   "$AI_URL/v1/chat/<thread-id>/history?user_id=42&limit=20"   -H "Authorization: Bearer $AI_TOKEN" | jq
```

## Telemetry

```bash
export TELEMETRY_TOKEN="<API_BEARER_TOKEN>"

curl -fsS "$TELEMETRY_URL/v1/dashboards"   -H "Authorization: Bearer $TELEMETRY_TOKEN" | jq
```

Esse token é estático no contrato atual, não JWT Keycloak.

PNG:

```bash
curl -fsS   "$TELEMETRY_URL/v1/dashboards/<dashboard-id>/charts/<chart-id>/png"   -H "Authorization: Bearer $TELEMETRY_TOKEN"   -o chart.png
```

## Knowledge MCP health

```bash
export MCP_URL="https://ms-midas-mcp.discloud.app"
curl -fsS "$MCP_URL/health" | jq
```

Chamadas em `/mcp/` exigem cliente MCP e `Authorization: Bearer <MCP_AUTH_TOKEN>`.

## Diagnóstico HTTP

Status + body:

```bash
code="$(
  curl -sS -o /tmp/ouros-response.json -w '%{http_code}'     "$SPRING_URL/farms"     -H "Authorization: Bearer $ACCESS_TOKEN"
)"
printf 'HTTP %s
' "$code"
cat /tmp/ouros-response.json | jq .
```

## Regra de segurança

Nunca cole em docs/issues/PRs:

- access token real;
- refresh token;
- password;
- API key;
- cookie de sessão;
- private key.
