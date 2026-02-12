# 32 — Boas Práticas no Desenvolvimento com Spring

Este capítulo reúne boas práticas modernas para desenvolvimento com **Spring Boot 3+**, **Spring Framework 6+** e **Java 17/21+**, cobrindo arquitetura, código, segurança, testes, desempenho, observabilidade e manutenção.  
O objetivo é orientar a construção de sistemas **robustos, escaláveis, seguros e evolutivos**.

---

## 📌 Sumário

1. [Arquitetura e Organização do Projeto](#1-arquitetura-e-organização-do-projeto)
2. [Boas Práticas de Código](#2-boas-práticas-de-código)
3. [Boas Práticas de Configuração](#3-boas-práticas-de-configuração)
4. [Boas Práticas de Persistência](#4-boas-práticas-de-persistência)
5. [Boas Práticas de Transações](#5-boas-práticas-de-transações)
6. [Boas Práticas de Segurança](#6-boas-práticas-de-segurança)
7. [Boas Práticas de APIs REST](#7-boas-práticas-de-apis-rest)
8. [Boas Práticas de Testes](#8-boas-práticas-de-testes)
9. [Boas Práticas de Desempenho](#9-boas-práticas-de-desempenho)
10. [Boas Práticas de Observabilidade](#10-boas-práticas-de-observabilidade)
11. [Boas Práticas de Manutenção e Evolução](#11-boas-práticas-de-manutenção-e-evolução)
12. [Boas Práticas de Modularidade e Evolução](#12-boas-práticas-de-modularidade-e-evolução)
13. [Tabela-Resumo](#13-tabela-resumo-de-boas-práticas)
14. [Conclusão](#14-conclusão)

---

# 1. Arquitetura e Organização do Projeto

## ✅ Use Arquitetura em Camadas ou Hexagonal

Estruture o projeto com separação clara de responsabilidades:

- **Camada Web** → Controllers, DTOs, validações.
- **Camada de Aplicação** → Serviços e casos de uso.
- **Camada de Domínio** → Entidades e regras de negócio.
- **Camada de Infraestrutura** → Persistência, mensageria e integrações externas.

### Estrutura sugerida

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
```

## ✅ Use Spring Modulith para modularidade explícita

- Separe módulos por contexto de negócio.
- Utilize `@ApplicationModule`, `@NamedInterface` e eventos para reforçar fronteiras.
- Evite dependências cíclicas entre módulos.

---

# 2. Boas Práticas de Código

## ✅ Prefira Injeção por Construtor

```java
@Service
public class PedidoService {

    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
```

### ❌ Evite `@Autowired` em campos

- Dificulta testes.
- Quebra imutabilidade.
- Reduz clareza de dependências.

---

## ✅ Use DTOs para entrada e saída da API

```java
public record CriarUsuarioDTO(
    @NotBlank String nome,
    @Email String email
) {}
```

**Nunca exponha entidades JPA diretamente nos endpoints.**

---

## ✅ Use `record` para objetos imutáveis (Java 16+)

```java
public record UsuarioDTO(Long id, String nome, String email) {}
```

---

## ✅ Prefira composição à herança

- Utilize interfaces e delegação.
- Evite hierarquias profundas e acoplamento excessivo.

---

# 3. Boas Práticas de Configuração

## ✅ Use `@ConfigurationProperties` em vez de `@Value`

```java
@ConfigurationProperties(prefix = "app.jwt")
public record JwtProperties(String secret, Duration expiration) {}
```

---

## ✅ Separe configurações por perfil

```yaml
# application.yml
spring:
  profiles:
    active: dev
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://...
```

---

## ❌ Não versionar segredos no repositório

- Use variáveis de ambiente.
- Utilize soluções como Vault, AWS Secrets Manager ou Azure Key Vault.

---

# 4. Boas Práticas de Persistência

## ✅ Use repositórios Spring Data

```java
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {}
```

---

## ✅ Use transações no nível de serviço

```java
@Transactional
public void processarPedido(Long id) {}
```

Evite `@Transactional` em controllers.

---

## ❌ Evite lógica de negócio pesada em entidades JPA

- Entidades devem representar estado e regras simples.
- Use serviços de domínio para orquestração.

---

## ✅ Evite N+1 queries

```java
@EntityGraph(attributePaths = "pedidos")
Optional<Usuario> findById(Long id);
```

Ou utilize `fetch join`.

---

## ✅ Use paginação e projeções

```java
Page<UsuarioResumo> findBy(Pageable pageable);
```

Nunca retorne listas massivas sem controle.

---

# 5. Boas Práticas de Transações

## ✅ Prefira transações declarativas

```java
@Transactional
public void criarPedido(Pedido pedido) {}
```

---

## ❌ Evite `@Transactional` em métodos privados

O proxy do Spring não intercepta chamadas internas.

---

## ✅ Use `readOnly = true` para consultas

```java
@Transactional(readOnly = true)
public List<Usuario> listar() {}
```

---

## ✅ Use `@TransactionalEventListener` para eventos pós-commit

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPagamento(PagamentoConfirmadoEvent event) {}
```

---

# 6. Boas Práticas de Segurança

## ✅ Proteja métodos com anotações

```java
@PreAuthorize("hasRole('ADMIN')")
public void excluirUsuario(Long id) {}
```

---

## ✅ Valide entradas sempre

- Bean Validation (`@Valid`)
- Sanitização de dados
- Limitação de tamanho de payload

---

## ❌ Nunca confie em dados do cliente

- Não confie em IDs ou roles enviados pelo frontend.
- Sempre valide no backend.

---

## ✅ Use HTTPS obrigatoriamente

TLS deve ser padrão em produção.

---

## ✅ Armazene senhas com hash forte

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

# 7. Boas Práticas de APIs REST

## ✅ Siga princípios REST

Use substantivos e HTTP status adequados:

```
GET    /api/usuarios
POST   /api/usuarios
GET    /api/usuarios/{id}
PUT    /api/usuarios/{id}
DELETE /api/usuarios/{id}
```

---

## ✅ Use versionamento de API

```
/api/v1/usuarios
```

Ou via header:

```
Accept: application/vnd.exemplo.v1+json
```

---

## ✅ Padronize respostas de erro

```json
{
  "timestamp": "2026-02-11T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "E-mail inválido",
  "path": "/api/usuarios"
}
```

---

## ✅ Documente APIs com OpenAPI/Swagger

- Utilize `@Operation`, `@Schema`, `@ApiResponse`.
- Gere documentação automática.

---

# 8. Boas Práticas de Testes

## ✅ Testes em todos os níveis

- Unitários → serviços e regras de negócio.
- Integração → repositórios e controllers.
- End-to-end → fluxos completos.

---

## ✅ Use escopos adequados

```java
@WebMvcTest(UsuarioController.class)
class UsuarioControllerTest {}
```

```java
@DataJpaTest
class UsuarioRepositoryTest {}
```

```java
@SpringBootTest
class IntegracaoTest {}
```

---

## ✅ Use `@MockBean` para isolar dependências

---

## ❌ Não dependa exclusivamente de testes manuais

Automação é obrigatória para sistemas profissionais.

---

# 9. Boas Práticas de Desempenho

## ✅ Use cache quando apropriado

```java
@Cacheable("usuarios")
public Usuario buscar(Long id) {}
```

---

## ✅ Use paginação e limites

Evite retornar grandes volumes sem controle.

---

## ✅ Use projeções e consultas otimizadas

Reduza carga de dados desnecessários.

---

## ✅ Use Virtual Threads (Java 21+) quando aplicável

```java
@Bean
public Executor taskExecutor() {
    return Executors.newVirtualThreadPerTaskExecutor();
}
```

---

## ❌ Evite bloqueios desnecessários em aplicações reativas

---

# 10. Boas Práticas de Observabilidade

## ✅ Exponha métricas

- Micrometer + Prometheus.
- Use `@Observed`, `@Timed`.

---

## ✅ Centralize logs

- Logs estruturados (JSON).
- Correlation ID.
- Trace ID.

---

## ✅ Use tracing distribuído

- OpenTelemetry.
- Jaeger ou Zipkin.

---

## ❌ Não ignore exceções silenciosamente

Sempre registre erros relevantes.

---

# 11. Boas Práticas de Manutenção e Evolução

## ✅ Versione banco de dados

Exemplo com Flyway:

```
V1__criar_tabelas.sql
V2__adicionar_coluna_status.sql
```

---

## ✅ Refatore continuamente

- Pequenas melhorias frequentes.
- Evite grandes reescritas disruptivas.

---

## ✅ Documente decisões arquiteturais (ADR)

```
adr/0001-escolha-arquitetura.md
```

---

## ✅ Automatize CI/CD

- Build
- Testes
- Análise estática
- Deploy automatizado

---

# 12. Boas Práticas de Modularidade e Evolução

## ✅ Use eventos para desacoplamento

Fluxo recomendado:

Domínio → Aplicação → Infraestrutura

---

## ✅ Evite dependências cíclicas

---

## ✅ Prefira contratos estáveis entre módulos

- Interfaces
- DTOs
- Eventos

---

# 13. Tabela-Resumo de Boas Práticas

| Área            | Boas Práticas                         |
| --------------- | ------------------------------------- |
| Arquitetura     | Camadas, Hexagonal, Modulith          |
| Código          | Injeção por construtor, DTOs, records |
| Configuração    | `@ConfigurationProperties`, perfis    |
| Persistência    | Repositórios, paginação, projeções    |
| Transações      | Declarativas, nível de serviço        |
| Segurança       | HTTPS, hashing, validação             |
| APIs            | REST, versionamento, documentação     |
| Testes          | Unitários, integração, E2E            |
| Performance     | Cache, virtual threads, otimização    |
| Observabilidade | Métricas, logs, tracing               |
| Evolução        | Migrations, ADR, CI/CD                |
| Modularidade    | Eventos, contratos, desacoplamento    |

---

# 14. Conclusão

Seguir boas práticas no desenvolvimento com Spring não é apenas uma questão de estilo, mas de **sustentabilidade técnica**.

Sistemas bem arquitetados, testados, seguros e observáveis são:

- Mais fáceis de manter.
- Mais simples de evoluir.
- Mais preparados para escalar.
- Mais alinhados às exigências do mercado.

Este capítulo consolida um guia prático para profissionais que desejam construir aplicações Spring modernas, robustas e preparadas para o longo prazo.

---

<p align="center">
<b>Finalizada a Boas Práticas no Desenvolvimento com Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="33-referencias.md">Referências Oficiais e Recursos Recomendados</a>
</p>
