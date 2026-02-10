# 🗄️ Persistência e Exploração de Dados com Spring Data

O **Spring Data** fornece um modelo unificado e consistente para acesso a dados em aplicações Spring, abstraindo detalhes de implementação de diferentes tecnologias de persistência, como bancos relacionais, NoSQL, armazenamento em memória e repositórios reativos.

Este capítulo explora os fundamentos do Spring Data, seus módulos, mecanismos internos, padrões de projeto e boas práticas para construir camadas de persistência robustas, performáticas e evolutivas.

---

## 🧠 Visão Geral do Spring Data

Spring Data não é um framework único, mas uma **família de projetos**, incluindo:

- Spring Data JPA
- Spring Data JDBC
- Spring Data MongoDB
- Spring Data Redis
- Spring Data Cassandra
- Spring Data Neo4j
- Spring Data R2DBC (reativo)
- Spring Data Elasticsearch

### 🎯 Objetivos principais

- Reduzir boilerplate.
- Padronizar o acesso a dados.
- Suportar múltiplos paradigmas (imperativo e reativo).
- Integrar com o ecossistema Spring.

---

## 🧩 Arquitetura Interna

### 🔸 Repositórios

Spring Data gera implementações automaticamente a partir de interfaces:

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
}
```

### 🔸 Infraestrutura Interna

Componentes principais:

- `RepositoryFactoryBeanSupport`
- `RepositoryFactory`
- `QueryLookupStrategy`
- `EntityInformation`
- `MappingContext`

Esses componentes permitem que o Spring Data:

- Inspecione entidades.
- Gere consultas automaticamente.
- Resolva métodos dinamicamente.
- Aplique proxies AOP para transações, validações e eventos.

---

# 🔹 JPA e Bancos Relacionais

## 🔧 Interfaces Principais

- `CrudRepository`
- `PagingAndSortingRepository`
- `JpaRepository`
- `JpaSpecificationExecutor`

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long>,
                                           JpaSpecificationExecutor<Pedido> {
}
```

---

## 🔍 Derivação de Queries por Nome

O Spring Data interpreta o nome do método e gera a query automaticamente:

```java
List<Usuario> findByNomeContainingIgnoreCase(String nome);
Optional<Usuario> findByEmail(String email);
List<Pedido> findByStatusAndDataAfter(Status status, LocalDate data);
```

---

## 🔧 Queries Customizadas com `@Query`

```java
@Query("SELECT u FROM Usuario u WHERE u.email = :email")
Optional<Usuario> buscarPorEmail(@Param("email") String email);
```

### 🔸 Queries Nativas

```java
@Query(value = "SELECT * FROM usuarios WHERE ativo = true", nativeQuery = true)
List<Usuario> buscarAtivos();
```

---

# 🔹 NoSQL: MongoDB, Redis, Cassandra, Neo4j

## 📦 MongoDB

```java
@Document(collection = "usuarios")
public class Usuario {

    @Id
    private String id;

    private String nome;
}
```

```java
public interface UsuarioRepository extends MongoRepository<Usuario, String> {
    List<Usuario> findByNome(String nome);
}
```

---

## ⚡ Redis

Usado principalmente como cache ou armazenamento em memória.

```java
@RedisHash("usuario")
public class Usuario {

    @Id
    private String id;

    private String nome;
}
```

```java
public interface UsuarioRepository extends CrudRepository<Usuario, String> {
}
```

---

## 🌐 Cassandra

```java
@Table("usuarios")
public class Usuario {

    @PrimaryKey
    private UUID id;

    private String nome;
}
```

```java
public interface UsuarioRepository extends CassandraRepository<Usuario, UUID> {
}
```

---

## 🔗 Neo4j (Grafos)

```java
@Node("Usuario")
public class Usuario {

    @Id
    @GeneratedValue
    private Long id;

    private String nome;
}
```

```java
public interface UsuarioRepository extends Neo4jRepository<Usuario, Long> {
}
```

---

# 🔹 Repositórios Reativos

Spring Data oferece suporte reativo com **Project Reactor**.

## 🔧 Exemplo com R2DBC

```java
@Table("usuarios")
public class Usuario {

    @Id
    private Long id;

    private String nome;
}
```

```java
public interface UsuarioRepository extends ReactiveCrudRepository<Usuario, Long> {
    Flux<Usuario> findByNome(String nome);
}
```

### ⚠️ Observações Importantes

- Não misture programação imperativa e reativa.
- Não use JPA com WebFlux.
- Use R2DBC para bancos relacionais em aplicações reativas.

---

# 🔹 Auditing, Soft Delete e Versionamento

## 🧾 Auditing Automático

```java
@EntityListeners(AuditingEntityListener.class)
public class Usuario {

    @CreatedDate
    private LocalDateTime criadoEm;

    @LastModifiedDate
    private LocalDateTime atualizadoEm;

    @CreatedBy
    private String criadoPor;

    @LastModifiedBy
    private String atualizadoPor;
}
```

```java
@EnableJpaAuditing
@Configuration
public class AuditoriaConfig {
}
```

---

## 🗑️ Soft Delete

```java
@SQLDelete(sql = "UPDATE usuarios SET ativo = false WHERE id = ?")
@Where(clause = "ativo = true")
@Entity
public class Usuario {

    @Id
    private Long id;

    private boolean ativo = true;
}
```

---

## 🔁 Versionamento Otimista

```java
@Version
private Long versao;
```

Previne sobrescrita concorrente silenciosa.

---

# 🔹 Multi-Tenancy (SaaS)

## 🧠 Modelos Comuns

- Database per tenant
- Schema per tenant
- Shared schema com coluna `tenant_id`

---

## 🔧 Exemplo (Tenant por Coluna)

```java
@Entity
public class Pedido {

    @Id
    private Long id;

    private String tenantId;
}
```

```java
@FilterDef(
    name = "tenantFilter",
    parameters = @ParamDef(name = "tenantId", type = String.class)
)
@Filter(
    name = "tenantFilter",
    condition = "tenant_id = :tenantId"
)
```

```java
session.enableFilter("tenantFilter")
       .setParameter("tenantId", tenantAtual);
```

### ⚠️ Complexidade

Multi-tenancy exige:

- Isolamento rigoroso de dados.
- Segurança reforçada.
- Observabilidade por tenant.
- Estratégias de migração e versionamento.

---

# 🧪 Testes com Spring Data

## 🔸 Testes de Repositório (JPA)

```java
@DataJpaTest
class UsuarioRepositoryTest {

    @Autowired
    private UsuarioRepository repository;

    @Test
    void deveSalvarUsuario() {
        Usuario usuario = new Usuario("Albert", "albert@email.com");
        Usuario salvo = repository.save(usuario);
        assertNotNull(salvo.getId());
    }
}
```

---

## 🔸 Testes Reativos (R2DBC)

```java
@DataR2dbcTest
class UsuarioReactiveRepositoryTest {

    @Autowired
    private UsuarioRepository repository;

    @Test
    void deveBuscarUsuarios() {
        StepVerifier.create(repository.findAll())
            .expectNextCount(3)
            .verifyComplete();
    }
}
```

---

# 🛡️ Boas Práticas

- Separe entidades de DTOs.
- Centralize lógica de negócio em serviços.
- Evite `fetch = EAGER`.
- Use paginação sempre que possível.
- Evite consultas nativas quando possível.
- Monitore consultas lentas.
- Utilize índices adequados.
- Evite lógica complexa em repositórios.
- Documente contratos de persistência.
- Teste queries críticas.

---

# 🧱 Conclusão do Capítulo

O Spring Data oferece uma plataforma poderosa e unificada para acesso a dados, permitindo que desenvolvedores foquem no domínio em vez de infraestrutura. Entretanto, seu uso eficaz exige compreensão de seus mecanismos internos, limitações e implicações arquiteturais.

Dominar Spring Data é fundamental para construir aplicações escaláveis, resilientes e orientadas a dados em ambientes modernos, distribuídos e cloud-native.
