# mobile-template

**Stack:** Android nativo, Kotlin, XML, ViewBinding.

**Repo:** [Ouros-App/mobile-template](https://github.com/Ouros-App/mobile-template)

## Objetivo

Template parametrizado para gerar apps Android com package/name próprios.

## Placeholders

O código usa:

- `{{PROJECT_NAME}}`;
- `{{APP_LABEL}}`;
- `{{PACKAGE_NAME}}`;
- `{{PACKAGE_PATH}}`;
- `{{APPLICATION_CLASS_NAME}}`.

## Config de geração

Exemplo:

```json
{
  "projectName": "mob-github-manager",
  "appLabel": "GitHub Manager",
  "packageName": "com.ouros.githubmanager",
  "applicationClassName": "GithubManagerApplication"
}
```

Campos são extraídos pelo script Bash.

## Processo de geração

```bash
cp template.config.example.json template.config.json
bash scripts/init-template.sh
```

Opcionalmente:

```bash
bash scripts/init-template.sh config.json caminho/saida
```

Sem destino explícito:

```text
out/<projectName>
```

O script:

1. valida presença de config;
2. lê os quatro campos obrigatórios;
3. converte package para path;
4. apaga/recria destino;
5. copia template com `rsync`;
6. substitui placeholders com Perl;
7. move pacote Java/Kotlin;
8. renomeia Application class;
9. falha se sobrar `{{...}}`.

## Dependências do script

Além de Bash:

- `rsync`;
- `perl`.

Isso importa em Windows, onde WSL/Git Bash podem ser necessários.

## Android config

```text
compileSdk 36
targetSdk 36
minSdk 33
Java compatibility 11
ViewBinding true
```

## Estrutura inicial

Inclui:

- `MainActivity`;
- `LoginActivity`;
- `HomeFragment`;
- `MainViewModel`;
- `MainRepository`;
- `ApiClient`;
- `AppTheme`;
- Application class;
- recursos XML;
- teste unitário exemplo;
- teste instrumentado exemplo.

Essas classes são deliberadamente mínimas.

## Local properties

`local.properties.example` existe para indicar o SDK path.

`local.properties` é local e não deve ser commitado.

## Limitações

O template não escolhe por você:

- Retrofit/Ktor/OkHttp;
- serialization;
- Room;
- DataStore;
- auth;
- offline sync;
- DI;
- navigation framework.

Essa neutralidade reduz acoplamento, mas significa que um app gerado não está pronto para produção.

## Consumidor atual

`ouros-android-app` deriva desse template e ainda conserva grande parte desse estado inicial.
