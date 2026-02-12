# 📡 Eventos, Mensageria e Streaming

Arquiteturas modernas são cada vez mais **orientadas a eventos**, **assíncronas** e **desacopladas**. O ecossistema Spring fornece suporte robusto para publicação de eventos, integração com sistemas de mensageria e plataformas de streaming de dados, permitindo construir aplicações resilientes, escaláveis e reativas.

Este capítulo cobre desde eventos internos no Spring até integração com brokers como RabbitMQ, Kafka e sistemas de streaming distribuído.

---

# 🧠 Conceitos Fundamentais

## 🔹 Evento

Um fato ocorrido no sistema, representado por um objeto imutável, que pode ser consumido por múltiplos interessados.

## 🔹 Mensageria

Troca de mensagens assíncronas entre sistemas ou componentes via brokers.

## 🔹 Streaming

Processamento contínuo de fluxos de dados em tempo real, com ordenação, particionamento e retenção.

---

# 🧩 Tipos de Comunicação

| Tipo           | Característica              |
| -------------- | --------------------------- |
| Síncrona       | Bloqueante, acoplada        |
| Assíncrona     | Não bloqueante, desacoplada |
| Event-driven   | Baseada em eventos, reativa |
| Message-driven | Baseada em mensagens        |
| Stream-driven  | Baseada em fluxo contínuo   |

---

# 🔁 Eventos Internos no Spring

## 🔧 ApplicationEventPublisher

```java
@Component
public class PedidoService {

    private final ApplicationEventPublisher publisher;

    public PedidoService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void confirmarPedido(Pedido pedido) {
        // lógica de negócio
        publisher.publishEvent(new PedidoConfirmadoEvent(pedido));
    }
}
```

---

## 🔸 Definindo um Evento

```java
public class PedidoConfirmadoEvent {

    private final Pedido pedido;

    public PedidoConfirmadoEvent(Pedido pedido) {
        this.pedido = pedido;
    }

    public Pedido getPedido() {
        return pedido;
    }
}
```

---

## 🔸 Consumindo Eventos

```java
@Component
public class EmailListener {

    @EventListener
    public void enviarEmail(PedidoConfirmadoEvent event) {
        // lógica de envio de email
    }
}
```

---

## 🔸 Eventos Transacionais

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPedido(PedidoConfirmadoEvent event) {
    // será executado somente após commit
}
```

### 📌 Fases disponíveis

- BEFORE_COMMIT
- AFTER_COMMIT
- AFTER_ROLLBACK
- AFTER_COMPLETION

---

# 📨 Mensageria com Spring AMQP (RabbitMQ)

## 🔧 Conceitos

- **Producer** → Publica mensagens.
- **Broker** → Intermediário (RabbitMQ).
- **Queue** → Fila de mensagens.
- **Exchange** → Roteador de mensagens.
- **Consumer** → Processa mensagens.

---

## 🔸 Configuração Básica

```java
@Configuration
public class RabbitConfig {

    @Bean
    public Queue filaPedidos() {
        return new Queue("pedidos.queue", true);
    }

    @Bean
    public DirectExchange exchangePedidos() {
        return new DirectExchange("pedidos.exchange");
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(filaPedidos())
                .to(exchangePedidos())
                .with("pedido.confirmado");
    }
}
```

---

## 🔸 Enviando Mensagens

```java
@Autowired
private RabbitTemplate rabbitTemplate;

public void publicarPedido(Pedido pedido) {
    rabbitTemplate.convertAndSend(
        "pedidos.exchange",
        "pedido.confirmado",
        pedido
    );
}
```

---

## 🔸 Consumindo Mensagens

```java
@RabbitListener(queues = "pedidos.queue")
public void processarPedido(Pedido pedido) {
    // lógica
}
```

---

# 📨 Mensageria com Apache Kafka

## 🔧 Conceitos

- **Producer** → Envia mensagens.
- **Topic** → Canal de mensagens.
- **Partition** → Divisão de paralelismo.
- **Consumer Group** → Grupo de consumidores.
- **Offset** → Posição no stream.

---

## 🔸 Produzindo Mensagens

```java
@Autowired
private KafkaTemplate<String, Pedido> kafkaTemplate;

public void publicarPedido(Pedido pedido) {
    kafkaTemplate.send(
        "pedidos-topic",
        pedido.getId(),
        pedido
    );
}
```

---

## 🔸 Consumindo Mensagens

```java
@KafkaListener(
    topics = "pedidos-topic",
    groupId = "pedido-group"
)
public void consumirPedido(Pedido pedido) {
    // lógica
}
```

---

## 🔸 Configuração Essencial (application.yml)

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: pedido-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

---

# 🌊 Streaming de Dados

Streaming é diferente de mensageria tradicional:

| Mensageria                        | Streaming                      |
| --------------------------------- | ------------------------------ |
| Mensagens pontuais                | Fluxo contínuo                 |
| Geralmente deletadas após consumo | Retidas por tempo configurável |
| Foco em entrega                   | Foco em processamento          |

---

# 🔄 Spring Cloud Stream

Spring Cloud Stream abstrai brokers (Kafka, RabbitMQ, Pulsar, etc.) através de **binders**.

---

## 🔧 Exemplo com Funções

```java
@Bean
public Function<Pedido, Pedido> processarPedido() {
    return pedido -> {
        // lógica
        return pedido;
    };
}
```

---

## 🔸 Configuração

```yaml
spring:
  cloud:
    stream:
      bindings:
        processarPedido-in-0:
          destination: pedidos-in
        processarPedido-out-0:
          destination: pedidos-out
      default-binder: kafka
```

---

# 🔄 Processamento de Streams com Kafka Streams

Spring Kafka oferece integração com Kafka Streams API.

```java
@Bean
public KStream<String, Pedido> processarPedidos(StreamsBuilder builder) {

    KStream<String, Pedido> stream =
        builder.stream("pedidos-topic");

    stream
        .filter((key, pedido) -> pedido.getValor() > 1000)
        .to("pedidos-alto-valor");

    return stream;
}
```

---

# 🔁 Padrões Arquiteturais

## 🔹 Event-Driven Architecture (EDA)

- Componentes reagem a eventos.
- Baixo acoplamento.
- Alta escalabilidade.

---

## 🔹 Publish/Subscribe

- Um produtor → múltiplos consumidores.
- Desacoplamento total.

---

## 🔹 Saga Pattern

Coordenação de transações distribuídas através de eventos e compensações.

---

## 🔹 Outbox Pattern

Garante consistência entre banco e mensageria.

### 📌 Fluxo

1. Salva entidade.
2. Salva evento na tabela outbox.
3. Publicador lê outbox e envia mensagem.
4. Marca como processado.

---

## 🔹 Event Sourcing

- Estado derivado da sequência de eventos.
- Alta auditabilidade.
- Complexidade maior.

---

# 🔐 Garantias de Entrega

| Garantia      | Significado                           |
| ------------- | ------------------------------------- |
| At-most-once  | Pode perder mensagens, nunca duplica  |
| At-least-once | Nunca perde, pode duplicar            |
| Exactly-once  | Não perde nem duplica (mais complexo) |

Kafka suporta **Exactly-Once Semantics (EOS)** com configurações adequadas.

---

# 🧪 Testes de Mensageria

## 🔸 Testes com Kafka

```java
@SpringBootTest
@EmbeddedKafka
class PedidoKafkaTest {

    @Autowired
    private KafkaTemplate<String, Pedido> kafkaTemplate;

    @Test
    void devePublicarPedido() {
        kafkaTemplate.send("pedidos-topic", new Pedido());
    }
}
```

---

## 🔸 Testes com RabbitMQ

Utilize **Testcontainers** ou brokers embarcados.

---

# 🛡️ Boas Práticas

- Utilize eventos imutáveis.
- Evite lógica de negócio pesada em listeners.
- Implemente idempotência.
- Use Dead Letter Queues (DLQ).
- Monitore consumo e latência.
- Trate falhas e retries adequadamente.
- Prefira mensageria a chamadas síncronas em sistemas distribuídos.
- Documente contratos de eventos.

---

# 🧱 Conclusão do Capítulo

Eventos, mensageria e streaming são pilares de arquiteturas modernas orientadas a serviços e microsserviços. O ecossistema Spring fornece abstrações poderosas e flexíveis para construir soluções desacopladas, resilientes e escaláveis.

Dominar esses conceitos permite criar sistemas que reagem em tempo real, escalam horizontalmente e mantêm consistência mesmo em ambientes distribuídos complexos.

<p align="center">
<b>Finalizada a Eventos, Mensageria e Streaming! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="12-spring-integration.md">Integração de Sistemas com Spring Integration</a>
</p>
