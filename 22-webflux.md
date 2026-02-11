# 22 — Programação Reativa com Spring WebFlux

## 🎯 Objetivo

Este capítulo apresenta a **programação reativa** com Spring WebFlux, demonstrando como construir aplicações **não bloqueantes**, altamente **concorrentes** e **escaláveis**, utilizando os tipos reativos `Mono` e `Flux` do Project Reactor. Também é apresentada uma comparação entre WebFlux e **Virtual Threads (Java 21+)**, destacando quando utilizar cada abordagem.

---

## 🧠 Fundamentos da Programação Reativa

A programação reativa no Spring é baseada na especificação **Reactive Streams**, cujos princípios fundamentais são:

- **Assíncrono**: nenhuma thread é bloqueada aguardando operações de IO.
- **Não bloqueante**: recursos são liberados enquanto operações externas estão em andamento.
- **Backpressure**: o consumidor controla a taxa de emissão de dados.
- **Orientado a eventos**: os dados fluem por pipelines reativos.

No ecossistema Spring:

- `Mono<T>` → representa **0 ou 1** elemento.
- `Flux<T>` → representa **0 a N** elementos (stream).

---

## ⚙️ Configuração do Spring WebFlux

Adicionar a dependência Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

O Spring Boot:

- Configura automaticamente o runtime reativo (Netty por padrão).
- Habilita suporte a REST reativo, SSE, WebSockets e streaming.
- Integra-se com Micrometer, tracing e segurança.

---

## 🌐 Controladores Reativos

```java
@RestController
@RequestMapping("/pedidos")
public class PedidoController {

    @GetMapping("/{id}")
    public Mono<Pedido> buscarPorId(@PathVariable Long id) {
        return pedidoService.buscarPorId(id);
    }

    @GetMapping
    public Flux<Pedido> listarTodos() {
        return pedidoService.listarTodos();
    }
}
```

### Diferenças em relação ao Spring MVC

- Métodos retornam `Mono` ou `Flux`, não objetos diretos.
- O framework gerencia a assinatura reativa e o streaming da resposta.
- Suporte nativo a **backpressure**.
- Baseado em modelo **event-loop**.

---

## 🔄 Trabalhando com Mono e Flux

### Exemplo básico

```java
Flux<Integer> numeros = Flux.range(1, 5)
    .map(n -> n * 2)
    .filter(n -> n > 5);

numeros.subscribe(System.out::println);
```

### Operadores mais utilizados

- `map`, `flatMap`, `filter`
- `zip`, `merge`, `concat`
- `onErrorResume`, `onErrorMap`
- `retry`, `timeout`, `delayElements`

Os operadores permitem criar pipelines funcionais e reativos.

---

## 🌍 WebClient — Cliente HTTP Reativo

```java
@Autowired
private WebClient webClient;

public Mono<Pedido> buscarPedido(Long id) {
    return webClient.get()
        .uri("/pedidos/{id}", id)
        .retrieve()
        .bodyToMono(Pedido.class);
}
```

### Vantagens

- Totalmente não bloqueante.
- Suporte a timeout, retry e circuit breaker.
- Integração com tracing e métricas.
- Encadeamento fluido com operadores reativos.

---

## 🗄️ Persistência Reativa

O Spring WebFlux integra-se com diversas tecnologias reativas:

- **R2DBC** → bancos relacionais reativos
- **MongoDB Reactive**
- **Redis Reactive**
- **Cassandra Reactive**
- Mensageria reativa (Kafka, RabbitMQ, Pulsar)

### Exemplo com R2DBC

```java
@Repository
public interface PedidoRepository extends ReactiveCrudRepository<Pedido, Long> {

    Flux<Pedido> findByStatus(String status);
}
```

---

## 🔁 Streaming, SSE e WebSockets

### Server-Sent Events (SSE)

```java
@GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> stream() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(i -> "Evento " + i);
}
```

Permite envio contínuo de eventos ao cliente.

### WebSockets

O Spring WebFlux oferece suporte nativo a WebSockets reativos, possibilitando comunicação bidirecional baseada em streams.

---

## 🧵 WebFlux vs Virtual Threads (Java 21+)

Com o Java 21, surgem as **Virtual Threads**, permitindo alto paralelismo com código imperativo tradicional.

| Característica       | WebFlux (Reativo)             | Virtual Threads (Java 21+) |
| -------------------- | ----------------------------- | -------------------------- |
| Modelo de execução   | Event-loop, não bloqueante    | Threads leves, bloqueantes |
| Backpressure         | Nativo                        | Não nativo                 |
| Código existente     | Precisa ser reativo           | Funciona sem reescrita     |
| Curva de aprendizado | Mais elevada                  | Mais simples               |
| Ideal para           | Streaming, eventos, pipelines | APIs REST tradicionais     |

### Quando usar cada um?

- **WebFlux**: sistemas orientados a eventos, streaming contínuo, alta taxa de IO.
- **Virtual Threads**: APIs REST síncronas com alta concorrência e menor complexidade arquitetural.

---

## ⚠️ Armadilhas Comuns

- Utilizar métodos bloqueantes (`block()`, JDBC, IO síncrono) em pipelines reativos.
- Usar `@Transactional` tradicional em fluxos reativos.
- Ignorar backpressure em grandes volumes de dados.
- Misturar Spring MVC e WebFlux no mesmo contexto sem isolamento adequado.

---

## 🧰 Boas Práticas

- Utilize `Mono` para operações unitárias.
- Utilize `Flux` para streams e grandes volumes.
- Evite `block()` em produção.
- Combine com Resilience4j (retry, circuit breaker).
- Monitore métricas e tracing (Micrometer + OpenTelemetry).
- Prefira bancos reativos em aplicações totalmente reativas.

---

## 📚 Referências

- https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html
- https://projectreactor.io/docs/core/release/reference/
- https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#webflux
- https://www.baeldung.com/spring-webflux

---

## ✅ Conclusão

O Spring WebFlux oferece uma abordagem robusta para construção de aplicações **reativas, não bloqueantes e altamente escaláveis**, especialmente adequadas para sistemas distribuídos e orientados a eventos.

Com a introdução das Virtual Threads no Java 21, novas arquiteturas tornam-se possíveis. Ainda assim, WebFlux permanece essencial em cenários que exigem **backpressure, streaming contínuo e pipelines reativos complexos**, consolidando-se como uma ferramenta estratégica no ecossistema Spring moderno.
