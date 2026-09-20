# Auto Review: contrato de webhook

O `ouros-autoreview-app` é um GitHub App acionado por comentário em Pull Request.

Ele não é uma API pública de produto. A superfície HTTP principal é um webhook autenticado pelo GitHub.

## Endpoints

### GET `/health`

```json
{"ok": true}
```

### POST `/webhooks/github`

Corpo máximo: **2 MiB**.

O servidor lê o corpo bruto para validar a assinatura antes de fazer parse JSON.

## Headers esperados

```http
X-Hub-Signature-256: sha256=<assinatura>
X-GitHub-Event: issue_comment
Content-Type: application/json
```

A assinatura é HMAC SHA-256 usando o webhook secret configurado.

## Quando o evento é processado

O app continua somente quando:

1. assinatura é válida;
2. evento é `issue_comment`;
3. `action == "created"`;
4. comentário pertence a uma PR;
5. body do comentário é exatamente o comando configurado;
6. payload contém `installation.id`;
7. autor possui permissão suficiente.

Comando padrão do projeto:

```text
/auto-review
```

Permissões aceitas para o autor:

- admin;
- maintain;
- push.

## Respostas HTTP

| Status | Situação |
| ---: | --- |
| 200 | health |
| 202 | evento ignorado legitimamente ou review aceita |
| 400 | JSON inválido ou installation ID ausente |
| 401 | assinatura do webhook inválida |
| 403 | autor sem permissão ou permission check falhou |

Evento que não interessa não vira erro. Exemplo:

```json
{"ignored": true}
```

Quando a review é aceita:

```json
{"accepted": true}
```

## Execução após 202

Depois de responder 202, o processo:

1. comenta que a review começou;
2. carrega estado da PR;
3. avalia gates;
4. roda review por IA;
5. publica APPROVED ou BLOCKED.

!!! warning "Sem fila persistente"
    O trabalho continua no próprio processo Node. Reinício durante uma review pode perder a execução.

## Gates principais

A review pode bloquear por:

- PR draft;
- mergeability inválida;
- checks pending/failing;
- review `CHANGES_REQUESTED`;
- threads relevantes CodeRabbit/Sonar abertas;
- score IA abaixo do mínimo;
- finding high/critical;
- SHA da PR mudar durante a análise.

O SHA é conferido novamente antes do approval para evitar aprovar uma versão diferente da revisada.

## Providers de IA

Ordem atual:

1. Groq;
2. NVIDIA NIM como fallback.

Pode haver duas chaves por provider; as disponíveis podem ser tentadas em paralelo.

## Segurança

- validar assinatura antes de parse/processar;
- nunca aceitar comando apenas pelo conteúdo do comentário;
- permission check é obrigatório;
- private key do GitHub App e webhook secret ficam fora do Git;
- corpo do webhook não deve ser logado indiscriminadamente.

## Teste manual

Não gere assinatura “fake” em produção. Para validar o fluxo completo, prefira redelivery de webhook no GitHub App ou um ambiente de teste com secret separado.
