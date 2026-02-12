# 31 — Anotações Modernas no Spring

Este capítulo apresenta as anotações modernas mais relevantes do ecossistema **Spring Boot 3+** e **Spring Framework 6+**, organizadas por categoria, com explicações práticas, exemplos e boas práticas.

O foco é **produtividade, clareza arquitetural e aderência aos padrões atuais**.

---

## 📌 1. Anotações de Configuração e Inicialização

### `@SpringBootApplication`

Combina:

- `@Configuration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

### `@ConfigurationProperties`

Mapeia propriedades externas para objetos tipados.

```java
@ConfigurationProperties(prefix = "app.security")
public record SecurityProperties(boolean enabled, String apiKey) {}
```

```yaml
app:
  security:
    enabled: true
    api-key: abc123
```

Ativar:

```java
@EnableConfigurationProperties(SecurityProperties.class)
```

---

### `@Import`

Importa classes de configuração.

```java
@Configuration
@Import(OutraConfiguracao.class)
public class AppConfig {}
```

---

### `@ConditionalOnProperty`, `@ConditionalOnBean`, `@ConditionalOnMissingBean`

```java
@Bean
@ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
public CacheManager cacheManager() {
    return new ConcurrentMapCacheManager();
}
```

---

## 📌 2. Anotações de Injeção de Dependência

### Estereótipos

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`

```java
@Service
public class UsuarioService {}
```

```java
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {}
```

---

### Injeção por construtor (recomendada)

```java
@Service
public class PedidoService {

    private final ClienteRepository repository;

    public PedidoService(ClienteRepository repository) {
        this.repository = repository;
    }
}
```

> `@Autowired` é opcional em construtores únicos.

---

### `@Qualifier`

```java
@Service
@Qualifier("emailNotificacao")
public class EmailNotificacaoService implements NotificacaoService {}
```

```java
public PedidoService(@Qualifier("emailNotificacao") NotificacaoService service) {}
```

---

### `@Primary`

Define o bean preferencial.

```java
@Primary
@Service
public class DefaultNotificacaoService implements NotificacaoService {}
```

---

## 📌 3. Anotações Web e REST (Spring MVC / WebFlux)

### Mapeamentos HTTP

```java
@GetMapping("/{id}")
public UsuarioDTO buscar(@PathVariable Long id) {}
```

---

### Parâmetros HTTP

- `@RequestBody`
- `@PathVariable`
- `@RequestParam`
- `@RequestHeader`

```java
@PostMapping
public ResponseEntity<UsuarioDTO> criar(@RequestBody @Valid UsuarioDTO dto) {}
```

---

### `@ResponseStatus`

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping
public UsuarioDTO criar(@RequestBody UsuarioDTO dto) {}
```

---

### Tratamento global de exceções

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(EntidadeNaoEncontradaException.class)
    public ResponseEntity<String> handleNotFound(Exception ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## 📌 4. Anotações de Validação (Jakarta Validation)

### `@Valid` / `@Validated`

```java
@PostMapping
public ResponseEntity<Void> salvar(@RequestBody @Valid UsuarioDTO dto) {}
```

---

### Principais anotações

```java
public record UsuarioDTO(
    @NotBlank String nome,
    @Email String email,
    @Size(min = 8) String senha,
    @Min(18) int idade
) {}
```

Outras:

- `@NotNull`
- `@NotEmpty`
- `@Pattern`
- `@Positive`
- `@Past`
- `@Future`

---

### Validação personalizada

```java
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EmailCorporativoValidator.class)
public @interface EmailCorporativo {
    String message() default "E-mail inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

## 📌 5. Anotações de Persistência (JPA / Hibernate)

### Entidades

```java
@Entity
@Table(name = "usuarios")
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nome;
}
```

---

### Relacionamentos

```java
@OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL)
private List<Pedido> pedidos;

@ManyToOne
@JoinColumn(name = "usuario_id")
private Usuario usuario;
```

---

### Auditoria

```java
@EntityListeners(AuditingEntityListener.class)
public class EntidadeBase {

    @CreatedDate
    private LocalDateTime criadoEm;

    @LastModifiedDate
    private LocalDateTime atualizadoEm;
}
```

Ativar:

```java
@EnableJpaAuditing
```

---

## 📌 6. Anotações de Transações

### `@Transactional`

```java
@Transactional
public void processarPedido(Long id) {}
```

---

### Configuração avançada

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.READ_COMMITTED,
    rollbackFor = Exception.class
)
public void executar() {}
```

---

### `@TransactionalEventListener`

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPagamento(PagamentoConfirmadoEvent event) {}
```

---

## 📌 7. Anotações de Cache

### `@EnableCaching`

```java
@EnableCaching
@Configuration
public class CacheConfig {}
```

---

### Operações de cache

```java
@Cacheable("usuarios")
public Usuario buscar(Long id) {}
```

```java
@CacheEvict(value = "usuarios", key = "#id")
public void remover(Long id) {}
```

---

## 📌 8. Agendamento e Execução Assíncrona

### `@EnableScheduling` e `@Scheduled`

```java
@EnableScheduling
@Configuration
public class SchedulingConfig {}
```

```java
@Scheduled(fixedRate = 60000)
public void executarTarefa() {}
```

---

### `@EnableAsync` e `@Async`

```java
@EnableAsync
@Configuration
public class AsyncConfig {}
```

```java
@Async
public CompletableFuture<Void> enviarEmail() {}
```

---

## 📌 9. Segurança (Spring Security 6+)

### `@EnableMethodSecurity`

```java
@EnableMethodSecurity
@Configuration
public class SecurityConfig {}
```

---

### Autorização declarativa

```java
@PreAuthorize("hasRole('ADMIN')")
public void excluirUsuario(Long id) {}
```

```java
@PostAuthorize("returnObject.usuario == authentication.name")
public Pedido buscarPedido(Long id) {}
```

---

### `@Secured` (legado)

```java
@Secured("ROLE_ADMIN")
public void atualizarSistema() {}
```

---

## 📌 10. Eventos

### Publicação

```java
applicationEventPublisher.publishEvent(new UsuarioCriadoEvent(this, usuario));
```

---

### `@EventListener`

```java
@EventListener
public void aoCriarUsuario(UsuarioCriadoEvent event) {}
```

---

### Listener assíncrono

```java
@Async
@EventListener
public void processarEvento(Evento event) {}
```

---

## 📌 11. Observabilidade

### `@Observed` (Micrometer)

```java
@Observed(name = "pedido.processar", contextualName = "processarPedido")
public void processarPedido() {}
```

---

### `@Timed`

```java
@Timed(value = "pedido.criar", description = "Tempo para criar pedidos")
public void criarPedido() {}
```

---

## 📌 12. HTTP Clients Declarativos

### `@HttpExchange` (Spring 6+)

```java
@HttpExchange("/api/pagamentos")
public interface PagamentoClient {

    @GetExchange("/{id}")
    Pagamento buscar(@PathVariable Long id);

    @PostExchange
    void criar(@RequestBody Pagamento pagamento);
}
```

Registro:

```java
@Bean
public PagamentoClient pagamentoClient(WebClient.Builder builder) {
    return HttpServiceProxyFactory
        .builder(WebClientAdapter.forClient(builder.baseUrl("http://pagamentos").build()))
        .build()
        .createClient(PagamentoClient.class);
}
```

---

## 📌 13. Anotações Reativas (WebFlux)

```java
@GetMapping("/usuarios")
public Flux<Usuario> listar() {}
```

```java
@GetMapping("/usuarios/{id}")
public Mono<Usuario> buscar(@PathVariable Long id) {}
```

---

## 📌 14. AOP (Aspect-Oriented Programming)

```java
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.exemplo.service..*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Executando: " + joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```

```java
@Pointcut("within(com.exemplo.service..*)")
public void serviceLayer() {}
```

---

## 📌 15. Testes

### `@SpringBootTest`

```java
@SpringBootTest
class UsuarioServiceTest {}
```

---

### `@WebMvcTest`

```java
@WebMvcTest(UsuarioController.class)
class UsuarioControllerTest {}
```

---

### `@DataJpaTest`

```java
@DataJpaTest
class UsuarioRepositoryTest {}
```

---

### `@MockBean`

```java
@MockBean
private UsuarioService usuarioService;
```

---

### `@TestConfiguration`

```java
@TestConfiguration
public class TestConfig {

    @Bean
    public Clock clock() {
        return Clock.fixed(Instant.now(), ZoneId.systemDefault());
    }
}
```

---

## 📌 16. Configuração Condicional e Modular

### `@Profile`

```java
@Profile("dev")
@Bean
public DataSource dataSourceDev() {}
```

---

### `@ConditionalOnExpression`

```java
@Bean
@ConditionalOnExpression("'${app.mode}'=='producao'")
public ServicoProducao servico() {}
```

---

### `@ImportResource`

```java
@ImportResource("classpath:legacy-context.xml")
```

---

## 📌 17. Documentação (OpenAPI)

```java
@Operation(summary = "Cria um usuário", description = "Endpoint para criação de usuários")
@ApiResponse(responseCode = "201", description = "Usuário criado com sucesso")
@PostMapping
public ResponseEntity<UsuarioDTO> criar(@RequestBody @Valid UsuarioDTO dto) {}
```

```java
@Schema(description = "Dados do usuário")
public record UsuarioDTO(
    @Schema(example = "João Silva") String nome,
    @Schema(example = "joao@email.com") String email
) {}
```

---

## 📌 18. Virtual Threads (Java 21+)

```java
@Bean
public Executor taskExecutor() {
    return Executors.newVirtualThreadPerTaskExecutor();
}
```

Uso:

```java
@Async
public void processar() {}
```

---

## 📌 19. Spring Modulith

```java
@ApplicationModule
package com.exemplo.pedidos;
```

```java
@NamedInterface("api")
package com.exemplo.pedidos.api;
```

```java
@Externalized
public class PedidoCriadoEvent {}
```

---

## 📌 20. Boas Práticas

✅ Prefira injeção por construtor.  
✅ Use `@ConfigurationProperties` em vez de `@Value`.  
✅ Combine validação com DTOs.  
✅ Documente APIs com OpenAPI.  
✅ Evite métodos transacionais excessivamente amplos.

❌ Não exponha entidades JPA diretamente na API.  
❌ Não misture responsabilidades em uma única classe.

---

## 📌 21. Tabela-Resumo

| Categoria       | Principais Anotações                                                  |
| --------------- | --------------------------------------------------------------------- |
| Inicialização   | `@SpringBootApplication`, `@ConfigurationProperties`, `@Import`       |
| Injeção         | `@Component`, `@Service`, `@Repository`, `@Qualifier`, `@Primary`     |
| Web             | `@RestController`, `@GetMapping`, `@RequestBody`, `@ControllerAdvice` |
| Validação       | `@Valid`, `@NotNull`, `@Email`, `@Size`                               |
| Persistência    | `@Entity`, `@Id`, `@OneToMany`, `@CreatedDate`                        |
| Transações      | `@Transactional`, `@TransactionalEventListener`                       |
| Cache           | `@EnableCaching`, `@Cacheable`, `@CacheEvict`                         |
| Segurança       | `@EnableMethodSecurity`, `@PreAuthorize`                              |
| Observabilidade | `@Observed`, `@Timed`                                                 |
| Testes          | `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`                      |

---

## 📌 22. Conclusão

As anotações modernas do Spring constituem a base do desenvolvimento produtivo, seguro, modular e observável.

Quando combinadas com boas práticas arquiteturais, permitem construir aplicações robustas, escaláveis e alinhadas aos padrões atuais do ecossistema Java com **Spring Boot 3+, Java 17/21+ e arquitetura moderna**.

---

<p align="center">
<b>Finalizada a Anotações Modernas no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="32-boas-praticas.md">Boas Práticas no Desenvolvimento com Spring</a>
</p>
