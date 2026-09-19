# Ambientes

Ouros possui componentes com ambientes locais, QA e produção. Nem todos os serviços possuem uma terceira cópia formal, então o ambiente deve ser identificado pelo **runtime e pelas dependências**, não só pelo nome do repo.

## Local

Objetivo:

- feedback rápido;
- testes;
- desenvolvimento isolado.

Características comuns:

- `.env`;
- localhost;
- containers locais;
- banco de dev/teste;
- providers externos opcionais ou mockados.

Nunca use credenciais de produção “só para facilitar” um teste local.

## QA

Objetivo:

- integração;
- drift;
- migrations;
- contratos;
- comportamento pré-produção.

### PostgreSQL QA

Possui repo dedicado e reconciliador.

Produção é autoridade do contrato gerenciado, mas QA preserva dados de teste e objetos inesperados por padrão.

### MongoDB QA

Banco dedicado `mongodb-ai-qa` com a mesma estrutura de índices esperada para IA.

### APIs

Quando houver branch/deploy QA, configure dependências para recursos QA. Evite um serviço “QA” apontando silenciosamente para banco prod.

## Produção

### PostgreSQL

`postgres-segundo-prod-database` é fonte de verdade do schema gerenciado.

### Mongo

`mongodb-ai-prod`.

### Keycloak

Endpoint confirmado:

```text
https://ouros-keycloak.discloud.app
```

### Auth

Endpoint referenciado pelo User Storage:

```text
https://ms-auth-service.discloud.app
```

## Separação de dados

Checklist antes de deploy:

- host de DB correto;
- nome do database correto;
- environment correto no Infisical;
- secret path correto;
- audience/issuer corretos;
- URLs MCP corretas;
- CORS correto;
- logs identificam ambiente.

## Naming

Não derive segurança apenas de nomes `qa`/`prod`. Valide a configuração efetiva.

Um env chamado QA com `DATABASE_URL` de prod continua sendo produção na prática.

## Promoção

Fluxo recomendado:

```text
local
  ↓
PR + CI
  ↓
QA
  ↓
validação
  ↓
main/prod
```

Para banco, respeite o fluxo de migration e reconciliador em vez de aplicar SQL manual como estado final.
