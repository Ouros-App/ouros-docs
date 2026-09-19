# Runbook: Midas, MCP ou IA degradados

## Sintomas

- chat responde fallback/default demais;
- erro 5xx;
- tools não são chamadas;
- dados de fazenda não aparecem;
- memória some;
- Qdrant falha;
- provider LLM timeout;
- resposta fica lenta.

## 1. Separe AI Server de MCP

AI Server:

```text
GET /health
GET /metrics
```

Knowledge MCP:

```text
GET /health
tool qdrant_status
tool postgres_status
```

## 2. Provider LLM

Cheque:

- Groq keys;
- NIM keys;
- timeout;
- modelo configurado;
- rate limit externo.

O serviço pode degradar sem necessariamente deixar de subir.

## 3. MCP não carrega tools

Cheque:

- `MCP_URL`;
- Bearer/JWT;
- `MCP_RESOURCE_URL`;
- issuer;
- TTL;
- conectividade.

## 4. Dados pessoais não aparecem

Cheque:

- `user_id` numérico;
- `MCP_USER_TYPE`;
- identidade autenticada;
- `get_user_context`;
- farms autorizadas;
- role `midas_ro`.

Não contorne scoping passando outro ID manualmente.

## 5. Qdrant

Cheque:

- URL/key;
- collection name;
- status da coleção;
- embedding model;
- ingestão recente.

Se busca começou a piorar após trocar modelo de embedding, suspeite de incompatibilidade dos vetores existentes.

## 6. MongoDB

Cheque:

- URI;
- database;
- índices;
- `thread_owners`;
- checkpoints;
- memória.

Erro de ownership de thread é comportamento de segurança, não corrupção automática.

## 7. Importação

Se preview funciona e write falha:

- conexão `MIDAS_IMPORT_DATABASE_URL`;
- role do importer;
- função `midas.import_resource_records`;
- request UUID;
- payload validado.

## 8. Latência

Quebre por estágio:

```text
router
specialist
MCP
LLM
Mongo
síntese
```

Use métricas e logs, não intuição.

## 9. Fallback operacional

Se provider principal cair e fallback existir, valide que:

- key está configurada;
- modelo existe;
- timeout não consome o budget inteiro antes do fallback.

## 10. Depois

- registrar provider/falha sem secret;
- criar teste;
- ajustar timeout/retry;
- atualizar observabilidade.
