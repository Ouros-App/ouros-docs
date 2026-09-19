# Desenvolvimento local

Guia rápido para subir e validar os principais tipos de projeto da organização.

## Pré-requisitos gerais

Úteis em praticamente qualquer fluxo:

```text
git
GitHub CLI (opcional)
Docker/Compose
curl
```

Ferramentas adicionais dependem da stack.

## Python / FastAPI

Repos principais:

- `ms-auth-service`;
- `ms-ai-server`;
- `ms-mcp-server-ouros-knowledge`;
- `ms-telemetry-dashboard-service`;
- `ms-github-manager`.

Versão recorrente:

```text
Python 3.12
```

### Ambiente virtual

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Subir FastAPI

Padrão mais comum:

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Alguns repos possuem entrypoint diferente. Confirme o README/`Dockerfile`.

### Testes

```bash
pytest
```

Quando o repo usa `unittest`, `pytest` normalmente ainda consegue descobrir casos, mas siga a CI do projeto como referência.

## Spring Boot

Repos:

- `ms-spring-api`;
- `ms-spring-template`.

### Java

`ms-spring-api` compila com:

```text
Java release 17
```

`ms-spring-template` usa:

```text
Java 21
```

Não assuma a mesma JDK para os dois.

### Rodar

```bash
./gradlew bootRun
```

### Testar

```bash
./gradlew test
```

### Build

```bash
./gradlew clean build
```

No Spring API:

```bash
./gradlew jacocoTestReport
```

### Banco

O Spring API não cria schema:

```text
spring.jpa.hibernate.ddl-auto=none
```

Você precisa apontar para um banco compatível já migrado.

## React / Vite

Repos:

- `ms-ouros-front-web`;
- `ms-webfront-template`.

Runtime de build observado:

```text
Node 22
```

### Instalar

```bash
npm install
```

### Dev server

```bash
npm run dev
```

### Validar

```bash
npm run lint
npm run build
```

!!! warning
    Variáveis `VITE_*` entram no bundle do navegador. Não coloque secrets nelas.

## Android

Repos:

- `ouros-android-app`;
- `mobile-template`.

Config atual:

```text
compileSdk 36
targetSdk 36
minSdk 33
Java compatibility 11
```

### Build debug

Linux/macOS:

```bash
./gradlew assembleDebug
```

Windows:

```powershell
.\gradlew.bat assembleDebug
```

### Teste unitário

```bash
./gradlew test
```

### Device/emulator

Para instalação via ADB:

```bash
adb devices
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

O app atual ainda precisa de implementação real de networking/auth antes de testar integração completa.

## PostgreSQL repos

Repos:

- `postgres-segundo-prod-database`;
- `postgres-segundo-qa-database`;
- `ouros-analytics-database`;
- `postgres-database-template`.

### Instalar dependências

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Aplicar runner clássico

```bash
python scripts/apply_sql.py
```

### QA

Use preferencialmente:

```bash
python scripts/reconcile_sql.py
```

QA tem semântica diferente de prod.

### Antes de apontar para banco real

Confirme:

```bash
env | grep '^POSTGRES_' | sed 's/=.*/=<redacted>/'
```

Isso mostra quais variáveis existem sem imprimir os valores.

## MongoDB repos

Repos:

- `mongodb-ai-prod-database`;
- `mongodb-ai-qa-database`;
- `mongodb-database-template`.

### Aplicar

```bash
python scripts/apply_mongo.py
```

Confirme antes:

- `MONGODB_URI`;
- database alvo;
- TLS/CA;
- ambiente.

## Keycloak

Repo:

`ouros-keycloak`.

### Componentes locais

O fluxo completo envolve:

- provider Java;
- PostgreSQL;
- Keycloak;
- IaC.

A CI é a referência mais fiel para uma integração reproduzível.

### Build do provider

Entre em `providers/ouros-user-storage` e use o wrapper/build Maven configurado pelo projeto quando necessário.

O Dockerfile espera:

```text
providers/ouros-user-storage/target/ouros-user-storage-1.0.0.jar
```

antes de montar a imagem final.

### Docker

O runtime usa:

```text
quay.io/keycloak/keycloak:26.7.3
```

## MkDocs / ouros-docs

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Build de CI:

```bash
mkdocs build --strict
```

## Docker

Quando houver Dockerfile:

```bash
docker build -t ouros-local .
docker run --rm -p 8000:8000 --env-file .env ouros-local
```

A porta real varia por serviço.

Não use esse comando cegamente no Keycloak ou Spring sem conferir o entrypoint.

## Secrets locais

Padrão:

```bash
cp .env.example .env
```

Depois preencha **somente credenciais de desenvolvimento/QA apropriadas**.

Nunca:

- copie secret prod para facilitar;
- comite `.env`;
- mande `.env` em issue;
- tire screenshot com token visível.

## Verificação de integração

Uma ordem segura:

```text
1. unit tests
2. build/lint
3. dependência local
4. health
5. readiness
6. integração autenticada
7. QA
```

## Problemas de versão

Cheque:

```bash
python --version
java -version
node --version
npm --version
docker --version
```

Se CI passa e local falha, compare primeiro versões de runtime antes de mexer no código.

## Debug sem vazar secrets

Bom:

```bash
env | cut -d= -f1 | sort
```

Ruim:

```bash
env
```

O segundo imprime secrets.

## Antes de enviar PR

Por stack:

### Python

```bash
pytest
```

### Java

```bash
./gradlew test
```

### Web

```bash
npm run lint
npm run build
```

### Docs

```bash
mkdocs build --strict
```

### Banco

Rode testes do executor e valide contra banco descartável/QA antes de produção.
