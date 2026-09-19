# Glossário

Vocabulário técnico e de domínio usado na organização.

## Produto e domínio

### Ouros

Plataforma B2B de gestão ambiental e operacional voltada a produtores integrados e empresas.

### Midas

Assistente de IA do Ouros. No código atual, o backend conversacional principal é `ms-ai-server`.

### Enterprise

Empresa integradora. Representada por `enterprises` no PostgreSQL.

### Farm

Fazenda/propriedade produtiva. Entidade central do domínio.

### Farm owner

Produtor/proprietário associado a uma fazenda.

No banco:

```text
farm_owners
```

No sistema de identidade:

```text
account_type=farm_owner
role=farm_owner
```

### Company employee

Funcionário vinculado a uma empresa.

No banco:

```text
company_employees
```

### Admin

Identidade administrativa legada em `adms`.

### Lot

Lote de aves entregue à fazenda, com quantidade recebida, entregue, perdas, custo e vínculo com empresa/fazenda.

### Water registry

Registro hídrico de uma fazenda.

Tabela:

```text
water_registries
```

### Energy registry

Registro de consumo de energia.

Tabela:

```text
energy_registries
```

## Identidade

### Keycloak

Identity Provider central usado para OIDC/OAuth2, tokens, roles e clients.

### Realm

Domínio lógico do Keycloak.

Realm principal:

```text
ouros
```

### Client

Aplicação registrada no Keycloak.

Tipos usados no IaC:

- mobile;
- web;
- service;
- microservice.

### Resource server

API que recebe access token e valida audience/issuer/assinatura.

### Issuer

Autoridade que emitiu o JWT.

Principal issuer:

```text
https://ouros-keycloak.discloud.app/realms/ouros
```

### Audience

Serviço/resource server para o qual o token foi emitido.

### JWKS

Conjunto público de chaves usado para validar assinatura dos JWTs.

### `sub`

Identificador do sujeito no Keycloak.

Não é sinônimo de ID numérico legado.

### `database_id`

ID da identidade correspondente no PostgreSQL legado.

### User Storage

SPI do Keycloak que permite resolver usuários fora do banco interno do Keycloak.

No Ouros, consulta o `ms-auth-service`.

## IA

### Agent

Nó de raciocínio/especialista dentro do LangGraph.

### Router

Componente que decide quais especialistas executar.

### Specialist

Agente especializado em tema como FAQ, sustentabilidade, ranking ou suporte.

### Default synthesizer

Agente final que sintetiza os resultados estruturados dos especialistas.

### Tool

Função que um modelo/agente pode chamar.

### MCP

Model Context Protocol.

No Ouros, o Knowledge MCP fornece conhecimento e dados contextuais controlados.

### Streamable HTTP

Transporte HTTP usado pelo servidor MCP atual.

### Qdrant

Banco vetorial usado para busca semântica.

### Embedding

Representação vetorial de texto usada em busca por similaridade.

### Checkpoint

Estado persistido do LangGraph para uma thread/conversa.

### Thread ownership

Vínculo entre `thread_id` e usuário dono da conversa.

### Guardrail

Validação aplicada a input/output para limitar comportamento indesejado e vazamento de dados.

## Bancos e dados

### OLTP

Banco transacional do produto.

No Ouros:

```text
postgres-segundo-prod-database
```

### Analytics

Banco derivado para leitura analítica, joins e agregações.

### Migration

Mudança versionada de schema/dados.

### `once`

Migration executada no máximo uma vez.

### `on_change`

Script reexecutado quando seu checksum muda.

### `always`

Script executado sempre.

### `never`

Script mantido no repositório mas não executado automaticamente.

### Baseline

Marcação de que um estado já existe e não deve ser reaplicado como seed/migration.

### Drift

Diferença entre o estado real do banco e o contrato esperado.

### Reconciler

Camada do QA que testa/aplica migrations contra o estado real, preservando dados e isolando falhas quando possível.

### Quarantine

Estado de migration problemática não crítica no QA.

### Source of truth

Sistema/repo considerado autoridade de determinado contrato.

Exemplo:

- schema transacional: PostgreSQL prod;
- tokens: Keycloak;
- docs sistêmicas: ouros-docs.

## Infra e engenharia

### Discloud

Plataforma de hospedagem usada por vários serviços.

### Infisical

Secret manager usado por vários serviços.

### GitHub Manager

Serviço interno que cria/configura repos da organização.

### Auto Review

GitHub App que avalia PRs após `/auto-review`.

### CodeRabbit

Review automatizado externo usado em PRs.

### CodeQL

Análise de segurança nativa do GitHub usada em alguns pipelines.

### `rerun-ci`

Label reconhecida por vários workflows para disparar novamente a CI.

### Readiness

Indica se o serviço está pronto para atender considerando dependências.

### Liveness/health

Indica que o processo está vivo.

Um serviço pode estar healthy e não ready.

### Correlation/request ID

Identificador usado para seguir uma requisição através de logs e integrações.
