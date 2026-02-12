# 🔗 Integração de Sistemas com Spring Integration

Sistemas corporativos raramente vivem isolados. Eles precisam se comunicar com bancos, ERPs, sistemas legados, APIs externas, filas, arquivos, dispositivos e serviços em nuvem. O **Spring Integration** fornece um modelo poderoso e extensível para construir essas integrações de forma **assíncrona**, **desacoplada** e **orientada a mensagens**, baseado nos padrões clássicos de integração (**Enterprise Integration Patterns – EIP**).

Este capítulo explora como projetar fluxos de integração robustos, observáveis e resilientes usando Spring Integration.

---

# 🧠 Conceitos Fundamentais

## 🔹 Message

Objeto que carrega dados (**payload**) e metadados (**headers**).

```java
Message<String> message = MessageBuilder
    .withPayload("Olá, mundo!")
    .setHeader("origem", "sistema-A")
    .build();
```

---

## 🔹 Message Channel

Canal por onde as mensagens fluem.

### Tipos comuns:

- `DirectChannel` (ponto-a-ponto, síncrono)
- `QueueChannel` (assíncrono, com fila)
- `PublishSubscribeChannel` (broadcast)
- `ExecutorChannel` (assíncrono com thread pool)
- `FluxMessageChannel` (reativo)

---

## 🔹 Message Handler

Componente que processa mensagens.

---

## 🔹 Message Endpoint

Elemento terminal ou intermediário do fluxo (adapter, transformer, router, filter, etc.).

---

## 🔹 Integration Flow

Definição declarativa de um pipeline de mensagens.

---

# 🧱 Arquitetura do Spring Integration

```
Producer → Channel → [Filter → Transformer → Router → Service Activator] → Channel → Consumer
```

Baseado em EIP, promove:

- Baixo acoplamento
- Processamento assíncrono
- Alta escalabilidade
- Observabilidade

---

# 🧩 Configuração Básica com Java DSL

```java
@Configuration
@EnableIntegration
public class IntegrationConfig {

    @Bean
    public IntegrationFlow fluxoBasico() {
        return IntegrationFlows.from("entradaChannel")
            .filter(String.class, msg -> msg.contains("pedido"))
            .transform(String.class, String::toUpperCase)
            .handle(m -> System.out.println("Processado: " + m.getPayload()))
            .get();
    }
}
```

---

# 🧩 Canais (Channels)

## 🔹 DirectChannel

```java
@Bean
public MessageChannel canalDireto() {
    return new DirectChannel();
}
```

---

## 🔹 QueueChannel

```java
@Bean
public MessageChannel canalFila() {
    return new QueueChannel();
}
```

---

## 🔹 PublishSubscribeChannel

```java
@Bean
public MessageChannel canalBroadcast() {
    return new PublishSubscribeChannel();
}
```

---

## 🔹 ExecutorChannel

```java
@Bean
public MessageChannel canalAssincrono() {
    return new ExecutorChannel(Executors.newCachedThreadPool());
}
```

---

# 🔁 Adaptadores (Adapters)

Adaptadores conectam o mundo externo ao fluxo de integração.

---

## 📥 Inbound Channel Adapter (Entrada)

Exemplo: leitura de arquivos de um diretório.

```java
@Bean
public IntegrationFlow arquivoInboundFlow() {
    return IntegrationFlows
        .from(Files.inboundAdapter(new File("/tmp/entrada"))
                .autoCreateDirectory(true)
                .patternFilter("*.txt"),
            e -> e.poller(Pollers.fixedDelay(5000)))
        .handle(m -> System.out.println("Arquivo recebido: " + m.getPayload()))
        .get();
}
```

---

## 📤 Outbound Channel Adapter (Saída)

Exemplo: gravação de arquivos.

```java
@Bean
public IntegrationFlow arquivoOutboundFlow() {
    return IntegrationFlows.from("canalSaida")
        .handle(Files.outboundAdapter(new File("/tmp/saida"))
                .autoCreateDirectory(true)
                .fileNameGenerator(m -> "saida.txt"))
        .get();
}
```

---

# 🔄 Transformadores (Transformers)

Transformam o payload da mensagem.

```java
@Bean
public IntegrationFlow transformadorFlow() {
    return IntegrationFlows.from("entrada")
        .transform(String.class, s -> s.toUpperCase())
        .channel("saida")
        .get();
}
```

---

# 🚦 Filtros (Filters)

Permitem ou bloqueiam mensagens com base em regras.

```java
@Bean
public IntegrationFlow filtroFlow() {
    return IntegrationFlows.from("entrada")
        .filter(String.class, s -> s.contains("OK"))
        .channel("saida")
        .get();
}
```

---

# 🔀 Roteadores (Routers)

Encaminham mensagens para diferentes canais com base em critérios.

```java
@Bean
public IntegrationFlow routerFlow() {
    return IntegrationFlows.from("entrada")
        .route(String.class,
            s -> s.startsWith("A") ? "canalA" : "canalB")
        .get();
}
```

---

# 🔧 Service Activator

Conecta um método Java ao fluxo de mensagens.

```java
@Bean
@ServiceActivator(inputChannel = "entrada", outputChannel = "saida")
public MessageHandler processador() {
    return message ->
        System.out.println("Processando: " + message.getPayload());
}
```

---

# 🔁 Gateways

Gateways conectam chamadas síncronas (métodos Java) ao mundo assíncrono da integração.

```java
@MessagingGateway
public interface PedidoGateway {

    @Gateway(
        requestChannel = "entrada",
        replyChannel = "saida"
    )
    String processarPedido(String pedido);
}
```

---

# 🧬 Agregadores e Divisores

## 🔹 Splitter

Divide uma mensagem em várias.

```java
@Bean
public IntegrationFlow splitterFlow() {
    return IntegrationFlows.from("entrada")
        .split()
        .channel("processamento")
        .get();
}
```

---

## 🔹 Aggregator

Agrupa mensagens relacionadas.

```java
@Bean
public IntegrationFlow aggregatorFlow() {
    return IntegrationFlows.from("processamento")
        .aggregate()
        .channel("saida")
        .get();
}
```

---

# 🔄 Retry, Error Handling e Resiliência

## 🔹 Canal de Erro

```java
@Bean
public MessageChannel errorChannel() {
    return new DirectChannel();
}

@ServiceActivator(inputChannel = "errorChannel")
public void tratarErro(ErrorMessage errorMessage) {
    System.err.println("Erro: " + errorMessage.getPayload());
}
```

---

## 🔹 Retry Advice

```java
@Bean
public RequestHandlerRetryAdvice retryAdvice() {
    RequestHandlerRetryAdvice advice = new RequestHandlerRetryAdvice();
    RetryTemplate retryTemplate = new RetryTemplate();
    advice.setRetryTemplate(retryTemplate);
    return advice;
}
```

---

## 🔹 Circuit Breaker

```java
@Bean
public CircuitBreakerAdvice circuitBreakerAdvice() {
    return new CircuitBreakerAdvice();
}
```

---

# 🌐 Integrações Comuns

Spring Integration oferece módulos para:

- HTTP/REST
- FTP/SFTP
- JMS
- Kafka
- RabbitMQ
- MQTT
- WebSockets
- E-mail (SMTP/IMAP/POP3)
- JDBC
- Sistemas legados (mainframe, arquivos, sockets)

---

## 🔹 Exemplo: HTTP Inbound Gateway

```java
@Bean
public IntegrationFlow httpInboundFlow() {
    return IntegrationFlows
        .from(Http.inboundGateway("/api/integracao")
            .requestMapping(m -> m.methods(HttpMethod.POST))
            .requestPayloadType(String.class))
        .handle((payload, headers) ->
            "Recebido: " + payload)
        .get();
}
```

---

## 🔹 Exemplo: JMS Outbound Adapter

```java
@Bean
public IntegrationFlow jmsOutboundFlow(
        ConnectionFactory connectionFactory) {

    return IntegrationFlows.from("entrada")
        .handle(Jms.outboundAdapter(connectionFactory)
            .destination("fila.externa"))
        .get();
}
```

---

# 🔍 Observabilidade

Spring Integration fornece métricas, tracing e logs:

- Micrometer Integration
- Spring Boot Actuator
- OpenTelemetry

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,integrations
```

---

# 🧪 Testes de Fluxos de Integração

## 🔹 Testando Fluxos

```java
@SpringBootTest
class IntegrationFlowTest {

    @Autowired
    private MessageChannel entrada;

    @Autowired
    private PollableChannel saida;

    @Test
    void deveProcessarMensagem() {

        entrada.send(
            MessageBuilder
                .withPayload("pedido OK")
                .build()
        );

        Message<?> resposta = saida.receive(5000);

        assertNotNull(resposta);
    }
}
```

---

# 🛡️ Boas Práticas

- Use mensagens imutáveis.
- Prefira canais assíncronos para I/O.
- Evite lógica pesada em adaptadores.
- Separe fluxos por domínio.
- Use retry e DLQ para resiliência.
- Instrumente seus fluxos.
- Documente contratos de mensagens.
- Teste fluxos críticos isoladamente.

---

# 🧱 Conclusão do Capítulo

Spring Integration fornece uma plataforma madura e poderosa para orquestrar comunicações complexas entre sistemas heterogêneos. Ao aplicar os padrões EIP com as abstrações do Spring, é possível construir soluções robustas, escaláveis e altamente observáveis, fundamentais para ambientes corporativos modernos e arquiteturas distribuídas.

Dominar Spring Integration significa dominar a arte de conectar sistemas com segurança, resiliência e elegância arquitetural.

---

<p align="center">
<b>Finalizada a Integração de Sistemas com Spring Integration! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="13-spring-state-machine.md">Máquinas de Estado com Spring State Machine</a>
</p>
