# Operação e diagnóstico de bancos

Consultas de leitura para entender qual estado de migration está aplicado.

## PostgreSQL: versão mais recente

```sql
SELECT
    versao,
    commit_id,
    comentario_commit,
    aplicado_em
FROM controle_versoes
ORDER BY versao DESC
LIMIT 1;
```

## PostgreSQL: scripts recentes

```sql
SELECT
    arquivo,
    checksum,
    commit_id,
    executado_em
FROM controle_scripts_sql
ORDER BY executado_em DESC;
```

Use para responder:

- esse arquivo já rodou?
- qual checksum foi aplicado?
- em qual commit?
- quando?

## Conferir um script específico

```sql
SELECT *
FROM controle_scripts_sql
WHERE arquivo = 'sql/exemplo.sql';
```

A identidade exata do arquivo depende de como o runner a registra.

## QA: execuções de reconciliação

```sql
SELECT *
FROM qa_reconciliation_runs
ORDER BY id DESC
LIMIT 20;
```

Campos incluem:

- commit QA;
- commit prod;
- início/fim;
- status;
- resumo.

## QA: itens de uma execução

```sql
SELECT *
FROM qa_reconciliation_items
WHERE run_id = <run_id>
ORDER BY attempted_at;
```

Procure status como:

- `APPLIED`;
- `RECONCILED`;
- `UNCHANGED`;
- `BASELINED`;
- `QUARANTINED`;
- `BLOCKED`;
- `DISABLED`;
- `FAILED`.

## QA: estado mais recente por arquivo

```sql
SELECT
    arquivo,
    checksum,
    status,
    prod_commit,
    last_error,
    last_attempt_at,
    last_applied_at
FROM qa_reconciliation_state
ORDER BY arquivo;
```

Essa tabela é especialmente útil quando uma execução anterior já terminou e você quer o estado consolidado.

## Validar integridade de leitura sem expor dados

Prefira contagens/agregações:

```sql
SELECT COUNT(*) FROM farms;
SELECT COUNT(*) FROM water_registries;
SELECT COUNT(*) FROM energy_registries;
SELECT COUNT(*) FROM lots;
```

Evite despejar dados pessoais no terminal/log só para testar conectividade.

## Verificar roles

PostgreSQL:

```sql
SELECT current_user;
```

E permissões de tabela:

```sql
SELECT
    grantee,
    table_schema,
    table_name,
    privilege_type
FROM information_schema.role_table_grants
WHERE grantee = current_user
ORDER BY table_schema, table_name, privilege_type;
```

Isso ajuda a confirmar se um serviço está usando role read-only ou excessivamente privilegiada.

## Verificar conexões ativas

```sql
SELECT
    usename,
    datname,
    state,
    COUNT(*) AS connections
FROM pg_stat_activity
GROUP BY usename, datname, state
ORDER BY connections DESC;
```

Não exponha query text contendo dados sensíveis em prints públicos.

## MongoDB: versão estrutural

No `mongosh`, examine as coleções de controle:

```javascript
db.controle_versoes.find().sort({ _id: -1 }).limit(10)
db.controle_scripts_mongo.find().sort({ executado_em: -1 }).limit(20)
```

Os campos exatos podem variar conforme evolução do executor; use `findOne()` para inspecionar shape antes de criar automação dependente dele.

## MongoDB: índices

```javascript
db.user_memories.getIndexes()
db.thread_owners.getIndexes()
db.checkpoints.getIndexes()
db.checkpoint_writes.getIndexes()
```

Compare com o repo de banco correspondente antes de criar índice manual.

## Regra operacional

Se o banco divergir do Git:

- em produção, trate Git/migrations como contrato;
- em QA, use o reconciliador;
- em Mongo, atualize o repo de banco;
- não faça correção manual e esqueça de versionar.

## Segurança

Consultas desta página são para diagnóstico de leitura.

Antes de qualquer `UPDATE`, `DELETE`, DDL ou grant:

1. confirme ambiente;
2. confirme usuário;
3. confirme backup/rollback;
4. prefira migration versionada;
5. registre motivo.
