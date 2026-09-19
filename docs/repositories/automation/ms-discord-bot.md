# ms-discord-bot

**Stack:** Python, discord.py, aiohttp, integração Tuya.

**Repo:** [Ouros-App/ms-discord-bot](https://github.com/Ouros-App/ms-discord-bot)

## Responsabilidade

Opera como console remoto simples do homelab pelo Discord:

- monitora health;
- muda status do canal;
- mostra consumo elétrico;
- liga tomada;
- solicita shutdown seguro;
- corta energia;
- reinicia.

## Monitoramento

Ao ficar ready:

1. cria `HomelabManager`;
2. faz health check inicial;
3. inicia loop a cada 60 s.

Quando o estado muda, atualiza canal e envia notificação.

Nome esperado do canal:

- online: `🟢-server-status`;
- offline: `🔴-server-status`.

Há cooldown de 30 s para rename.

## Comandos

### `!homelab-status`

Retorna:

- online/offline;
- potência atual;
- energia total;
- detalhe do health.

### `!homelab-ligar`

1. verifica se já está online;
2. liga tomada Tuya;
3. consulta health a cada 5 s;
4. tenta por até 24 ciclos;
5. atualiza status.

### `!homelab-desligar`

1. verifica online;
2. chama shutdown seguro;
3. espera 5 s;
4. corta energia via Tuya.

!!! warning "Comportamento importante"
    O retorno de `safe_shutdown()` não é usado para impedir o corte de energia. Depois de esperar 5 s, o código tenta desligar a tomada mesmo se o endpoint de shutdown falhar.

### `!homelab-reboot`

Desliga, espera 5 s e liga novamente.

## Contrato do health service

GET na URL configurada precisa retornar HTTP 200 e:

```json
{"status":"online"}
```

O monitor envia header `ngrok-skip-browser-warning`.

## Shutdown

O código deriva uma URL a partir do health URL e faz POST em `/api/shutdown`.

Payload atual:

```json
{"token":"internal"}
```

!!! warning
    Esse token literal é um contrato frágil. Se este fluxo continuar sendo usado, autenticação do shutdown deveria migrar para secret real ou mecanismo local autenticado.

## Tuya

`TuyaController` recebe:

- access ID;
- access secret;
- device ID.

Também fornece potência e energia para o comando de status.

## Variáveis

- `DISCORD_TOKEN`;
- `GUILD_ID`;
- `STATUS_CHANNEL_ID`;
- `TUYA_ACCESS_ID`;
- `TUYA_ACCESS_SECRET`;
- `TUYA_DEVICE_ID`;
- `HOMELAB_HEALTH_URL`;
- fallback IP/porta.

## Testes

Não há suíte automatizada no estado observado.

## Estrutura

O diretório está grafado `contoller/` no repo. Preserve isso ao referenciar imports atuais até uma refatoração coordenada.
