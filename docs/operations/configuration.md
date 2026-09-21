# Referência de configuração

Resumo central dos principais parâmetros de runtime. Use esta página para orientação rápida; o código/`.env.example` do próprio serviço continua sendo a fonte final para uma versão específica.

## Portas e probes

| Serviço | Porta padrão | Liveness | Readiness |
| --- | ---: | --- | --- |
| Auth Service | 8000 | `/health` | `/ready` |
| AI Server | 8000 | `/health` | não possui probe separado documentado |
| Knowledge MCP | 8000 | `/health` | diagnóstico via tools/status |
| Telemetry | 8000 | `/health` | `/ready` |
| Spring API | 8000 no `.env.example`; runtime pode usar 8080 | `/health` | não possui probe separado documentado |
| GitHub Manager | 8000 local; Discloud inicia em 8080 | `/health` | não possui probe separado |
| Keycloak | 8080 internamente | health nativo Keycloak | readiness depende do runtime/DB/reconciliação |

## Auth Service

### Runtime geral

```text
APP_NAME
APP_VERSION
ENVIRONMENT
APP_PORT
FORWARDED_ALLOW_IPS
```

Defaults observados:

```text
APP_NAME=ouros-auth-service
APP_VERSION=0.1.0
ENVIRONMENT=development
APP_PORT=8000
FORWARDED_ALLOW_IPS=127.0.0.1
```

### Banco

```text
DATABASE_URL                 secret
DATABASE_MIN_POOL_SIZE       default 1
DATABASE_MAX_POOL_SIZE       default 10
DATABASE_COMMAND_TIMEOUT_SECONDS default 5
```

Em produção, a role deve ser read-only para identidade.

### Redis

```text
REDIS_URL                    secret/opcional
```

Sem Redis, o serviço pode usar fallback local para rate limit, mas o limite deixa de ser compartilhado entre réplicas.

### Rate limit

```text
AUTH_RATE_LIMIT_IP_BURST=3
AUTH_RATE_LIMIT_IP_BURST_WINDOW_SECONDS=10
AUTH_RATE_LIMIT_IP_PER_MINUTE=5
AUTH_RATE_LIMIT_IP_PER_15_MINUTES=20
AUTH_RATE_LIMIT_EMAIL_PER_15_MINUTES=5
```

### Confiança Keycloak

Não secrets:

```text
KEYCLOAK_ISSUER_URL=https://ouros-keycloak.discloud.app/realms/ouros
KEYCLOAK_INTERNAL_AUDIENCE=ms-auth-service-internal
KEYCLOAK_INTERNAL_CLIENT_ID=keycloak-user-storage
```

O broker first-party também usa client ID/secret/scope próprios; o secret deve vir do secret manager/runtime.

## AI Server

### App

```text
APP_PORT=8000
LLM_TEMPERATURE=0.2
LLM_TIMEOUT_SECONDS=30
LLM_TOTAL_TIMEOUT_SECONDS=60
```

### MongoDB

```text
MONGODB_URI                 secret quando contém credencial
MONGODB_URI_DOCKER
MONGODB_DATABASE=mongodb-ai-prod
```

QA deve usar explicitamente `mongodb-ai-qa`.

### Groq

Modelos observados:

```text
GROQ_FAST_MODEL=openai/gpt-oss-20b
GROQ_MODEL=openai/gpt-oss-120b
```

As API keys não aparecem preenchidas no `.env.example`; devem vir do runtime/Infisical.

### NVIDIA NIM

```text
NVIDIA_NIM_FAST_MODEL=nvidia/nemotron-3-nano-30b-a3b
NVIDIA_NIM_MODEL=nvidia/nemotron-3-super-120b-a12b
NVIDIA_NIM_BASE_URL=https://integrate.api.nvidia.com/v1
```

### Auth

```text
AUTH_JWT_ISSUER
AUTH_JWT_AUDIENCE
AUTH_REQUIRE_USER_JWT=false
```

Quando `AUTH_REQUIRE_USER_JWT=true`, o serviço passa a exigir identidade JWT compatível com o `user_id` da requisição.

### MCP

Modo interoperável com o Knowledge MCP atual:

```text
MCP_URL=https://ms-midas-mcp.discloud.app/mcp/
MCP_ACCESS_TOKEN=<mesmo valor de MCP_AUTH_TOKEN no MCP>
MCP_USER_TYPE=farm_owner
```

O AI Server também possui `MCP_JWT_SECRET`, `MCP_JWT_ISSUER_URL`, `MCP_RESOURCE_URL` e TTL para gerar JWT curto por usuário. Porém, o Knowledge MCP atual usa verifier estático e **não aceita esses JWTs**.

Não habilite o modo JWT isoladamente até o MCP possuir verifier compatível.

## Knowledge MCP

### Qdrant

```text
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY             secret
QDRANT_COLLECTION_NAME=ouros_knowledge
SEARCH_TOP_K=5
```

### Embeddings

```text
NVIDIA_API_KEY             secret
NVIDIA_BASE_URL=https://integrate.api.nvidia.com/v1
NVIDIA_EMBEDDING_MODEL=nvidia/nemotron-3-embed-1b
```

### Extração/importação

```text
NVIDIA_NIM_URL=https://integrate.api.nvidia.com/v1/chat/completions
NVIDIA_NIM_MODEL=meta/llama-3.1-70b-instruct
NVIDIA_NIM_TIMEOUT=60
IMPORT_MARKDOWN_MAX_CHARS=120000
```

### PostgreSQL

```text
MIDAS_DATABASE_URL          secret; leitura
MIDAS_IMPORT_DATABASE_URL   secret; importação controlada
MIDAS_DB_CONNECT_TIMEOUT=10
```

Não use a mesma role para leitura e escrita só para simplificar.

### MCP

```text
MCP_AUTH_TOKEN              secret, >= 32 chars
MCP_RESOURCE_URL=http://localhost:8000/mcp
```

Em deploy público, `MCP_RESOURCE_URL` deve refletir a URL pública real do resource server.

## Telemetry Dashboard

### App

```text
APP_PORT=8000
PROJECT_NAME="Telemetry Dashboard Service"
LOG_LEVEL=INFO
DASHBOARD_CATALOG_PATH=data/dashboards.json
```

### Databricks

Não secrets:

```text
DATABRICKS_HOST
DATABRICKS_TOKEN_URL        opcional
HTTP_TIMEOUT_SECONDS=10
HTTP_MAX_RETRIES=2
SQL_WAIT_TIMEOUT_SECONDS=10
HTTP_RETRY_BACKOFF_SECONDS=0.1
TOKEN_REFRESH_MARGIN_SECONDS=60
CHART_CACHE_TTL_SECONDS=30
```

Secrets exigidos em runtime:

```text
DATABRICKS_CLIENT_ID
DATABRICKS_CLIENT_SECRET
KEYCLOAK_ISSUER_URL
KEYCLOAK_AUDIENCE
KEYCLOAK_JWKS_URL
KEYCLOAK_REQUIRED_ROLE
```

Credenciais Databricks podem ser carregadas do Infisical. A configuração Keycloak deve permanecer coerente com o resource server `ms-telemetry-dashboard-service`.

### CORS

```text
CORS_ORIGINS=[]
```

Use JSON array de origens explícitas. O serviço rejeita wildcard em configuração válida.

## Keycloak

### Admin bootstrap

```text
KC_BOOTSTRAP_ADMIN_USERNAME
KC_BOOTSTRAP_ADMIN_PASSWORD
```

Use apenas para criar o primeiro administrador.

### Admin IaC

```text
KC_IAC_ADMIN_USERNAME
KC_IAC_ADMIN_PASSWORD
KC_IAC_REALM=ouros
```

É a identidade administrativa permanente usada pelo reconciliador.

### Recuperação

```text
KC_RECOVERY_ADMIN_USERNAME
KC_RECOVERY_ADMIN_PASSWORD
```

Somente recuperação emergencial.

### Host e banco

```text
KC_HOSTNAME=https://ouros-keycloak.discloud.app
KC_DB_URL=jdbc:postgresql://keycloak-db:5432/keycloak
KC_DB_USERNAME=keycloak
KCRAW_DB_PASSWORD            secret
```

O repo orienta preferir `KCRAW_DB_PASSWORD` para preservar caracteres como `$` literalmente.

## Spring API

Configuração de runtime observada:

```text
SERVER_PORT=8080
KEYCLOAK_ISSUER_URL=https://ouros-keycloak.discloud.app/realms/ouros
KEYCLOAK_JWK_SET_URL=https://ouros-keycloak.discloud.app/realms/ouros/protocol/openid-connect/certs
KEYCLOAK_AUDIENCE=ms-spring-api
KEYCLOAK_CLIENT_ID=ms-spring-api
CORS_ALLOWED_ORIGINS=
```

Além disso, o serviço precisa do datasource PostgreSQL e pode receber secrets pelo Infisical.

Não há mais `app.jwt.secret`/emissor JWT local no contrato atual: o Spring valida tokens do Keycloak por JWKS.

## GitHub Manager

### GitHub

```text
GITHUB_ORG_LOGIN=Ouros-App
GH_TOKEN                       secret
TEMPLATE_SUFFIX=-template
DEFAULT_BRANCH=main
GH_TIMEOUT_SECONDS=120
```

### App

```text
APP_PORT=8000
PROJECT_NAME
PROJECT_DESCRIPTION
APP_NAME=ms-github-manager
VERSION=0.1.0
```

### Login da UI

```text
AUTH_USERNAME=admin
AUTH_PASSWORD                   secret
SESSION_SECRET                  secret
SESSION_TTL_SECONDS=28800
AUTH_COOKIE_SECURE=true
```

### Métricas

```text
METRICS_TOKEN                   secret
```

### Sonar legado

O exemplo ainda possui `SONAR_CLOUD_TOKEN`, refletindo integrações históricas do manager. Não trate isso como exigência organizacional universal.

## Infisical

Há duas convenções de nomes observadas entre serviços:

Auth Service:

```text
INFISICAL_SITE_URL
INFISICAL_CLIENT_ID
INFISICAL_CLIENT_SECRET
INFISICAL_PROJECT_ID
INFISICAL_ENVIRONMENT
INFISICAL_SECRET_PATH
```

Serviços mais antigos/alternativos podem usar:

```text
INFISICAL_PROJECT_ID
INFISICAL_ENV
INFISICAL_PATH
INFISICAL_TOKEN
```

!!! warning
    Não padronize nomes apenas na documentação. Uma mudança de convenção precisa alterar código/deploy de cada consumidor.

## Classificação prática

### Pode ficar no Git

Normalmente:

- porta;
- timeout;
- model name;
- audience;
- issuer público;
- hostname público;
- feature flag sem segredo;
- nome de collection/database.

### Deve ficar fora do Git

Sempre:

- password;
- token;
- API key;
- client secret;
- private key;
- URI com credenciais;
- webhook secret;
- session secret.

## Debug de configuração

Quando um serviço “sobe mas não funciona”:

1. compare `.env.example` com Settings/config atual;
2. cheque se o secret vem de Infisical e não do env local;
3. valide nome exato da variável;
4. confirme ambiente/path do Infisical;
5. confirme hostname/audience/issuer;
6. use readiness quando existir;
7. evite imprimir o valor do secret no log.
