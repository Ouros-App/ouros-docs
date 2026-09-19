# Impacto de mudanças

Playbooks para alterações que atravessam mais de um repositório.

## Alterar login/autenticação

Revise, nesta ordem:

1. `ouros-keycloak`;
2. `ms-auth-service`;
3. banco de produção, se identidade/schema mudou;
4. resource servers que validam claims/audience;
5. web;
6. Android;
7. AI Server, se `user_id`/JWT mudar;
8. documentação.

### Exemplo: adicionar claim

Mudança preferida:

```text
1. adicionar claim no Keycloak
2. consumidores passam a aceitar/usar
3. validar rollout
4. somente depois tornar obrigatório
```

Evite tornar um claim obrigatório no consumidor antes de todos os tokens poderem carregá-lo.

## Alterar schema PostgreSQL

Revise:

- `postgres-segundo-prod-database`;
- `postgres-segundo-qa-database`;
- `ms-spring-api`;
- `ms-auth-service`, se tocar identidade;
- Knowledge MCP, se tocar contexto/importação;
- analytics/sync;
- testes e docs.

### Rename de coluna

Prefira rollout expand/contract:

```text
adicionar nova coluna
    ↓
backfill/dual-read ou dual-write
    ↓
migrar consumidores
    ↓
validar
    ↓
remover coluna antiga em migration posterior
```

Evite rename destrutivo de uma vez quando existem vários consumidores.

## Alterar dados de fazenda

Possíveis consumidores:

- Spring API;
- Knowledge MCP;
- AI Server via MCP;
- analytics;
- dashboards futuros;
- web/mobile.

Se a mudança for apenas de apresentação, não altere schema sem necessidade.

## Criar nova tool do Midas

Revise:

1. qual repo deve possuir a tool?
2. ela precisa dado do usuário?
3. qual role de DB?
4. qual agente pode chamá-la?
5. precisa entrar na allowlist do AI Server?
6. `user_id` deve ser injetado pelo backend?
7. qual limite de retorno?
8. precisa timeout?
9. mutation exige confirmação/preview?
10. como será auditada?

Para tool de escrita, siga o padrão de importação:

```text
interpretação/preview
   ↓
validação
   ↓
operação controlada
   ↓
banco
```

Não dê SQL arbitrário ao modelo.

## Trocar modelo de embedding

Repos afetados:

- Knowledge MCP;
- pipeline/CLI de ingestão;
- Qdrant.

Pergunta crítica:

> vetores existentes continuam comparáveis ao novo modelo?

Se não, a troca exige reindexação da coleção ou nova coleção/versionamento.

## Alterar memória do AI Server

Revise:

- código de memory store;
- índices em MongoDB;
- políticas de privacidade/filtro;
- compatibilidade dos documentos existentes;
- testes de thread ownership;
- TTL/retenção, se introduzidos.

## Alterar dashboard

Se a mudança é só representação de Databricks:

- Telemetry;
- consumidores do HTML/PNG;
- testes de chart mapping.

Se a mudança cria dashboards por usuário diretamente do analytics, isso é uma mudança arquitetural diferente e deve decidir explicitamente qual serviço passa a consultar o banco analítico.

## Criar novo microserviço

Fluxo recomendado:

1. definir responsabilidade que não pertence a serviço existente;
2. escolher template;
3. criar via GitHub Manager ou template;
4. revisar CI gerado;
5. definir auth/audience;
6. definir secrets;
7. definir health/readiness;
8. definir observabilidade;
9. definir deploy;
10. registrar no Keycloak se for resource server;
11. adicionar página no `ouros-docs`;
12. atualizar diagrama/dependências.

## Alterar template

Pergunte se a mudança deve afetar:

- somente futuros repos;
- repos existentes também.

Template não atualiza derivados automaticamente.

Se a mudança corrige vulnerabilidade/padrão importante, abra PRs também nos consumidores existentes.

## Alterar branch protection/check obrigatório

Revise:

- nome exato do job/check;
- workflow que o produz;
- `rerun-ci`;
- Auto Review;
- merge queue/auto-merge, se utilizado.

Um check obrigatório com nome que nenhum workflow publica pode congelar a branch inteira.

## Alterar CI de banco

Risco maior que CI comum.

Valide:

- concurrency;
- ambiente alvo;
- secrets;
- paths que disparam apply;
- retry;
- idempotência;
- rollback;
- baseline;
- logs/auditoria.

## Alterar secret

Não é só “trocar valor”.

Mapeie:

```text
secret manager
   ↓
runtime(s)
   ↓
consumidor(es)
   ↓
rotação/redeploy
   ↓
revogação do antigo
```

## Checklist final de rollout transversal

- [ ] mudança backward-compatible primeiro;
- [ ] migration versionada;
- [ ] QA validado;
- [ ] consumidores mapeados;
- [ ] feature/config flag quando necessário;
- [ ] observabilidade suficiente;
- [ ] rollback conhecido;
- [ ] docs atualizadas;
- [ ] contrato antigo removido apenas depois da migração.
