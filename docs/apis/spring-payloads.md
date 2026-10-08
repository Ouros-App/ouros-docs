# Spring API: payloads e validações

Referência dos DTOs atuais do `ms-spring-api`.

Para autorização e rotas, veja [Spring API](spring-api-reference.md).

## Convenção de nomes

A forma canônica de vários campos é `snake_case`.

Muitos DTOs aceitam aliases camelCase para compatibilidade:

```json
{
  "id_farm": 1,
  "registration_date": "2026-09-19"
}
```

pode, em campos anotados com `@JsonAlias`, aceitar equivalentes como `idFarm` e `registrationDate`.

Respostas usam a forma definida por `@JsonProperty`.

## Address

### Create

```json
{
  "zip_code": "12345678",
  "state": "SP",
  "city": "Campinas",
  "number": "555",
  "country": "BR"
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| zip_code | sim | não vazio, até 50 |
| state | sim | exatamente 2 letras |
| city | sim | até 100 |
| number | sim | até 50 |
| country | sim | exatamente 2 letras |

Normalização:

- trim;
- `state` uppercase;
- `country` uppercase.

### PATCH

Mesmos campos, todos opcionais.

### Response

```text
id
zip_code
state
city
number
country
```

## Enterprise

### Create

```json
{
  "name": "Agro Ouros S.A.",
  "email": "contato@example.com",
  "document_number": "<cnpj-valido>",
  "telephone": "11999999999",
  "id_address": 1
}
```

Ou crie o endereço inline:

```json
{
  "name": "Agro Ouros S.A.",
  "email": "contato@example.com",
  "document_number": "<cnpj-valido>",
  "telephone": "11999999999",
  "address": {
    "zip_code": "12345678",
    "state": "SP",
    "city": "Campinas",
    "number": "555",
    "country": "BR"
  }
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| name | sim | até 100 |
| email | sim | email válido, até 50 |
| document_number | sim | CNPJ válido |
| telephone | sim | 10..13 dígitos |
| id_address | condicional | positivo |
| address | condicional | AddressRequestDTO |

Exatamente **uma** forma de endereço deve existir:

```text
id_address XOR address
```

### PATCH

Campos opcionais:

- name;
- email;
- document_number;
- telephone;
- id_address.

A autorização é mais restritiva que o DTO: company employee da própria empresa pode editar dados permitidos, mas mudança de CNPJ/endereço continua restrita a ADM no service layer.

### Response

```text
id
name
email
document_number
telephone
id_address
```

## Farm

### Create

```json
{
  "name": "Fazenda Santa Maria",
  "area_property": 150.50,
  "region": "Sudeste",
  "poultry_capacity": 50000,
  "place": "Gleba 4",
  "id_address": 1,
  "chickens_now": 3200,
  "foto_url": "https://example.com/farm.jpg",
  "id_enterprise": 1
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| name | sim | 1..100 |
| area_property | sim | > 0 |
| region | sim | 1..50 |
| poultry_capacity | sim | >= 0 |
| place | sim | 1..50 |
| id_address | condicional | positivo |
| address | condicional | endereço inline |
| chickens_now | não | >= 0 |
| foto_url | não | até 2048 |
| id_enterprise | sim | positivo |

Também usa `id_address XOR address`.

### PATCH

Mutáveis:

- name;
- area_property;
- region;
- poultry_capacity;
- place;
- chickens_now;
- foto_url.

Não muda endereço nem empresa por esse DTO de PATCH.

### Response

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

## Company employee

### Create

```json
{
  "name": "João da Silva",
  "document_number": "<cpf-valido>",
  "email": "joao@example.com",
  "telephone": "11987654321",
  "password": "<senha-forte>",
  "id_enterprise": 1
}
```

| Campo | Regra |
| --- | --- |
| name | obrigatório, até 100 |
| document_number | CPF válido |
| email | válido, até 50 |
| telephone | 10..13 dígitos |
| password | 8..20, maiúscula, minúscula, número e especial |
| id_enterprise | obrigatório, positivo |

CPF tem pontuação removida na normalização.

### PATCH

- email;
- telephone;
- password.

### Response

```text
id
name
document_number
email
telephone
id_enterprise
```

Password nunca aparece na resposta.

## Farm owner

### Create

```json
{
  "name": "Sebastião da Silva",
  "document_number": "<cpf-valido>",
  "email": "produtor@example.com",
  "telephone": "11987654321",
  "password": "<senha-forte>",
  "id_farm": 1,
  "foto_url": "https://example.com/profile.jpg"
}
```

| Campo | Regra |
| --- | --- |
| name | obrigatório, até 100 |
| document_number | CPF válido |
| email | válido, até 255 |
| telephone | 10..13 dígitos |
| password | 8..20 + complexidade |
| id_farm | obrigatório, positivo |
| foto_url | opcional, até 2048 |

### PATCH

```json
{
  "email": "novo@example.com",
  "telephone": "11999998888",
  "first_access": false,
  "foto_url": "https://example.com/new-profile.jpg"
}
```

Campos:

- email;
- telephone;
- password;
- first_access;
- foto_url.

Aliases históricos de `first_access` ainda aceitos:

```text
firstAccess
first_acess
firstAcess
```

A forma canônica é `first_access`.

### Response

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

## Lot

### Create

```json
{
  "received_chickens": 50000,
  "delivered_chickens": null,
  "date_birth": "2026-09-01",
  "delivery_date": null,
  "gain": 0,
  "losts": 0,
  "cost": 0,
  "id_enterprise": 1,
  "id_farm": 1
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| received_chickens | sim | >= 0 |
| delivered_chickens | não | >= 0 e <= recebidas |
| date_birth | sim | data |
| delivery_date | não | >= date_birth |
| gain | não | >= 0 |
| losts | não | >= 0 |
| cost | não | >= 0 |
| id_enterprise | não | positivo; pode ser inferido |
| id_farm | sim | positivo |

Se `delivery_date` é omitida, o service pode inicializá-la conforme a regra atual do ciclo.

### PATCH

Pode alterar:

- received_chickens;
- delivered_chickens;
- date_birth;
- delivery_date;
- gain;
- losts;
- cost.

Validações cruzadas continuam valendo sobre o estado efetivo.

### Response

```text
id
received_chickens
delivered_chickens
date_birth
delivery_date
gain
losts
cost
id_enterprise
id_farm
```

## Water registry

### Create

```json
{
  "registration_date": "2026-09-19",
  "start_hydrometer": 120.5000,
  "end_hydrometer": 135.8000,
  "id_farm": 1
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| registration_date | sim | data passada ou presente |
| start_hydrometer | sim | > 0; até 15 inteiros + 4 decimais |
| end_hydrometer | sim | > 0; >= inicial |
| id_farm | depende da role | positivo |

FO pode omitir `id_farm`; ADM/CE precisam informar.

### PATCH

- end_hydrometer;
- start_hydrometer;
- registration_date.

### Response

```text
id
registration_date
start_hydrometer
end_hydrometer
id_farm
```

## Energy registry

### Create

```json
{
  "registration_date": "2026-09-19",
  "energy_consumption": 450.75,
  "id_farm": 1
}
```

| Campo | Obrigatório | Regra |
| --- | ---: | --- |
| registration_date | sim | passada ou presente |
| energy_consumption | sim | > 0 |
| id_farm | depende da role | positivo |

FO pode omitir `id_farm`; ADM/CE precisam informar.

### PATCH

- registration_date;
- energy_consumption.

### Response

```text
id
registration_date
energy_consumption
id_farm
```

## Query params: atenção aos nomes

O contrato atual não é uniforme:

| Endpoint | Query param |
| --- | --- |
| `GET /farm-owners` | `farmId` |
| `GET /lots` | `id_farm`, `id_enterprise` |
| `GET /water-registries` | `farm_id` |
| `GET /energy-registries` | `farm_id` |

Não normalize isso apenas no cliente sem confirmar a rota: o nome faz parte do contrato HTTP atual.

## PATCH vazio

Os DTOs possuem `hasUpdates()`. Um payload sem campos úteis normalmente não deve ser usado como “touch”.

## Passwords

Embora DTOs de criação/atualização ainda recebam password para identidades legadas, respostas nunca devolvem o campo.

Nunca registre payloads de criação de usuário integralmente em logs.
