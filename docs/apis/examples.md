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

## Login legado → Spring API

Este exemplo usa o broker legado para diagnóstico. **Não implemente esse fluxo no Android**. O mobile usa Authorization Code + PKCE conforme [Autenticação no Android](../guides/mobile-authentication.md).

Para um diagnóstico legado sem colocar a senha nos argumentos do `curl`:

```bash
read -r -p "Email: " OUROS_EMAIL
read -r -s -p "Senha: " OUROS_PASSWORD
printf '\n'

ACCESS_TOKEN="$(
  jq -n \
    --arg email "$OUROS_EMAIL" \
    --arg password "$OUROS_PASSWORD" \
    '{email:$email,password:$password}' |
  curl -fsS "$AUTH_URL/v1/auth/token" \
    -H 'content-type: application/json' \
    --data-binary @- |
  jq -r .access_token
)"

unset OUROS_PASSWORD
```

Chamar Spring:

```bash
curl -fsS "$SPRING_URL/farms" \
  -H "Authorization: Bearer $ACCESS_TOKEN" |
  jq
```

O broker existe para compatibilidade. Para o fluxo mobile real, gere um token pelo Browser Flow + PKCE e carregue-o antes dos exemplos seguintes:

```bash
python3 scripts/test-mobile-auth.py --output ./mobile-auth-tokens.json
ACCESS_TOKEN="$(jq -er '.initial.access_token' ./mobile-auth-tokens.json)"
trap 'rm -f ./mobile-auth-tokens.json' EXIT
```

O arquivo contém access/refresh/id token e deve existir apenas durante o diagnóstico.

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
read -r -p "Email: " OUROS_EMAIL
read -r -s -p "Senha: " OUROS_PASSWORD
printf '\n'

jq -n \
  --arg email "$OUROS_EMAIL" \
  --arg password "$OUROS_PASSWORD" \
  '{email:$email,password:$password,account_type:"farm_owner"}' |
curl -fsS "$AUTH_URL/v1/auth/credentials/verify" \
  -H 'content-type: application/json' \
  --data-binary @- |
jq

unset OUROS_PASSWORD
```

## AI Server

No fluxo mobile, o mesmo JWT Keycloak usado no Spring também possui audience `ms-ai-server`.

```bash
curl -fsS "$AI_URL/v1/chat" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H 'content-type: application/json' \
  --data-binary '{
    "message":"Como está meu consumo de energia?"
  }' | jq
```

O backend deriva a identidade do JWT; não troque de usuário fornecendo identificadores arbitrários.

## Histórico Midas

```bash
curl -fsS "$AI_URL/v1/chat/<thread-id>/history?limit=20" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | jq
```

## Telemetry

O mesmo access token mobile contém a audience do Telemetry. As rotas atuais também exigem role `admin`.

```bash
curl -fsS "$TELEMETRY_URL/v1/dashboards" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | jq
```

Um usuário autenticado sem role `admin` recebe 403.

PNG:

```bash
curl -fsS "$TELEMETRY_URL/v1/dashboards/<dashboard-id>/charts/<chart-id>/png" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -o chart.png
```

## Knowledge MCP health

```bash
export MCP_URL="https://ms-midas-mcp.discloud.app"
curl -fsS "$MCP_URL/health" | jq
```

Chamadas em `/mcp/` exigem cliente MCP e JWT Keycloak com audience `ms-mcp-server-ouros-knowledge`. No caminho normal, o Android não chama o MCP diretamente: o AI Server encaminha o JWT autenticado.

## Diagnóstico HTTP

Status + body:

```bash
code="$(
  curl -sS -o /tmp/ouros-response.json -w '%{http_code}'     "$SPRING_URL/farms"     -H "Authorization: Bearer $ACCESS_TOKEN"
)"
printf 'HTTP %s
' "$code"
jq . /tmp/ouros-response.json 2>/dev/null || cat /tmp/ouros-response.json
```

## Regra de segurança

Nunca cole em docs/issues/PRs:

- access token real;
- refresh token;
- password;
- API key;
- cookie de sessão;
- private key.
