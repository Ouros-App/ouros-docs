# Troubleshooting

## “A API está viva, mas não funciona”

Cheque diferença entre liveness e readiness.

Exemplos:

- Auth: `/health` pode estar 200 enquanto `/ready` falha por PostgreSQL/Redis;
- Telemetry: `/health` pode estar 200 enquanto credenciais Databricks estão incompletas.

## 401

Perguntas:

1. token foi enviado?
2. issuer correto?
3. expirou?
4. audience contém o serviço?
5. assinatura/JWKS válidos?
6. endpoint ainda usa JWT legado ou Keycloak?
7. Bearer estático é esperado naquele serviço?

## 403

Pode significar autenticação válida, mas falta de autorização/ownership.

No Auth interno, credencial humana inválida retorna 403 para reservar 401 à autenticação da própria integração.

## 429 no Auth

Rate limit de tentativa de credencial.

Cheque IP/email e `Retry-After`.

Não desative rate limiting como primeira solução.

## 502/504 no Telemetry

- 502: integração Databricks falhou;
- 504: timeout.

Cheque OAuth, permissões do service principal, workspace e SQL Warehouse.

## Midas não usa dados pessoais

Cheque:

- `MCP_URL`;
- token/JWT MCP;
- `MCP_RESOURCE_URL`;
- identidade numérica;
- allowlist do agente;
- `MIDAS_DATABASE_URL`;
- ownership da farm.

## Midas responde sem IA

Chaves de Groq/NVIDIA podem ser opcionais para startup. Serviço pode subir e cair em respostas default quando nenhum provider está funcional.

## Thread de IA inacessível

Ownership de `thread_id` é persistido. Uma thread criada por usuário A não deve ser reutilizada por B.

## Migration SQL não roda

Cheque:

- modo `once/on_change/never/always`;
- checksum em `controle_scripts_sql`;
- ordem em `config.yaml`;
- baseline;
- commit identificado;
- grants/owner.

## QA não sincroniza tudo

Abra o relatório do reconciliador.

Procure:

- `QUARANTINED`;
- `BLOCKED`;
- dependência;
- migration crítica;
- drift local.

Não conclua que “sync inteiro quebrou” só porque um item ficou isolado.

## Unique index Mongo falha

Pode existir duplicidade nos dados atuais. O executor de prod evita aplicar índice unique sobre conflito.

Corrija dados conscientemente antes de retry.

## Frontend “loga” mas API retorna 401

O login atual de `ms-ouros-front-web` é demo e não obtém token real.

## Android não acessa API

No snapshot atual:

- `ApiClient` vazio;
- repo não declara `INTERNET` no Manifest.

Networking ainda precisa ser implementado.

## FastAPI template não builda Docker

O Dockerfile do `ms-fastapi-template` espera `.env` e arquivos de workflow dentro de `app/templates/workflows/` que não existem na árvore atual.

## Auto Review bloqueado

Cheque:

- PR draft;
- mergeability;
- checks pending/failing;
- review `CHANGES_REQUESTED`;
- threads CodeRabbit/Sonar abertas;
- score de IA;
- finding high/critical;
- SHA mudou durante review.

## “CodeQL obrigatório fica pendente”

Confirme que o workflow realmente cria check com o **mesmo nome/contexto** exigido pela branch protection.

## Pages não publica

No `ouros-docs`:

1. Pages precisa estar habilitado para GitHub Actions uma vez;
2. build strict precisa passar;
3. deploy job precisa de `pages: write` e `id-token: write`.
