# Guia: mudar o banco sem quebrar produção

## 1. Classifique a mudança

Use on_change para definição idempotente.

Use once para migration histórica/transformação.

Use never para script manual/legado que não deve rodar automaticamente.

## 2. Prefira expand/contract

Para renomear campo:

~~~text
adicionar novo
  ↓
backend aceita antigo + novo
  ↓
migrar dados
  ↓
migrar clientes
  ↓
parar escrita antiga
  ↓
remover antigo depois
~~~

## 3. Valide dados antes de constraints

Antes de exigir valor positivo, unique ou NOT NULL, encontre registros incompatíveis.

Exemplo:

~~~sql
SELECT *
FROM tabela
WHERE valor < 0;
~~~

## 4. Unique index

Procure duplicatas antes:

~~~sql
SELECT chave, COUNT(*)
FROM tabela
GROUP BY chave
HAVING COUNT(*) > 1;
~~~

Não apague registros automaticamente só para o índice passar.

## 5. Seed

Seed inicial só pertence a banco realmente vazio. O prod atual usa baseline para impedir dataload inicial em banco já preenchido.

## 6. Atualize config.yaml

A posição na execution_order e o modo são parte do contrato.

## 7. Valide quatro cenários

1. banco vazio;
2. versão anterior;
3. versão anterior com dados;
4. reexecução quando on_change.

## 8. QA

Use o reconciliador e leia estado APPLIED, RECONCILED, QUARANTINED, BLOCKED ou FAILED.

Não limpe o QA só para esconder drift que reproduz um bug real.

## 9. Compatibilidade com API

Durante rollout, a versão antiga da API pode continuar ativa.

Nova coluna deve tolerar isso até a migração de consumidores terminar.

## 10. Permissões

Aplicações usam roles mínimas. Bootstrap cria/provisiona role; migration aplica grants.

Não dê root ao serviço para simplificar migration.

## 11. Produção

O estado final deve vir do Git/workflow.

Hotfix manual inevitável precisa ser refletido no repo imediatamente.

## 12. Rollback

Antes de migration irreversível, saiba:

- backup;
- restore;
- forward-fix;
- compatibilidade com API anterior.

Rollback de código não desfaz dados automaticamente.

## Checklist

- [ ] modo correto;
- [ ] idempotência;
- [ ] ordem;
- [ ] dados existentes;
- [ ] constraints;
- [ ] índices;
- [ ] QA;
- [ ] API antiga/nova;
- [ ] permissions;
- [ ] rollback;
- [ ] docs.
