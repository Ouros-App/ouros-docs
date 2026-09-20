# Keycloak: clients, audiences e User Storage

Referência do IaC atual em `ouros-keycloak/iac`.

## Tipos de client suportados

O reconciliador reconhece cinco tipos:

| Tipo | Uso | Característica |
| --- | --- | --- |
| `mobile` | app nativo | public client + Authorization Code + PKCE S256 |
| `web` | SPA/web | public client + Authorization Code + PKCE S256 |
| `service` | machine-to-machine | confidential + service account |
| `microservice` | resource server | audience/scope para API |
| `password-broker` | bridge first-party do Auth Service | confidential + direct access grant controlado |

Implicit Flow fica desabilitado.

## Resources realmente versionados hoje

Em `iac/resources/`:

```text
keycloak-user-storage.conf
ms-auth-service-broker.conf
ms-auth-service-internal.conf
ms-spring-api.conf
ms-telemetry-dashboard-service.conf
```

Mobile/web aparecem como tipos suportados e exemplos, mas não como resource files ativos no snapshot atual.

Isso evita confundir:

> “o IaC sabe criar” com “o client já existe em produção”.

## Matriz atual

| Client | Tipo | Audience(s) | Uso |
| --- | --- | --- | --- |
| `keycloak-user-storage` | service | `ms-auth-service-internal` | chama Auth interno |
| `ms-auth-service-broker` | password-broker | `ms-spring-api` | login first-party e token para Spring |
| `ms-auth-service-internal` | microservice | `ms-auth-service-internal` | resource server do Auth interno |
| `ms-spring-api` | microservice | `ms-spring-api` | resource server de domínio |
| `ms-telemetry-dashboard-service` | microservice | `ms-telemetry-dashboard-service` | resource preparado para Telemetry |

## `keycloak-user-storage`

Config:

```text
CLIENT_TYPE=service
CLIENT_ID=keycloak-user-storage
AUDIENCES=ms-auth-service-internal
```

Papel:

- identidade machine-to-machine do provider User Storage;
- obtém token via Client Credentials;
- chama `/internal/v1/**` no Auth Service;
- access token inclui audience `ms-auth-service-internal`.

O secret é gerado pelo Keycloak e injetado no componente User Storage pelo reconciliador.

## `ms-auth-service-broker`

Config atual:

```text
CLIENT_TYPE=password-broker
CLIENT_ID=ms-auth-service-broker
AUDIENCES=ms-spring-api
```

Papel:

- usado somente pelo `ms-auth-service`;
- recebe credenciais first-party;
- pede token ao Keycloak;
- produz access token com audience aceita pelo Spring;
- secret fica no secret management do Auth Service.

Fluxo:

```text
POST /v1/auth/token
      ↓
ms-auth-service-broker
      ↓
Keycloak
      ↓
aud=ms-spring-api
      ↓
ms-spring-api
```

Browser/mobile não devem conhecer o client secret desse broker.

## `ms-auth-service-internal`

Config:

```text
CLIENT_TYPE=microservice
CLIENT_ID=ms-auth-service-internal
AUDIENCE=ms-auth-service-internal
SCOPE_NAME=ms-auth-service-internal-audience
MAPPER_NAME=ms-auth-service-internal-audience
```

É o resource server lógico das rotas internas do Auth Service.

O audience mapper adiciona:

```text
aud = ms-auth-service-internal
```

ao access token dos consumidores aos quais o scope foi anexado.

## `ms-spring-api`

Config:

```text
CLIENT_TYPE=microservice
CLIENT_ID=ms-spring-api
AUDIENCE=ms-spring-api
SCOPE_NAME=ms-spring-api-audience
MAPPER_NAME=ms-spring-api-audience
```

O Spring atual valida:

- assinatura JWKS;
- issuer;
- timestamps;
- `aud=ms-spring-api`.

O código usa `AudienceValidator` além dos validators padrão do Spring Security.

## `ms-telemetry-dashboard-service`

Config:

```text
CLIENT_TYPE=microservice
CLIENT_ID=ms-telemetry-dashboard-service
AUDIENCE=ms-telemetry-dashboard-service
SCOPE_NAME=ms-telemetry-dashboard-audience
MAPPER_NAME=ms-telemetry-dashboard-audience
```

Isso prepara o Keycloak para emitir tokens destinados ao Telemetry.

!!! note "Migração da aplicação ainda incompleta"
    O Telemetry atual autentica rotas de negócio por `API_BEARER_TOKEN` estático. Ter o resource server no IaC não significa que o código já valide JWT Keycloak.

## Exemplos suportados pelo IaC

### Mobile

```text
CLIENT_TYPE=mobile
CLIENT_ID=ouros-mobile
REDIRECT_URIS=com.ouros.app:/oauth2redirect
AUDIENCES=ms-example-api
```

Características:

- public client;
- sem client secret;
- Authorization Code;
- PKCE S256;
- redirect URI controlado pelo app.

### Web

```text
CLIENT_TYPE=web
CLIENT_ID=ouros-web
REDIRECT_URIS=https://app.example.com/*
WEB_ORIGINS=https://app.example.com
AUDIENCES=ms-example-api
```

Características:

- public client;
- PKCE S256;
- redirect URIs e origins explícitos.

### Service

```text
CLIENT_TYPE=service
CLIENT_ID=ouros-worker
AUDIENCES=ms-example-api
```

Uso:

- Client Credentials;
- service account;
- secret gerado no Keycloak.

### Microservice

```text
CLIENT_TYPE=microservice
CLIENT_ID=ms-example-api
AUDIENCE=ms-example-api
SCOPE_NAME=ms-example-api-audience
MAPPER_NAME=ms-example-api-audience
```

Não é login interativo. Representa o recurso que deve aparecer em `aud`.

## Audience scopes

O reconciliador cria client scope e mapper do tipo:

```text
oidc-audience-mapper
```

Comportamento observado:

- inclui audience no access token;
- não inclui no ID token;
- inclui em introspection;
- mantém mapper idempotente por nome.

## `ouros-identity`

O reconciliador mantém o client scope:

```text
ouros-identity
```

Claims de domínio:

- `database_id`;
- `account_type`;
- `farm_id`;
- `enterprise_id`;
- `first_access`.

`database_id` é especialmente importante para o Spring, porque pode evitar lookup local por email na conversão do principal.

Claims ajudam no contexto, mas não substituem authorization/ownership de negócio.

## User Storage

Config observada:

```text
NAME=ouros-auth-service
PROVIDER_ID=ouros-auth-service
AUTH_SERVICE_URL=https://ms-auth-service.discloud.app
SERVICE_CLIENT_ID=keycloak-user-storage
TOKEN_URL=http://127.0.0.1:8080/realms/ouros/protocol/openid-connect/token
PRIORITY=0
CACHE_POLICY=NO_CACHE
```

### Por que `TOKEN_URL` usa localhost?

O reconciliador/provider roda junto do Keycloak e pede o service token diretamente ao runtime local:

```text
127.0.0.1:8080
```

Isso evita uma ida pela rota pública para falar com o próprio Keycloak.

### Cache

```text
NO_CACHE
```

O provider consulta a fonte federada em vez de manter cópia duradoura da identidade legada.

### Importação de usuário

O fluxo federado é read-only. O hash legado continua no PostgreSQL e a validação de password é delegada ao Auth Service.

## Como o secret chega ao provider

```mermaid
sequenceDiagram
    participant I as sync-user-storage.sh
    participant K as Keycloak Admin API
    participant C as keycloak-user-storage client
    participant P as User Storage component

    I->>K: resolve client UUID
    I->>K: GET client-secret
    K-->>I: secret
    I->>I: arquivo temporário + umask 077
    I->>K: create/update component
    I->>I: cleanup + unset
```

O secret não pertence ao Git.

## Ordem de reconciliação

`sync-clients.sh` usa duas passadas:

1. cria `microservice` e seus audience scopes;
2. reconcilia mobile/web/service/password-broker.

Motivo:

> consumidores podem referenciar audiences sem depender da ordem alfabética dos arquivos.

## Adicionar nova API

Exemplo:

```text
CLIENT_TYPE=microservice
CLIENT_ID=ms-nova-api
AUDIENCE=ms-nova-api
SCOPE_NAME=ms-nova-api-audience
MAPPER_NAME=ms-nova-api-audience
```

Depois:

1. rode `bash iac/validate.sh`;
2. adicione a audience aos clients consumidores;
3. faça deploy/reconciliação;
4. configure issuer + JWKS + audience na API;
5. teste token certo, expirado e audience errada;
6. teste roles/ownership.

Veja [Guia: nova API com Keycloak](../guides/new-keycloak-api.md).

## Adicionar mobile/web real

Não copie exemplo sem trocar:

- client ID;
- redirect URI;
- web origin;
- audiences.

Web:

- HTTPS em produção;
- origins exatas;
- sem client secret no browser.

Mobile:

- PKCE S256;
- redirect URI controlado pelo app;
- armazenamento seguro de tokens.

## Falhas comuns

### Token válido, API retorna 401

Compare:

```text
iss
aud
exp
assinatura
```

No Spring, confirme também role reconhecida.

### Client existe, audience não aparece

Cheque:

- scope criado;
- mapper criado;
- scope anexado ao client consumidor;
- token foi reemitido depois da mudança.

### User Storage não encontra usuário

Cheque:

- Auth Service;
- service client;
- client secret;
- token URL local;
- audience interna;
- provider Java;
- PostgreSQL legado.

### Alterei um `.conf` e nada mudou

O reconciliador roda no startup/deploy correspondente. Veja logs `[keycloak-iac]`.

## Semântica de remoção

A reconciliação é deliberadamente não destrutiva.

Remover um `.conf` do Git não significa apagar automaticamente o client no Keycloak.

Exclusão exige operação administrativa explícita.
