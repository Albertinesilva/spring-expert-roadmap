# 31 — Anotações Modernas no Spring

Este capítulo apresenta as anotações modernas mais importantes do ecossistema Spring (Spring Boot 3+, Spring Framework 6+), organizadas por categoria, com explicações práticas, exemplos e boas práticas. O foco é produtividade, clareza arquitetural e aderência aos padrões atuais.

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

@ConfigurationProperties
Mapeia propriedades externas para objetos tipados.

@ConfigurationProperties(prefix = "app.security")
public record SecurityProperties(boolean enabled, String apiKey) {}

app:
  security:
    enabled: true
    api-key: abc123
Ativar:

@EnableConfigurationProperties(SecurityProperties.class)
@Import
Importa classes de configuração.

@Configuration
@Import(OutraConfiguracao.class)
public class AppConfig {}

@ConditionalOnProperty, @ConditionalOnBean, @ConditionalOnMissingBean
@Bean
@ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
public CacheManager cacheManager() {
    return new ConcurrentMapCacheManager();
}

📌 2. Anotações de Injeção de Dependência
@Component, @Service, @Repository, @Controller, @RestController

@Service
public class UsuarioService {}

@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {}

@Autowired (opcional em construtores modernos)
@Service
public class PedidoService {

    private final ClienteRepository repository;

    public PedidoService(ClienteRepository repository) {
        this.repository = repository;
    }
}

@Qualifier
Quando há múltiplas implementações.

@Service
@Qualifier("emailNotificacao")
public class EmailNotificacaoService implements NotificacaoService {}

@Autowired
public PedidoService(@Qualifier("emailNotificacao") NotificacaoService service) {}

@Primary
Define bean preferencial.

@Primary
@Service
public class DefaultNotificacaoService implements NotificacaoService {}

📌 3. Anotações Web e REST (Spring MVC / WebFlux) @RequestMapping, @GetMapping, @PostMapping, etc.

@GetMapping("/{id}")
public UsuarioDTO buscar(@PathVariable Long id) {}

@RequestBody, @PathVariable, @RequestParam, @RequestHeader
@PostMapping
public ResponseEntity<UsuarioDTO> criar(@RequestBody @Valid UsuarioDTO dto) {}

@ResponseStatus
@ResponseStatus(HttpStatus.CREATED)
@PostMapping
public UsuarioDTO criar(@RequestBody UsuarioDTO dto) {}

@ControllerAdvice e @ExceptionHandler
@ControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(EntidadeNaoEncontradaException.class)
    public ResponseEntity<String> handleNotFound(Exception ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}

@RestControllerAdvice
Combina @ControllerAdvice + @ResponseBody.

📌 4. Anotações de Validação (Bean Validation / Jakarta Validation)

@Valid / @Validated
@PostMapping
public ResponseEntity<Void> salvar(@RequestBody @Valid UsuarioDTO dto) {}
Principais anotações
public record UsuarioDTO(
    @NotBlank String nome,
    @Email String email,
    @Size(min = 8) String senha,
    @Min(18) int idade
) {}

Outras:

@NotNull

@NotEmpty

@Pattern

@Positive, @PositiveOrZero

@Past, @Future

Validação personalizada
@Target({ FIELD })
@Retention(RUNTIME)
@Constraint(validatedBy = EmailCorporativoValidator.class)
public @interface EmailCorporativo {
    String message() default "E-mail inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

📌 5. Anotações de Persistência (JPA / Hibernate) Entidades

@Entity
@Table(name = "usuarios")
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nome;
}

Relacionamentos
@OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL)
private List<Pedido> pedidos;

@ManyToOne
@JoinColumn(name = "usuario_id")
private Usuario usuario;

Auditoria
@EntityListeners(AuditingEntityListener.class)
public class EntidadeBase {

    @CreatedDate
    private LocalDateTime criadoEm;

    @LastModifiedDate
    private LocalDateTime atualizadoEm;
}
Ativar:

@EnableJpaAuditing

📌 6. Anotações de Transações

@Transactional
@Transactional
public void processarPedido(Long id) {}

Configurações avançadas
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.READ_COMMITTED,
    rollbackFor = Exception.class
)
public void executar() {}

@TransactionalEventListener
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPagamento(PagamentoConfirmadoEvent event) {}

📌 7. Anotações de Cache @EnableCaching

@EnableCaching
@Configuration
public class CacheConfig {}

@Cacheable, @CachePut, @CacheEvict
@Cacheable("usuarios")
public Usuario buscar(Long id) {}

@CacheEvict(value = "usuarios", key = "#id")
public void remover(Long id) {}

📌 8. Anotações de Agendamento e Execução Assíncrona

@EnableScheduling, @Scheduled
@EnableScheduling
@Configuration
public class SchedulingConfig {}

@Scheduled(fixedRate = 60000)
public void executarTarefa() {}

@EnableAsync, @Async
@EnableAsync
@Configuration
public class AsyncConfig {}

@Async
public CompletableFuture<Void> enviarEmail() {}
📌 9. Anotações de Segurança (Spring Security 6+) @EnableMethodSecurity

@EnableMethodSecurity
@Configuration
public class SecurityConfig {}

@PreAuthorize, @PostAuthorize
@PreAuthorize("hasRole('ADMIN')")
public void excluirUsuario(Long id) {}

@PostAuthorize("returnObject.usuario == authentication.name")
public Pedido buscarPedido(Long id) {}

@Secured (legado, mas ainda suportado)
@Secured("ROLE_ADMIN")
public void atualizarSistema() {}

📌 10. Anotações de Eventos

Publicação de eventos
applicationEventPublisher.publishEvent(new UsuarioCriadoEvent(this, usuario));

@EventListener
@EventListener
public void aoCriarUsuario(UsuarioCriadoEvent event) {}

@Async em listeners
@Async
@EventListener
public void processarEvento(Evento event) {}

📌 11. Anotações de Observabilidade

@Observed (Micrometer / Spring Boot 3+)
@Observed(name = "pedido.processar", contextualName = "processarPedido")
public void processarPedido() {}

@Timed
@Timed(value = "pedido.criar", description = "Tempo para criar pedidos")
public void criarPedido() {}

📌 12. Anotações para HTTP Clients

@HttpExchange (Spring 6+)
@HttpExchange("/api/pagamentos")
public interface PagamentoClient {

    @GetExchange("/{id}")
    Pagamento buscar(@PathVariable Long id);

    @PostExchange
    void criar(@RequestBody Pagamento pagamento);
}

Registrar:

@Bean
public PagamentoClient pagamentoClient(WebClient.Builder builder) {
    return HttpServiceProxyFactory
        .builder(WebClientAdapter.forClient(builder.baseUrl("http://pagamentos").build()))
        .build()
        .createClient(PagamentoClient.class);
}

📌 13. Anotações Reativas (WebFlux)

@RestController com Reactor
@GetMapping("/usuarios")
public Flux<Usuario> listar() {}

@GetMapping("/usuarios/{id}")
public Mono<Usuario> buscar(@PathVariable Long id) {}

@Controller + @ResponseBody também é válido.
📌 14. Anotações de AOP (Aspect-Oriented Programming)

@Aspect, @Around, @Before, @AfterReturning, @AfterThrowing
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.exemplo.service..*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Executando: " + joinPoint.getSignature());
        return joinPoint.proceed();
    }
}

@Pointcut
@Pointcut("within(com.exemplo.service..*)")
public void serviceLayer() {}

📌 15. Anotações de Testes

@SpringBootTest
@SpringBootTest
class UsuarioServiceTest {}

@WebMvcTest
@WebMvcTest(UsuarioController.class)
class UsuarioControllerTest {}

@DataJpaTest
@DataJpaTest
class UsuarioRepositoryTest {}

@MockBean
@MockBean
private UsuarioService usuarioService;

@TestConfiguration
@TestConfiguration
public class TestConfig {

    @Bean
    public Clock clock() {
        return Clock.fixed(Instant.now(), ZoneId.systemDefault());
    }
}

📌 16. Anotações de Configuração Condicional e Modular

@Profile
@Profile("dev")
@Bean
public DataSource dataSourceDev() {}

@ConditionalOnExpression
@Bean
@ConditionalOnExpression("'${app.mode}'=='producao'")

public ServicoProducao servico() {}
@ImportResource
@ImportResource("classpath:legacy-context.xml")

📌 17. Anotações de Documentação (OpenAPI / Swagger)

@Operation, @ApiResponse, @Schema
@Operation(summary = "Cria um usuário", description = "Endpoint para criação de usuários")
@ApiResponse(responseCode = "201", description = "Usuário criado com sucesso")

@PostMapping
public ResponseEntity<UsuarioDTO> criar(@RequestBody @Valid UsuarioDTO dto) {}

@Schema(description = "Dados do usuário")
public record UsuarioDTO(
    @Schema(example = "João Silva") String nome,
    @Schema(example = "joao@email.com") String email
) {}


📌 18. Anotações de Virtual Threads (Java 21+ com Spring Boot 3.2+)

@EnableAsync com virtual threads
@Bean
public Executor taskExecutor() {
    return Executors.newVirtualThreadPerTaskExecutor();
}
Usado com:

@Async
public void processar() {}
📌 19. Anotações de Spring Modulith
@ApplicationModule
@ApplicationModule
package com.exemplo.pedidos;

import org.springframework.modulith.ApplicationModule;
@NamedInterface
@NamedInterface("api")
package com.exemplo.pedidos.api;

import org.springframework.modulith.NamedInterface;
@Externalized
@Externalized
public class PedidoCriadoEvent {}

📌 20. Boas Práticas no Uso de Anotações

✅ Prefira injeção por construtor.

✅ Evite lógica pesada em classes anotadas com @Controller.

✅ Use @ConfigurationProperties em vez de @Value.

✅ Combine validação com DTOs, não diretamente em entidades.

✅ Evite abusar de @Transactional em métodos muito amplos.

✅ Documente endpoints com OpenAPI.

❌ Evite misturar responsabilidades (ex: @Service que também é @Controller).

❌ Não exponha entidades JPA diretamente em APIs.

📌 21. Tabela-Resumo de Anotações Modernas

Categoria	Anotações
Inicialização	@SpringBootApplication, @ConfigurationProperties, @Import
Injeção	@Component, @Service, @Repository, @Autowired, @Qualifier, @Primary
Web	@RestController, @RequestMapping, @GetMapping, @RequestBody, @ControllerAdvice
Validação	@Valid, @NotNull, @Email, @Size, @Pattern
Persistência	@Entity, @Id, @OneToMany, @ManyToOne, @CreatedDate
Transações	@Transactional, @TransactionalEventListener
Cache	@EnableCaching, @Cacheable, @CacheEvict
Agendamento	@EnableScheduling, @Scheduled
Assíncrono	@EnableAsync, @Async
Segurança	@EnableMethodSecurity, @PreAuthorize, @PostAuthorize
Eventos	@EventListener, @Async
Observabilidade	@Observed, @Timed
HTTP Clients	@HttpExchange, @GetExchange, @PostExchange
AOP	@Aspect, @Around, @Before, @AfterReturning
Testes	@SpringBootTest, @WebMvcTest, @DataJpaTest, @MockBean
Modularidade	@ApplicationModule, @NamedInterface, @Externalized
Documentação	@Operation, @ApiResponse, @Schema
Virtual Threads	@EnableAsync, Executors.newVirtualThreadPerTaskExecutor()

📌 22. Conclusão
As anotações modernas do Spring formam a espinha dorsal do desenvolvimento produtivo, seguro, modular e observável. Combinadas com boas práticas arquiteturais, elas permitem criar aplicações robustas, escaláveis e alinhadas aos padrões atuais do ecossistema Java.

Este capítulo serve como referência prática e técnica para uso profissional em projetos reais com Spring Boot 3+, Java 17/21+ e arquitetura moderna.

```
