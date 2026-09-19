# Runbook: falha de migration ou apply

## Primeiro princípio

**Não aplique SQL manual destrutivo para “destravar a pipeline” sem entender o estado.**

## Produção

### 1. Identifique o script

Use logs do workflow e:

- `controle_scripts_sql`;
- `controle_versoes`.

Determine:

- arquivo;
- modo;
- checksum;
- commit;
- última execução.

### 2. Classifique

- DDL idempotente?
- migration `once`?
- seed?
- mudança de dados?
- grant/role?
- view/function?

### 3. Preserve dados

Antes de qualquer correção:

- não rode `TRUNCATE`;
- não recrie tabela como atalho;
- não rerode seed sobre banco preenchido;
- não altere checksum histórico sem intenção.

### 4. Corrija no Git

O estado desejado deve voltar para o repo.

Se precisa hotfix:

```text
branch
 ↓
SQL seguro
 ↓
CI
 ↓
merge
 ↓
apply
```

## QA

Use o reconciliador.

Cheque:

- `qa_reconciliation_runs`;
- `qa_reconciliation_items`;
- `qa_reconciliation_state`;
- status `QUARANTINED/BLOCKED/FAILED`.

Uma migration isolada não significa que todas falharam.

## Falha por drift

Descubra qual objeto divergiu.

Opções:

- tornar migration mais idempotente;
- baselinar estado legítimo;
- corrigir drift manual de QA;
- ajustar dependência;
- corrigir migration upstream.

## Falha de seed

Se o banco já contém dados, confirme se o baseline deveria impedir execução.

Seed inicial não é mecanismo de reconciliação de dados vivos.

## MongoDB

Se índice unique falhar:

- procure duplicatas;
- não apague automaticamente;
- defina estratégia de merge/correção;
- reaplique depois.

## Rollback

Nem toda migration é reversível.

Antes de mergear migration crítica, registre:

- backup/restore;
- rollback SQL quando seguro;
- estratégia forward-fix;
- compatibilidade com versão anterior da API.

## Após o incidente

- teste com snapshot representativo;
- adicionar regression test;
- documentar causa;
- revisar `on_change` vs `once`;
- revisar alertas do workflow.
