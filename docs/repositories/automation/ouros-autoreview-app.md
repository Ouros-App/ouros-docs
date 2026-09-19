# ouros-autoreview-app

**Stack:** Node.js, TypeScript, Express, Octokit, GitHub App, Groq/NVIDIA NIM.

**Repo:** [Ouros-App/ouros-autoreview-app](https://github.com/Ouros-App/ouros-autoreview-app)

## Responsabilidade

GitHub App da organização acionada pelo comentário:

```text
/auto-review
```

Ela avalia gates e, quando todos passam, envia uma review `APPROVE` ancorada no SHA revisado.

## Webhook

Endpoint:

```http
POST /webhooks/github
```

Saúde:

```http
GET /health
```

O webhook:

1. lê corpo raw;
2. valida `x-hub-signature-256` com HMAC SHA-256;
3. aceita apenas evento `issue_comment`;
4. exige comentário em PR;
5. exige action `created`;
6. exige conteúdo exatamente igual ao `COMMAND`;
7. obtém installation ID;
8. verifica permissão do autor;
9. responde `202`;
10. executa a revisão.

## Quem pode acionar

O código exige permissão GitHub equivalente a:

- `admin`;
- `maintain`;
- `push`.

## Gates

### PR

Bloqueia quando:

- draft;
- `mergeable !== true`;
- novos commits aparecem durante a análise.

O SHA é carregado no início e comparado novamente antes de aprovar.

### CI

Lê:

- combined commit statuses;
- check runs.

Bloqueia pending/failure/error/cancelled/timed out/action required.

### Sonar

O código não exige que exista um check Sonar específico. Porém, **se houver** um check cujo nome corresponda aos identificadores Sonar configurados, ele precisa terminar em uma conclusão aceita.

Isso significa que repos sem Sonar ainda podem passar, desde que os demais checks estejam em estado aceitável.

### Reviews e threads

Bloqueia:

- qualquer latest review `CHANGES_REQUESTED`;
- CodeRabbit com changes requested;
- threads não resolvidas originadas de CodeRabbit ou Sonar.

### IA

Default:

```text
MIN_SCORE=85
```

Bloqueia se:

- score abaixo do limite;
- finding `high` ou `critical`;
- review por IA falhar.

## Provedores de IA

Ordem:

1. Groq;
2. NVIDIA NIM como fallback.

Cada provedor pode ter duas chaves. As requests das chaves disponíveis são disparadas em paralelo e a primeira review válida vence.

Configuração típica:

- base URL;
- model;
- timeout;
- key 1/key 2.

!!! note "Nome legado na mensagem"
    O formatter ainda chama o score de “NIM score” mesmo quando a review pode ter vindo do Groq. É apenas nomenclatura interna/UI, não garantia do provedor usado.

## GitHub App permissions

Documentadas pelo projeto:

- Pull requests: read/write;
- Issues: read/write;
- Checks: read;
- Commit statuses: read;
- Contents: read;
- Metadata: read.

## Infisical

Pode carregar secrets via Universal Auth antes de importar a configuração principal.

Segredos sensíveis:

- App ID/private key;
- webhook secret;
- Groq/NIM keys.

## Segurança

Pontos fortes observados:

- assinatura do webhook validada em tempo constante;
- usuário acionador passa por permission check;
- approval é ancorado no commit;
- SHA é revalidado depois do trabalho caro;
- unresolved review threads relevantes bloqueiam;
- findings high/critical bloqueiam.

## Comportamento assíncrono

O servidor responde `202` antes de terminar a review, mas a execução continua **dentro do processo Node atual**.

Isso não é uma fila persistente. Reinício do processo durante uma review pode abortar o trabalho.

## Uso

```text
/auto-review
```

O app publica primeiro uma mensagem de início e depois uma decisão `APPROVED` ou `BLOCKED`.

## Manutenção

Ao mudar padrões da org, revise:

- nomes dos checks;
- necessidade de Sonar;
- aliases CodeRabbit;
- score mínimo;
- modelos;
- categorias/severidades;
- permissões do App.
