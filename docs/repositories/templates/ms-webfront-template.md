# ms-webfront-template

**Stack:** React 19.1, TypeScript 5.8, Vite 7, Tailwind CSS 3.4, React Router 7.

**Repo:** [Ouros-App/ms-webfront-template](https://github.com/Ouros-App/ms-webfront-template)

## Objetivo

Scaffold de SPA React com estrutura suficiente para iniciar um produto sem começar de um `App.tsx` vazio.

## Estrutura

```text
src/
├── components/
│   ├── common/
│   ├── complex/
│   └── layout/
├── context/
├── hooks/
├── pages/
├── reducers/
├── routes/
├── services/
├── types/
├── utils/
├── App.tsx
└── main.tsx
```

## Dependências principais

Runtime:

- React;
- React DOM;
- React Router;
- DOMPurify.

Dev:

- TypeScript;
- Vite;
- ESLint;
- Tailwind;
- PostCSS/Autoprefixer.

## Scripts

```bash
npm run dev
npm run build
npm run preview
npm run lint
npm run docker:run
npm run docker:compose
```

`build` executa:

```text
tsc -b && vite build
```

## Configuração

`.env.example`:

```text
APP_NAME=ms-webfront-template
VITE_APP_TITLE=MS Webfront Template
VITE_API_URL=http://localhost:8000/api
VITE_STORAGE_VERSION=v1
PORT=4173
```

Variáveis prefixadas `VITE_` ficam disponíveis no bundle frontend e **não podem conter secrets**.

## Docker

Multi-stage:

1. `node:22-alpine` instala e faz build;
2. `nginx:1.27-alpine` serve `dist/`.

Config Nginx vive em `docker/nginx/default.conf`.

## Arquitetura inicial

O template já traz conceitos de:

- auth context;
- private routes;
- API client;
- services;
- pages;
- hooks reutilizáveis;
- validators/sanitizers;
- componentes básicos.

Isso é scaffold, não implementação de segurança.

## Testes

Não existe script de testes automatizados no `package.json` atual.

Para projeto real, adicionar pelo menos:

- testes unitários de utilitários;
- componentes críticos;
- auth/session;
- integração do API client.

## Segurança frontend

- nunca colocar client secrets em `VITE_*`;
- sanitizar HTML quando renderizar conteúdo não confiável;
- preferir cookies HttpOnly/BFF ou secure platform flow conforme a arquitetura;
- não confiar em `PrivateRoute` como autorização real;
- backend sempre precisa autorizar.

## Consumidor atual

`ms-ouros-front-web` foi criado a partir desta estrutura e ainda mantém vários artefatos do template.
