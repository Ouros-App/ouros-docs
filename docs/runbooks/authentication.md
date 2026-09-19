# Runbook: falha de autenticação

Use este runbook quando usuários não conseguem logar, tokens deixam de funcionar ou APIs começam a responder 401/403 em massa.

## 1. Identifique o estágio

Fluxo simplificado:

```text
Client
 ↓
Keycloak / Auth Service
 ↓
PostgreSQL identidade
 ↓
JWT
 ↓
Resource server
```

Pergunta inicial:

> a falha acontece antes ou depois da emissão do token?

## 2. Teste liveness/readiness

Auth Service:

```text
GET /health
GET /ready
```

Se `/health=200` e `/ready=503`, foque em dependências.

## 3. Verifique Keycloak

Cheque:

- discovery endpoint;
- JWKS;
- database do Keycloak;
- reconciliação IaC;
- User Storage;
- client usado.

Não rotacione secrets às cegas antes de identificar a camada.

## 4. Token não é emitido

Possíveis causas:

- Auth Service sem PostgreSQL;
- User Storage sem service token válido;
- client do broker inválido;
- credencial humana inválida;
- rate limit;
- identidade ambígua;
- Keycloak indisponível.

## 5. Token é emitido, API responde 401

Valide claims:

- `iss`;
- `aud`;
- `exp`;
- assinatura;
- `azp` quando aplicável.

Compare com configuração do resource server.

## 6. API responde 403

Token pode estar válido, mas faltar:

- role;
- ownership;
- farm/enterprise scope;
- autorização de negócio.

Não transforme 403 em 200 só para “resolver”.

## 7. Muitos 429

O Auth Service aplica rate limit por IP/email.

Cheque se:

- há ataque/brute force;
- proxy está fazendo todos usuários parecerem o mesmo IP;
- Redis está indisponível;
- limites foram configurados incorretamente.

## 8. Rollback

Se incidente começou após alteração de auth:

- restaure config/client anterior quando possível;
- mantenha token format backward-compatible;
- evite reverter migration de identidade que já transformou dados sem análise.

## 9. Evidências a guardar

Sem secrets:

- timestamp;
- request ID;
- status;
- issuer/audience esperados;
- client ID;
- endpoint afetado;
- versão/commit;
- erro sanitizado.

## 10. Depois do incidente

- adicionar teste que reproduz;
- revisar readiness;
- atualizar docs;
- considerar alerta para a causa observada.
