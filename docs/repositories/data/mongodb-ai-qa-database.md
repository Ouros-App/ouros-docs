# mongodb-ai-qa-database

**Papel:** contrato MongoDB da IA no ambiente de QA.

**Repo:** [Ouros-App/mongodb-ai-qa-database](https://github.com/Ouros-App/mongodb-ai-qa-database)

## Diferença para produção

O schema/indexação observados são equivalentes ao repo de produção. A diferença principal é o banco selecionado:

```text
mongodb-ai-qa
```

Isso permite testar persistência do Midas sem misturar:

- memórias;
- ownership de threads;
- checkpoints;
- writes.

## Índices

As mesmas quatro áreas são gerenciadas:

- `user_memories`: unicidade por usuário + memória e ordenação por `updated_at`;
- `thread_owners`: `thread_id` único;
- `checkpoints`: chave composta de checkpoint;
- `checkpoint_writes`: chave composta de write/task.

## Aplicação

`colecoes.json`:

- `on_change`;
- não transacional;
- explicitamente idempotente.

## Uso correto

Configure o `ms-ai-server` de QA com:

- URI do cluster/servidor de QA;
- `MONGODB_DATABASE=mongodb-ai-qa`.

Nunca aponte um teste destrutivo para `mongodb-ai-prod`.

## Versionamento e execução

O executor e as coleções de controle seguem o mesmo padrão do repo de produção.

Veja [MongoDB AI prod](mongodb-ai-prod-database.md) para detalhes do contrato estrutural.
