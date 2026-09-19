# Classificação de dados

Classificação prática para orientar armazenamento, logs, acesso e documentação.

## Nível 0 — Público

Exemplos:

- URL pública do Keycloak;
- issuer;
- JWKS;
- nomes de serviços;
- documentação;
- model names;
- portas públicas;
- código open source.

Pode aparecer em Git e docs.

Ainda assim, “público” não significa que toda metadata operacional precise ser exposta sem motivo.

## Nível 1 — Interno operacional

Exemplos:

- nomes de ambientes;
- IDs de dashboard não sensíveis;
- topology interna;
- nomes de roles;
- métricas agregadas;
- request IDs;
- nomes de collections;
- commit SHA;
- status de migration.

Pode aparecer em logs internos e docs técnicas.

Evite exposição desnecessária em UI pública.

## Nível 2 — Dados de negócio

Exemplos:

- consumo de água;
- consumo de energia;
- lotes;
- capacidade produtiva;
- perdas;
- custo;
- metas;
- pagamentos;
- desempenho de fazenda.

Regras:

- acesso por role/ownership;
- evitar logs crus;
- analytics deve aplicar princípio de minimização;
- exportações precisam respeitar escopo.

## Nível 3 — Dados pessoais

Exemplos no schema:

- nome;
- email;
- document number;
- telefone;
- foto;
- vínculo usuário ↔ fazenda/empresa.

Regras:

- não usar como debug público;
- acesso mínimo;
- cuidado em analytics;
- evitar enviar ao LLM quando não necessário;
- mascarar em exemplos.

## Nível 4 — Credenciais e secrets

Exemplos:

- password;
- hash de password;
- access token;
- refresh token;
- API key;
- private key;
- client secret;
- session secret;
- webhook secret;
- URI com senha;
- credencial de banco.

Regras:

- nunca no Git;
- nunca em `VITE_*`;
- nunca em logs;
- armazenar em secret manager/runtime;
- rotacionar após suspeita de exposição.

## IA e documentos

### Prompt do usuário

Pode conter Nível 2 ou 3.

Não deve ser logado integralmente por padrão.

### Memória persistente

Pode conter informação contextual do usuário.

O AI Server possui filtro para rejeitar credenciais, mas isso não transforma qualquer texto em dado “seguro”.

### Documento importado

Pode conter PII ou dado operacional.

Evite:

- logar base64;
- logar Markdown completo;
- reter arquivo além do necessário sem política.

### Embedding

Embedding não deve ser tratado automaticamente como dado anônimo.

Ele ainda é derivado de conteúdo e pode exigir a mesma governança do texto de origem.

## Analytics

Objetivo é reduzir custo/risco de análise, não criar uma cópia irrestrita do prod.

Pergunte por coluna:

- é necessária?
- precisa de ID direto?
- pode ser pseudonimizada?
- precisa de granularidade individual?
- qual retenção?

## Logs

### Permitido

- status;
- duração;
- rota normalizada;
- request ID;
- operação;
- tipo de erro sanitizado;
- commit.

### Evitar

- body completo;
- email sem necessidade;
- document number;
- prompt;
- SQL com valores pessoais.

### Proibido

- senha;
- token;
- secret;
- private key.

## Métricas

Labels de Prometheus devem ter baixa cardinalidade.

Não use como label:

- user ID;
- email;
- farm ID arbitrário;
- thread ID;
- request ID.

Isso gera explosão de cardinalidade e pode vazar identificadores.

## Exemplos de documentação

Ruim:

```text
DATABASE_URL=postgres://<usuario>:<senha>@<host>/<database>
```

Bom:

```text
DATABASE_URL=<postgres-connection-string>
```

Ruim:

```json
{"email":"pessoa-real@empresa.com","document_number":"123..."}
```

Bom:

```json
{"email":"usuario@example.com","document_number":"<documento>"}
```

## Regra de minimização

Se um serviço não precisa de um campo, não entregue o campo.

Isso vale para:

- SQL;
- DTO;
- JWT claim;
- MCP result;
- analytics;
- log;
- LLM context.

## Ao criar campo novo

Documente:

1. classificação;
2. fonte;
3. owner;
4. leitores;
5. escritores;
6. retenção;
7. se entra em analytics;
8. se entra em IA;
9. se aparece em logs;
10. como corrigir/apagar.
