# GitHub Manager: referência de API

API interna do `ms-github-manager` para criação e padronização de repositórios da organização.

## Segurança

O serviço usa **sessão por cookie assinado**, não JWT.

Cookie:

- nome `session`;
- HttpOnly;
- SameSite=Lax;
- Secure configurável;
- HMAC SHA-256;
- TTL configurável.

O middleware exige sessão em todas as rotas não públicas.

Rotas públicas:

```text
GET  /health
GET  /ui
POST /auth/login
POST /auth/logout
GET  /auth/session
/static/**
```

`/metrics` possui autenticação própria por `METRICS_TOKEN`.

## Login

### POST `/auth/login`

```json
{
  "username": "admin",
  "password": "<senha>"
}
```

Sucesso:

- 200;
- cookie `session` definido.

Falhas:

- 401: credenciais inválidas;
- 429: mais de 10 tentativas/minuto por IP no limiter local.

### GET `/auth/session`

Valida a sessão atual.

### POST `/auth/logout`

Remove o cookie de sessão.

## Templates

### GET `/templates`

Retorna templates GitHub disponíveis para a organização.

Exemplo:

```json
[
  {
    "name": "ms-fastapi-template",
    "description": "Template para APIs FastAPI",
    "private": false,
    "url": "https://github.com/Ouros-App/ms-fastapi-template"
  }
]
```

Falha do GitHub upstream: 502.

## Criar repo cru

### POST `/repositories/bare`

Status: **202 Accepted**.

Request:

```json
{
  "name": "orders-api",
  "description": "API de pedidos",
  "visibility": "private",
  "language": "springboot"
}
```

`visibility`:

```text
private
public
internal
```

`language`:

```text
generic
frontend
springboot
fastapi
android
postgres
```

Para `language=postgres`, o nome precisa terminar com `-database`.

Response:

```json
{
  "creation_id": "8ff5a88d-d2c4-4c27-9c2d-13e0a19599e1",
  "status": "queued",
  "repository": "Ouros-App/orders-api",
  "message": "Criacao de repositorio cru iniciada."
}
```

## Criar a partir de template

### POST `/repositories/from-template`

Status: **202 Accepted**.

Request comum:

```json
{
  "name": "billing-api",
  "template_name": "ms-fastapi-template",
  "description": "API de faturamento",
  "visibility": "private"
}
```

### Template PostgreSQL

Também exige:

```json
{
  "postgres": {
    "host": "<host>",
    "port": 5432,
    "database": "app_db",
    "user": "app_user",
    "password": "<senha>",
    "root_database": "postgres",
    "root_user": "root_user",
    "root_password": "<senha-root>"
  }
}
```

O nome do repo precisa terminar com `-database`.

### Template MongoDB

Também exige nome terminando em `-database` e:

```json
{
  "mongodb": {
    "connection_url": "mongodb+srv://<usuario>:<senha>@<host>/<database>"
  }
}
```

!!! warning
    Esses payloads contêm secrets operacionais. Não os registre em logs, screenshots, issues ou exemplos com valores reais.

## Status da criação

### GET `/repositories/creations/{creation_id}`

`status`:

```text
queued
running
done
failed
```

Response:

```json
{
  "creation_id": "8ff5a88d-d2c4-4c27-9c2d-13e0a19599e1",
  "status": "running",
  "repository": "Ouros-App/billing-api",
  "mode": "template",
  "started_at": "2026-09-19T18:00:00Z",
  "finished_at": null,
  "current_step": "Aplicando CI/CD",
  "steps": [
    "Criando repositorio",
    "Aplicando CI/CD"
  ],
  "error": null,
  "url": null
}
```

404 quando `creation_id` não existe.

!!! warning "Estado em memória"
    O tracking de criação não é persistido. Reiniciar o processo perde o histórico de `creation_id` em andamento/concluídos.

## Status e falhas

| Status | Situação |
| ---: | --- |
| 200 | login/session/listagem/status concluído |
| 202 | criação assíncrona aceita |
| 401 | sessão inválida, login inválido ou metrics token inválido/ausente |
| 404 | creation_id desconhecido |
| 422 | payload inválido |
| 429 | limite local de login |
| 502 | falha ao consultar GitHub em operações que traduzem GitHubManagerError |

## Validação 422 e secrets

O serviço remove o campo `input` dos erros Pydantic antes de responder.

Motivo: payloads de criação de banco podem conter passwords/connection strings.

## Métricas

`GET /metrics` exige:

```http
Authorization: Bearer <METRICS_TOKEN>
```

Sem token configurado ou token incorreto: 401.

## Uso via curl

Login e guardar cookie:

```bash
curl -fsS "$GITHUB_MANAGER_URL/auth/login" \
  -H 'content-type: application/json' \
  --data-binary '{"username":"admin","password":"<senha>"}' \
  -c /tmp/ouros-manager.cookies
```

Listar templates:

```bash
curl -fsS "$GITHUB_MANAGER_URL/templates" \
  -b /tmp/ouros-manager.cookies |
  jq
```
