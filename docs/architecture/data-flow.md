# Fluxos e linhagem de dados

## Registro de água e energia

Fluxo transacional esperado:

```mermaid
sequenceDiagram
    participant Client as Web/Android
    participant API as ms-spring-api
    participant DB as PostgreSQL prod

    Client->>API: criar registro
    API->>DB: INSERT water/energy registry
    DB-->>API: registro persistido
    API-->>Client: resposta
```

Tabelas principais:

- `water_registries`;
- `energy_registries`.

O PostgreSQL é a fonte de verdade.

## Consumo → analytics

O banco de produção já inclui objetos para sincronização analítica.

```mermaid
flowchart LR
    W[water_registries] --> PROD[(PostgreSQL prod)]
    E[energy_registries] --> PROD
    PROD -->|role read-only + updated_at| SYNC[processo de sync]
    SYNC --> ANA[(Analytics DB)]
    ANA --> DASH[consumidores analíticos]
```

No snapshot atual, o repo de analytics ainda não materializa o modelo final.

Portanto, a seta `Analytics DB → dashboards` é arquitetura de destino, não garantia de implementação já concluída.

## Dados do usuário → Midas

```mermaid
sequenceDiagram
    participant U as Usuário
    participant AI as ms-ai-server
    participant MCP as Knowledge MCP
    participant DB as PostgreSQL

    U->>AI: mensagem
    AI->>MCP: get_user_context
    MCP->>DB: query fixa por identidade
    DB-->>MCP: farms/escopo
    MCP-->>AI: contexto autorizado
    AI->>MCP: get_user_farm_data
    MCP->>DB: dados limitados
    DB-->>MCP: consumos/lotes/metas
    MCP-->>AI: dados filtrados
    AI-->>U: resposta sintetizada
```

Pontos de segurança:

- `user_id` é injetado pelo backend;
- `farm_id` é validado contra farms autorizadas;
- o LLM não recebe SQL arbitrário.

## Documento → conhecimento vetorial

```mermaid
flowchart LR
    DOC[documento] --> ING[CLI/pipeline de ingestão]
    ING --> CHUNK[chunks]
    CHUNK --> EMB[embedding NVIDIA]
    EMB --> QD[(Qdrant)]
    Q[query] --> QEMB[embedding]
    QEMB --> QD
    QD --> R[resultados]
    R --> MCP[search_knowledge]
```

Compatibilidade crítica:

> o modelo de embedding usado para consultar precisa ser compatível com o usado na indexação.

Trocar o modelo pode exigir reindexar a coleção.

## PDF/XLSX → importação de consumo

```mermaid
flowchart TD
    F[PDF/XLSX] --> PREP[prepare_resource_import]
    PREP --> MD[conversão para Markdown]
    MD --> NIM[extração estruturada por IA]
    NIM --> PREVIEW[preview de registros]
    PREVIEW --> REVIEW[revisão/decisão]
    REVIEW --> IMPORT[import_user_resource_records]
    IMPORT --> FN[midas.import_resource_records]
    FN --> DB[(PostgreSQL)]
```

Separação importante:

- preview não escreve;
- escrita passa por função controlada;
- `request_id` suporta idempotência;
- conexão de importação é separada da conexão read-only.

## Identidade legada → Keycloak → Spring

```mermaid
sequenceDiagram
    participant C as Client
    participant A as ms-auth-service
    participant K as Keycloak
    participant SPI as User Storage SPI
    participant DB as PostgreSQL
    participant S as ms-spring-api

    C->>A: POST /v1/auth/token
    A->>K: password broker
    K->>SPI: resolve/validate user
    SPI->>A: lookup / verify com service JWT
    A->>DB: SELECT identidade + hash
    DB-->>A: dados
    A-->>SPI: identidade válida
    SPI-->>K: user attributes
    K-->>A: JWT aud=ms-spring-api
    A-->>C: access + refresh token
    C->>S: Bearer access token
    S->>K: valida assinatura via JWKS/cache
```

O password hash não migra automaticamente para o banco interno do Keycloak.

## JWT → resource server

```mermaid
flowchart LR
    K[Keycloak] -->|JWT| C[Client]
    C -->|Bearer JWT| API[Resource server]
    API -->|fetch/cache| JWKS[JWKS]
    API --> V{validar}
    V -->|iss aud exp assinatura| OK[Ação autorizada]
```

A autorização de negócio continua dependendo de roles/ownership, não apenas de token válido.

## Thread do Midas → MongoDB

```mermaid
flowchart LR
    CHAT[/v1/chat] --> OWNER[thread_owners]
    CHAT --> CP[checkpoints]
    CHAT --> CW[checkpoint_writes]
    CHAT --> MEM[user_memories]
```

Cada estrutura tem índice próprio versionado no repo Mongo.

## Databricks → gráfico

```mermaid
sequenceDiagram
    participant C as Client
    participant T as Telemetry
    participant D as Databricks

    C->>T: GET dashboard/chart
    T->>D: OAuth/API/SQL
    D-->>T: metadata/rows
    T-->>C: JSON, PNG ou HTML Chart.js
```

O serviço aplica cache curto para gráficos e normaliza falhas externas em 502/504.

## Produção → QA

Somente contrato gerenciado:

```text
prod main
  ↓
sync de sql/config/runner
  ↓
upstream.lock
  ↓
QA reconciler
```

Não há replicação de dados de produção como parte desse fluxo.

## Linhagem e privacidade

Ao criar um novo dado, documente:

1. origem;
2. fonte de verdade;
3. onde é replicado;
4. quem pode ler;
5. quem pode escrever;
6. retenção;
7. anonimização;
8. índices;
9. consumidor final;
10. como apagar/corrigir quando necessário.
