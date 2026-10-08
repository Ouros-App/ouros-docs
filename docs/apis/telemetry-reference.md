# Telemetry Dashboard: referência de API

Contrato atual do `ms-telemetry-dashboard-service`.

## Auth

Rotas de negócio validam JWT RS256 emitido pelo Keycloak:

```http
Authorization: Bearer <access_token>
```

Configuração esperada:

```text
issuer   = https://ouros-keycloak.discloud.app/realms/ouros
audience = ms-telemetry-dashboard-service
role     = admin
```

O serviço resolve a chave pelo JWKS, valida issuer/audience/timestamps e exige a realm role `admin`.

!!! note "Mobile"
    O token do `ouros-mobile` contém a audience do Telemetry, mas um `farm_owner` ou `company_employee` ainda recebe `403` enquanto as rotas atuais permanecerem admin-only. Não faça novo login para corrigir 403.

## Rotas

| Método | Rota | Auth | Resposta |
| --- | --- | --- | --- |
| GET | `/` | pública | mensagem |
| GET | `/health` | pública | liveness |
| GET | `/ready` | pública | readiness |
| GET | `/metrics` | pública | Prometheus |
| GET | `/v1/dashboards` | JWT Keycloak + `admin` | JSON |
| GET | `/v1/dashboards/{id}` | JWT Keycloak + `admin` | JSON |
| GET | `/v1/dashboards/{id}/charts` | JWT Keycloak + `admin` | JSON |
| GET | `/v1/dashboards/{id}/charts/{chart_id}/png` | JWT Keycloak + `admin` | image/png |
| GET | `/v1/dashboards/{id}/charts/{chart_id}/chartjs` | JWT Keycloak + `admin` | text/html |

## Readiness

`/ready` verifica:

- `DATABRICKS_HOST`;
- `DATABRICKS_CLIENT_ID`;
- `DATABRICKS_CLIENT_SECRET`;
- configuração completa de issuer/audience Keycloak;
- URLs HTTPS válidas;
- role exigida não vazia;
- limites de timeout/cache.

Pronto:

```json
{"status":"ok","errors":[]}
```

Configuração incompleta retorna 503 com lista sanitizada em `errors`.

## Dashboards

### Listar

```http
GET /v1/dashboards
Authorization: Bearer <jwt>
```

### Dashboard individual

```http
GET /v1/dashboards/{dashboard_id}
```

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

Resposta usa `Cache-Control: private, max-age=30`.

### Chart.js

```http
GET /v1/dashboards/{dashboard_id}/charts/{chart_id}/chartjs
```

Retorna HTML self-contained e usa `Cache-Control: no-store`.

## Erros

| Status | Causa |
| ---: | --- |
| 401 | JWT ausente, inválido, expirado, issuer/audience incorreto |
| 403 | JWT válido, mas role `admin` ausente |
| 404 | dashboard/chart não encontrado |
| 502 | integração Databricks falhou |
| 503 | autenticação/config/JWKS indisponível |
| 504 | request Databricks excedeu timeout |

## Segurança de dados

A autenticação JWT não transforma dashboards globais em dashboards user-scoped. Enquanto o provider usa credenciais Databricks compartilhadas e a autorização permanece `admin`, não remova a role obrigatória apenas para permitir acesso mobile.

A abertura para produtores/funcionários exige desenho separado de escopo/ownership de dashboard.
