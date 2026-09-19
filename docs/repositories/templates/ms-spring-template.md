# ms-spring-template

**Stack:** Java 21, Spring Boot 3.4.0, Gradle, JaCoCo.

**Repo:** [Ouros-App/ms-spring-template](https://github.com/Ouros-App/ms-spring-template)

## Objetivo

Template simples para novos serviços HTTP Spring Boot.

## API inicial

- `GET /`;
- `GET /health`.

Porta padrão:

```text
8000
```

## Dependências

`build.gradle` mantém o template propositalmente enxuto:

- Spring Boot Web;
- Spring Boot Test;
- JaCoCo;
- plugin SonarQube.

Compilação:

```text
Java release 21
```

## Estrutura

```text
src/
├── main/
│   ├── java/com/ourosapp/springtemplate/
│   │   ├── SpringTemplateApplication.java
│   │   └── controller/
│   │       ├── HomeController.java
│   │       └── HealthController.java
│   └── resources/application.properties
└── test/
    └── java/.../SpringTemplateApplicationTests.java
```

## Execução

```bash
./gradlew bootRun
```

Build/test:

```bash
./gradlew test
./gradlew build
```

## Docker

Build stage:

```text
gradle:8.14.4-jdk21
```

Runtime:

```text
eclipse-temurin:21-jre
```

O container expõe 8000.

O Compose atual faz:

```text
host 8080 → container 8000
```

## Configuração

`.env.example` define `SERVER_PORT=8000`.

## JaCoCo

`test` finaliza gerando relatório JaCoCo. XML e HTML ficam habilitados, útil para CI/Sonar ou outras ferramentas de cobertura.

## Limites do template

Não inclui por padrão:

- JPA;
- banco;
- security;
- OpenAPI;
- Infisical;
- exception handler;
- observabilidade;
- arquitetura de domínio.

Isso é positivo para não obrigar serviços simples a carregar dependências desnecessárias.

## Comparação com ms-spring-api

O `ms-spring-api` já evoluiu muito além deste scaffold:

- Java 17, não 21;
- JPA/Postgres;
- Security/JWT;
- OpenAPI;
- Infisical;
- CRUD de domínio.

Não copie configurações entre ambos assumindo compatibilidade automática.

## Criação de novos serviços

Se o serviço precisar do padrão moderno de identidade Ouros, prefira adicionar validação Keycloak/JWKS em vez de reproduzir o JWT legado do Spring API.
