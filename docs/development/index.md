# Desenvolvimento

## Fluxo padrão

```text
issue/tarefa
   ↓
branch
   ↓
implementação + testes
   ↓
documentação
   ↓
PR
   ↓
CI / CodeRabbit / checks
   ↓
merge
```

## Escolha o repo certo

Evite colocar responsabilidade no lugar “mais fácil”.

Exemplos:

- schema → repo de banco;
- token/identidade → Keycloak/Auth;
- CRUD de domínio → Spring API;
- raciocínio/agentes → AI Server;
- acesso contextual para IA → MCP;
- rendering Databricks → Telemetry.

## Mudança de API

Ao alterar contrato:

1. schema/request/response;
2. testes;
3. OpenAPI;
4. consumidores;
5. docs centrais;
6. compatibilidade de rollout.

## Mudança de banco

Schema primeiro no repo de banco.

Nunca dependa de `ddl-auto` para produção.

## Conventional Commits

A org usa/gerou validações no estilo:

```text
feat(scope): ...
fix(scope): ...
docs: ...
chore: ...
ci: ...
```

## Branch protection

Política varia por repo. Não copie uma regra de um repo para todos automaticamente.

O `ouros-docs`, por exemplo, usa PR sem approval humano obrigatório, com CI e CodeQL.

## Code review automatizado

O `ouros-autoreview-app` pode ser acionado com:

```text
/auto-review
```

Ele é gate auxiliar, não substitui ownership técnico quando uma mudança tem risco arquitetural ou de dados.

## Templates

Novos repos podem nascer via `ms-github-manager`, mas os templates são scaffolds e podem carregar padrões antigos.

Após gerar:

- remova placeholders;
- confira CI;
- ajuste auth;
- ajuste deploy;
- revise Sonar/CodeQL conforme política atual;
- documente domínio.

## Documentação

Toda mudança que invalida uma página desta wiki deve atualizar a página na mesma PR quando viável.

Veja [Escrevendo documentação](writing-docs.md).
