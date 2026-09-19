# Dependências externas

Registro dos serviços externos usados pelo ecossistema Ouros.

## Visão geral

| Dependência | Consumidores principais | Papel |
| --- | --- | --- |
| Discloud | vários serviços | hospedagem/runtime |
| Infisical | Auth, AI, Telemetry, Auto Review e outros | secret management |
| GitHub | toda a org | código, CI/CD, PRs, Pages |
| Keycloak | clientes/APIs | identidade/OIDC |
| PostgreSQL | Spring, Auth, MCP, Keycloak, analytics | persistência relacional |
| MongoDB | AI Server | memória/checkpoints |
| Redis | Auth Service | rate limit compartilhado |
| Qdrant | Knowledge MCP | busca vetorial |
| Groq | AI Server, Auto Review | inferência LLM |
| NVIDIA NIM | AI/MCP/Auto Review | LLM e embeddings |
| Databricks | Telemetry | dashboards/SQL |
| CodeRabbit | PRs | code review automatizado |
| SonarCloud | alguns fluxos legados | quality checks |

## Discloud

Usado para hospedar componentes como:

- Keycloak;
- Spring API;
- Auto Review;
- GitHub Manager;
- Discord bot.

Características observadas:

- `discloud.config`;
- autorestart;
- sites/bots;
- VLAN em alguns serviços.

Falha externa pode afetar múltiplos componentes ao mesmo tempo.

## Infisical

Secret manager.

Usado para evitar colocar credenciais no Git.

Consumidores observados carregam secrets por:

- Universal Auth;
- token/config legado em alguns serviços.

Ponto de atenção:

- há mais de uma convenção de nomes de env entre os repos.

Falha pode impedir startup ou deixar serviço sem dependência externa.

## GitHub

Funções:

- source control;
- Actions;
- branch protection;
- PR/reviews;
- Pages;
- GitHub App Auto Review;
- API consumida pelo GitHub Manager.

### Credenciais privilegiadas

GitHub Manager e Auto Review merecem atenção extra, pois têm capacidade de modificar recursos da organização/repositórios.

## Keycloak

Apesar de ser mantido pela org, funciona como infraestrutura de identidade para os demais serviços.

Endpoint público observado:

```text
https://ouros-keycloak.discloud.app
```

Dependências:

- PostgreSQL próprio;
- Auth Service;
- provider Java;
- IaC.

## PostgreSQL

Há funções diferentes:

### Banco transacional

Fonte de verdade do domínio.

Consumidores:

- Spring API;
- Auth Service;
- Knowledge MCP;
- sync para analytics.

### Keycloak PostgreSQL

Armazena estado interno do IdP.

### Analytics PostgreSQL

Projeção derivada para análise.

## MongoDB

Persistência do AI Server:

- memórias;
- ownership de thread;
- checkpoints;
- checkpoint writes.

O schema/índices são versionados separadamente do código da IA.

## Redis

Opcional no Auth Service.

Uso:

- rate limiting compartilhado.

Sem Redis, o serviço pode cair para estado local in-process, que não é compartilhado entre réplicas.

## Qdrant

Usado pelo Knowledge MCP.

Armazena embeddings de conhecimento.

Variáveis principais:

- URL;
- API key;
- collection;
- embedding model.

Risco operacional:

- collection existe, mas embeddings são incompatíveis após troca de modelo.

## Groq

Provider LLM primário em partes do ecossistema.

Usos observados:

- AI Server;
- Auto Review.

Auto Review pode disparar duas keys em paralelo e usar a primeira resposta válida.

## NVIDIA

Usos distintos:

### NIM chat

- fallback/modelo alternativo;
- extração de importações;
- Auto Review fallback.

### Embeddings

Knowledge MCP usa modelo NVIDIA para vetorização/busca.

Não confunda chave/modelo de chat com embedding.

## Databricks

Fonte atual do Telemetry Dashboard Service.

Necessidades:

- host;
- client ID;
- client secret;
- token endpoint;
- acesso aos dashboards/warehouse.

Falha é traduzida normalmente para 502/504 no serviço.

## CodeRabbit

Review automatizado em PRs.

O Auto Review também examina estado/reviews/threads do CodeRabbit.

Uma mudança de nome/login/comportamento desse bot pode impactar gates automáticos.

## SonarCloud

Ainda aparece:

- em alguns workflows/templates;
- em lógica do GitHub Manager;
- como check opcional interpretado pelo Auto Review.

Não é política obrigatória universal da org.

## Matriz de impacto de indisponibilidade

| Dependência caiu | Efeito esperado |
| --- | --- |
| Keycloak | login/token e validações novas podem falhar |
| Auth Service | User Storage/login federado falha |
| PostgreSQL prod | CRUD, auth legado/contexto Midas podem falhar |
| MongoDB | memória/checkpoints/thread state degradam/falham |
| Qdrant | busca semântica falha |
| Groq | AI pode cair para fallback |
| NVIDIA | fallback/embedding/import podem falhar |
| Databricks | dashboards retornam erro/timeout |
| Redis | rate limit pode usar fallback local |
| Infisical | startup/secret loading pode falhar |
| GitHub | CI, deploy, manager e review ficam indisponíveis |
| Discloud | serviços hospedados podem cair simultaneamente |

## Regra de integração externa

Todo client externo deveria definir:

- timeout;
- erro explícito;
- retry limitado;
- observabilidade;
- segredo fora do Git;
- comportamento de fallback;
- teste de falha;
- documentação de ownership.
