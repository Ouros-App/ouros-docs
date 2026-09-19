# Segurança

Segurança no Ouros é distribuída entre identidade, bancos, secrets, APIs e automações.

## Princípios observados

1. menor privilégio;
2. secrets fora do Git;
3. identidade central;
4. banco de aplicação sem poder administrativo quando possível;
5. leitura separada de escrita;
6. migrations versionadas;
7. validação local de JWT por resource server;
8. escopo de usuário injetado pelo backend, não escolhido pelo LLM.

## Secrets

Nunca versionar:

- passwords;
- API keys;
- tokens GitHub;
- client secrets;
- private keys;
- connection strings reais;
- webhook secrets.

Veja [Gestão de secrets](../operations/secrets.md).

## Identidade

Keycloak é a autoridade central para tokens novos.

APIs devem validar token, não confiar em claims apenas decodificados.

## Banco

### Auth Service

Usa conexão read-only em nível de aplicação e deve usar role com `SELECT` apenas em produção.

### MCP

Separa:

- `midas_ro`: contexto/leitura;
- `midas_importer`: permissão controlada para função de importação.

### Analytics

Sincronização deve ler produção com role limitada, nunca usar credencial root.

## IA

### Identidade de tools

O AI Server vincula `user_id` no backend para memória/MCP.

Isso evita que o modelo simplesmente invente outro ID para acessar dados.

### Fazenda

O provider compara `farm_id` solicitado com farms autorizadas.

### Memória

Há filtro para impedir armazenamento de credenciais e conteúdo inválido.

### Prompt/tool boundary

Resultados de especialistas chegam ao sintetizador como dados. Isso reduz superfície de instrução indireta entre agentes.

## Webhooks GitHub

Auto Review valida HMAC SHA-256 no corpo raw usando comparação timing-safe.

## GitHub Manager

É um serviço privilegiado. O token permite criar e modificar infraestrutura de repositórios. Proteja como credencial de administração.

## Frontend

Variáveis `VITE_*` são públicas no bundle.

Nunca coloque secret nelas.

## Mobile

Não armazenar token em armazenamento trivial ou log.

## CORS

- Spring: allowlist configurável, sem credenciais;
- Telemetry: rejeita `*` na configuração;
- CORS não é mecanismo de autenticação.

## Logs

Não registrar:

- Authorization header;
- passwords;
- refresh tokens;
- secrets;
- payload de credencial.

Logs devem incluir contexto operacional como request ID, rota, status e duração.

## Dependências

Use Dependabot/alertas e versões controladas. Repo de docs e alguns executores pinam dependências diretas.

## Checklist de review de segurança

- [ ] novo endpoint exige auth?
- [ ] audience correta?
- [ ] autorização por ownership/role?
- [ ] input validado?
- [ ] output expõe dado sensível?
- [ ] secret está fora do repo?
- [ ] DB role tem permissão mínima?
- [ ] mutation é idempotente/retry-safe?
- [ ] timeout existe?
- [ ] log não vaza token?
- [ ] migration preserva dados?
