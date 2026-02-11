# 🧪 Testes no Ecossistema Spring

Testes são parte essencial da arquitetura de aplicações modernas. No ecossistema Spring, testar não é apenas verificar resultados, mas garantir **contratos**, **comportamento**, **integração**, **resiliência** e **segurança** ao longo de todo o ciclo de vida da aplicação.

Este capítulo aborda estratégias, ferramentas e práticas para testes **unitários**, de **camada**, de **integração**, de **contrato**, **reativos** e **end-to-end (E2E)**.

---

# 🧠 Filosofia de Testes no Spring

O Spring promove testes:

- **Isolados**, por meio de mocks e test doubles
- **Integrados**, com contextos reais e infraestrutura efêmera
- **Declarativos**, utilizando anotações e configuração mínima
- **Reprodutíveis**, com ambientes controlados

O objetivo não é apenas "passar nos testes", mas **confiar no sistema em produção**.

---

# 🧩 Tipos de Testes

## 🔹 Testes Unitários

Validam uma única classe ou método de forma isolada.

- Não carregam contexto Spring
- Uso frequente de **Mockito**
- Rápidos e determinísticos

```java
@ExtendWith(MockitoExtension.class)
class CalculadoraServiceTest {

    @InjectMocks
    private CalculadoraService service;

    @Mock
    private Repositorio repositorio;

    @Test
    void deveSomarCorretamente() {
        assertEquals(4, service.somar(2, 2));
    }
}
```

---

## 🔹 Testes de Camada (Layered Tests)

Validam uma camada específica com **contexto Spring parcial**.

Exemplos:

- Controllers
- Repositórios
- Serialização JSON
- Clientes HTTP

---

## 🔹 Testes de Integração

Validam a integração entre múltiplos componentes.

- Contexto Spring completo
- Banco real (geralmente via **Testcontainers**)
- Comunicação real entre camadas

```java
@SpringBootTest
class PedidoIntegrationTest {

    @Test
    void fluxoCompletoDePedido() {
        // cenário completo
    }
}
```

---

## 🔹 Testes End-to-End (E2E)

Validam o sistema como um todo, da entrada à saída.

- Simulam comportamento real do usuário
- Incluem infraestrutura externa
- Podem envolver autenticação, filas, banco, cache etc.

---

## 🔹 Testes de Contrato

Validam acordos entre produtores e consumidores de APIs.

- Evitam quebras de compatibilidade
- Fundamentais em microsserviços
- Automatizam validações entre serviços

---

# 🧪 Test Slices

Test Slices carregam apenas partes específicas do contexto Spring, tornando os testes **mais rápidos e focados**.

---

## 🔹 `@WebMvcTest`

Testa apenas controllers e camada web.

```java
@WebMvcTest(PedidoController.class)
class PedidoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PedidoService service;

    @Test
    void deveRetornarPedido() throws Exception {
        mockMvc.perform(get("/pedidos/1"))
            .andExpect(status().isOk());
    }
}
```

---

## 🔹 `@DataJpaTest`

Testa repositórios JPA com banco em memória por padrão.

```java
@DataJpaTest
class PedidoRepositoryTest {

    @Autowired
    private PedidoRepository repository;

    @Test
    void deveSalvarPedido() {
        Pedido pedido = new Pedido();
        repository.save(pedido);
        assertNotNull(pedido.getId());
    }
}
```

---

## 🔹 `@JsonTest`

Testa serialização e desserialização JSON.

```java
@JsonTest
class PedidoJsonTest {

    @Autowired
    private JacksonTester<Pedido> json;

    @Test
    void deveSerializarPedido() throws Exception {
        Pedido pedido = new Pedido("123");
        assertThat(json.write(pedido))
            .hasJsonPathStringValue("$.codigo");
    }
}
```

---

## 🔹 `@RestClientTest`

Testa clientes HTTP de forma isolada.

```java
@RestClientTest(PedidoClient.class)
class PedidoClientTest {

    @Autowired
    private PedidoClient client;

    @Autowired
    private MockRestServiceServer server;

    @Test
    void deveBuscarPedido() {
        server.expect(requestTo("/pedidos/1"))
            .andRespond(withSuccess("{\"id\":1}", MediaType.APPLICATION_JSON));

        client.buscarPedido(1L);
    }
}
```

---

# 📜 Spring Cloud Contract

Permite definir contratos que garantem compatibilidade entre serviços.

## 🔹 Exemplo de Contrato (Groovy DSL)

```groovy
Contract.make {
    request {
        method GET()
        url '/pedidos/1'
    }
    response {
        status OK()
        body(
            id: 1,
            status: "CRIADO"
        )
        headers {
            contentType(applicationJson())
        }
    }
}
```

## 🔹 Benefícios

- O produtor gera testes automaticamente
- O consumidor valida compatibilidade
- Reduz falhas em produção por mudanças inesperadas

---

# 🐳 Testcontainers

Permite subir bancos, filas e serviços reais via Docker durante os testes.

## 🔹 Exemplo com PostgreSQL

```java
@Testcontainers
@SpringBootTest
class PedidoIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void overrideProps(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void devePersistirPedido() {
        // teste real com banco real
    }
}
```

---

# ☁️ Testes Cloud-Native

## 🔹 LocalStack (AWS)

Simula serviços AWS localmente (S3, SQS, SNS, DynamoDB etc.).

```yaml
spring:
  cloud:
    aws:
      endpoint: http://localhost:4566
      region: us-east-1
```

```java
@SpringBootTest
class AwsIntegrationTest {
}
```

---

# ⚡ Testes Reativos

## 🔹 WebFlux Controller

```java
@WebFluxTest(PedidoController.class)
class PedidoControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void deveRetornarPedido() {
        webTestClient.get().uri("/pedidos/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.id").isEqualTo(1);
    }
}
```

---

## 🔹 Testando `Flux` e `Mono`

```java
@Test
void deveEmitirValores() {
    Flux<Integer> flux = Flux.just(1, 2, 3);

    StepVerifier.create(flux)
        .expectNext(1, 2, 3)
        .verifyComplete();
}
```

---

# ⏳ Testes Assíncronos

```java
@Test
void deveExecutarAssincrono() throws Exception {
    CompletableFuture<String> future = service.processarAsync();
    assertEquals("OK", future.get(5, TimeUnit.SECONDS));
}
```

---

# 🧪 Testes End-to-End (E2E)

## 🔹 Com Spring Boot + MockMvc

```java
@SpringBootTest
@AutoConfigureMockMvc
class PedidoE2ETest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void fluxoCompleto() throws Exception {

        mockMvc.perform(post("/pedidos"))
            .andExpect(status().isCreated());

        mockMvc.perform(get("/pedidos"))
            .andExpect(status().isOk());
    }
}
```

---

## 🔹 Com Infraestrutura Real

Pode incluir:

- Banco de dados
- Mensageria
- Autenticação
- Cache
- API Gateway

---

# 🔐 Testes de Segurança

```java
@WithMockUser(username = "admin", roles = {"ADMIN"})
@Test
void devePermitirAcessoAdmin() {
}
```

```java
@Test
void deveNegarAcessoSemToken() throws Exception {
    mockMvc.perform(get("/admin"))
        .andExpect(status().isUnauthorized());
}
```

---

# ⚠️ Armadilhas Comuns

- Testes lentos por carregar contexto completo desnecessariamente
- Dependência de dados externos
- Falta de isolamento entre testes
- Uso excessivo de mocks em testes de integração
- Ignorar testes de contrato em ambientes distribuídos

---

# 🧱 Boas Práticas

- Prefira **Test Slices** para testes rápidos
- Use **Testcontainers** para ambientes realistas
- Separe testes unitários, integração e E2E
- Automatize no pipeline CI/CD
- Trate testes como código de produção
- Priorize qualidade sobre cobertura numérica
- Teste falhas, exceções e cenários negativos

---

# 🧠 Conclusão

O Spring fornece uma das infraestruturas de teste mais completas do ecossistema Java, permitindo validar desde classes isoladas até sistemas distribuídos inteiros.

Testar no Spring não é apenas validar funcionalidades — é garantir **confiabilidade**, **evolução segura**, **resiliência** e **qualidade arquitetural**.

Sem testes, não há arquitetura sustentável.
