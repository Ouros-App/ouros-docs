# Telemetry Dashboard: referência de API

Contrato atual de `ms-telemetry-dashboard-service`.

## Autenticação

As rotas de negócio usam **token estático** configurado em `API_BEARER_TOKEN`, enviado como:

```http
Authorization: Bearer <api-bearer-token>
```

Esse mecanismo **não é JWT Keycloak** no código atual: o serviço não valida assinatura, issuer, JWKS nem audience. O resource server já existe no IaC do Keycloak, mas a migração da aplicação ainda não foi concluída.

Meta/health/readiness/metrics têm tratamento próprio conforme a implementação.

## GET `/health`

Liveness.

## GET `/ready`

Valida configuração necessária para Databricks.

Response:

```json
{
  "status": "ready",
  "errors": []
}
```

Quando não pronto, a rota usa **503** e lista erros sanitizados.

## GET `/metrics`

Prometheus.

No código atual, `/metrics` **não exige Bearer**: a rota não usa `require_bearer` e `app/main.py` não adiciona middleware global de autenticação. Portanto, trate o endpoint como público enquanto esse contrato não mudar.

## GET `/v1/dashboards`

Retorna:

```json
{
  "items": [
    {
      "id": "dashboard-local-id",
      "title": "Título",
      "description": "Descrição",
      "provider": "databricks"
    }
  ]
}
```

Falhas externas:

- Databricks timeout → **504**;
- integração Databricks → **502**.

## GET `/v1/dashboards/{dashboard_id}`

Retorna `DashboardPublic`.

Não encontrado: **404**.

## GET `/v1/dashboards/{dashboard_id}/charts`

Response:

```json
{
  "items": [
    {
      "id": "chart-id",
      "title": "Título",
      "type": "bar"
    }
  ]
}
```

Tipos suportados pelo schema:

```text
counter
bar
line
pie
```

## GET `/v1/dashboards/{dashboard_id}/charts/{chart_id}/png`

Retorna imagem PNG.

Erros:

- dashboard/chart ausente → 404;
- timeout → 504;
- integração → 502.

O serviço define cache privado curto para PNG.

## GET `/v1/dashboards/{dashboard_id}/charts/{chart_id}/chartjs`

Retorna HTML self-contained com Chart.js.

A resposta usa `no-store`.

O código escapa conteúdo relevante antes de embutir JSON/HTML.

## Modelo interno de chart

Uma definição de chart possui:

```text
id
title
type
warehouse_id
dataset_query
fields[]
encodings{}
```

Cada field:

```text
name
expression
```

O provider converte metadata/serialized dashboard do Databricks para esse contrato interno.

## DashboardRecord local

Campos:

| Campo | Regra |
| --- | --- |
| `id` | lowercase/dígitos/hífen, até 64 |
| `provider` | literalmente `databricks` |
| `title` | 1..200 |
| `description` | até 1000 |
| `dashboard_id` | 1..200 |
| `enabled` | bool, default true |

O catálogo local está vazio no snapshot analisado; descoberta real pode vir do provider Databricks.

## Databricks

Modelos internos incluem:

### Token

```text
access_token
expires_in?
```

### Dashboard summary

```text
dashboard_id
display_name
lifecycle_state = ACTIVE | TRASHED
```

### Dashboard definition

```text
dashboard_id
serialized_dashboard
warehouse_id?
```

## Request ID

O serviço suporta `X-Request-ID`.

Use em debug:

```bash
curl -i "$TELEMETRY_URL/health" \
  -H 'X-Request-ID: debug-telemetry-001'
```

## Retry e timeout

Configuração central inclui:

- HTTP timeout;
- máximo de retries;
- retry backoff;
- SQL wait timeout;
- margem de refresh OAuth.

Não faça retry adicional agressivo no cliente sem considerar os retries internos.

## Caching

Default observado:

```text
CHART_CACHE_TTL_SECONDS=30
```

Ao depurar resultado “velho”, considere o cache antes de assumir query errada.

## Compatibilidade

Clientes devem tratar:

- 401/403 de autenticação conforme ambiente;
- 404 como recurso ausente;
- 502 como dependência Databricks/integrador;
- 504 como timeout recuperável conforme contexto.
