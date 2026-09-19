# Gestão de secrets

## Onde secrets aparecem

O ecossistema usa mais de um mecanismo porque cada tipo de execução tem necessidades diferentes.

### Infisical

Serviços como `ms-auth-service` e `ouros-autoreview-app` suportam Universal Auth.

Bootstrap típico:

```text
INFISICAL_SITE_URL
INFISICAL_CLIENT_ID
INFISICAL_CLIENT_SECRET
INFISICAL_PROJECT_ID
INFISICAL_ENVIRONMENT
INFISICAL_SECRET_PATH
```

Essas credenciais permitem buscar os secrets reais do serviço.

### GitHub Actions

Usado para:

- conexões de banco em workflows;
- tokens de ferramentas;
- credenciais necessárias à esteira.

### Runtime/Discloud

Configura variáveis de inicialização e bootstrap.

### Local

`.env`, ignorado pelo Git.

## O que pode ficar versionado

Valores não sensíveis:

- host público;
- issuer;
- audience;
- nomes de client;
- porta default;
- nome de database;
- timeout default;
- feature flag não secreta.

## O que não pode

- password;
- token;
- API key;
- refresh token;
- private key;
- connection string contendo credencial;
- client secret;
- webhook secret.

## `.env.example`

Serve como **schema humano** de configuração:

```dotenv
API_KEY=
DATABASE_URL=
APP_PORT=8000
```

Nunca preencha com secret “fake mas parecido com real” que alguém possa reutilizar por engano.

## Frontend

Tudo que começa com `VITE_` pode parar no JavaScript entregue ao navegador.

Portanto é configuração pública.

## Keycloak

Clients mobile/web são públicos e não devem ter secret.

Clients `service` são confidenciais e o secret é gerado/mantido operacionalmente, não versionado.

## Banco

Passwords de roles auxiliares criadas por SQL devem ser injetadas por ambiente/workflow, não codificadas no arquivo SQL.

## Rotação

Ao rotacionar:

1. identifique produtor e consumidores;
2. suporte overlap se necessário;
3. atualize secret manager;
4. redeploy consumidores;
5. valide;
6. revogue antigo;
7. não faça commit com o novo valor.

## Incidente

Se secret entrar no Git:

1. considere comprometido;
2. rotacione imediatamente;
3. remova do código/histórico conforme processo;
4. avalie logs/uso;
5. não confie apenas em apagar a linha no commit seguinte.
