# Lacunas e inconsistências atuais

Este inventário registra diferenças observadas entre o estado atual e uma plataforma totalmente integrada. Não é uma lista de culpa nem um ranking de prioridade.

## Identidade em transição

Hoje coexistem:

- Keycloak como issuer central;
- Auth Service como bridge para credenciais legadas;
- Spring API já migrado para JWT Keycloak/JWKS/audience;
- Bearer estático no Telemetry e Knowledge MCP;
- Bearer compartilhado/JWT HS256 local no AI Server.

Implicação:

- clientes não podem assumir que o mesmo token é aceito por todos os serviços;
- novos resource servers devem seguir o padrão Keycloak do Spring;
- migrações de Telemetry/AI/MCP precisam preservar compatibilidade durante o rollout.

## AI Server e Knowledge MCP têm modos de auth incompatíveis

O AI Server contém código para gerar JWT HS256 curto por usuário para o MCP quando `MCP_JWT_SECRET` é usado.

O Knowledge MCP atual usa `StaticTokenVerifier` e aceita somente igualdade exata com `MCP_AUTH_TOKEN`.

Portanto, o modo interoperável hoje é:

```text
AI Server MCP_ACCESS_TOKEN
        ==
Knowledge MCP MCP_AUTH_TOKEN
```

O caminho JWT per-user exige implementar um verifier JWT compatível no MCP antes de ser habilitado.

## Web ainda é scaffold

`ms-ouros-front-web` ainda mantém:

- nome de pacote do template;
- páginas de `products`;
- login demo;
- token literal;
- API client sem auth/refresh;
- services genéricos.

Implicação:

> build verde não significa frontend integrado ao Ouros.

## Android ainda é scaffold

`ouros-android-app` possui:

- `ApiClient` vazio;
- `MainRepository` vazio;
- Activities/Fragment mínimos;
- sem permissão `INTERNET` observada;
- sem sync offline implementado.

## Analytics ainda não materializa o modelo

`ouros-analytics-database/config.yaml` possui:

```yaml
execution_order: []
```

A infraestrutura de versionamento existe, mas o modelo analítico final ainda não está versionado ali.

## Telemetry ainda depende de Databricks

O serviço atual consulta Databricks.

Isso é diferente da ideia futura de dashboards por usuário baseados no banco analítico.

Não documente acesso ao analytics como feature pronta do Telemetry até existir no código.

## FastAPI template com Docker inconsistente

O Dockerfile do `ms-fastapi-template`:

- copia `.env` durante build;
- verifica arquivos em `app/templates/workflows/`.

Esses workflows não existem na árvore atual.

O Dockerfile precisa de correção antes de ser considerado base confiável.

## Templates de app com CI desigual

Alguns templates de aplicação possuem apenas metadata semanal, enquanto produtos derivados já possuem CI completa.

Implicação:

- regressão pode nascer no template e só aparecer no repo gerado.

## Observabilidade desigual

Prometheus bem definido:

- AI Server;
- Telemetry;
- GitHub Manager.

Sem endpoint Prometheus observado:

- Auth Service;
- Spring API;
- Knowledge MCP;
- Auto Review;
- Discord bot.

## README do Spring API ficou para trás

O README local ainda descreve o serviço como muito menor do que o código real.

A documentação central corrige essa visão, mas o README do repo ainda merece atualização.

## Convenções Infisical não são uniformes

Existem pelo menos dois conjuntos de nomes:

```text
INFISICAL_CLIENT_ID / CLIENT_SECRET / ENVIRONMENT / SECRET_PATH
```

e

```text
INFISICAL_TOKEN / ENV / PATH
```

Isso aumenta custo cognitivo e risco de deploy incorreto.

## GitHub Manager mantém estado de criação em memória

`creation_id` e progresso não são persistidos.

Reinício do processo perde histórico operacional dessas criações.

## GitHub Manager ainda carrega padrão Sonar

O manager/template de workflows ainda possui lógica Sonar em algumas famílias.

A política atual da organização não exige Sonar universalmente.

Risco:

- novo repo nascer com regra antiga.

## Auto Review usa nomenclatura “NIM score”

O fluxo pode usar Groq como provider primário, mas o texto de resultado ainda pode chamar o score de “NIM score”.

É uma inconsistência de nomenclatura, não necessariamente de cálculo.

## Auto Review não tem fila persistente

Após responder 202 ao webhook, a revisão continua no processo Node.

Se o processo reiniciar, o job pode ser perdido.

## Bot do homelab corta energia após tentativa de shutdown

No fluxo atual:

1. chama shutdown seguro;
2. espera 5 s;
3. tenta desligar tomada.

O retorno do shutdown seguro não bloqueia o corte.

Também há um token literal `internal` no payload de shutdown.

## Perfil público da organização tem placeholders automáticos

`.github/profile/README.md` contém blocos de linguagens/repos ainda aguardando coleta automática.

## README do Knowledge MCP diverge das tools atuais

A tabela do README local descreve argumentos de importação que não aparecem na assinatura atual de `app/mcp_server.py`, incluindo confirmação explícita em nível de tool.

A documentação central segue o código executável e registra essa diferença para evitar clientes construídos contra um contrato inexistente.

## Code Mode é experimento, não produção

O fork `ms-mcp-server-ouros-knowledge-codemode` não possui todas as tools do MCP principal.

Não deve ser tratado como drop-in replacement.

## Documentação e implementação podem divergir

Essa própria página existe porque o projeto evolui rápido.

Regra:

> quando descobrir divergência, corrija o código ou a documentação, mas não deixe os dois contando histórias diferentes.
