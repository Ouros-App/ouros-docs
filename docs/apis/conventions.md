# Convenções de API

Convenções **observadas** no ecossistema atual. Onde os serviços divergem, a divergência é documentada em vez de fingir uniformidade.

## Autenticação

### JWT Keycloak

Usado pelo `ms-spring-api`.

```http
Authorization: Bearer <access-token>
```

O resource server valida:

- assinatura via JWKS;
- `iss`;
- timestamps;
- `aud=ms-spring-api`;
- role reconhecida.

### Bearer estático

Usado atualmente por Telemetry e Knowledge MCP.

O header é igual, mas **o token não é um JWT Keycloak**.

### AI Server

Aceita dois modos:

1. Bearer compartilhado (`AUTH_BEARER_TOKEN`);
2. JWT HS256 (`AUTH_JWT_SECRET`), opcionalmente validando issuer/audience.

Não confunda esse JWT com o fluxo JWKS do Keycloak.

### Sessão

GitHub Manager usa cookie `session` assinado por HMAC.

## Content-Type

Para JSON:

```http
Content-Type: application/json
```

Respostas de gráfico podem ser:

- `image/png`;
- `text/html`;
- JSON.

MCP usa Streamable HTTP conforme o protocolo MCP.

## Naming

Payloads de domínio do Spring usam majoritariamente **snake_case**:

```json
{
  "id_farm": 1,
  "registration_date": "2026-09-19",
  "energy_consumption": 450.75
}
```

Alguns DTOs aceitam aliases camelCase por compatibilidade. Para clientes novos, prefira a forma canônica documentada/OpenAPI.

## Datas

Use ISO 8601.

Data sem horário:

```text
2026-09-19
```

Timestamp:

```text
2026-09-19T18:45:00Z
```

## Status HTTP

| Status | Semântica comum |
| ---: | --- |
| 200 | leitura/atualização concluída |
| 201 | recurso criado; Spring também envia `Location` |
| 202 | operação aceita e continua assíncrona |
| 204 | remoção concluída sem corpo |
| 400 | regra/entrada semanticamente inválida |
| 401 | autenticação ausente ou inválida |
| 403 | autenticado, mas sem autorização/ownership |
| 404 | recurso não encontrado |
| 409 | conflito de integridade/estado |
| 422 | validação Pydantic/FastAPI |
| 429 | rate limit |
| 502 | integração externa falhou |
| 503 | serviço/configuração/dependência não pronta |
| 504 | dependência externa excedeu timeout |

## Erros Spring: ProblemDetail

Exemplo de validação:

```json
{
  "type": "about:blank",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Erro de validação nos campos da requisição",
  "errors": {
    "email": "E-mail inválido"
  },
  "timestamp": "2026-09-19T18:45:00Z"
}
```

Conflitos de banco viram 409.

## Erros FastAPI

Formato simples:

```json
{
  "detail": "Invalid or missing bearer token"
}
```

Validação 422 normalmente retorna `detail` como lista de erros.

O GitHub Manager remove o campo `input` da resposta de validação para evitar ecoar passwords/secrets.

## Rate limit

Auth Service responde 429 e inclui:

```http
Retry-After: <segundos>
```

Cliente deve respeitar o header, não fazer retry imediato.

## Request ID

Telemetry aceita/gera `X-Request-ID` e o devolve nos logs/contexto.

Use um valor não sensível:

```http
X-Request-ID: mobile-7f31c9
```

## Paginação

Não existe paginação uniforme.

- histórico do Midas: cursor `before` + `limit`;
- várias listas Spring retornam todo o escopo autorizado;
- tools MCP aplicam limites próprios;
- Databricks pode paginar internamente sem expor esse mecanismo ao cliente do Telemetry.

Não invente `page`/`offset` se o endpoint não documenta.

## Retry

Seguro normalmente:

- GET idempotente;
- 502/503/504 com backoff quando o contrato permitir;
- 429 respeitando `Retry-After`.

Cuidado:

- POST de criação não deve ser repetido cegamente;
- importação Midas usa `request_id` para idempotência;
- mutations de banco precisam de contrato explícito.

## Secrets e exemplos

Use placeholders:

```text
<access-token>
<senha>
<client-secret>
```

Nunca copie valor real para README, issue, PR ou curl versionado.
