# ms-spring-api

**Stack:** Java 17, Spring Boot 3.4, Gradle, Spring Security, JPA/Hibernate, PostgreSQL, JWT, springdoc OpenAPI.

**Repo:** [Ouros-App/ms-spring-api](https://github.com/Ouros-App/ms-spring-api)

## Responsabilidade

API transacional principal do domínio Ouros. O código atual já implementa recursos de endereço, empresa, fazenda, proprietário, funcionário, lotes, registros de água e energia, além de login JWT legado.

!!! warning "README local defasado"
    O README da `main` ainda descreve a API como um esqueleto com apenas `/` e `/health`. O código atual possui dezenas de operações CRUD e autenticação. Esta página segue os controllers e configurações reais.

## Arquitetura interna

```text
Controller
   ↓
Service
   ↓
Repository (Spring Data JPA)
   ↓
Entity
   ↓
PostgreSQL
```

Pacotes relevantes:

- `controller/`: contrato HTTP;
- `service/`: regras de negócio e autorização contextual;
- `repository/`: acesso JPA;
- `entity/`: mapeamento das tabelas;
- `dto/`: payloads de entrada/saída;
- `security/`: JWT, principal autenticado e filtro;
- `config/`: Security, OpenAPI, Infisical e tratamento global de erros.

## Segurança

A API é stateless. CSRF fica desativado e o token é enviado por `Authorization: Bearer <jwt>`.

Rotas públicas:

- `GET /`;
- `GET /health`;
- `POST /adms/login`;
- `POST /company-employees/login`;
- `POST /farm-owners/login`;
- `/v3/api-docs/**`;
- `/swagger-ui/**`.

Todo o restante exige autenticação no `SecurityFilterChain`.

O CORS vem de `CORS_ALLOWED_ORIGINS`; sem origens configuradas, nenhuma origem externa é liberada.

## Endpoints

### Meta e login

| Método | Rota | Uso |
| --- | --- | --- |
| GET | `/` | Disponibilidade básica. |
| GET | `/health` | Health check. |
| POST | `/adms/login` | Login de administrador. |
| POST | `/company-employees/login` | Login de funcionário de empresa. |
| POST | `/farm-owners/login` | Login de proprietário de fazenda. |

### Endereços

| Método | Rota |
| --- | --- |
| POST | `/addresses` |
| GET | `/addresses/{id}` |
| PATCH | `/addresses/{id}` |

### Empresas

| Método | Rota |
| --- | --- |
| POST | `/enterprises` |
| GET | `/enterprises` |
| GET | `/enterprises/{id}` |
| PATCH | `/enterprises/{id}` |

### Fazendas

| Método | Rota |
| --- | --- |
| POST | `/farms` |
| GET | `/farms` |
| GET | `/farms/{id}` |
| PATCH | `/farms/{id}` |
| DELETE | `/farms/{id}` |

`GET /farms` usa o `UserPrincipal` autenticado para limitar a visão conforme o usuário.

### Proprietários

| Método | Rota |
| --- | --- |
| POST | `/farm-owners` |
| GET | `/farm-owners/me` |
| GET | `/farm-owners` |
| GET | `/farm-owners/{id}` |
| PATCH | `/farm-owners/{id}` |
| DELETE | `/farm-owners/{id}` |

### Funcionários de empresa

| Método | Rota |
| --- | --- |
| POST | `/company-employees` |
| GET | `/company-employees/me` |
| GET | `/company-employees/{id}` |
| PATCH | `/company-employees/{id}` |
| DELETE | `/company-employees/{id}` |

### Lotes

| Método | Rota |
| --- | --- |
| POST | `/lots` |
| GET | `/lots` |
| GET | `/lots/{id}` |
| PATCH | `/lots/{id}` |
| DELETE | `/lots/{id}` |

### Água

| Método | Rota |
| --- | --- |
| POST | `/water-registries` |
| GET | `/water-registries` |
| GET | `/water-registries/{id}` |
| PATCH | `/water-registries/{id}` |
| DELETE | `/water-registries/{id}` |

### Energia

| Método | Rota |
| --- | --- |
| POST | `/energy-registries` |
| GET | `/energy-registries` |
| GET | `/energy-registries/{id}` |
| PATCH | `/energy-registries/{id}` |
| DELETE | `/energy-registries/{id}` |

## Banco e JPA

`spring.jpa.hibernate.ddl-auto=none`. A API **não é dona da criação automática do schema**. Mudanças estruturais devem ser versionadas no repositório de banco.

O datasource local pode ser configurado por `application-local.properties`. Em deploy, o projeto possui integração com Infisical para injetar secrets antes da inicialização.

## Configuração

Variáveis importantes:

- `SERVER_PORT`, padrão 8080;
- datasource PostgreSQL;
- secret/configuração JWT;
- `JWT_EXPIRATION_MS` / `APP_JWT_EXPIRATION_MS`;
- `CORS_ALLOWED_ORIGINS`;
- bootstrap do Infisical quando utilizado.

## OpenAPI

O projeto inclui `springdoc-openapi-starter-webmvc-ui`. Use:

- `/swagger-ui/index.html`;
- `/v3/api-docs`.

## Testes

A suíte é extensa para o tamanho do serviço e cobre:

- controllers com MockMvc;
- services;
- DTOs/entities;
- JWT e filtro;
- OpenAPI;
- Infisical;
- tratamento global de exceções.

Comandos:

```bash
./gradlew test
./gradlew clean build jacocoTestReport
```

## Dependências externas

- PostgreSQL de produção/QA;
- Infisical em ambientes configurados;
- clientes mobile/web;
- durante a migração de identidade, o `ms-auth-service`/Keycloak é o caminho novo e o login JWT deste serviço deve ser tratado como legado até a migração terminar.

## Manutenção

Ao adicionar um recurso:

1. criar/alterar migration no repo de banco;
2. atualizar Entity;
3. Repository;
4. DTOs;
5. Service;
6. Controller;
7. testes;
8. OpenAPI/documentação central.
