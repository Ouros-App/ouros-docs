# ms-spring-api

**Stack:** Java 17, Spring Boot 3.4, Gradle, OAuth2 Resource Server, JPA/Hibernate, PostgreSQL, springdoc OpenAPI.

**Repo:** [Ouros-App/ms-spring-api](https://github.com/Ouros-App/ms-spring-api)

## Responsabilidade

!!! warning "README do repo está defasado"
    O README da `main` ainda descreve apenas `/` e `/health` e afirma que não existem CRUD/auth implementados. Esta página segue o código executável atual, que já possui CRUD completo e OAuth2 Resource Server Keycloak.

API transacional principal do domínio Ouros:

- endereços;
- empresas;
- fazendas;
- produtores;
- funcionários;
- lotes;
- água;
- energia.

## Arquitetura

```text
HTTP / JWT Keycloak
      ↓
Controller
      ↓
Service (regra + ownership)
      ↓
Repository / JPA
      ↓
PostgreSQL
```

## Segurança atual

A API já está migrada para **Keycloak**.

`SecurityConfig` configura:

- stateless session;
- OAuth2 Resource Server;
- JWKS;
- issuer;
- audience `ms-spring-api`;
- converter de roles/claims;
- CORS por allowlist.

Rotas públicas:

```text
/
 /health
/error
/v3/api-docs/**
/swagger-ui/**
```

Não existem mais endpoints locais de login na `main` atual.

Fluxo first-party:

```text
cliente
  ↓ POST /v1/auth/token
ms-auth-service
  ↓
Keycloak
  ↓ JWT aud=ms-spring-api
cliente
  ↓ Authorization: Bearer
ms-spring-api
```

## Principal autenticado

`KeycloakJwtAuthenticationConverter` resolve:

- `sub` → Keycloak ID;
- `database_id` → ID local, quando disponível;
- email → fallback de lookup local;
- realm/client/simple roles → authorities Spring.

Roles de negócio:

```text
ADM
COMPANY_EMPLOYEE
FARM_OWNER
```

As regras finas de acesso ficam no service layer.

## API

A referência detalhada, incluindo matriz de autorização e payloads, está em [Spring API](../../apis/spring-api-reference.md).

OpenAPI:

- `/v3/api-docs`;
- `/swagger-ui/index.html`.

## Banco

`spring.jpa.hibernate.ddl-auto=none`.

Schema/migrations pertencem a `postgres-segundo-prod-database`, não ao Hibernate.

## Configuração principal

```text
SERVER_PORT
KEYCLOAK_ISSUER_URL
KEYCLOAK_JWK_SET_URL
KEYCLOAK_AUDIENCE=ms-spring-api
KEYCLOAK_CLIENT_ID=ms-spring-api
CORS_ALLOWED_ORIGINS
```

Além do datasource PostgreSQL e bootstrap Infisical conforme o ambiente.

## Erros

`GlobalExceptionHandler` usa `ProblemDetail` para:

- regras de negócio;
- validação;
- conflitos de integridade.

Veja [Convenções de API](../../apis/conventions.md).

## Testes

Cobertura inclui:

- controllers/MockMvc;
- services;
- DTOs;
- JPA;
- audience validator;
- converter Keycloak;
- `UserPrincipal`;
- OpenAPI;
- Infisical;
- tratamento de exceções.

```bash
./gradlew test
./gradlew clean build jacocoTestReport
```

## Dependências

- PostgreSQL;
- Keycloak/JWKS;
- Auth Service indiretamente no fluxo de login;
- Infisical quando configurado;
- clientes web/mobile.

## Manutenção

Ao adicionar endpoint:

1. migration se schema mudar;
2. Entity/Repository;
3. DTO;
4. Service com ownership;
5. Controller;
6. testes;
7. OpenAPI;
8. docs central.
