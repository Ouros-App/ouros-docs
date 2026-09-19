# postgres-database-template

**Stack:** Python, PostgreSQL, psycopg2, YAML.

**Repo:** [Ouros-App/postgres-database-template](https://github.com/Ouros-App/postgres-database-template)

## Objetivo

Template para versionar um PostgreSQL por arquivos SQL, sem depender de ORM migrations.

## Componentes

```text
config.yaml
scripts/apply_sql.py
sql/versionamento.sql
sql/atualiza_controle_versoes_identity.sql
tests/test_apply_sql.py
.github/workflows/
```

## Configuração

`config.yaml` define:

- engine;
- conexão alvo;
- conexão bootstrap;
- owner;
- path de SQL;
- tabela de versão;
- schema de versionamento;
- ordem de execução.

## Conexões

### Bootstrap

Usada para garantir role/database quando necessário:

- `POSTGRES_ROOT_DB`;
- `POSTGRES_ROOT_USER`;
- `POSTGRES_ROOT_PASSWORD`.

### Owner

Usada para aplicar objetos no banco alvo:

- `POSTGRES_DB`;
- `POSTGRES_USER`;
- `POSTGRES_PASSWORD`.

## Modos

| Modo | Uso |
| --- | --- |
| `once` | migration imutável |
| `on_change` | definição idempotente |
| `always` | rotina segura para repetição |
| `never` | legado/desativado |

## Estado inicial

O template executa:

```yaml
- file: atualiza_controle_versoes_identity.sql
  mode: once
```

Projetos derivados devem substituir/adicionar migrations de domínio.

## Controle

`controle_versoes` registra o estado global.

`controle_scripts_sql` rastreia arquivo/checksum/commit e permite decidir se um script precisa rodar.

## CI

O template inclui:

- compilação/validação Python;
- testes;
- validação de scaffold;
- políticas para bloquear SQL destrutivo automático;
- Apply SQL On Main.

## Operações destrutivas

O CI base procura operações como:

- `TRUNCATE`;
- `DROP DATABASE`;
- `DROP TABLE`;
- `DROP SCHEMA`.

Isso é proteção de esteira, não substituto de revisão de migration.

## Uso

```bash
cp .env.example .env
pip install -r requirements.txt
python scripts/apply_sql.py
```

## Repositórios derivados

- `postgres-segundo-prod-database`;
- `postgres-segundo-qa-database`;
- `ouros-analytics-database`.

Cada um evoluiu ou deverá evoluir sua política própria.
