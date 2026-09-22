# Decisões arquiteturais observadas

Esta página não é um conjunto de leis eternas. É um registro das decisões que o código atual expressa, para que futuras mudanças sejam conscientes.

## 1. Keycloak como issuer central

**Estado:** adotado no caminho principal de identidade e nos resource servers usados pelo mobile.

Decisão observada:

- novos access/refresh tokens devem ser emitidos pelo Keycloak;
- Auth Service verifica credenciais legadas;
- serviços novos não devem criar issuer JWT próprio.

Motivo técnico:

- sessão/roles/audience centralizadas;
- JWKS padrão;
- menos duplicação de auth.

Estado observado:

- Spring API, AI Server, Knowledge MCP e Telemetry validam JWT Keycloak por JWKS, issuer e audience;
- Auth Service continua como bridge para credenciais legadas e não assina o JWT final;
- o Android usa `ouros-mobile` com Authorization Code + PKCE;
- o AI Server usa o JWT do usuário como subject token em Standard Token Exchange v2 e encaminha ao Knowledge MCP apenas o JWT delegado;
- Telemetry adiciona uma barreira de autorização: realm role `admin`.

## 2. Credenciais legadas continuam no banco de negócio

O User Storage do Keycloak é read-only e delega validação para Auth Service.

Isso evita uma migração explosiva de passwords.

Consequência:

- Keycloak conhece a identidade federada;
- hash legado continua no PostgreSQL;
- alteração de senha exige desenho específico, não escrita arbitrária do provider.

## 3. Schema pertence ao repo de banco

APIs consomem schema, mas não são a fonte de verdade dele.

No Spring:

```text
hibernate.ddl-auto=none
```

Consequência:

- mudança estrutural deve virar SQL versionado;
- deploy de API e migration podem ser coordenados;
- estado do banco fica auditável por commit.

## 4. Produção é autoridade do contrato; QA preserva caos útil

QA não é cópia descartável.

Decisão:

- contrato gerenciado vem de prod;
- dados de teste permanecem;
- drift inesperado é reportado/reconciliado;
- migration não crítica pode ser isolada.

Isso transforma QA em ambiente de teste realista, não em fotografia limpa que quebra na primeira intervenção manual.

## 5. Analytics é projeção derivada

O banco analítico não deve receber escrita transacional de negócio como fonte primária.

Decisão:

- prod continua fonte de verdade;
- analytics lê com role limitada;
- joins/agregações/limpeza acontecem fora do OLTP.

## 6. Estado da IA fica separado do domínio

Memória/checkpoints/thread ownership usam MongoDB dedicado.

Motivo:

- ciclo de vida diferente;
- estruturas específicas do LangGraph;
- não poluir schema transacional.

Índices são versionados em repos próprios.

## 7. Tools de usuário não recebem identidade livre do LLM

No AI Server, `user_id` é injetado pelo backend.

Farm IDs também passam por verificação contra o contexto autorizado.

Objetivo:

- modelo não pode trocar de identidade só por gerar outro argumento.

## 8. IA não recebe SQL arbitrário para mutações

Importação segue:

```text
arquivo
  ↓
extração/preview
  ↓
revisão
  ↓
função PostgreSQL controlada
```

A tool não ganha poder de executar SQL livre.

## 9. Importação usa idempotência por request ID

O schema de importação mantém `request_id UUID`.

Isso permite evitar duplicação acidental em retries.

## 10. Knowledge e geração ficam separados

Knowledge MCP concentra:

- Qdrant;
- contexto MIDAS;
- acesso controlado a dados.

AI Server concentra:

- roteamento;
- especialistas;
- síntese;
- memória;
- policy de tools.

Essa separação reduz acoplamento entre protocolo de conhecimento e lógica conversacional.

## 11. Especialistas retornam estrutura, sintetizador retorna linguagem

Agentes especialistas produzem JSON com fatos/recomendações/status.

O agente default faz a síntese final.

Benefícios:

- fan-out/fan-in claro;
- menos respostas concorrentes;
- melhor controle de tool use;
- resultado intermediário testável.

## 12. Templates são ponto de partida, não policy engine

Um template pode ficar desatualizado.

Um repo derivado não recebe correções automaticamente.

Portanto:

- mudança crítica no template pode exigir backport para consumidores;
- GitHub Manager também precisa evoluir quando a política muda.

## 13. CI de banco e deploy de banco são separados

Primeiro validação, depois apply em `main`.

Isso reduz a chance de PR executar mutation em ambiente final.

QA acrescenta camada de reconciliação própria.

## 14. Docs são centralizadas

README:

> como operar aquele repositório.

Ouros Docs:

> como o sistema inteiro funciona.

A documentação central deve registrar divergências sem reescrever silenciosamente a realidade.

## 15. Exceções de autenticação são isoladas

O padrão para usuários é Keycloak + JWT RS256/JWKS/audience.

As exceções atuais são explícitas:

- `ms-auth-service-broker`: compatibilidade first-party legada durante o rollout;
- `ms-ai-server-debug`: Direct Access Grant confidencial, exclusivo do console de debug;
- service clients: Client Credentials para identidade máquina-a-máquina.

Novos clientes mobile/web não devem copiar essas exceções.

## Como mudar uma decisão

Quando uma dessas decisões deixar de fazer sentido:

1. documente problema atual;
2. proponha alternativa;
3. liste consumidores;
4. defina migration/rollout;
5. atualize código;
6. atualize esta página;
7. preserve histórico no Git.

A pior arquitetura é a que muda sem ninguém perceber que uma decisão mudou.
