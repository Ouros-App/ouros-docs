# Spring API: referência de contrato

Referência baseada nos controllers e DTOs atuais de `ms-spring-api`.

!!! note
    Para o schema OpenAPI exato da versão implantada, use `/v3/api-docs` ou Swagger UI. Esta página explica as regras que mais importam para quem implementa cliente.

## Autenticação

Com exceção das rotas públicas de meta/login/OpenAPI, o serviço exige um JWT legado enviado em `Authorization: Bearer <token>`.

Esse JWT é assinado localmente com **HS256** usando `app.jwt.secret` e contém claims como `sub=email`, `id` e `role`.

!!! warning "Token Keycloak ainda não é aceito aqui"
    O access token retornado por `ms-auth-service /v1/auth/token` é emitido pelo Keycloak e **não é aceito pelo Spring API atual**. O `JwtUtil` do Spring valida apenas a assinatura HMAC configurada em `app.jwt.secret`; ele ainda não valida issuer/JWKS/audience do Keycloak. Enquanto essa migração não acontecer, use o token produzido pelas rotas de login legadas do próprio Spring para chamar os endpoints protegidos.

Rotas públicas observadas:

```text
GET  /
GET  /health
POST /adms/login
POST /company-employees/login
POST /farm-owners/login
/v3/api-docs/**
/swagger-ui/**
```

## Login legado

Request:

```json
{
  "email": "usuario@example.com",
  "password": "<senha>"
}
```

Regras:

- email obrigatório e válido;
- password obrigatório.

Response:

```json
{
  "token": "<jwt>",
  "first_access": true
}
```

`first_access` pode não aparecer para tipos de conta aos quais não se aplica.

## Convenções de JSON

Os DTOs aceitam vários aliases camelCase, mas a forma canônica de resposta tende a usar snake_case:

```text
zip_code
document_number
id_enterprise
id_farm
registration_date
energy_consumption
start_hydrometer
end_hydrometer
received_chickens
delivered_chickens
date_birth
delivery_date
first_access
foto_url
```

Para clientes novos, prefira snake_case.

## Endereços

### POST `/addresses`

Status de sucesso: **201**.

Request:

```json
{
  "zip_code": "12345678",
  "state": "SP",
  "city": "Campinas",
  "number": "555",
  "country": "BR"
}
```

Validações:

- `zip_code`: obrigatório, até 50 chars;
- `state`: exatamente 2 letras;
- `city`: obrigatório, até 100;
- `number`: obrigatório, até 50;
- `country`: exatamente 2 letras.

`state` e `country` são normalizados para uppercase.

Response:

```json
{
  "id": 1,
  "zip_code": "12345678",
  "state": "SP",
  "city": "Campinas",
  "number": "555",
  "country": "BR"
}
```

### GET `/addresses/{id}`

Status: **200**.

### PATCH `/addresses/{id}`

Todos os campos são opcionais. Mantêm os limites/formato da criação.

## Empresas

### POST `/enterprises`

Status: **201**.

Campos:

| Campo | Regra |
| --- | --- |
| `name` | obrigatório, até 100 |
| `email` | obrigatório, email válido, até 50 |
| `document_number` | CNPJ válido |
| `telephone` | 10 a 13 dígitos |
| `id_address` | ID positivo |
| `address` | objeto AddressRequestDTO |

Regra importante:

> informe **exatamente uma** forma de endereço: `id_address` **ou** `address`, nunca ambas e nunca nenhuma.

Exemplo usando endereço existente:

```json
{
  "name": "Empresa Exemplo",
  "email": "contato@example.com",
  "document_number": "<cnpj-valido>",
  "telephone": "11999999999",
  "id_address": 10
}
```

Response contém:

```text
id, name, email, document_number, telephone, id_address
```

### GET `/enterprises`

Retorna lista autorizada para o principal autenticado.

### GET `/enterprises/{id}`

Retorna uma empresa.

### PATCH `/enterprises/{id}`

Campos opcionais:

- name;
- email;
- document_number;
- telephone;
- id_address.

## Fazendas

### POST `/farms`

Status: **201**.

Campos:

| Campo | Regra |
| --- | --- |
| `name` | obrigatório, até 100 |
| `area_property` | obrigatório, > 0 |
| `region` | obrigatório, até 50 |
| `poultry_capacity` | obrigatório, >= 0 |
| `place` | obrigatório, até 50 |
| `id_address` | positivo, alternativo a `address` |
| `address` | endereço completo alternativo |
| `chickens_now` | >= 0, opcional |
| `foto_url` | até 2048 chars |
| `id_enterprise` | obrigatório, positivo |

Também exige exatamente uma forma de endereço.

Exemplo:

```json
{
  "name": "Fazenda Exemplo",
  "area_property": 125.50,
  "region": "Sudeste",
  "poultry_capacity": 50000,
  "place": "Campinas",
  "id_address": 10,
  "chickens_now": 42000,
  "foto_url": "https://example.com/fazenda.jpg",
  "id_enterprise": 3
}
```

Response:

```text
id
name
area_property
region
poultry_capacity
place
id_address
chickens_now
foto_url
id_enterprise
```

### GET `/farms`

A lista é derivada do `UserPrincipal`, portanto não deve ser tratada como “todas as fazendas do banco”.

### PATCH `/farms/{id}`

Campos mutáveis atuais:

- name;
- area_property;
- region;
- poultry_capacity;
- place;
- chickens_now;
- foto_url.

### DELETE `/farms/{id}`

Status: **204**.

## Farm owners

### POST `/farm-owners`

Status: **201**.

Campos:

| Campo | Regra |
| --- | --- |
| `name` | obrigatório, até 100 |
| `document_number` | CPF válido |
| `email` | obrigatório, válido, até 255 |
| `telephone` | 10 a 13 dígitos |
| `password` | 8–20, maiúscula, minúscula, número e especial |
| `id_farm` | obrigatório, positivo |
| `foto_url` | opcional, até 2048 |

Password nunca aparece no DTO de resposta.

Response:

```text
id
name
document_number
email
telephone
id_farm
first_access
foto_url
```

### GET `/farm-owners/me`

Retorna o produtor associado ao principal autenticado.

### GET `/farm-owners?farmId=<id>`

`farmId` é query param opcional. O service aplica autorização de acordo com o principal.

### PATCH `/farm-owners/{id}`

Campos mutáveis atuais:

- email;
- telephone;
- password;
- first_access;
- foto_url.

### DELETE `/farm-owners/{id}`

Status: **204**.

## Company employees

### POST `/company-employees`

Status: **201**.

Campos:

- `name`: obrigatório, até 100;
- `document_number`: CPF válido;
- `email`: obrigatório, válido, até 50;
- `telephone`: 10–13 dígitos;
- `password`: política 8–20 + complexidade;
- `id_enterprise`: obrigatório e positivo.

Response:

```text
id
name
document_number
email
telephone
id_enterprise
```

### GET `/company-employees/me`

Retorna o funcionário autenticado.

### PATCH `/company-employees/{id}`

Mutáveis:

- email;
- telephone;
- password.

### DELETE `/company-employees/{id}`

Status: **204**.

## Registros de energia

### POST `/energy-registries`

Status: **201**.

Request:

```json
{
  "registration_date": "2026-09-19",
  "energy_consumption": 450.75,
  "id_farm": 1
}
```

Regras:

- data obrigatória;
- data não pode ser futura;
- consumo obrigatório e > 0;
- `id_farm` positivo quando informado.

O DTO documenta `id_farm` como opcional para farm owner e necessário/inferível conforme o tipo de principal.

### GET `/energy-registries?farm_id=<id>`

`farm_id` é opcional e passa pelo service de autorização.

### PATCH `/energy-registries/{id}`

Mutáveis:

- registration_date;
- energy_consumption.

### DELETE

`DELETE /energy-registries/{id}` → **204**.

## Registros de água

### POST `/water-registries`

Status: **201**.

Request:

```json
{
  "registration_date": "2026-09-19",
  "start_hydrometer": 120.5000,
  "end_hydrometer": 135.8000,
  "id_farm": 1
}
```

Validações:

- data obrigatória e não futura;
- leituras obrigatórias e > 0;
- até 15 dígitos inteiros e 4 casas;
- leitura final >= leitura inicial;
- `id_farm` positivo quando informado.

### GET `/water-registries?farm_id=<id>`

Lista conforme o principal autorizado.

### PATCH `/water-registries/{id}`

Mutáveis:

- start_hydrometer;
- end_hydrometer;
- registration_date.

A validação cruzada precisa continuar válida após a atualização efetiva.

### DELETE

Status: **204**.

## Lotes

### POST `/lots`

Status: **201**.

Campos:

| Campo | Regra |
| --- | --- |
| `received_chickens` | obrigatório, >= 0 |
| `delivered_chickens` | opcional, >= 0 |
| `date_birth` | obrigatório |
| `delivery_date` | opcional |
| `gain` | >= 0 |
| `losts` | >= 0 |
| `cost` | >= 0 |
| `id_enterprise` | positivo quando informado |
| `id_farm` | obrigatório, positivo |

Validações cruzadas:

- `delivered_chickens <= received_chickens`;
- `delivery_date >= date_birth`.

O DTO documenta `id_enterprise` como inferível a partir do usuário/fazenda quando omitido.

### GET `/lots?id_farm=<id>&id_enterprise=<id>`

Ambos os filtros são opcionais.

### PATCH `/lots/{id}`

Mutáveis:

- received_chickens;
- delivered_chickens;
- date_birth;
- delivery_date;
- gain;
- losts;
- cost.

### DELETE

Status: **204**.

## Erros de validação

Os DTOs usam Jakarta Validation. Exemplos:

- campo obrigatório ausente;
- CPF/CNPJ inválido;
- email inválido;
- data futura;
- número negativo;
- regra cruzada inconsistente.

O formato exato da resposta de erro pertence ao `GlobalExceptionHandler` da versão implantada e deve ser conferido no OpenAPI/testes.

## Compatibilidade

Ao alterar um DTO:

1. considere aliases existentes;
2. não remova campo usado por cliente numa única etapa;
3. atualize OpenAPI;
4. atualize web/mobile;
5. atualize esta referência.
