# 26 — Virtual Threads (Java 21+) no Ecossistema Spring

As **Virtual Threads**, introduzidas oficialmente no Java 21 (Projeto Loom), revolucionam o modelo de concorrência ao permitir milhões de threads leves, com custo quase desprezível, mantendo o modelo de programação síncrono tradicional.

No ecossistema Spring, elas possibilitam construir aplicações altamente concorrentes, escaláveis e simples, sem a necessidade obrigatória de adotar programação reativa.

---

## 📌 Sumário

- [1. O que são Virtual Threads](#1-o-que-são-virtual-threads)
- [2. Diferença entre Platform Threads e Virtual Threads](#2-diferença-entre-platform-threads-e-virtual-threads)
- [3. Por que Virtual Threads mudam tudo](#3-por-que-virtual-threads-mudam-tudo)
- [4. Virtual Threads no Spring Boot](#4-virtual-threads-no-spring-boot)
- [5. Configuração no Spring Boot](#5-configuração-no-spring-boot)
- [6. Integração com Spring MVC](#6-integração-com-spring-mvc)
- [7. Integração com JDBC e JPA](#7-integração-com-jdbc-e-jpa)
- [8. Virtual Threads vs WebFlux](#8-virtual-threads-vs-webflux)
- [9. Limitações e Armadilhas](#9-limitações-e-armadilhas)
- [10. Casos de Uso Ideais](#10-casos-de-uso-ideais)
- [11. Boas Práticas](#11-boas-práticas)
- [12. Conclusão](#12-conclusão)

---

## 1. O que são Virtual Threads

Virtual Threads são threads gerenciadas pela **JVM**, e não diretamente pelo sistema operacional. Elas:

- Consomem poucos KB de memória.
- São criadas e destruídas rapidamente.
- São desacopladas das _platform threads_ (threads do SO).
- Permitem escalabilidade massiva utilizando código síncrono tradicional.

Elas são especialmente eficientes para aplicações **IO-bound** (operações de rede, banco de dados, APIs externas).

---

## 2. Diferença entre Platform Threads e Virtual Threads

| Característica        | Platform Threads       | Virtual Threads               |
| --------------------- | ---------------------- | ----------------------------- |
| Gerenciamento         | Sistema Operacional    | JVM                           |
| Custo de memória      | Alto (~1MB por thread) | Muito baixo (~KBs)            |
| Quantidade viável     | Centenas / milhares    | Milhões                       |
| Bloqueio              | Bloqueia thread do SO  | Suspensão controlada pela JVM |
| Modelo de programação | Síncrono tradicional   | Síncrono tradicional          |

---

## 3. Por que Virtual Threads mudam tudo

Antes do Projeto Loom:

- Alta escala → exigia programação reativa ou assíncrona.
- Código simples → limitado em escalabilidade.

Com Virtual Threads:

- Escala **e** simplicidade coexistem.
- Código imperativo síncrono torna-se altamente escalável.
- Reduz-se significativamente a complexidade arquitetural.

---

## 4. Virtual Threads no Spring Boot

O **Spring Boot 3.2+** oferece suporte direto a Virtual Threads:

- Tomcat, Jetty e Undertow suportam virtual threads.
- `@Async`, `TaskExecutor`, `Spring MVC`, JDBC e JPA podem operar sobre virtual threads.
- Não é necessário alterar o modelo de programação da aplicação.

---

## 5. Configuração no Spring Boot

### ✔ Ativando Virtual Threads

Em `application.properties`:

```properties
spring.threads.virtual.enabled=true
```

Ou em YAML:

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

Ao habilitar essa configuração, o servidor web e executores internos passam a utilizar virtual threads automaticamente.

---

## 6. Integração com Spring MVC

```java
@RestController
@RequestMapping("/api/pedidos")
public class PedidoController {

    @GetMapping("/{id}")
    public Pedido buscar(@PathVariable Long id) {
        // Código bloqueante tradicional
        return pedidoService.buscarPorId(id);
    }
}
```

Mesmo com chamadas bloqueantes (JPA, REST, IO), a aplicação pode escalar para milhares de requisições simultâneas graças às virtual threads.

---

## 7. Integração com JDBC e JPA

Virtual threads são ideais para workloads **IO-bound**:

```java
public Pedido buscar(Long id) {
    return pedidoRepository.findById(id)
            .orElseThrow();
}
```

O bloqueio ocorre apenas na virtual thread, sem impactar diretamente as threads do sistema operacional.

⚠️ Atenção: o pool de conexões (ex.: HikariCP) ainda precisa ser corretamente dimensionado.

---

## 8. Virtual Threads vs WebFlux

| Aspecto              | Virtual Threads        | WebFlux (Reativo)        |
| -------------------- | ---------------------- | ------------------------ |
| Modelo               | Imperativo síncrono    | Funcional reativo        |
| Curva de aprendizado | Baixa                  | Alta                     |
| Interoperabilidade   | Total com libs legadas | Pode exigir adaptações   |
| Uso de recursos      | Muito eficiente        | Muito eficiente          |
| Melhor para          | APIs REST, CRUD, IO    | Streaming e backpressure |

➡️ Em muitos cenários, Virtual Threads reduzem a necessidade de adoção do WebFlux.

---

## 9. Limitações e Armadilhas

❌ Código CPU-bound intenso não se beneficia significativamente  
❌ Uso excessivo de `ThreadLocal` exige cautela  
❌ Bloqueios com `synchronized` podem reduzir eficiência  
❌ Bibliotecas que criam threads manualmente podem ignorar virtual threads

---

## 10. Casos de Uso Ideais

✔ APIs REST tradicionais com alta concorrência  
✔ Sistemas com muitas chamadas IO (banco, HTTP, mensageria)  
✔ Aplicações que desejam simplicidade e escalabilidade  
✔ Migração de sistemas síncronos legados para ambientes cloud

---

## 11. Boas Práticas

✔ Utilize Java 21 ou superior  
✔ Ative virtual threads via configuração global  
✔ Evite pools de threads manuais desnecessários  
✔ Monitore bloqueios e uso de recursos  
✔ Teste sob carga real  
✔ Combine com observabilidade (Micrometer, tracing distribuído)

---

## 12. Conclusão

As Virtual Threads representam uma evolução significativa no modelo de concorrência Java, oferecendo escalabilidade massiva sem abandonar a simplicidade do código síncrono.

No ecossistema Spring, elas permitem desenvolver sistemas modernos, performáticos e de fácil manutenção, reduzindo a necessidade de modelos reativos complexos em grande parte dos cenários tradicionais.
