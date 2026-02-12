# 📦 Spring GraphQL

O **Spring GraphQL** fornece suporte oficial ao GraphQL no ecossistema Spring, permitindo a construção de APIs flexíveis, fortemente tipadas e orientadas a dados, como alternativa ou complemento às APIs REST tradicionais.

Este capítulo aborda desde os fundamentos do GraphQL até sua integração profunda com Spring Boot, Spring Data, segurança, validação, observabilidade e arquitetura moderna.

---

## 🧠 O que é GraphQL?

GraphQL é uma linguagem de consulta para APIs e um runtime para executar essas consultas. Diferente do REST, onde cada recurso possui um endpoint, no GraphQL existe normalmente **um único endpoint** que permite ao cliente:

- Definir exatamente quais campos deseja.
- Navegar por relacionamentos.
- Evitar over-fetching e under-fetching.
- Compor consultas complexas em uma única requisição.

---

## 🧩 Arquitetura do Spring GraphQL

Principais componentes:

- **Schema GraphQL (`.graphqls`)** — Define tipos, queries, mutations e subscriptions.
- **Data Fetchers / Resolvers** — Métodos Java que resolvem campos do schema.
- **GraphQlService** — Executa consultas.
- **WebGraphQlHandler** — Integra com a camada web.
- **Instrumentation** — Observabilidade e métricas.
- **Execution Graph** — Representação interna da consulta.

Spring GraphQL integra-se nativamente com:

- Spring Boot
- Spring MVC / WebFlux
- Spring Security
- Spring Data
- Spring Validation
- Micrometer
- OpenTelemetry

---

## 📦 Dependências

### 🔹 Maven

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

### 🔹 Opcional para UI (GraphiQL / Playground via Web)

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

## 📜 Definição do Schema

Exemplo em `src/main/resources/graphql/schema.graphqls`:

```graphql
type Usuario {
  id: ID!
  nome: String!
  email: String!
}

type Query {
  usuario(id: ID!): Usuario
  usuarios: [Usuario!]!
}

type Mutation {
  criarUsuario(nome: String!, email: String!): Usuario
}
```

---

## 🔧 Resolvers com `@Controller`

```java
@Controller
public class UsuarioGraphQLController {

    private final UsuarioService service;

    public UsuarioGraphQLController(UsuarioService service) {
        this.service = service;
    }

    @QueryMapping
    public Usuario usuario(@Argument Long id) {
        return service.buscarPorId(id);
    }

    @QueryMapping
    public List<Usuario> usuarios() {
        return service.listarTodos();
    }

    @MutationMapping
    public Usuario criarUsuario(@Argument String nome, @Argument String email) {
        return service.criar(nome, email);
    }
}
```

---

## 🔁 Mapeamento de Campos Relacionados

```java
@SchemaMapping(typeName = "Usuario", field = "enderecos")
public List<Endereco> enderecos(Usuario usuario) {
    return enderecoService.buscarPorUsuario(usuario.getId());
}
```

Esse método será chamado **apenas se o campo `enderecos` for solicitado pelo cliente**.

---

## 🧠 Data Fetching e Performance

### 🔸 N+1 Problem

Assim como no JPA, GraphQL pode sofrer do problema **N+1** ao resolver relacionamentos.

### ✅ Solução: DataLoader

```java
@Bean
public DataLoaderRegistry dataLoaderRegistry() {
    DataLoaderRegistry registry = new DataLoaderRegistry();

    registry.register("enderecosLoader",
        DataLoader.newMappedDataLoader(keys ->
            CompletableFuture.supplyAsync(() ->
                enderecoService.buscarPorUsuarios(keys)
            )
        )
    );

    return registry;
}
```

```java
@SchemaMapping(typeName = "Usuario", field = "enderecos")
public CompletableFuture<List<Endereco>> enderecos(
        Usuario usuario,
        DataLoader<Long, List<Endereco>> loader) {

    return loader.load(usuario.getId());
}
```

---

## 🔐 Segurança com Spring Security

### 🔸 Autorização por método

```java
@PreAuthorize("hasRole('ADMIN')")
@MutationMapping
public Usuario excluirUsuario(@Argument Long id) {
    return service.excluir(id);
}
```

### 🔸 Autenticação com JWT

Spring GraphQL integra-se automaticamente com o filtro de segurança configurado para REST.

### 🔸 Segurança por campo

```java
@SchemaMapping(typeName = "Usuario", field = "email")
@PreAuthorize("hasRole('ADMIN')")
public String email(Usuario usuario) {
    return usuario.getEmail();
}
```

---

## 🛡️ Validação de Entrada

```java
public record CriarUsuarioInput(
    @NotBlank String nome,
    @Email String email
) {}
```

```java
@MutationMapping
public Usuario criarUsuario(@Argument @Valid CriarUsuarioInput input) {
    return service.criar(input.nome(), input.email());
}
```

---

## 🚨 Tratamento de Erros

### 🔸 Exemplo com `DataFetcherExceptionResolver`

```java
@Component
public class GraphQLExceptionHandler implements DataFetcherExceptionResolver {

    @Override
    public Mono<DataFetcherExceptionResolverResult> resolveException(
            Throwable ex,
            DataFetchingEnvironment env) {

        GraphQLError error = GraphqlErrorBuilder.newError(env)
            .message(ex.getMessage())
            .errorType(ErrorType.BAD_REQUEST)
            .build();

        return Mono.just(
            DataFetcherExceptionResolverResult
                .newResult()
                .error(error)
                .build()
        );
    }
}
```

---

## 🧪 Testes com Spring GraphQL

### 🔸 Teste de Query

```java
@GraphQlTest(UsuarioGraphQLController.class)
class UsuarioGraphQLControllerTest {

    @Autowired
    private GraphQlTester graphQlTester;

    @Test
    void deveBuscarUsuario() {
        graphQlTester.document("""
            query {
              usuario(id: 1) {
                id
                nome
                email
              }
            }
        """)
        .execute()
        .path("usuario.nome")
        .entity(String.class)
        .isEqualTo("Albert");
    }
}
```

---

## 📊 Observabilidade

Spring GraphQL integra-se com:

- **Micrometer** — Métricas por operação.
- **OpenTelemetry** — Tracing distribuído.
- **Spring Actuator** — Health checks.

### 🔧 Exemplo de configuração

```yaml
management:
  metrics:
    enable:
      graphql: true
```

---

## 🔄 Subscriptions (WebSocket)

### 🔸 Definição no schema

```graphql
type Subscription {
  usuarioCriado: Usuario
}
```

### 🔸 Implementação

```java
@SubscriptionMapping
public Flux<Usuario> usuarioCriado() {
    return usuarioService.streamUsuariosCriados();
}
```

---

## 🌐 GraphQL vs REST

| Critério       | REST      | GraphQL              |
| -------------- | --------- | -------------------- |
| Endpoints      | Múltiplos | Normalmente único    |
| Over-fetching  | Comum     | Evitado              |
| Under-fetching | Comum     | Evitado              |
| Versionamento  | Explícito | Implícito por schema |
| Tipagem        | Fraca     | Forte (schema)       |
| Cache HTTP     | Simples   | Mais complexo        |
| Complexidade   | Menor     | Maior                |

---

## 🧠 Quando usar GraphQL?

### ✅ Recomendado quando:

- Há múltiplos clientes com necessidades diferentes.
- O frontend exige flexibilidade.
- O domínio possui muitos relacionamentos.
- Deseja-se evitar múltiplas chamadas REST.

### ❌ Evitar quando:

- A API é simples.
- Cache HTTP é essencial.
- Há limitação de recursos ou baixa maturidade da equipe.

---

## 🧱 Boas Práticas

- Use DTOs/Input Types, não entidades diretamente.
- Controle complexidade de queries (depth, cost analysis).
- Use DataLoader para evitar N+1.
- Documente o schema.
- Versione por evolução do schema (depreciação).
- Não exponha lógica sensível no schema.
- Monitore queries lentas.
- Limite tamanhos de payload.

---

## 🧩 Evolução de Schema

```graphql
type Usuario {
  id: ID!
  nome: String!
  email: String! @deprecated(reason: "Use contato.email")
  contato: Contato
}
```

---

## 🧱 Conclusão do Capítulo

O Spring GraphQL oferece uma abordagem moderna, flexível e fortemente tipada para construção de APIs orientadas a dados. Seu uso eficaz exige domínio de schema design, resolvers, segurança, performance e observabilidade.

Em arquiteturas modernas, GraphQL não substitui necessariamente REST, mas complementa-o, oferecendo maior expressividade, eficiência de rede e controle do cliente sobre os dados retornados.

Com as práticas e padrões corretos, o Spring GraphQL pode ser uma poderosa ferramenta no arsenal de desenvolvimento de APIs. Explore, experimente e adapte conforme as necessidades do seu projeto!

---

<p align="center">
<b>Finalizada a Spring GraphQL! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="09-persistencia-spring-data.md">Persistência e Exploração de Dados com Spring Data</a>
</p>
