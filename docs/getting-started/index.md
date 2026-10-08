# Começando no Ouros

Este guia é para quem acabou de chegar no projeto e precisa descobrir rapidamente **o que rodar, onde mexer e qual serviço chamar**.

## 1. Entenda o mapa

Antes de clonar qualquer coisa:

| Necessidade | Repo principal |
| --- | --- |
| CRUD e domínio de fazendas/lotes/consumo | `ms-spring-api` |
| Login e credenciais legadas | `ms-auth-service` |
| Tokens, roles e OIDC | `ouros-keycloak` |
| Midas / chat de IA | `ms-ai-server` |
| Conhecimento, contexto e importação para IA | `ms-mcp-server-ouros-knowledge` |
| Dashboards Databricks | `ms-telemetry-dashboard-service` |
| Schema PostgreSQL prod | `postgres-segundo-prod-database` |
| Schema PostgreSQL QA | `postgres-segundo-qa-database` |
| Estrutura Mongo da IA | `mongodb-ai-*-database` |
| Web | `ms-ouros-front-web` |
| Android | `ouros-android-app` |

Veja [Arquitetura](../architecture/index.md) para o desenho completo.

## 2. Escolha o ambiente

### Local

Use quando:

- desenvolvendo lógica;
- rodando testes;
- alterando frontend/mobile;
- validando migrations antes de integração.

Normalmente exige `.env` local baseado em `.env.example`.

### QA

Use para:

- validar integração entre serviços;
- testar migrations contra estado real e possivelmente “sujo”;
- reproduzir drift;
- testar mudanças que não devem tocar produção.

O QA PostgreSQL é mutável e possui reconciliador próprio.

### Produção

Use apenas para:

- tráfego real;
- dados reais;
- deploys aprovados pela esteira;
- migrations já validadas.

Produção é fonte de verdade do **contrato gerenciado** do banco.

## 3. Autenticação

Novo fluxo de identidade:

```text
cliente
  ↓
ms-auth-service / Keycloak
  ↓
JWT emitido pelo Keycloak
  ↓
resource server
```

Não implemente um novo emissor JWT dentro de cada API.

O `ms-spring-api` já é um resource server Keycloak. Para o fluxo first-party, obtenha o access token em `ms-auth-service /v1/auth/token` e envie-o ao Spring como Bearer.

Veja [Autenticação](../apis/authentication.md).

## 4. Rodando serviços principais

### FastAPI

Padrão comum:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
uvicorn app.main:app --reload
```

Consulte o README do serviço porque porta/config podem variar.

### Spring

```bash
./gradlew test
./gradlew bootRun
```

### Frontend

```bash
npm install
npm run dev
```

### Android

Abra o repo no Android Studio ou use Gradle Wrapper.

## 5. Mudando banco

Não altere schema “na mão” como solução definitiva.

Fluxo recomendado:

```text
nova necessidade
  ↓
repo de banco
  ↓
SQL + config.yaml
  ↓
CI
  ↓
QA
  ↓
produção
  ↓
API passa a depender do novo contrato
```

Para PostgreSQL, leia [Bancos de dados](../databases/index.md).

## 6. Secrets

Nunca coloque valor real em:

- README;
- issue;
- commit;
- exemplo de curl;
- Dockerfile;
- `VITE_*`;
- config versionada.

Use:

- Infisical onde o serviço já integra Universal Auth;
- secrets do GitHub para Actions;
- variáveis/secrets do runtime de deploy;
- `.env` apenas local, ignorado pelo Git.

Veja [Secrets](../operations/secrets.md).

## 7. Antes de abrir PR

Cheque:

- testes;
- lint/build;
- documentação impactada;
- migration quando schema mudou;
- nenhum secret no diff;
- contrato compatível com consumidores;
- observabilidade para caminhos novos.

## 8. Ordem de leitura recomendada

Para onboarding geral:

1. esta página;
2. [Arquitetura](../architecture/index.md);
3. [Catálogo de repos](../repositories/index.md);
4. [APIs](../apis/index.md);
5. [Segurança](../security/index.md);
6. página específica do repo que você vai alterar.

## 9. Regra para documentação

Quando README e código divergirem, confirme o comportamento no código/configuração atual e registre a divergência. Não propague documentação antiga só porque ela já existe.
