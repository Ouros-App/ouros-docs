# Observabilidade

Observabilidade no Ouros ainda não é uniforme. Esta página registra o que existe hoje e onde estão as lacunas.

## Matriz

| Componente | Health | Readiness | Métricas | Logs/correlação |
| --- | --- | --- | --- | --- |
| AI Server | sim | não separado | Prometheus | logs controlados; evita conteúdo sensível |
| Telemetry | sim | sim | Prometheus | JSON + request ID |
| GitHub Manager | sim | não separado | Prometheus | logging Python |
| Auth Service | sim | sim | não observado Prometheus no snapshot | logs de serviço |
| Spring API | sim | não separado | não observado Prometheus/Actuator | logs Spring |
| Keycloak | nativo | runtime/IaC | métricas nativas habilitadas | logs Keycloak |
| Knowledge MCP | sim | status via tools | não observado endpoint Prometheus | logs Python |
| Auto Review | sim | não separado | não observado | console estruturável |
| Bancos | n/a | n/a | externas ao repo | tabelas de auditoria/versionamento |

## AI Server

`GET /metrics` expõe Prometheus e exige autenticação.

Métricas observadas cobrem:

- requests HTTP;
- latência;
- status;
- resultado do chat;
- agentes;
- tools usadas.

Nomes incluem métricas próprias, como:

```text
ai_server_http_requests
```

A configuração de logging declara explicitamente a intenção de não registrar conteúdo de mensagens nem credenciais.

### O que observar

- taxa de erro por rota;
- p95/p99 de `/v1/chat`;
- timeouts de provider;
- quantidade de fallback;
- uso/falha de MCP;
- tools por agente;
- falha de persistência Mongo;
- rejeições de guardrail.

## Telemetry Dashboard

É o serviço com observabilidade mais estruturada no snapshot.

Métricas incluem:

```text
http_requests_total
http_request_duration_seconds
databricks_requests_total
```

!!! warning "Endpoint atualmente público"
    No código analisado, `GET /metrics` do Telemetry não exige Bearer e não há middleware global de autenticação em `app/main.py`. Trate o endpoint como público enquanto esse contrato não mudar.

O cliente Databricks também registra duração e erros específicos.

### Request ID

O middleware aceita/gera request ID e o injeta no contexto de logging.

Isso permite seguir:

```text
request HTTP
   ↓
service
   ↓
Databricks
   ↓
response
```

sem colocar credenciais nos logs.

### Logs

Formato JSON com campos controlados.

Isso facilita ingestão em Grafana/Loki ou outro backend de logs.

## GitHub Manager

Possui Prometheus:

- contador HTTP;
- duração HTTP;
- métricas padrão de processo.

`/metrics` exige `METRICS_TOKEN`:

```http
Authorization: Bearer <METRICS_TOKEN>
```

Se o token não estiver configurado **ou** estiver incorreto, o middleware atual retorna 401.

## Keycloak

O Dockerfile habilita:

```text
KC_HEALTH_ENABLED=true
KC_METRICS_ENABLED=true
```

Use métricas nativas do Keycloak para:

- volume de autenticação;
- erros;
- comportamento do runtime;
- saúde do IdP.

Não confunda saúde do processo Keycloak com sucesso da federação para Auth Service.

## Auth Service

Tem separação clara:

- `/health`: processo vivo;
- `/ready`: dependências prontas.

Readiness verifica PostgreSQL e Redis quando configurado.

No snapshot analisado, não foi identificado endpoint Prometheus equivalente aos serviços AI/Telemetry/GitHub Manager.

### Sinais importantes

- 401;
- 403 interno;
- 409 de identidade ambígua;
- 429 rate limit;
- 503 readiness;
- latência de bcrypt;
- falha de JWKS/Keycloak;
- falha do token broker.

## Spring API

Possui `/health` customizado.

Não foi observado Spring Boot Actuator/Prometheus no build atual.

Para troubleshooting, hoje o serviço depende mais de:

- logs Spring;
- status HTTP;
- testes;
- disponibilidade do datasource.

## Knowledge MCP

Diagnóstico funcional via tools:

- `qdrant_status`;
- `postgres_status`.

Além disso, `GET /health` cobre liveness do processo.

Isso é útil para distinguir:

```text
MCP vivo
Qdrant falhando
Postgres falhando
```

No snapshot atual não foi observado endpoint Prometheus.

## Banco PostgreSQL

Observabilidade de migration/versionamento é persistida no próprio banco.

### Produção

- `controle_scripts_sql`;
- `controle_versoes`.

### QA

Além das anteriores:

- `qa_reconciliation_runs`;
- `qa_reconciliation_items`;
- `qa_reconciliation_state`.

Essas tabelas são parte do diagnóstico operacional, não “lixo interno”.

## MongoDB

O executor mantém:

- `controle_scripts_mongo`;
- `controle_contadores`;
- `controle_versoes`.

Use isso para descobrir qual script/commit realmente foi aplicado.

## Auto Review

O serviço escreve eventos e erros em console, mas não possui Prometheus observado.

Sinais importantes:

- webhook 401;
- webhook 403;
- review start;
- falha de permission check;
- falha de provider AI;
- bloqueadores publicados na PR.

Uma melhoria futura útil seria métricas de:

- reviews iniciadas;
- aprovadas/bloqueadas;
- duração;
- falha por gate;
- provider utilizado.

## Discord Bot

Observabilidade atual é simples:

- stdout;
- mensagens no Discord;
- estado do canal;
- health do homelab.

Não há métricas estruturadas.

## Padrão recomendado para novos serviços

Mínimo:

1. `/health`;
2. `/ready` se houver dependências externas;
3. request/correlation ID;
4. logs estruturados;
5. duração/status HTTP;
6. contadores específicos de integrações;
7. nenhuma credencial/payload sensível em log;
8. timeout explícito;
9. métrica de retry/falha externa.

## Alertas úteis

Quando a stack de monitoramento estiver centralizada, bons alertas seriam:

- readiness > 0 por N minutos;
- erro 5xx acima de baseline;
- p95 de chat/telemetry acima do SLO;
- falha de Databricks contínua;
- falha de MCP/Qdrant;
- Keycloak indisponível;
- apply/reconciler de banco falhou;
- rate limit de Auth subindo abruptamente;
- Auto Review falhando em provider AI.

## Regra de privacidade

Nunca use observabilidade como desculpa para logar:

- password;
- token;
- refresh token;
- private key;
- prompt completo com dado pessoal;
- documento importado;
- connection string com senha.
