# Catálogo de repositórios

A organização `Ouros-App` possui **26 repositórios** no snapshot de 19/09/2026.

Esta seção documenta o papel técnico de cada um. A documentação central descreve o sistema; o README local continua útil para comandos específicos do próprio repo.

## Serviços centrais

| Repositório | Papel |
| --- | --- |
| `ms-spring-api` | API transacional Spring Boot do domínio Ouros. |
| `ms-auth-service` | Verificação de credenciais e bridge de identidade para Keycloak. |
| `ouros-keycloak` | IdP OIDC, clients-as-code e User Storage federado. |
| `ms-ai-server` | Orquestração LangGraph do Midas. |
| `ms-mcp-server-ouros-knowledge` | MCP de conhecimento, contexto MIDAS e importação controlada. |
| `ms-mcp-server-ouros-knowledge-codemode` | Fork experimental reduzido para testes de Code Mode. |
| `ms-telemetry-dashboard-service` | API de dashboards Databricks e renderização de gráficos. |

## Dados

| Repositório | Papel |
| --- | --- |
| `postgres-segundo-prod-database` | Schema e migrations do banco transacional de produção. |
| `postgres-segundo-qa-database` | QA reconciliado a partir do contrato de produção. |
| `ouros-analytics-database` | Banco PostgreSQL analítico, atualmente em bootstrap. |
| `mongodb-ai-prod-database` | Estrutura MongoDB da IA em produção. |
| `mongodb-ai-qa-database` | Estrutura MongoDB da IA em QA. |

## Clientes e aplicações

| Repositório | Papel |
| --- | --- |
| `ms-ouros-front-web` | Frontend React atual, ainda próximo do template. |
| `ouros-android-app` | App Android Kotlin/XML atual, ainda próximo do template. |
| `ouros-crud-primeiro` | Repositório do CRUD do primeiro ano; `main` expõe apenas README no snapshot atual. |

## Automação e plataforma

| Repositório | Papel |
| --- | --- |
| `ms-github-manager` | Criação padronizada de repositórios e scaffolds. |
| `ouros-autoreview-app` | GitHub App acionado por `/auto-review`. |
| `ms-discord-bot` | Controle/monitoramento de homelab via Discord e Tuya. |
| `.github` | Perfil público e metadados da organização. |
| `ouros-docs` | Este portal. |

## Templates

| Repositório | Papel |
| --- | --- |
| `ms-fastapi-template` | Base FastAPI. |
| `ms-spring-template` | Base Spring Boot. |
| `ms-webfront-template` | Base React/Vite/Tailwind. |
| `mobile-template` | Base Android Kotlin/XML parametrizada. |
| `postgres-database-template` | Executor/versionador PostgreSQL. |
| `mongodb-database-template` | Executor/versionador MongoDB. |

!!! note
    O catálogo registra **estado atual**, não maturidade planejada. Repositórios com nomes “prod”, “app” ou “analytics” podem ainda ter partes em implantação.
