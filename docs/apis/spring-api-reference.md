# Spring API: referência de contrato

Referência do `ms-spring-api` atual, validada contra a `main` em 19/09/2026.

## Resumo

- **Framework:** Spring Boot 3.4 / Java 17
- **Modelo:** REST + JPA/PostgreSQL
- **Auth:** OAuth2 Resource Server / Keycloak
- **Audience:** `ms-spring-api`
- **OpenAPI:** `/v3/api-docs`
- **Swagger UI:** `/swagger-ui/index.html`
- **Sessão:** stateless
- **Schema DB:** não é criado pelo Hibernate (`ddl-auto=none`)

## Autenticação atual

Envie o access token Keycloak:

```http
Authorization: Bearer <access-token>
```

O `JwtDecoder` valida:

1. assinatura via JWKS;
2. issuer;
3. timestamps;
4. audience `ms-spring-api`.

Defaults de produção:

```text
issuer  = https://ouros-keycloak.discloud.app/realms/ouros
jwks    = https://ouros-keycloak.discloud.app/realms/ouros/protocol/openid-connect/certs
audience = ms-spring-api
```

!!! important
    Os antigos endpoints de login local do Spring não existem mais na `main` atual. Para login first-party, use `POST /v1/auth/token` no Auth Service.

## Rotas públicas

```text
GET /
GET /health
/error
/v3/api-docs
/v3/api-docs/**
/v3/api-docs.yaml
/swagger-ui/**
/swagger-ui.html
```

Todo o restante exige JWT válido.

## Claims e principal

O converter aceita roles de:

- `realm_access.roles`;
- `resource_access["ms-spring-api"].roles`;
- claim simples `role`.

Aliases reconhecidos:

| Token | Authority interna |
| --- | --- |
| ADMIN, ADM, ROLE_ADMIN, ROLE_ADM | `ROLE_ADM` |
| COMPANY_EMPLOYEE | `ROLE_COMPANY_EMPLOYEE` |
| FARM_OWNER | `ROLE_FARM_OWNER` |

Prioridade quando há mais de uma: **ADM → COMPANY_EMPLOYEE → FARM_OWNER**.

Identidade de negócio:

- `sub` vira `keycloakId`;
- `database_id`, quando presente, vira o ID local;
- se `database_id` faltar e houver email, o serviço tenta resolver o usuário no PostgreSQL;
- se o principal ficar sem ID local, operações que dependem de ownership retornam 401.

## Matriz de autorização

Legenda: **ADM** = administrador, **CE** = company employee, **FO** = farm owner.

| Recurso/operação | ADM | CE | FO |
| --- | --- | --- | --- |
| criar endereço | sim | sim | sim |
| ler/editar endereço | qualquer | somente endereço da empresa/fazendas da empresa | somente endereço da própria fazenda |
| criar empresa | sim | não | não |
| listar empresa | todas | própria empresa | não |
| ler empresa | qualquer | própria empresa | não |
| editar empresa | qualquer | própria empresa; CNPJ/endereço continuam restritos a ADM | não |
| criar fazenda | sim | somente na própria empresa | não |
| listar/ler fazenda | todas | fazendas da empresa | própria fazenda |
| editar/remover fazenda | sim | somente da própria empresa | não |
| criar farm owner | sim | somente fazenda da própria empresa | não |
| listar farm owners | todos/por fazenda | fazendas da empresa | produtores da própria fazenda |
| ler/editar farm owner | qualquer | produtor de fazenda da empresa | somente o próprio cadastro |
| remover farm owner | sim | produtor de fazenda da empresa | não |
| criar company employee | sim | somente na própria empresa | não |
| ler company employee | qualquer | funcionários da própria empresa | não |
| editar/remover company employee | sim | somente o próprio cadastro | não |
| criar lote | sim | própria empresa | não |
| listar/ler lote | todos | própria empresa | própria fazenda |
| editar/remover lote | sim | própria empresa | não |
| água/energia: criar | qualquer fazenda | fazenda da própria empresa | própria fazenda |
| água/energia: listar/ler/editar/remover | todas | fazendas da empresa | própria fazenda |

!!! note "Autorização vive no Service"
    O controller autentica a request, mas as regras de ownership ficam no service layer. Um ID informado pelo cliente nunca é prova suficiente de acesso.

## Mapa de endpoints

### Endereços

| Método | Rota | Sucesso |
| --- | --- | ---: |
| POST | `/addresses` | 201 |
| GET | `/addresses/{id}` | 200 |
| PATCH | `/addresses/{id}` | 200 |

Não há endpoint de listagem nem DELETE.

### Empresas

| Método | Rota | Sucesso |
| --- | --- | ---: |
| POST | `/enterprises` | 201 |
| GET | `/enterprises` | 200 |
| GET | `/enterprises/{id}` | 200 |
| PATCH | `/enterprises/{id}` | 200 |

### Fazendas

| Método | Rota | Sucesso |
| --- | --- | ---: |
| POST | `/farms` | 201 |
| GET | `/farms` | 200 |
| GET | `/farms/{id}` | 200 |
| PATCH | `/farms/{id}` | 200 |
| DELETE | `/farms/{id}` | 204 |

### Farm owners

| Método | Rota | Observação |
| --- | --- | --- |
| POST | `/farm-owners` | ADM/CE |
| GET | `/farm-owners/me` | FO autenticado |
| GET | `/farm-owners?farm_id=<id>` | filtro opcional |
| GET | `/farm-owners/{id}` | ownership por role |
| PATCH | `/farm-owners/{id}` | parcial |
| DELETE | `/farm-owners/{id}` | FO não pode remover |

### Company employees

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
| GET | `/lots?id_farm=<id>&id_enterprise=<id>` |
| GET | `/lots/{id}` |
| PATCH | `/lots/{id}` |
| DELETE | `/lots/{id}` |

### Água

| Método | Rota |
| --- | --- |
| POST | `/water-registries` |
| GET | `/water-registries?farm_id=<id>` |
| GET | `/water-registries/{id}` |
| PATCH | `/water-registries/{id}` |
| DELETE | `/water-registries/{id}` |

### Energia

| Método | Rota |
| --- | --- |
| POST | `/energy-registries` |
| GET | `/energy-registries?farm_id=<id>` |
| GET | `/energy-registries/{id}` |
| PATCH | `/energy-registries/{id}` |
| DELETE | `/energy-registries/{id}` |

## Regras de payload mais importantes

### Empresa/fazenda e endereço

Na criação, use **uma** das formas:

- `id_address` existente; ou
- objeto `address` completo.

Enviar ambas ou nenhuma gera 400.

### Registros de água

```json
{
  "registration_date": "2026-09-19",
  "start_hydrometer": 120.5000,
  "end_hydrometer": 135.8000,
  "id_farm": 1
}
```

Regras:

- data não futura;
- leituras positivas;
- leitura final >= inicial;
- FO pode omitir `id_farm`: o backend usa sua fazenda;
- ADM/CE precisam informar fazenda.

### Registros de energia

```json
{
  "registration_date": "2026-09-19",
  "energy_consumption": 450.75,
  "id_farm": 1
}
```

FO pode omitir `id_farm`; ADM/CE não.

### Lotes

Regras cruzadas:

- `delivered_chickens <= received_chickens`;
- `delivery_date >= date_birth`;
- FO não pode criar/editar/remover lote;
- para CE, a empresa é limitada ao vínculo do funcionário.

## Criação e Location

POSTs de recurso retornam 201 e usam `Location` apontando para o novo ID.

Exemplo:

```http
HTTP/1.1 201 Created
Location: /farms/42
```

## Erros

O Spring padroniza erros de negócio/validação em `ProblemDetail`.

### Validação

```json
{
  "type": "about:blank",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Erro de validação nos campos da requisição",
  "errors": {
    "email": "E-mail inválido"
  },
  "timestamp": "2026-09-19T18:45:00Z"
}
```

### Integridade

`DataIntegrityViolationException` vira 409:

```json
{
  "type": "about:blank",
  "title": "Data Integrity Violation",
  "status": 409,
  "detail": "Conflito de integridade de dados ou registro duplicado no banco de dados"
}
```

### Autenticação

Token ausente/inválido: 401.

Token válido sem role reconhecida também é rejeitado durante conversão.

## CORS

`CORS_ALLOWED_ORIGINS` define as origens. Sem configuração, a lista fica vazia.

- métodos: GET, POST, PUT, DELETE, OPTIONS, PATCH;
- headers: `*`;
- credentials: false.

## Exemplo: login e chamada

Primeiro obtenha token pelo Auth Service:

```bash
TOKEN="$(
  curl -fsS "$AUTH_URL/v1/auth/token"     -H 'content-type: application/json'     --data-binary '{"email":"usuario@example.com","password":"<senha>"}' |
  jq -r .access_token
)"
```

Depois:

```bash
curl -fsS "$SPRING_URL/farms"   -H "Authorization: Bearer $TOKEN"
```

O broker Keycloak está configurado para incluir audience `ms-spring-api`.

## Evoluindo o contrato

Mudou DTO/rota/role?

1. atualize testes;
2. atualize OpenAPI;
3. preserve compatibilidade quando possível;
4. atualize clientes;
5. atualize esta página.
