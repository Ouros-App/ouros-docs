# Guia: proteger uma nova API com Keycloak

Exemplo fictício: ms-reports-service.

## 1. Criar resource server no IaC

Crie um arquivo em iac/resources com:

~~~text
CLIENT_TYPE="microservice"
CLIENT_ID="ms-reports-service"
AUDIENCE="ms-reports-service"
SCOPE_NAME="ms-reports-service-audience"
MAPPER_NAME="ms-reports-service-audience"
~~~

## 2. Validar

~~~bash
bash iac/validate.sh
~~~

## 3. Definir consumidores

Clients que precisam chamar a API devem receber a audience correspondente.

## 4. Reconciliar

No deploy/startup, confirme nos logs keycloak-iac:

- client;
- client scope;
- audience mapper.

## 5. Configurar a API

Trust não secreto:

~~~text
ISSUER=https://ouros-keycloak.discloud.app/realms/ouros
AUDIENCE=ms-reports-service
~~~

## 6. Validar token

Cheque:

- assinatura/JWKS;
- issuer;
- expiration;
- audience.

Depois aplique roles e ownership.

Decodificar claims sem validação criptográfica não é autenticação.

## 7. Service-to-service

Para backend sem usuário:

- client type service;
- Client Credentials;
- secret no secret manager;
- audience explícita.

## 8. Web/mobile

Public clients:

- Authorization Code;
- PKCE S256;
- sem client secret no cliente.

## 9. Claims de domínio

O scope ouros-identity transporta contexto como database_id, account_type, farm_id, enterprise_id e first_access.

Claims ajudam no contexto, mas não substituem ownership de negócio quando uma ação é sensível.

## 10. Testes

- token correto;
- expirado;
- issuer errado;
- audience errada;
- role ausente;
- recurso de outro usuário.

## 11. Migração de Bearer legado

~~~text
aceitar JWT em paralelo
  ↓
migrar clientes
  ↓
observar
  ↓
remover Bearer antigo
~~~

## Checklist

- [ ] resource config;
- [ ] audience;
- [ ] consumers;
- [ ] issuer/JWKS;
- [ ] roles;
- [ ] ownership;
- [ ] testes;
- [ ] secrets;
- [ ] docs.
