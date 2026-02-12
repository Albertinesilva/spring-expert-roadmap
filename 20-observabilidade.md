# 20 — Observabilidade (Metrics, Tracing e Logging) no Spring

## 🎯 Objetivo

Este capítulo aborda como o Spring Boot oferece suporte completo à **observabilidade**, permitindo monitorar, medir e diagnosticar aplicações de forma eficiente. Os principais pilares são:

- **Metrics**: métricas de performance e utilização de recursos
- **Tracing**: rastreamento distribuído de requisições
- **Logging**: registro estruturado de eventos e exceções
- **Health Checks**: monitoramento da saúde da aplicação

O objetivo é permitir que aplicações Java escaláveis e distribuídas possam ser **observadas, monitoradas e mantidas** com confiabilidade.

---

# 🧠 Conceitos Fundamentais

**Observabilidade** é a capacidade de inferir o estado interno de um sistema a partir de sinais externos.

Em arquiteturas modernas (microserviços e cloud-native), ela se apoia em quatro pilares:

1. **Metrics** → dados quantitativos (latência, taxa de erro, throughput, memória)
2. **Logs** → eventos detalhados da execução
3. **Tracing** → rastreamento de requisições entre múltiplos serviços
4. **Health Checks** → verificação da saúde da aplicação e suas dependências

Esses pilares são complementares e devem ser usados em conjunto.

---

# 📊 Métricas com Micrometer

O Spring Boot utiliza o **Micrometer** como facade para coleta de métricas, com suporte a múltiplos backends:

- Prometheus
- Datadog
- New Relic
- CloudWatch
- Grafana
- Entre outros

---

## 1️⃣ Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

---

## 2️⃣ Exposição de Endpoints

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,metrics,prometheus"
```

Endpoints importantes:

- `/actuator/metrics`
- `/actuator/prometheus`
- `/actuator/health`

---

## 3️⃣ Métricas Customizadas

```java
@Autowired
private MeterRegistry meterRegistry;

public void processarPedido() {

    Timer timer = meterRegistry.timer("pedido.processamento");

    timer.record(() -> {
        // lógica de processamento
    });
}
```

Também é possível usar:

- `Counter` → contadores
- `Gauge` → valores instantâneos
- `DistributionSummary` → distribuição de valores

Exemplo:

```java
Counter counter = meterRegistry.counter("pedido.criados");
counter.increment();
```

---

# 🔄 Tracing Distribuído com OpenTelemetry

Tracing permite rastrear uma requisição através de múltiplos serviços.

O Spring Boot 3 utiliza **Micrometer Tracing** com suporte a OpenTelemetry.

---

## 1️⃣ Dependência

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

---

## 2️⃣ Exemplo de Uso Manual

```java
@Autowired
private Tracer tracer;

public void executar() {

    Span span = tracer.nextSpan()
                      .name("processarPedido")
                      .start();

    try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
        // lógica de negócio
    } finally {
        span.end();
    }
}
```

### Conceitos importantes

- **Trace** → conjunto completo da requisição
- **Span** → unidade individual de trabalho
- **TraceId** → identifica toda a requisição
- **SpanId** → identifica um trecho específico

---

## 🔗 Correlação com Logs

Ao usar tracing corretamente, logs passam a incluir:

- `traceId`
- `spanId`

Isso permite rastrear facilmente falhas em sistemas distribuídos.

---

# 📝 Logging no Spring Boot

O Spring Boot utiliza por padrão:

- SLF4J (facade)
- Logback (implementação)

---

## 1️⃣ Configuração Básica

```yaml
logging:
  level:
    root: INFO
    com.exemplo: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n"
```

---

## 2️⃣ Logs Estruturados (JSON)

Configuração no `logback-spring.xml`:

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

Ideal para integração com:

- ELK Stack (Elasticsearch + Logstash + Kibana)
- Grafana Loki
- Splunk

---

# ❤️ Health Checks (Actuator)

O Spring Boot Actuator fornece endpoints de monitoramento da aplicação.

---

## 1️⃣ Configuração

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info"
```

---

## 2️⃣ Endpoint Padrão

```
/actuator/health
```

Retorna:

```json
{
  "status": "UP"
}
```

---

## 3️⃣ Health Indicator Customizado

```java
@Component
public class BancoHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {

        try {
            // lógica para checar banco
            return Health.up()
                    .withDetail("Banco", "OK")
                    .build();

        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

---

## 4️⃣ Liveness e Readiness (Kubernetes)

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true
```

Endpoints:

- `/actuator/health/liveness`
- `/actuator/health/readiness`

### Diferença:

- **Liveness** → aplicação está viva?
- **Readiness** → está pronta para receber tráfego?

Essencial para ambientes Kubernetes.

---

# 📈 Integração com Prometheus e Grafana

```yaml
management:
  metrics:
    export:
      prometheus:
        enabled: true
  endpoints:
    web:
      exposure:
        include: "metrics,prometheus,health"
```

Fluxo típico:

Spring Boot → Prometheus → Grafana

Grafana permite dashboards com:

- Latência
- Throughput
- Erros
- Uso de CPU/memória
- Métricas customizadas

---

# 🧰 Boas Práticas

✔️ Combine métricas + tracing + logging  
✔️ Use logs estruturados em JSON  
✔️ Configure sampling para tracing em produção  
✔️ Monitore latência, taxa de erro e consumo de recursos  
✔️ Configure readiness e liveness em ambientes cloud  
✔️ Adicione tags às métricas para contexto (ex: status, endpoint, tipo)

---

# ⚠️ Armadilhas Comuns

❌ Métricas sem contexto ou tags  
❌ Logs excessivos em produção  
❌ Tracing sem correlação com logs  
❌ Não monitorar dependências externas (DB, APIs, filas)  
❌ Não proteger endpoints do Actuator

---

# 📚 Referências

- https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html
- https://micrometer.io/
- https://opentelemetry.io/
- https://www.baeldung.com/spring-boot-actuators

---

# ✅ Conclusão

O Spring Boot fornece um ecossistema robusto e integrado para observabilidade, permitindo monitorar métricas, rastrear requisições distribuídas, registrar eventos estruturados e verificar a saúde da aplicação.

Ao combinar **metrics + tracing + logging + health checks**, é possível construir sistemas resilientes, escaláveis e preparados para ambientes modernos de Cloud e microsserviços.

---

<p align="center">
<b>Finalizada a Observabilidade (Metrics, Tracing e Logging) no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="21-clientes-http.md">Clientes HTTP e Integrações no Spring</a>
</p>
