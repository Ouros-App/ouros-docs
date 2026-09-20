# Telemetry Dashboard: referência de API

Contrato atual do `ms-telemetry-dashboard-service`.

## Auth

Rotas de negócio usam token estático:

```http
Authorization: Bearer <API_BEARER_TOKEN>
```

Não há validação de JWT Keycloak no código atual.

O IaC já possui resource server `ms-telemetry-dashboard-service`, então a infraestrutura para migração existe, mas a aplicação ainda usa `compare_digest` contra o token configurado.

## Rotas

| Método | Rota | Auth | Resposta |
| --- | --- | --- | --- |
| GET | `/` | pública | mensagem |
| GET | `/health` | pública | liveness |
| GET | `/ready` | pública | readiness |
| GET | `/metrics` | pública | Prometheus |
| GET | `/v1/dashboards` | Bearer estático | JSON |
| GET | `/v1/dashboards/{id}` | Bearer estático | JSON |
| GET | `/v1/dashboards/{id}/charts` | Bearer estático | JSON |
| GET | `/v1/dashboards/{id}/charts/{chart_id}/png` | Bearer estático | image/png |
| GET | `/v1/dashboards/{id}/charts/{chart_id}/chartjs` | Bearer estático | text/html |

!!! warning "Metrics público"
    `/metrics` não exige Bearer na versão atual.

## Readiness

`/ready` valida configuração obrigatória, incluindo:

- `API_BEARER_TOKEN`;
- `DATABRICKS_HOST`;
- `DATABRICKS_CLIENT_ID`;
- `DATABRICKS_CLIENT_SECRET`.

Pronto:

```json
{"status":"ok","errors":[]}
```

Não pronto: 503 com lista sanitizada em `errors`.

## Dashboards

### Listar

```http
GET /v1/dashboards
```

```json
{
  "items": [
    {
      "id": "operacao",
      "title": "Operação",
      "description": "Indicadores",
      "provider": "databricks"
    }
  ]
}
```

### Dashboard individual

```http
GET /v1/dashboards/{dashboard_id}
```

404 quando não existe/visível.

### Charts

```http
GET /v1/dashboards/{dashboard_id}/charts
```

Tipos públicos:

```text
counter
bar
line
pie
```

### PNG

```http
GET /v1/dashboards/{dashboard_id}/charts/{chart_id}/png
```

Header:

```http
Cache-Control: private, max-age=30
Content-Type: image/png
```

### Chart.js

```http
GET /v1/dashboards/{dashboard_id}/charts/{chart_id}/chartjs
```

Retorna HTML self-contained/iframe-friendly.

Header:

```http
Cache-Control: no-store
```

O JSON embutido é escapado para `<`, `>` e `&`, e o título passa por escape HTML.

## Erros

| Status | Causa |
| ---: | --- |
| 401 | Bearer ausente/incorreto |
| 404 | dashboard/chart não encontrado |
| 502 | integração Databricks falhou |
| 503 | auth/config não configurada ou readiness falhou |
| 504 | request Databricks excedeu timeout |

Se `API_BEARER_TOKEN` não estiver configurado, uma rota protegida retorna **503**, não 401.

## Request ID

O serviço suporta `X-Request-ID` para correlação.

```bash
curl -i "$TELEMETRY_URL/health"   -H 'X-Request-ID: debug-telemetry-001'
```

## Retry/cache

O cliente Databricks já aplica timeout/retry configurado. Evite empilhar retries agressivos no consumidor.

Cache de charts: default 30 s.

## Migração futura para Keycloak

Resource server já versionado:

```text
CLIENT_ID=ms-telemetry-dashboard-service
AUDIENCE=ms-telemetry-dashboard-service
```

Uma migração segura deve aceitar JWT Keycloak em paralelo, migrar consumidores e só depois remover o token estático.
