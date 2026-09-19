# ms-mcp-server-ouros-knowledge-codemode

**Status:** fork experimental.

**Repo:** [Ouros-App/ms-mcp-server-ouros-knowledge-codemode](https://github.com/Ouros-App/ms-mcp-server-ouros-knowledge-codemode)

## Objetivo

Versão experimental do Knowledge MCP criada para testar uma arquitetura orientada a Code Mode sem comprometer o servidor principal.

No snapshot atual, o fork ainda mantém praticamente a mesma base FastAPI/FastMCP, porém possui **menos ferramentas** que o repo principal.

## Tools atuais

- `search_knowledge(query, limit)`;
- `qdrant_status()`;
- `postgres_status()`;
- `get_user_context(user_type, user_id)`;
- `get_user_farm_data(user_type, user_id, limit)`.

Não estão presentes no `mcp_server.py` atual do fork:

- `prepare_resource_import`;
- `import_user_resource_records`.

## Configuração

Mantém Qdrant, NVIDIA embeddings, MIDAS read-only e autenticação MCP. Diferentemente do principal, a configuração atual não declara a conexão dedicada de importação nem parâmetros NIM de extração.

## Como tratar este repo

!!! warning
    Não trate este fork como substituto de produção. Ele deve permanecer isolado enquanto a hipótese de Code Mode é validada.

Critérios úteis para comparação:

- quantidade de tools expostas;
- flexibilidade para geração de gráficos/código;
- superfície de segurança;
- custo de tokens;
- latência;
- capacidade de restringir acesso por usuário/fazenda;
- observabilidade;
- compatibilidade com Discloud.

## Sincronização

Mudanças de segurança/bugfix do MCP principal que também se aplicam ao experimento devem ser portadas conscientemente. Não faça merge cego se a diferença fizer parte do experimento.
