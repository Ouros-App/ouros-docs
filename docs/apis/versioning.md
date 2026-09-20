# Versionamento e evolução das APIs

O ecossistema ainda não usa uma estratégia única de versionamento HTTP.

Esta página documenta o estado atual e as regras recomendadas para evoluir contratos sem quebrar clientes.

## Estado atual

| Serviço | Prefixo de negócio | OpenAPI |
| --- | --- | --- |
| Spring API | sem versão: `/farms`, `/lots`, etc. | `/v3/api-docs` + Swagger UI |
| Auth Service | `/v1/auth/**` | FastAPI `/openapi.json` / `/docs` |
| Auth interno | `/internal/v1/**` | oculto do OpenAPI público |
| AI Server | `/v1/chat/**` | FastAPI OpenAPI |
| Telemetry | `/v1/dashboards/**` | FastAPI OpenAPI |
| GitHub Manager | sem prefixo global | FastAPI OpenAPI |
| Knowledge MCP | protocolo MCP em `/mcp/` | REST health em FastAPI; tools via MCP |
| Auto Review | `/webhooks/github` | não usa OpenAPI |

`v3` em `/v3/api-docs` é a versão do endpoint de documentação springdoc/OpenAPI, **não** a versão da API de domínio.

## Compatibilidade primeiro

Mudança aditiva normalmente deve evitar nova versão.

Exemplos compatíveis:

- endpoint novo;
- campo opcional novo;
- novo valor que clientes já tratam como desconhecido;
- filtro/query param opcional;
- novo claim não obrigatório.

Exemplos potencialmente breaking:

- remover/renomear campo;
- tornar campo opcional em obrigatório;
- mudar tipo;
- alterar semântica de status;
- restringir ownership/role de forma que consumidor existente deixe de funcionar;
- trocar mecanismo de autenticação sem janela de coexistência.

## Spring sem `/v1`

O Spring atual usa URLs de domínio estáveis:

```text
/farms
/lots
/water-registries
/energy-registries
```

Não introduza `/v2` apenas porque um DTO ganhou campo.

Para breaking change real, primeiro avalie:

1. expand/contract;
2. aliases temporários;
3. campo novo em paralelo;
4. feature/config flag;
5. migration de clientes.

Só crie uma versão paralela quando coexistência do contrato antigo for realmente necessária.

## APIs já prefixadas com `/v1`

Auth, AI e Telemetry já expõem namespaces `/v1`.

Isso não significa que toda mudança exige `/v2`.

`/v2` deve representar contrato incompatível que precisa coexistir com `/v1` por um período definido.

## API interna

`/internal/v1` é deliberadamente separado do contrato público.

Regras:

- não aparecer no OpenAPI público;
- service auth obrigatória;
- não ser consumido por web/mobile;
- breaking changes precisam ser coordenadas com o User Storage/provider que consome essas rotas.

## MCP

A versão do protocolo MCP via header/protocolo não equivale à versão das tools.

Ao mudar uma tool:

- preserve nome/parâmetros existentes quando possível;
- campos novos devem ser opcionais por padrão;
- output precisa permanecer parseável;
- atualize allowlists do AI Server;
- teste consumidores MCP.

## OpenAPI como contrato executável

### Spring

```text
GET /v3/api-docs
GET /swagger-ui/index.html
```

### FastAPI

Normalmente:

```text
GET /openapi.json
GET /docs
GET /redoc
```

Rotas internas podem ser explicitamente ocultadas.

## O que a wiki adiciona ao OpenAPI

OpenAPI é excelente para:

- shape;
- tipos;
- required fields;
- status declarados.

A wiki deve explicar o que geralmente não cabe bem no schema:

- ownership;
- fluxo entre serviços;
- qual token usar;
- fonte de verdade;
- compatibilidade;
- rollout;
- efeitos colaterais;
- lacunas conhecidas.

## Deprecação

Não há mecanismo formal de deprecação global observado hoje.

Quando precisar depreciar:

1. marque no OpenAPI/documentação;
2. identifique consumidores;
3. mantenha janela de compatibilidade;
4. adicione métrica/log de uso quando possível;
5. migre clientes;
6. só então remova;
7. registre a remoção na PR e docs.

## Checklist de mudança de contrato

- [ ] é realmente breaking?
- [ ] existe alternativa aditiva?
- [ ] consumidores foram identificados?
- [ ] OpenAPI foi atualizado?
- [ ] auth/ownership mudou?
- [ ] migration de dados é necessária?
- [ ] rollback existe?
- [ ] exemplos foram atualizados?
- [ ] wiki foi atualizada?
