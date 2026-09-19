# APIs do Ouros

Esta página responde **qual interface usar**. Para detalhes internos, abra a página do respectivo repo.

## Matriz

| Serviço | Interface | Principal uso | Auth atual |
| --- | --- | --- | --- |
| `ms-spring-api` | REST | domínio transacional | JWT legado Spring |
| `ms-auth-service` | REST | login/credenciais | público + service JWT interno |
| `ouros-keycloak` | OIDC/OAuth2 | tokens/sessões/roles | protocolos Keycloak |
| `ms-ai-server` | REST | chat Midas | Bearer/JWT configurável |
| Knowledge MCP | MCP Streamable HTTP + REST health | tools de conhecimento/dados | Bearer MCP |
| Telemetry | REST | dashboards/gráficos | Bearer estático |
| GitHub Manager | REST + UI | engenharia interna | sessão/cookie |
| Auto Review | webhook HTTP | automação GitHub | HMAC GitHub |

## “Qual API eu chamo?”

### Preciso autenticar um usuário

Use `ms-auth-service` e/ou Keycloak, conforme fluxo do cliente.

Não adicione login novo em outro microserviço.

### Preciso criar/editar fazenda, lote, consumo ou perfil

Use `ms-spring-api`.

### Preciso conversar com o Midas

Use `ms-ai-server /v1/chat`.

O cliente não deve chamar diretamente as tools MCP para reproduzir lógica que pertence ao AI Server, exceto se estiver construindo um consumidor MCP autorizado específico.

### Preciso buscar conhecimento/documentos

Por dentro do ecossistema de IA, use o Knowledge MCP.

### Preciso consultar dados do usuário para resposta de IA

Use tools user-scoped do Knowledge MCP, com identidade vinculada pelo backend.

### Preciso renderizar dashboard Databricks

Use `ms-telemetry-dashboard-service`.

## Contratos principais

### Auth Service

```http
POST /v1/auth/token
POST /v1/auth/credentials/verify
GET  /health
GET  /ready
```

### AI Server

```http
POST /v1/chat
GET  /v1/chat/{thread_id}/history
GET  /metrics
```

### Telemetry

```http
GET /v1/dashboards
GET /v1/dashboards/{id}
GET /v1/dashboards/{id}/charts
GET /v1/dashboards/{id}/charts/{chart_id}/png
GET /v1/dashboards/{id}/charts/{chart_id}/chartjs
```

### Spring API

Recursos:

- `/addresses`;
- `/enterprises`;
- `/farms`;
- `/farm-owners`;
- `/company-employees`;
- `/lots`;
- `/water-registries`;
- `/energy-registries`.

## OpenAPI

Quando disponível, prefira OpenAPI/Swagger como referência de **shape exato** do endpoint.

A documentação central deve responder:

- por que chamar;
- como autenticar;
- fluxo;
- ownership;
- erros;
- relações entre serviços.

Evite copiar manualmente centenas de schemas gerados, pois eles envelhecem mais rápido.

## Health vs readiness

Não confunda:

- **health/liveness**: processo está vivo;
- **readiness**: dependências/config estão utilizáveis.

`ms-auth-service` e telemetry possuem distinção explícita.

## Erros entre serviços

Ao criar integração:

1. preserve status úteis;
2. defina timeout;
3. defina retry somente onde seguro;
4. não faça retry cego de mutação;
5. logue request/correlation ID, não token;
6. converta falha externa em erro de integração claro.

## URLs públicas confirmadas no código

- Keycloak: `https://ouros-keycloak.discloud.app`;
- Auth Service: `https://ms-auth-service.discloud.app`;
- AI Server é referenciado como `ms-ai-server.discloud.app` na configuração de métricas;
- MCP de conhecimento é referenciado como `https://ms-midas-mcp.discloud.app/mcp/`.

!!! note
    Não invente host de produção a partir do nome do repo. Só documentamos URLs que aparecem explicitamente no código/configuração.
