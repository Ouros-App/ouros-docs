# Runbook: dashboards e Databricks

## Sintomas

- `/ready=503`;
- 502 em dashboards;
- 504 em dashboards;
- PNG/Chart.js vazio;
- OAuth falha;
- latência alta.

## 1. Health e readiness

```text
GET /health
GET /ready
```

`/health=200` só indica processo vivo.

## 2. Ready 503

Revise:

- `KEYCLOAK_ISSUER_URL`;
- `KEYCLOAK_AUDIENCE=ms-telemetry-dashboard-service`;
- `KEYCLOAK_JWKS_URL` quando sobrescrito;
- `KEYCLOAK_REQUIRED_ROLE=admin`;
- `DATABRICKS_HOST`;
- `DATABRICKS_CLIENT_ID`;
- `DATABRICKS_CLIENT_SECRET`.

## 3. 401 do próprio serviço

Bearer da API está ausente/incorreto.

Isso é diferente de erro OAuth Databricks.

## 4. 502

Integração Databricks falhou.

Cheque logs/métricas:

- token;
- API;
- SQL;
- permissões;
- dashboard ID.

## 5. 504

Timeout.

Cheque:

- `HTTP_TIMEOUT_SECONDS`;
- `SQL_WAIT_TIMEOUT_SECONDS`;
- retries;
- estado do warehouse;
- latência externa.

Não aumente timeout indefinidamente antes de descobrir a causa.

## 6. Request ID

Reproduza passando:

```http
X-Request-ID: incidente-123
```

Use o mesmo ID para localizar os logs relacionados.

## 7. Chart incorreto

Cheque:

- metadata do chart;
- fields;
- encodings;
- rows retornadas;
- tipo do gráfico.

Chart.js usa fallback para fields quando encodings não estão completos.

## 8. Cache

TTL padrão observado:

```text
30 segundos
```

Confirme se o problema é cache antes de culpar o Databricks.

## 9. Segurança

Não logar:

- client secret;
- OAuth token;
- query contendo dado sensível.

## 10. Depois

- salvar request ID;
- adicionar fixture/teste;
- revisar retry;
- documentar dashboard/encoding problemático.
