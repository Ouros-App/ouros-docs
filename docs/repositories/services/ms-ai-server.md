# ms-ai-server

**Stack:** Python 3.12, FastAPI, LangGraph/LangChain, MongoDB, Groq, NVIDIA NIM, MCP.

**Repo:** [Ouros-App/ms-ai-server](https://github.com/Ouros-App/ms-ai-server)

## Responsabilidade

Backend do Midas. Recebe uma mensagem, autentica o usuário, roteia intenções para especialistas, permite uso controlado de memória/MCP e produz uma resposta final sintetizada.

## API

| Método | Rota | Autenticação |
| --- | --- | --- |
| GET | `/` | Bearer |
| GET | `/health` | Bearer |
| GET | `/metrics` | Bearer |
| POST | `/v1/chat` | Bearer/JWT conforme configuração |
| GET | `/v1/chat/{thread_id}/history` | Bearer/JWT conforme configuração |

Swagger/ReDoc/OpenAPI permanecem públicos.

### Chat

Payload:

```json
{
  "user_id": "6",
  "message": "Como está meu consumo de energia?",
  "thread_id": "opcional"
}
```

Limites do schema:

- `user_id`: 1..128 chars;
- `message`: 1..8000;
- `thread_id`: 1..128; UUID automático quando omitido.

Resposta:

- `thread_id`;
- `message`;
- `agents`;
- `tools`.

## Grafo de agentes

```mermaid
flowchart LR
    G[Input guardrail] --> R[Router]
    R --> F[faq]
    R --> S[sustainability]
    R --> K[ranking]
    R --> U[support]
    R --> B[fallback]
    F --> C[collect]
    S --> C
    K --> C
    U --> C
    B --> C
    C --> D[default synthesizer]
    D --> O[Output guardrail]
```

O router pode selecionar até quatro especialistas.

Há roteamento determinístico por palavras-chave antes de recorrer a um LLM, reduzindo custo/latência para intenções óbvias.

## Contrato dos especialistas

Especialistas devem retornar JSON estruturado com:

- `status`;
- `facts`;
- `recommendations`;
- `missing_data`;
- `sources`.

O `default` recebe esses resultados como **dados, não instruções**, e produz a única resposta natural final.

## Models

Perfis rápidos são usados por router, guardrails e especialistas leves. Ranking/sustentabilidade podem usar perfil mais potente.

Groq é o caminho primário conforme disponibilidade; NVIDIA NIM atua como fallback do perfil.

## Memória

Tools injetadas pelo backend:

- `recall_user_memories`;
- `save_user_memory`.

O `user_id` não é fornecido pelo modelo à tool; ele é fechado no backend.

Memórias passam por validação para evitar credenciais e conteúdo inadequado.

## MCP

Allowlist por agente:

| Agente | Tools |
| --- | --- |
| faq | `search_knowledge`, `get_user_context` |
| sustainability | `search_knowledge`, `get_user_context`, `get_user_farm_data` |
| ranking | `get_user_context`, `get_user_farm_data`, `postgres_status` |
| support | `search_knowledge`, `get_user_context` |
| fallback | `search_knowledge` |

Tools de usuário têm identidade vinculada pelo backend. O modelo não escolhe `user_id`.

O provider também filtra `farm_id` contra a lista autorizada retornada pelo contexto do usuário.

## Persistência

MongoDB armazena:

- memórias;
- ownership de threads;
- checkpoints;
- checkpoint writes.

O schema/indexação é gerenciado pelos repositórios `mongodb-ai-*-database`.

## Configuração

Grupos principais:

- MongoDB: `MONGODB_URI`, `MONGODB_DATABASE`;
- Groq: keys e modelos fast/powerful;
- NIM: key, modelos, base URL;
- auth: issuer Keycloak, audience `ms-ai-server` e JWKS;
- MCP: URL/resource e encaminhamento do JWT Keycloak autenticado ao Knowledge MCP;
- timeouts/temperatura.

## Identidade

Quando `AUTH_REQUIRE_USER_JWT=true`, o `sub`/user ID autenticado precisa coincidir com `user_id`. Threads também possuem owner persistido e não podem trocar de usuário posteriormente.

### Interop com Knowledge MCP

O provider pode gerar JWT HS256 por usuário via `MCP_JWT_SECRET`, mas o Knowledge MCP atual só aceita token estático por igualdade exata.

No estado atual, use:

```text
MCP_ACCESS_TOKEN (AI Server)
=
MCP_AUTH_TOKEN (Knowledge MCP)
```

até existir um verifier JWT compatível no MCP.

## Observabilidade

`/metrics` cobre HTTP, latência, resultado de chat, agentes e tools.

## Testes

Há testes para API, grafo, guardrails, prompts, modelos, MCP, ownership, tools, métricas, config e Infisical.
