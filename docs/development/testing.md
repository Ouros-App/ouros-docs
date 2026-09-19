# Testes e qualidade

A estratégia de testes varia bastante entre os repositórios. Esta página mostra a cobertura estrutural observada e onde ainda há buracos.

## Serviços mais cobertos

### ms-spring-api

A suíte cobre:

- controllers/MockMvc;
- services;
- DTOs;
- entities;
- repositories;
- JWT;
- filtro de autenticação;
- SecurityConfig;
- OpenAPI;
- Infisical;
- tratamento global de exceções.

Comandos principais:

```bash
./gradlew test
./gradlew clean build jacocoTestReport
```

### ms-auth-service

Testes cobrem:

- rotas públicas;
- rotas internas;
- bcrypt;
- usuário inexistente/timing;
- rate limit;
- banco;
- Redis/fallback;
- Infisical;
- broker Keycloak;
- JWT/JWKS interno;
- configuração.

### ms-ai-server

Cobertura observada para:

- API;
- grafo LangGraph;
- roteamento;
- guardrails;
- prompts;
- models;
- MCP adapter;
- ownership de thread;
- memória;
- métricas;
- configuração;
- Infisical.

Pontos especialmente importantes:

- `user_id` vinculado;
- farm scoping;
- auth de métricas;
- fallback de provider.

### Knowledge MCP

Testes para:

- autenticação;
- transporte MCP;
- knowledge/Qdrant;
- banco;
- importação;
- CLI;
- Infisical.

### Telemetry

Cobre:

- rotas;
- Bearer auth;
- readiness;
- charts;
- Chart.js;
- PNG;
- config;
- provider Databricks;
- HTTP client;
- logging;
- request ID;
- Infisical.

### Keycloak

A CI é mais próxima de integração real:

- PostgreSQL;
- Keycloak real;
- provider Java;
- clients;
- PKCE;
- Client Credentials;
- User Storage;
- login federado;
- refresh;
- claims;
- roles;
- JWKS;
- idempotência do IaC.

## Bancos

### PostgreSQL prod

Testa:

- executor;
- integração contra PostgreSQL;
- compatibilidade de dataload/schema;
- regras de segurança da migration.

O CI também tenta impedir DDL/destruição indevida em migrations automáticas.

### PostgreSQL QA

Além do executor:

- testes do reconciliador;
- dependências;
- quarentena;
- política de drift.

### MongoDB

Testes do executor verificam:

- expansão de env;
- comportamento de script;
- retomada após falha não transacional.

## Automação

### GitHub Manager

Testes cobrem:

- API;
- schemas;
- criação;
- manager GitHub;
- scaffolds/helpers.

Como o serviço altera a organização, mocks/unit tests são importantes, mas integração controlada com repo descartável também é uma boa camada quando disponível.

### Auto Review

O repo possui teste para a camada de provider/review AI.

Os gates GitHub dependem de comportamento da API; mudanças nessa área devem ser revisadas com atenção mesmo quando unit tests passam.

## Clientes

### Web

No `package.json` atual não há script de testes automatizados.

Existem:

```bash
npm run lint
npm run build
```

Isso pega erros estáticos/build, mas não valida comportamento.

Lacunas:

- auth;
- routing;
- API client;
- componentes;
- integração.

### Android

Há apenas exemplos básicos de:

- unit test;
- instrumented test.

Como a lógica real ainda é mínima, a cobertura de produto também é mínima.

## Templates

### FastAPI

`tests/` não contém casos reais no snapshot.

### Spring

Possui apenas teste de contexto.

### Web

Sem test runner configurado.

### Mobile

Exemplos padrão.

### Templates de banco

Possuem testes reais porque o executor/versionador é a parte central do template.

## Documentação

`ouros-docs` valida:

```bash
mkdocs build --strict
```

Além disso, para esta documentação da organização foi feita validação adicional de:

- links Markdown relativos;
- fences de código balanceadas;
- nav apontando para arquivos existentes.

## Pirâmide recomendada

```text
          poucos E2E
       integração real
    service/component tests
       unit tests
  lint / typecheck / build
```

Não tente transformar tudo em E2E.

## Contratos que merecem integração real

Prioridade alta:

- Keycloak ↔ Auth Service;
- Auth Service ↔ PostgreSQL;
- Spring API ↔ PostgreSQL;
- MCP ↔ PostgreSQL;
- MCP ↔ Qdrant;
- AI Server ↔ MCP;
- Telemetry ↔ Databricks;
- apply/reconciler ↔ banco real temporário.

## Testando migrations

Uma migration “compila” e ainda pode destruir dados.

Valide:

1. banco vazio;
2. banco com schema anterior;
3. banco com dados;
4. reexecução se `on_change`;
5. rollback/dry-run quando aplicável;
6. constraints com dados extremos;
7. permissions da role real.

## Testando auth

Casos mínimos:

- token válido;
- expirado;
- issuer errado;
- audience errada;
- role faltando;
- usuário válido sem ownership;
- senha errada;
- usuário inexistente;
- rate limit;
- refresh;
- rotação de chave/JWKS.

## Testando IA

Não valide só “texto bonito”.

Valide contratos:

- rota escolhida;
- tool allowlist;
- identidade;
- farm scope;
- timeout;
- structured JSON do especialista;
- fallback;
- memória;
- injection/guardrail;
- ausência de secret no output.

## Quality gates

Checks automáticos devem ser determinísticos.

IA de review pode ser gate adicional, mas CI funcional precisa continuar sendo a fonte de verdade para:

- build;
- testes;
- typecheck;
- migration validation;
- security scanning.

## Quando um teste falhar

Não conserte o teste para “ficar verde” antes de responder:

1. o comportamento mudou de propósito?
2. o contrato documentado mudou?
3. o teste representa regra real?
4. existe dado/fixture inválido?
5. é flake externo?
6. precisa retry ou isolamento melhor?

Teste verde sem contrato correto é só uma lâmpada pintada. 
