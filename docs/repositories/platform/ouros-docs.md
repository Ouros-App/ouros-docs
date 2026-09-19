# ouros-docs

**Stack:** MkDocs + Material for MkDocs, Markdown, Mermaid, GitHub Pages.

**Repo:** [Ouros-App/ouros-docs](https://github.com/Ouros-App/ouros-docs)

## Responsabilidade

Portal técnico central da organização.

Regra:

> README explica um repositório. `ouros-docs` explica o sistema.

## Fonte

Conteúdo em:

```text
docs/
```

Navegação em:

```text
mkdocs.yml
```

## Preview local

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
mkdocs serve
```

## Validação

```bash
mkdocs build --strict
```

O mesmo build é executado no CI.

## Publicação

Push na `main` aciona o workflow de GitHub Pages.

As permissões seguem least privilege:

- build: leitura;
- deploy: `pages: write` + `id-token: write`.

O Pages precisa ser habilitado uma vez com source GitHub Actions.

## Política de merge

O repo foi desenhado para baixa fricção:

- PR obrigatória;
- zero approvals humanos obrigatórios;
- CI obrigatório;
- CodeQL mantido como check nativo;
- auto-merge disponível;
- Sonar removido por baixo valor em repo majoritariamente Markdown.

## Convenções

- PT-BR por padrão;
- nomes técnicos podem permanecer em inglês;
- não duplicar Swagger inteiro;
- exemplos devem ser executáveis quando possível;
- nunca colocar secrets;
- mudança de comportamento documentado deve atualizar docs;
- divergências entre README e código devem ser explicitadas.

## Diagramas

Mermaid está habilitado via `pymdownx.superfences`.

## Dependências

`mkdocs-material` fica pinado e Dependabot propõe upgrades.

## Manutenção

Ao adicionar repo ou serviço:

1. criar página em `docs/repositories/`;
2. ligar no catálogo;
3. atualizar arquitetura se necessário;
4. adicionar ao `nav`;
5. rodar build strict.
