# Catálogo de migrations PostgreSQL

Referência da `execution_order` atual de `postgres-segundo-prod-database`.

## Regras de modo

### `on_change`

Reexecuta quando o checksum muda. Use apenas para definições idempotentes que não alterem dados de usuário de forma irreversível.

### `once`

Migration imutável, executada no máximo uma vez. Boa para transformação histórica de schema/dados.

### `never`

Arquivo mantido no repositório, mas excluído do apply automático.

## Ordem atual

| Ordem | Arquivo | Modo | Responsabilidade |
| ---: | --- | --- | --- |
| 1 | `banco_ouros_fisico.sql` | on_change | schema físico/base do domínio |
| 2 | `atualiza_updated_at_analytics.sql` | on_change | timestamps/triggers para sync analítico |
| 3 | `analytics_sync_user.sql` | on_change | grants read-only do sync analytics |
| 4 | `keycloak_user_link.sql` | on_change | vínculo UUID Keycloak ↔ usuários legados |
| 5 | `ms_auth_service.sql` | on_change | grants mínimos para Auth Service |
| 6 | `triggers_logs.sql` | on_change | auditoria de alterações em tabelas centrais |
| 7 | `atualiza_lots_farm-owners.sql` | once | evolução histórica de lotes/farm owners |
| 8 | `atualiza_farms-chicken-left.sql` | once | campos de aves/foto e tabela chicken_left |
| 9 | `dataload_inicial.sql` | once + baseline | seed inicial apenas em banco vazio |
| 10 | `atualiza_consumo_mensal.sql` | never | ajuste manual/legado de consumo |
| 11 | `views_galinhas_consumo.sql` | on_change | views derivadas de consumo |
| 12 | `dataload_lots_farm-owners.sql` | never | carga/ajuste legado de lotes e first_access |
| 13 | `atualiza_password.sql` | once | amplia colunas de password para TEXT |
| 14 | `midas-user.sql` | on_change | schema/views read-only do Midas |
| 15 | `midas-resource-import.sql` | on_change | importação controlada e auditada do Midas |

## `banco_ouros_fisico.sql`

Fundação estrutural do domínio. Cria tabelas centrais, relacionamentos, metas, planos, pagamentos e tabelas de log.

!!! danger
    Evite transformação destrutiva de dados nesse arquivo. Para mudança histórica de dados, prefira migration `once` dedicada.

## `atualiza_updated_at_analytics.sql`

Cria `public.atualiza_updated_at()` e garante `updated_at TIMESTAMPTZ` em tabelas selecionadas, com trigger `analytics_updated_at_<tabela>`.

Tabelas observadas incluem addresses, enterprises, farms, lots, water/energy registries, plans, payments, goals, tips, categories e reviews.

Objetivo: suportar sync incremental para analytics.

## `analytics_sync_user.sql`

Aplica grants à role `analytics_sync_ro`: USAGE no schema e SELECT somente nas tabelas relevantes. Não concede escrita nem acesso a sequences.

## `keycloak_user_link.sql`

Adiciona `keycloak_user_id UUID` nullable em farm_owners, company_employees e adms, com índices unique parciais.

Objetivo: mapear `sub` Keycloak para usuários legados sem exigir migração instantânea de todas as contas.

## `ms_auth_service.sql`

Aplica acesso read-only à role `ms_auth_service_ro` sobre farm_owners, company_employees e adms. O SQL revoga privilégios amplos antes dos grants.

## `triggers_logs.sql`

Mantém auditoria por triggers para tabelas como payments, lots e farms, gravando alterações nas respectivas tabelas de log.

## `atualiza_lots_farm-owners.sql`

Migration `once` aditiva. Adiciona `losts` e `cost` a lots e trata o typo histórico `first_acess` → `first_access` quando apenas a coluna antiga existe.

Se as duas colunas existirem, a migration não escolhe automaticamente qual valor prevalece.

## `atualiza_farms-chicken-left.sql`

Adiciona `chickens_now` e `foto_url` a farms, `foto_url` a farm_owners e garante a tabela chicken_left.

## `dataload_inicial.sql`

Seed inicial de desenvolvimento/teste. O runner só executa quando as tabelas centrais estão realmente vazias.

O baseline verifica dados em addresses, enterprises, farms, farm_owners, company_employees e adms. Se qualquer uma tiver dado, o script é registrado como baseline e não injeta a carga.

## `atualiza_consumo_mensal.sql`

Modo `never`. Recalcula leituras de água do mês anterior e ajusta certos consumos de energia. Por alterar dados de negócio artificialmente, fica fora do apply automático.

## `views_galinhas_consumo.sql`

Cria views como `chickens_per_liter`, `chickens_per_kwh` e `integrated_consumption`, agregando consumo do mês anterior por fazenda.

Usa `NULLIF(f.chickens_now, 0)` para evitar divisão por zero.

## `dataload_lots_farm-owners.sql`

Modo `never`. Atualiza losts/cost em lots e first_access em farm_owners. É carga histórica/manual.

## `atualiza_password.sql`

Migration `once` que altera colunas password para TEXT em farm_owners, company_employees e adms. Não altera o hash/valor.

## `midas-user.sql`

Cria/usa schema `midas` e aplica grants à role `midas_ro`. Expõe views controladas para o Midas, incluindo farms, enterprises, consumos, lotes e farm owners.

A view de farm owners seleciona apenas campos necessários ao contexto de IA em vez de expor a tabela de autenticação inteira.

## `midas-resource-import.sql`

Contrato de importação histórica controlada pelo MCP.

Características observadas:

- schema `midas`;
- `pgcrypto` no schema esperado;
- auditoria por `request_id UUID`;
- hash SHA-256 do payload;
- status accepted/rejected;
- contadores de recebidos/inseridos/duplicados/rejeitados;
- errors JSONB;
- schema version;
- timestamps;
- unique indexes de chave natural;
- função `midas.import_resource_records(...)` com `SECURITY DEFINER` e search_path fixado.

Tabelas centrais: `midas.resource_import_requests` e `midas.importer_identities`.

O script aborta se encontrar duplicatas legadas incompatíveis que exijam revisão de DBA.

## Roles técnicas

### `analytics_sync_ro`

Lê subconjunto do OLTP para sync analítico. Não escreve.

### `ms_auth_service_ro`

Lê somente identidades necessárias ao Auth Service.

### `midas_ro`

Acessa views do schema `midas` para contexto da IA.

### `midas_importer`

Usada pela conexão de importação e deve receber somente permissões para o contrato controlado de write.

## Ao adicionar migration

1. escolha `once`, `on_change` ou `never` conscientemente;
2. posicione em `execution_order`;
3. modele dependências no QA;
4. valide em banco vazio;
5. valide sobre versão anterior com dados;
6. teste reexecução se `on_change`;
7. documente consumidores;
8. nunca esconda mutation de dados dentro de “schema idempotente”.
