# CI/CD e deploy

Esta página resume **o que realmente existe nos repositórios**, não o que seria desejável existir.

## Visão rápida

| Família | CI de PR | Deploy/apply automático | Runtime observado |
| --- | --- | --- | --- |
| Serviços centrais | maioria sim | varia por serviço | Docker / Discloud / JVM / Python |
| Bancos | sim | sim, via apply em `main` | GitHub Actions + servidor externo |
| Clientes | sim nos apps atuais | não há deploy final consolidado documentado aqui | Vite/Nginx e Android |
| Templates | em geral incompleto | apenas templates de banco têm apply | scaffold |
| Docs | sim | GitHub Pages | MkDocs |
| Automação | sim nos serviços principais | Discloud em alguns casos | Node/Python |

## Serviços centrais

### `ms-spring-api`

Possui:

- `.github/workflows/ci-cd.yml`;
- Dockerfile;
- `docker-compose.yml`;
- `discloud.config`.

Discloud:

```text
TYPE=site
ID=ms-spring-api
MAIN=app.jar
START=java -jar app.jar --server.port=8080
RAM=512
AUTORESTART=true
```

O build do container usa Gradle/JDK e produz JAR.

### `ms-auth-service`

Possui CI de PR e Dockerfile Python 3.12.

Secrets/configuração são injetados em runtime; o container não deve conter credenciais embutidas.

### `ouros-keycloak`

Possui:

- CI dedicado;
- Dockerfile baseado em Keycloak 26.7.3;
- `discloud.config`;
- build do provider Java;
- testes reais de integração com PostgreSQL/Keycloak.

Discloud observado:

```text
ID=ouros-keycloak
TYPE=site
MAIN=Dockerfile
RAM=2048
AUTORESTART=true
```

### `ms-ai-server`

Possui:

- CI de PR;
- Dockerfile Python 3.12;
- `docker-compose.yml`;
- configuração por Infisical/runtime.

O Compose já prevê variáveis para MCP, auth e providers de IA.

### `ms-mcp-server-ouros-knowledge`

Possui CI e Dockerfile Python 3.12.

O serviço é preparado para execução HTTP do FastMCP/REST e depende de configuração externa para Qdrant, PostgreSQL e NVIDIA.

### `ms-mcp-server-ouros-knowledge-codemode`

É experimental. Não deve herdar automaticamente a política de deploy do MCP principal sem validação, porque a finalidade do fork é justamente testar uma arquitetura alternativa.

### `ms-telemetry-dashboard-service`

Possui CI de PR e Dockerfile Python 3.12.

Deploy depende de credenciais Databricks e Bearer de API fornecidos em runtime.

## Bancos

### PostgreSQL produção

`postgres-segundo-prod-database` possui:

- `ci-cd.yml`;
- `apply-sql-on-main.yml`;
- metadata semanal de README.

`Apply SQL On Main` roda em push para `main` quando caminhos relevantes mudam.

Fluxo:

```text
PR
 ↓
CI
 ↓
merge em main
 ↓
Apply SQL On Main
 ↓
scripts/apply_sql.py
 ↓
PostgreSQL prod
```

### PostgreSQL QA

Além de CI, possui:

- `sync-prod-sql.yml` a cada 15 minutos e manualmente;
- `reconciler-tests.yml`;
- workflow de reconciliação em `main`.

O QA não executa simplesmente o mesmo apply de produção: ele usa a camada de reconciliação e preserva drift/dados de teste.

### Analytics PostgreSQL

Possui CI e `apply-sql-on-main.yml`, mesmo estando ainda em estado de bootstrap de schema.

Isso significa que a esteira já existe antes da modelagem analítica real.

### MongoDB prod/QA

Ambos possuem:

- CI;
- `apply-mongo-on-main.yml`;
- metadata semanal.

O apply Mongo usa concurrency group:

```text
apply-mongodb-main
```

com `cancel-in-progress: false`, evitando cancelar uma aplicação estrutural em andamento por causa de outro push.

## Clientes

### `ms-ouros-front-web`

Possui `ci-cd.yml` e Dockerfile multi-stage:

```text
Node 22 build
   ↓
Vite dist/
   ↓
Nginx 1.27
```

A existência do Dockerfile não significa que o frontend já esteja funcionalmente integrado ao produto. O código ainda está próximo do template.

### `ouros-android-app`

Possui `ci-cd.yml`.

O app ainda está em estágio inicial de implementação; CI não implica que networking/auth/offline já estejam completos.

## Automação

### `ms-github-manager`

Possui:

- CI;
- Dockerfile;
- `discloud.config`.

Discloud observado:

```text
TYPE=site
ID=proxy-ouros
MAIN=main.py
START=uvicorn main:app --host 0.0.0.0 --port 8080
RAM=512
```

### `ouros-autoreview-app`

Possui:

- CI;
- Dockerfile Node 22;
- `discloud.config`.

Discloud:

```text
ID=ouros-autoreview
TYPE=site
MAIN=src/server.ts
RAM=512
VERSION=22
BUILD=npm install && npm run build
START=npx tsx src/server.ts
```

### `ms-discord-bot`

No estado observado, possui workflow de metadata, mas não um pipeline CI completo equivalente aos serviços principais.

Deploy Discloud:

```text
TYPE=bot
MAIN=run.py
NAME=ouros-bot
RAM=200
AUTORESTART=true
VLAN=true
```

## Templates

### Templates de aplicação

`ms-fastapi-template`, `ms-spring-template`, `ms-webfront-template` e `mobile-template` possuem automação de metadata, mas não devem ser tratados como tendo a mesma esteira de CI dos produtos derivados.

Isso é uma diferença importante: gerar um repo a partir do template não garante automaticamente que o template em si esteja sendo testado de ponta a ponta.

### Templates de banco

`postgres-database-template` e `mongodb-database-template` possuem CI e workflows de apply, porque o executor faz parte do próprio produto-template.

## Documentação

`ouros-docs` possui:

- `ci-cd.yml`;
- `deploy-docs.yml`;
- build `mkdocs build --strict`;
- CodeQL para Actions;
- GitHub Pages.

Push na `main` aciona a publicação.

## Workflow de metadata

Muitos repos possuem `readme-metadata.yml`, normalmente semanal:

```text
cron: 17 4 * * 1
```

Esse workflow atualiza badges/contribuidores e **não substitui CI funcional**.

## Reexecutando CI sem commit artificial

### Label `rerun-ci`

Vários workflows da organização escutam o evento `labeled` e executam novamente quando a label é:

```text
rerun-ci
```

### `ouros-docs`: dispatch manual

O workflow de CI do portal também aceita `workflow_dispatch`.

Via GitHub CLI:

```bash
gh workflow run ci-cd.yml \
  -R Ouros-App/ouros-docs \
  --ref <branch>
```

Isso é útil quando um commit criado por integração/bot não gera `pull_request.synchronize`.

Depois acompanhe:

```bash
gh run list -R Ouros-App/ouros-docs --branch <branch>
```

O dispatch manual executa `ci` e, após sucesso, `codeql`. O job de Conventional Commits é específico de eventos de pull request.

## Regras para novos repos

Ao criar um novo repositório, confirme separadamente:

1. CI de PR existe?
2. quais checks são realmente obrigatórios na branch protection?
3. deploy é automático ou manual?
4. existe ambiente QA?
5. secrets vêm de onde?
6. o runtime é Docker, Discloud direto, GitHub Pages ou apenas library/template?
7. rollback existe?
8. migration/apply é serializado?
9. o pipeline testa o comportamento real ou só lint/build?

!!! warning
    Não assuma que `ms-github-manager` ou um template refletem automaticamente a política mais recente da organização. Sempre confira o workflow gerado no repo final.
