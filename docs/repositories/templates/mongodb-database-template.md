# mongodb-database-template

**Stack:** Python, MongoDB/PyMongo, YAML, JSON de comandos.

**Repo:** [Ouros-App/mongodb-database-template](https://github.com/Ouros-App/mongodb-database-template)

## Objetivo

Template para versionar comandos estruturais MongoDB de forma controlada.

## Estrutura

```text
config.yaml
mongo/colecoes.json
scripts/apply_mongo.py
tests/test_apply_mongo.py
.github/workflows/
```

## Configuração

```yaml
database:
  engine: mongodb
  connection_url: ${MONGODB_URI}
  tls: ${MONGODB_TLS}
  tls_ca_file: ${MONGODB_TLS_CA_FILE}
  scripts_path: mongo
  version_collection: controle_versoes
  execution_order:
    - file: colecoes.json
      mode: on_change
```

No template genérico, o database pode ser derivado da própria URI.

## Formato dos scripts

Arquivos em `mongo/` contêm uma lista JSON de comandos MongoDB.

O `colecoes.json` do template está vazio no snapshot atual, servindo como placeholder.

## Modos

- `always`;
- `on_change`;
- `once`;
- `never`.

## Transações

Entradas são transacionais por padrão quando o ambiente/operação permite.

Para operação não transacional, declare:

```yaml
transactional: false
idempotent: true
```

Isso obriga o autor a reconhecer que a operação pode ser parcialmente aplicada e deve tolerar retry.

## Retomada

O executor mantém progresso de scripts não transacionais para conseguir retomar depois de falha.

Coleções de controle incluem:

- `controle_scripts_mongo`;
- `controle_contadores`;
- `controle_versoes`.

## TLS

Variáveis:

- `MONGODB_URI`;
- `MONGODB_TLS`;
- `MONGODB_TLS_CA_FILE`.

## CI/CD

Inclui validação do executor/config/scripts e workflow para aplicar após push em `main`.

## Repositórios derivados

- `mongodb-ai-prod-database`;
- `mongodb-ai-qa-database`.

Esses repos adicionam índices específicos para memória e checkpointer da IA.
