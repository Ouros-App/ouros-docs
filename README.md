# Ouros Docs

Portal central de documentação técnica da plataforma Ouros.

A documentação é escrita em Markdown, publicada com [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) e validada automaticamente pelo CI.

## Escrevendo documentação

1. Crie ou edite arquivos em `docs/`.
2. Se adicionar uma nova página, inclua-a no `nav` de `mkdocs.yml`.
3. Rode o preview local:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
mkdocs serve
```

Acesse `http://127.0.0.1:8000`.

Antes de mergear, o CI executa:

```bash
mkdocs build --strict
```

## Estrutura

```text
docs/
├── index.md
├── getting-started/
├── architecture/
├── repositories/
├── apis/
├── databases/
├── security/
├── operations/
└── development/
```

As páginas em `repositories/` documentam os repositórios da organização; `operations/` concentra ambientes, CI/CD, secrets e troubleshooting.

Tudo que entra na `main` é publicado automaticamente no GitHub Pages.

## Bootstrap do GitHub Pages

O Pages precisa ser habilitado apenas uma vez no repositório, usando **GitHub Actions** como fonte de publicação. Depois disso, o workflow `Deploy Docs` cuida das publicações automaticamente.

Com GitHub CLI e permissão de administração:

```bash
gh api --method POST repos/Ouros-App/ouros-docs/pages -f build_type=workflow
```
