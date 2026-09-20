# Keycloak: clients, audiences e User Storage

Referência do IaC atual em `ouros-keycloak/iac`.

## Resources versionados

```text
keycloak-user-storage.conf
ms-auth-service-broker.conf
ms-auth-service-internal.conf
ms-spring-api.conf
ms-telemetry-dashboard-service.conf
```

A lista acima representa clients/resources realmente gerenciados hoje. Mobile/web continuam disponíveis como **tipos suportados/exemplos**, não como resource files ativos.

## Matriz

| Client | Tipo | Audience(s) | Uso |
| --- | --- | --- | --- |
| `keycloak-user-storage` | service | `ms-auth-service-internal` | chama Auth interno |
| `ms-auth-service-broker` | password-broker | `ms-spring-api` | login first-party e token para Spring |
| `ms-auth-service-internal` | microservice | `ms-auth-service-internal` | resource server do Auth interno |
| `ms-spring-api` | microservice | `ms-spring-api` | resource server de domínio |
| `ms-telemetry-dashboard-service` | microservice | `ms-telemetry-dashboard-service` | resource preparado para Telemetry |

## Spring API

Config ativa:

```text
CLIENT_TYPE="microservice"
CLIENT_ID="ms-spring-api"
AUDIENCE="ms-spring-api"
SCOPE_NAME="ms-spring-api-audience"
MAPPER_NAME="ms-spring-api-audience"
```

O Spring atual valida essa audience via `AudienceValidator`.

## Broker de login

```text
CLIENT_TYPE="password-broker"
CLIENT_ID="ms-auth-service-broker"
AUDIENCES="ms-spring-api"
```

Isso é o elo entre:

```text
POST /v1/auth/token
      ↓
Keycloak token endpoint
      ↓
access token aud=ms-spring-api
      ↓
Spring API
```

O client secret do broker vive no secret management do Auth Service.

## Auth interno

`ms-auth-service-internal` define o resource usado pelo User Storage.

`keycloak-user-storage` é um service client que recebe audience interna e chama:

```text
/internal/v1/**
```

## Telemetry

O resource server já existe no IaC, mas o serviço ainda não valida JWT Keycloak. Hoje usa `API_BEARER_TOKEN`.

Não confunda “client criado no Keycloak” com “aplicação já migrada”.

## Tipos suportados pelo reconciliador

| Tipo | Fluxo |
| --- | --- |
| mobile | Authorization Code + PKCE S256 |
| web | Authorization Code + PKCE S256 |
| service | Client Credentials |
| microservice | resource server/audience |
| password-broker | confidential direct grant controlado |

Implicit Flow fica desabilitado.

## `ouros-identity`

O IaC mantém claims de domínio como:

- `database_id`;
- `account_type`;
- `farm_id`;
- `enterprise_id`;
- `first_access`.

`database_id` é especialmente importante para o Spring: quando presente, ele evita lookup local por email na conversão do principal.

## User Storage

Config observada:

```text
NAME=ouros-auth-service
PROVIDER_ID=ouros-auth-service
AUTH_SERVICE_URL=https://ms-auth-service.discloud.app
SERVICE_CLIENT_ID=keycloak-user-storage
TOKEN_URL=http://127.0.0.1:8080/realms/ouros/protocol/openid-connect/token
CACHE_POLICY=NO_CACHE
```

O token de serviço é obtido localmente no próprio runtime do Keycloak e o client secret é injetado no componente sem ir para o Git.

## Adicionar nova API

1. crie `iac/resources/<api>.conf`;
2. defina audience igual ao resource server;
3. valide IaC;
4. adicione a audience aos clients consumidores;
5. configure issuer/JWKS/audience na API;
6. teste token correto, audience errada e role faltando.

Veja [Guia: nova API com Keycloak](../guides/new-keycloak-api.md).
