# ouros-android-app

**Stack atual:** Android nativo, Kotlin, XML, ViewBinding, AndroidX/Material.

**Repo:** [Ouros-App/ouros-android-app](https://github.com/Ouros-App/ouros-android-app)

## Estado real

O repo foi inicializado a partir de `mobile-template` e ainda está muito próximo dele.

Sinais objetivos:

- README ainda descreve “Mobile Template”;
- badges apontam para `mobile-template`;
- `ApiClient` é um objeto vazio;
- `MainRepository` é uma classe vazia;
- `LoginActivity` e `HomeFragment` não possuem implementação funcional;
- `AndroidManifest.xml` não declara `android.permission.INTERNET` no snapshot analisado.

Portanto, o app ainda não possui integração HTTP funcional com os microserviços.

## Configuração Android

```text
namespace     com.ourosapp.ourosandroidapp
applicationId com.ourosapp.ourosandroidapp
compileSdk    36
targetSdk     36
minSdk        33
Java          11
versionCode   1
versionName   1.0
```

`ViewBinding` está habilitado.

## Componentes atuais

### `MainActivity`

Inflaciona `ActivityMainBinding` e define a view raiz.

### `LoginActivity`

Existe como `AppCompatActivity`, ainda sem comportamento implementado.

### `HomeFragment`

Existe como `Fragment`, ainda sem comportamento implementado.

### `MainViewModel`

Scaffold para camada de apresentação.

### `MainRepository`

Placeholder para camada de dados.

### `ApiClient`

Placeholder para cliente HTTP.

## Manifest

A aplicação declara:

- classe `OurosAndroidAppApplication`;
- ícones;
- tema;
- `MainActivity` como launcher/exported.

No estado atual não há declaração de permissão de rede.

## Testes

Existem apenas exemplos de:

- teste unitário;
- teste instrumentado Espresso/AndroidX.

A CI do repo deve ser tratada separadamente do template, pois o README copiado não reflete necessariamente os workflows atuais.

## Relação com autenticação

Quando implementado, o app precisa usar um fluxo compatível com a arquitetura de identidade atual.

Opções arquiteturais existentes no ecossistema:

- Authorization Code + PKCE com client mobile do Keycloak;
- contrato `POST /v1/auth/token` do `ms-auth-service` para clientes first-party quando essa decisão for mantida.

Tokens devem ficar em armazenamento seguro da plataforma, não em preferences comuns ou logs.

## Offline-first

O perfil público do Ouros define offline-first como requisito de produto, mas **o código atual deste repo ainda não implementa a camada de sincronização offline**.

Não confunda requisito de produto com capacidade já entregue.

Uma implementação futura precisa definir:

- armazenamento local;
- fila de mutações;
- estratégia de conflitos;
- ownership dos IDs;
- retry/backoff;
- estado de sync por registro;
- comportamento de autenticação offline;
- observabilidade de sincronização.

## Próximos passos técnicos

1. habilitar/configurar networking;
2. escolher biblioteca HTTP e serialization;
3. implementar autenticação;
4. definir armazenamento seguro;
5. implementar repository real;
6. mapear endpoints de domínio;
7. definir persistência local/offline;
8. adicionar testes de API/repository;
9. atualizar README e branding do template.
