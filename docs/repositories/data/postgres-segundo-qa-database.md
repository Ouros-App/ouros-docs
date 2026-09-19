# postgres-segundo-qa-database

**Papel:** banco PostgreSQL de QA reconciliado a partir de produção.

**Repo:** [Ouros-App/postgres-segundo-qa-database](https://github.com/Ouros-App/postgres-segundo-qa-database)

## Filosofia

QA é mutável. Desenvolvedores e DBAs podem criar drift, cenários incompletos e objetos temporários. O objetivo do reconciliador não é apagar isso, mas garantir que o **contrato gerenciado de produção** continue aplicável.

```text
prod/main
   ↓
sincroniza contrato SQL
   ↓
QA reconciler
   ├─ preflight
   ├─ dependências
   ├─ dry-run transacional
   ├─ rollback do probe
   └─ aplicação real quando segura
```

## Fonte upstream

`upstream.lock` registra:

- repo: `Ouros-App/postgres-segundo-prod-database`;
- branch: `main`;
- commit exato sincronizado.

O workflow de sync consulta produção periodicamente e também pode ser acionado manualmente.

## O que é sincronizado

O contrato inclui:

- `sql/`;
- `config.yaml`;
- `scripts/apply_sql.py`;
- `requirements.txt`;
- `upstream.lock`.

QA mantém configuração própria do ambiente e o reconciliador adicional.

## Política

`reconcile.yaml` define:

- `continue_on_error: true`;
- `reapply_on_change: true`;
- `preserve_data: true`;
- objetos inesperados: `report_only`.

## Dry-run real

Antes de aplicar uma migration, o reconciliador executa contra **o estado real do QA** dentro de transação e faz rollback.

Isso detecta problemas que lint estático não enxerga, como:

- coluna já alterada manualmente;
- objeto com tipo inesperado;
- constraint conflitante;
- dependência faltando;
- migration que só falha diante do estado atual.

## Estados

| Estado | Significado geral |
| --- | --- |
| `APPLIED` | migration aplicada normalmente. |
| `RECONCILED` | drift corrigido/reconciliado. |
| `UNCHANGED` | nada a aplicar. |
| `BASELINED` | tratado como já presente/estado base. |
| `QUARANTINED` | falhou, mas foi isolado por não ser crítico. |
| `BLOCKED` | dependência/condição impede execução. |
| `DISABLED` | migration não habilitada. |
| `FAILED` | falha não recuperada. |

Uma migration não crítica problemática não precisa derrubar o restante da sincronização.

## Criticidade

`banco_ouros_fisico.sql` é marcado crítico. Outros arquivos declaram dependências explícitas, por exemplo:

- analytics depende do schema físico;
- auth depende de schema físico e vínculo Keycloak;
- importação Midas depende do usuário/objetos Midas.

## Auditoria

O reconciliador mantém:

- `qa_reconciliation_runs`;
- `qa_reconciliation_items`;
- `qa_reconciliation_state`.

`controle_versoes` só avança quando o resultado geral fica `HEALTHY`.

## Dados de teste

O sync **não espelha dados de produção** e não deve apagar dados do QA.

Isso é intencional: o contrato de schema vem de prod, enquanto dados/cenários são propriedade do ambiente de teste.

## Quando usar o executor clássico

O repo ainda contém `scripts/apply_sql.py`, mas o fluxo oficial de QA é o reconciliador:

```bash
python scripts/reconcile_sql.py
```

## Diagnóstico de falha

Ao investigar:

1. identifique o item em `qa_reconciliation_items`;
2. veja se ficou `QUARANTINED`, `BLOCKED` ou `FAILED`;
3. confira dependências em `reconcile.yaml`;
4. reproduza o SQL no estado atual do QA;
5. corrija o contrato em produção quando o problema for do SQL upstream;
6. evite “arrumar” QA apagando dados só para a migration passar.
