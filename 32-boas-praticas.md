# 32 — Boas Práticas no Desenvolvimento com Spring

Este capítulo reúne boas práticas modernas para desenvolvimento com Spring Boot 3+, Spring Framework 6+ e Java 17/21+, cobrindo arquitetura, código, segurança, testes, desempenho, observabilidade e manutenção. O objetivo é orientar a construção de sistemas robustos, escaláveis, seguros e fáceis de evoluir.

---

## 📌 1. Arquitetura e Organização do Projeto

### ✅ Use Arquitetura em Camadas ou Hexagonal

Estruture o projeto com separação clara de responsabilidades:

- **Camada Web** → Controllers, DTOs, validações.
- **Camada de Aplicação** → Serviços, casos de uso.
- **Camada de Domínio** → Entidades, regras de negócio.
- **Camada de Infraestrutura** → Persistência, mensageria, clientes externos.

Estrutura sugerida:

```text
com.exemplo.app
├── config
├── web
│   ├── controller
│   └── dto
├── application
│   └── service
├── domain
│   └── model
├── infrastructure
│   ├── repository
│   ├── messaging
│   └── client
└── Application.java
✅ Use Spring Modulith para modularidade explícita
Separe módulos por contexto de negócio.

Use @ApplicationModule, @NamedInterface e eventos.

📌 2. Boas Práticas de Código
✅ Prefira Injeção por Construtor
@Service
public class PedidoService {

    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
❌ Evite @Autowired em campos
Dificulta testes.

Quebra imutabilidade.

Reduz clareza de dependências.

✅ Use DTOs para entrada e saída da API
public record CriarUsuarioDTO(
    @NotBlank String nome,
    @Email String email
) {}
Nunca exponha entidades JPA diretamente nos endpoints.

✅ Use record para DTOs e objetos imutáveis (Java 16+)
public record UsuarioDTO(Long id, String nome, String email) {}
✅ Prefira composição à herança
Use interfaces e delegação.

Evite hierarquias profundas.

📌 3. Boas Práticas de Configuração
✅ Use @ConfigurationProperties em vez de @Value
@ConfigurationProperties(prefix = "app.jwt")
public record JwtProperties(String secret, Duration expiration) {}
✅ Separe configurações por perfil
# application.yml
spring:
  profiles:
    active: dev
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://...
❌ Não versionar segredos no repositório
Use variáveis de ambiente.

Use Vault, AWS Secrets Manager, Azure Key Vault, etc.

📌 4. Boas Práticas de Persistência
✅ Use repositórios Spring Data
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {}
✅ Use transações no nível de serviço, não no controller
@Transactional
public void processarPedido(Long id) {}
❌ Evite lógica de negócio dentro de entidades JPA pesadas
Mantenha entidades focadas em estado e regras simples.

Use serviços de domínio para orquestração.

✅ Evite N+1 com @EntityGraph ou fetch join
@EntityGraph(attributePaths = "pedidos")
Optional<Usuario> findById(Long id);
✅ Use paginação e projeções para grandes volumes
Page<UsuarioResumo> findBy(Pageable pageable);
📌 5. Boas Práticas de Transações
✅ Prefira transações declarativas
@Transactional
public void criarPedido(Pedido pedido) {}
❌ Evite @Transactional em métodos privados
O proxy não intercepta chamadas internas.

✅ Use readOnly = true para consultas
@Transactional(readOnly = true)
public List<Usuario> listar() {}
✅ Use @TransactionalEventListener para eventos pós-commit
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPagamento(PagamentoConfirmadoEvent event) {}
📌 6. Boas Práticas de Segurança
✅ Proteja métodos com anotações
@PreAuthorize("hasRole('ADMIN')")
public void excluirUsuario(Long id) {}
✅ Valide entrada sempre
Bean Validation (@Valid).

Sanitização.

Limite de tamanho de payload.

❌ Não confie em dados vindos do cliente
Nunca confie em IDs, roles ou status enviados pelo frontend.

✅ Use HTTPS sempre
TLS obrigatório em produção.

✅ Armazene senhas com hash forte (BCrypt, Argon2)
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
📌 7. Boas Práticas de APIs REST
✅ Siga princípios REST
Use substantivos nos endpoints.

Use HTTP status codes corretamente.

Use métodos HTTP conforme o significado.

GET    /api/usuarios
POST   /api/usuarios
GET    /api/usuarios/{id}
PUT    /api/usuarios/{id}
DELETE /api/usuarios/{id}
✅ Use versionamento de API
/api/v1/usuarios
ou

Accept: application/vnd.exemplo.v1+json
✅ Padronize respostas de erro
{
  "timestamp": "2026-02-11T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "E-mail inválido",
  "path": "/api/usuarios"
}
✅ Documente APIs com OpenAPI/Swagger
Use @Operation, @Schema, @ApiResponse.

Gere documentação automaticamente.

📌 8. Boas Práticas de Testes
✅ Escreva testes automatizados em todos os níveis
Testes unitários → serviços, regras de negócio.

Testes de integração → repositórios, controllers.

Testes end-to-end → fluxos completos.

✅ Use escopos de teste adequados
@WebMvcTest(UsuarioController.class)
class UsuarioControllerTest {}
@DataJpaTest
class UsuarioRepositoryTest {}
@SpringBootTest
class IntegracaoTest {}
✅ Use @MockBean para isolar dependências
❌ Não dependa exclusivamente de testes manuais
Automatização é obrigatória para sistemas profissionais.

📌 9. Boas Práticas de Desempenho
✅ Use cache onde faz sentido
@Cacheable("usuarios")
public Usuario buscar(Long id) {}
✅ Use paginação e limites
Nunca retorne listas gigantes.

✅ Use projeções e consultas otimizadas
✅ Use virtual threads quando apropriado (Java 21+)
@Bean
public Executor taskExecutor() {
    return Executors.newVirtualThreadPerTaskExecutor();
}
❌ Evite bloqueios desnecessários em aplicações reativas
📌 10. Boas Práticas de Observabilidade
✅ Exponha métricas
Micrometer + Prometheus.

Use @Observed, @Timed.

✅ Centralize logs
JSON logs.

Correlation ID.

Trace ID.

✅ Use tracing distribuído
OpenTelemetry.

Jaeger, Zipkin.

❌ Não ignore erros silenciosamente
Sempre registre exceções relevantes.

📌 11. Boas Práticas de Manutenção e Evolução
✅ Versione banco de dados com Flyway ou Liquibase
V1__criar_tabelas.sql
V2__adicionar_coluna_status.sql
✅ Refatore continuamente
Pequenas melhorias frequentes.

Evite grandes reescritas.

✅ Documente decisões arquiteturais (ADR)
adr/0001-escolha-arquitetura.md
✅ Automatize pipelines CI/CD
Build.

Testes.

Análise estática.

Deploy.

📌 12. Boas Práticas de Modularidade e Evolução
✅ Use eventos para desacoplamento
Domínio → aplicação → infraestrutura.

✅ Evite dependências cíclicas
✅ Prefira contratos estáveis entre módulos
Interfaces.

DTOs.

Eventos.

📌 13. Tabela-Resumo de Boas Práticas
Área	Boas Práticas
Arquitetura	Camadas, Hexagonal, Modulith
Código	Injeção por construtor, DTOs, records
Configuração	@ConfigurationProperties, perfis
Persistência	Repositórios, paginação, projeções
Transações	Declarativas, nível de serviço
Segurança	HTTPS, hashing, validação
APIs	REST, versionamento, documentação
Testes	Unitários, integração, E2E
Performance	Cache, virtual threads, otimização
Observabilidade	Métricas, logs, tracing
Evolução	Migrations, ADR, CI/CD
Modularidade	Eventos, contratos, desacoplamento
📌 14. Conclusão
Seguir boas práticas no desenvolvimento com Spring não é apenas uma questão de estilo, mas de sustentabilidade técnica. Sistemas bem arquitetados, testados, seguros e observáveis são mais fáceis de manter, evoluir e escalar.

Este capítulo consolida um guia prático para profissionais que desejam construir aplicações Spring modernas, robustas e alinhadas às exigências do mercado atual.
```
