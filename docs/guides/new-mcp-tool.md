# Guia: adicionar tool ao Midas

Uma tool é uma fronteira de segurança.

## 1. Escolha o lugar certo

Use Knowledge MCP quando a capacidade envolve Qdrant, PostgreSQL, contexto de usuário, importação ou integração externa de conhecimento.

Use tool local do AI Server quando a capacidade é estritamente de memória/orquestração interna.

## 2. Exponha capacidade de domínio

Evite:

~~~text
execute_sql(sql)
~~~

Prefira:

~~~text
get_energy_summary(period)
~~~

O modelo não deve ganhar acesso genérico à infraestrutura.

## 3. Identidade

Para dado de usuário:

- user_id deve ser vinculado pelo backend;
- farm/enterprise precisam de scoping;
- argumentos do modelo não substituem autorização.

## 4. Banco

Leitura:

- query fixa;
- role read-only;
- limite;
- timeout.

Escrita:

- role separada;
- função controlada;
- request/idempotency key;
- auditoria;
- preview quando IA interpreta o dado.

## 5. Output bounded

Retorne estrutura compacta, por exemplo:

~~~json
{
  "status": "ok",
  "items": [],
  "meta": {"count": 0}
}
~~~

Não despeje centenas de registros no contexto do LLM.

## 6. Allowlist

Criar a tool no MCP não significa entregá-la a todos os agentes.

Atualize apenas a allowlist dos especialistas que realmente precisam dela.

## 7. Timeout e erro

Diferencie input inválido, forbidden/ownership, not found, timeout e dependência indisponível.

Uma tool não pode consumir todo o budget do chat antes da síntese final.

## 8. Observabilidade

Registre chamada, status e duração. Não use user ID ou farm ID como label Prometheus.

## 9. Testes

- input válido/inválido;
- identity scope;
- farm scope;
- limite;
- timeout;
- dependência falhando;
- output shape.

## 10. Tool de escrita

Padrão recomendado pelo fluxo atual:

~~~text
interpretar
  ↓
preview
  ↓
validar
  ↓
mutation controlada
  ↓
auditar
~~~

## Checklist

- [ ] capacidade de domínio;
- [ ] sem SQL arbitrário;
- [ ] identidade fechada;
- [ ] role mínima;
- [ ] output bounded;
- [ ] timeout;
- [ ] allowlist;
- [ ] testes;
- [ ] métricas;
- [ ] docs.
