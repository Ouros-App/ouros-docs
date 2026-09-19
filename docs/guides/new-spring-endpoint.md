# Guia: adicionar endpoint no Spring API

Fluxo alinhado ao ms-spring-api atual.

## 1. Confira se o recurso já existe

Procure Entity, Repository, Service, DTO, controller e migration semelhantes. Não crie um segundo caminho para a mesma regra.

## 2. Banco primeiro quando houver schema novo

Se a feature exige coluna, tabela, constraint, index, role ou view, altere primeiro o repo PostgreSQL.

O Spring usa:

~~~text
spring.jpa.hibernate.ddl-auto=none
~~~

A Entity não cria schema em produção.

## 3. Separe as camadas

~~~text
Controller
  ↓
Service
  ↓
Repository
  ↓
Entity
  ↓
PostgreSQL
~~~

- Controller: HTTP e validação de entrada.
- Service: regra de negócio e autorização contextual.
- Repository: acesso a dados.
- DTO: contrato público.
- Entity: mapeamento do schema.

## 4. DTO request

Use Jakarta Validation para formato, range, tamanho e datas.

Exemplo:

~~~java
public record ExampleRequestDTO(
    @NotBlank String name,
    @Positive Long idFarm
) {}
~~~

Para campos JSON snake_case, o padrão atual combina JsonProperty com aliases quando precisa preservar compatibilidade.

## 5. DTO response

Não retorne Entity diretamente. Um DTO de resposta impede exposição acidental de password/campos internos e estabiliza o contrato.

## 6. Service e ownership

O Service precisa responder:

- o principal pode ver esse ID?
- a farm pertence ao usuário?
- a enterprise corresponde ao vínculo?
- a mutation está dentro do escopo autorizado?

Não confie no ID enviado pelo cliente como prova de ownership.

## 7. Controller

Convenções observadas:

- POST: 201;
- GET: 200;
- PATCH: 200;
- DELETE: 204.

O principal autenticado normalmente entra via `@AuthenticationPrincipal`, mantendo explícito o tipo usado pela aplicação:

~~~java
public ResponseEntity<?> exemplo(
    @AuthenticationPrincipal UserPrincipal principal
) {
    // ...
}
~~~

## 8. Segurança

Rotas novas ficam autenticadas por padrão no SecurityConfig atual, a menos que sejam adicionadas explicitamente à allowlist pública.

Não torne rota pública apenas para facilitar teste.

## 9. Testes

Mínimo:

- controller/MockMvc;
- service;
- payload inválido;
- not found;
- ownership inválido;
- caso autorizado;
- mutation;
- resposta sem dado sensível.

## 10. Rollout de contrato

Para campo novo obrigatório:

~~~text
backend aceita ausência
  ↓
clientes passam a enviar
  ↓
medir/migrar
  ↓
campo passa a ser obrigatório
~~~

Para rename/removal, mantenha período de compatibilidade.

## Checklist

- [ ] migration quando necessário;
- [ ] Entity;
- [ ] request/response DTOs;
- [ ] Service;
- [ ] Repository;
- [ ] Controller;
- [ ] ownership;
- [ ] testes;
- [ ] OpenAPI;
- [ ] docs;
- [ ] consumidores revisados.
