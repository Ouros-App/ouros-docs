# ms-fastapi-template

**Stack:** Python 3.12, FastAPI 0.115.6, Uvicorn 0.34.

**Repo:** [Ouros-App/ms-fastapi-template](https://github.com/Ouros-App/ms-fastapi-template)

## Objetivo

Template mínimo para criar novos microserviços FastAPI com uma separação de pastas já definida.

## Estrutura

```text
app/
├── api/
│   └── routes.py
├── core/
│   └── config.py
├── models/
├── repositories/
├── schemas/
│   └── common.py
├── services/
└── main.py
tests/
```

A intenção é começar simples e permitir crescimento por camadas sem exigir refatoração estrutural imediata.

## API inicial

O template expõe apenas:

| Método | Rota | Resposta |
| --- | --- | --- |
| GET | `/` | disponibilidade |
| GET | `/health` | `{"status":"ok"}` |

FastAPI também fornece:

- `/docs`;
- `/redoc`;
- `/openapi.json`.

## Dependências

Fixadas em `requirements.txt`:

```text
fastapi==0.115.6
uvicorn[standard]==0.34.0
```

## Execução local

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

## Scripts Docker

O template possui `run.sh` e `run_compose.sh` para administrar instâncias numeradas.

Variáveis do `.env.example`:

- `APP_PORT=8000`;
- `APP_NAME=fastapi_microservice`.

## Problema conhecido do Dockerfile

!!! danger "Dockerfile inconsistente com a árvore atual"
    O Dockerfile faz `COPY .env .`, mas `.env` não deve ser versionado e pode não existir durante o build.

Além disso, o Dockerfile executa testes de existência para:

```text
app/templates/workflows/fastapi.yml
app/templates/workflows/frontend.yml
app/templates/workflows/springboot.yml
app/templates/workflows/generic.yml
```

Esses arquivos **não existem na árvore atual do template**.

Consequência: o build Docker, como está, não deve ser considerado funcional por padrão.

## Como novos serviços deveriam evoluir

Ao gerar um serviço real:

1. remover comportamento/arquivos que pertencem só ao template;
2. definir Settings por ambiente;
3. implementar schemas;
4. service/repository quando necessário;
5. autenticação;
6. observabilidade;
7. testes;
8. Docker build reproduzível;
9. README específico do domínio.

## Testes

O diretório `tests/` contém apenas `__init__.py` no estado atual. Não há casos reais.

## Relação com ms-github-manager

O manager reconhece templates FastAPI e pode usá-los como base de criação. Mudanças estruturais neste template precisam considerar o fluxo automatizado de geração.
