# ms-telemetry-dashboard-service

**Stack:** Python 3.12, FastAPI, Databricks APIs, Chart.js, renderização PNG.

**Repo:** [Ouros-App/ms-telemetry-dashboard-service](https://github.com/Ouros-App/ms-telemetry-dashboard-service)

## Responsabilidade

API para descobrir dashboards acessíveis no Databricks, listar seus gráficos e renderizar visualizações em formatos consumíveis por outros clientes.

A fonte principal é o workspace Databricks. O catálogo local `data/dashboards.json` está vazio no snapshot atual.

## Fluxo

```text
FastAPI route
   ↓
DashboardService
   ↓
DatabricksDashboardProvider
   ↓
DatabricksHttpClient + OAuth
   ↓
Databricks workspace / SQL
```

## Rotas públicas

| Método | Rota | Uso |
| --- | --- | --- |
| GET | `/` | Disponibilidade. |
| GET | `/health` | Liveness. |
| GET | `/ready` | Valida configuração obrigatória. |
| GET | `/metrics` | Prometheus. |

## Rotas protegidas por Bearer

| Método | Rota | Retorno |
| --- | --- | --- |
| GET | `/v1/dashboards` | Dashboards ativos visíveis. |
| GET | `/v1/dashboards/{dashboard_id}` | Metadados de um dashboard. |
| GET | `/v1/dashboards/{dashboard_id}/charts` | Lista de gráficos. |
| GET | `/v1/dashboards/{dashboard_id}/charts/{chart_id}/png` | PNG. |
| GET | `/v1/dashboards/{dashboard_id}/charts/{chart_id}/chartjs` | HTML self-contained com Chart.js. |

## Erros

- `404`: dashboard/chart não encontrado;
- `502`: falha de integração Databricks;
- `504`: timeout Databricks;
- `503` em `/ready`: configuração incompleta/inválida.

## Chart.js

A rota Chart.js serializa metadados e rows em JSON, escapa caracteres relevantes para HTML e monta uma página independente.

Tipos como `counter` recebem tratamento específico. Para gráficos normais, o código usa encodings quando disponíveis e fallback para os primeiros fields.

## Cache

`CHART_CACHE_TTL_SECONDS` controla cache de gráfico, com default 30 s. PNG também envia `Cache-Control: private, max-age=30`; HTML Chart.js usa `no-store`.

## Configuração

Obrigatórios para readiness:

- `DATABRICKS_HOST`;
- `DATABRICKS_CLIENT_ID`;
- `DATABRICKS_CLIENT_SECRET`;
- configuração Keycloak completa (issuer + audience);
- `KEYCLOAK_REQUIRED_ROLE` válida.

Outros:

- token URL;
- timeout/retries/backoff;
- margem de refresh;
- TTL de cache;
- timeout SQL;
- CORS explícito.

`DATABRICKS_HOST` e token URL precisam ser HTTPS. `CORS_ORIGINS` rejeita `*`.

## Autenticação

As rotas de negócio validam JWT RS256 do Keycloak por issuer/JWKS e audience `ms-telemetry-dashboard-service`. Depois da autenticação, o serviço exige a realm role `admin`.

O token mobile já possui a audience necessária, mas produtores/funcionários continuam recebendo 403 até existir autorização user-scoped para dashboards.

## Observabilidade

- logs JSON;
- request ID;
- status e duração;
- tentativas Databricks;
- métricas Prometheus;
- sem logging de tokens/secrets/query payloads.

## Testes

Cobrem API, auth, charts, configuração, service, provider HTTP, Infisical, logging e rotas.
