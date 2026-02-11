# 23 — Cloud Native, Configuração Distribuída e Gateways

## 🎯 Objetivo

Este capítulo aborda como construir aplicações **Cloud Native** com Spring, explorando:

- Configuração distribuída
- Service Discovery
- API Gateway e roteamento
- Resiliência (Circuit Breaker, Retry, Rate Limiting)
- Integração com provedores de nuvem (AWS, GCP, Azure)

O foco é compreender não apenas o uso das ferramentas, mas também **como funcionam internamente** e **quais são os trade-offs arquiteturais envolvidos**.

---

## ☁️ O que significa Cloud Native?

Aplicações **Cloud Native** são projetadas para:

- Executar em **ambientes distribuídos**
- Serem **resilientes a falhas**
- Escalar horizontalmente
- Utilizar **configuração externa**
- Integrar-se com **containers, Kubernetes e infraestrutura como código**

O ecossistema Spring oferece suporte completo por meio do **Spring Cloud**, permitindo implementação de padrões modernos de arquitetura distribuída.

---

## ⚙️ Configuração Distribuída — Spring Cloud Config

### 📌 Conceito

O **Spring Cloud Config** permite externalizar e centralizar configurações em:

- Git
- HashiCorp Vault
- Amazon S3
- Sistemas de arquivos

Essas configurações podem ser compartilhadas entre múltiplos serviços de forma centralizada e versionada.

---

### 🚀 Config Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

---

### 🔗 Cliente Config

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
spring:
  application:
    name: pedido-service
  config:
    import: optional:configserver:http://localhost:8888
```

- As propriedades são carregadas antes da inicialização completa do contexto.
- Permite atualização dinâmica com **Spring Cloud Bus**.

---

## 🔍 Service Discovery — Eureka, Consul e similares

### 📌 Conceito

Service Discovery permite que serviços se registrem e sejam descobertos dinamicamente, eliminando dependência de IPs fixos.

Benefícios:

- Registro automático
- Balanceamento de carga
- Failover transparente

---

### 🛰️ Eureka Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@EnableEurekaServer
@SpringBootApplication
public class DiscoveryServerApplication { }
```

---

### 📡 Cliente Eureka

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

---

## 🚪 API Gateway — Spring Cloud Gateway

### 📌 Conceito

O **Spring Cloud Gateway** atua como:

- Ponto único de entrada (API Gateway)
- Roteador de requisições
- Aplicador de políticas transversais (segurança, rate limit, logging)

---

### ⚙️ Dependência

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

---

### 🔀 Configuração de Rotas

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: pedido-service
          uri: lb://PEDIDO-SERVICE
          predicates:
            - Path=/pedidos/**
          filters:
            - StripPrefix=1
```

---

### 🔧 Filtros Comuns

- Autenticação e autorização
- Rate limiting
- Logging e tracing
- Reescrita de headers e paths

---

## 🛡️ Resiliência — Circuit Breaker, Retry e Rate Limiting

### 🔌 Dependência Resilience4j

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

---

### ⚡ Circuit Breaker

```java
@CircuitBreaker(name = "pedidoService", fallbackMethod = "fallback")
public Pedido buscarPedido(Long id) {
    // chamada remota
}
```

---

### 🔁 Retry

```java
@Retry(name = "pedidoService")
public Pedido buscarPedido(Long id) {
    // chamada remota
}
```

---

### 🚦 Rate Limiting

```java
@RateLimiter(name = "pedidoService")
public Pedido buscarPedido(Long id) {
    // chamada remota
}
```

Esses mecanismos protegem o sistema contra:

- Sobrecarga
- Instabilidade de dependências externas
- Ataques de negação de serviço

---

## 🔄 Load Balancing

O Spring fornece balanceamento no cliente com:

- **Spring Cloud LoadBalancer**
- Integração com `WebClient` e `RestClient`

```java
@Bean
@LoadBalanced
public WebClient.Builder webClientBuilder() {
    return WebClient.builder();
}
```

---

## ☁️ Integração com Provedores de Nuvem

### AWS — Spring Cloud AWS

- S3
- SQS
- SNS
- RDS
- Secrets Manager
- Parameter Store

### GCP — Spring Cloud GCP

- Pub/Sub
- Firestore
- Cloud Storage
- BigQuery

### Azure — Spring Cloud Azure

- Blob Storage
- Event Hubs
- Service Bus
- Key Vault

---

## 🧩 Kubernetes e Cloud Native Patterns

Spring Boot integra-se com Kubernetes via:

- ConfigMaps e Secrets
- Liveness e Readiness Probes
- Horizontal Pod Autoscaler (HPA)
- Service Mesh (Istio, Linkerd)

### 📐 Padrões Arquiteturais Comuns

- Externalized Configuration
- Service Discovery
- Circuit Breaker
- Bulkhead
- Rate Limiting
- Blue/Green Deployment
- Canary Deployment

---

## ⚠️ Armadilhas Comuns

- Centralizar configuração sem versionamento adequado
- Não tratar falhas de rede como regra
- Ausência de timeout e retry em chamadas HTTP
- Ignorar observabilidade em ambientes distribuídos

---

## 🧰 Boas Práticas

- Externalize todas as configurações sensíveis
- Utilize Gateway para centralizar políticas transversais
- Aplique circuit breakers e timeouts por padrão
- Monitore métricas, logs e tracing distribuídos
- Teste falhas com práticas de **chaos engineering**

---

## 📚 Referências

- https://spring.io/projects/spring-cloud
- https://docs.spring.io/spring-cloud-config/docs/current/reference/html/
- https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/
- https://resilience4j.readme.io/
- https://docs.spring.io/spring-boot/docs/current/reference/html/cloud.html

---

## ✅ Conclusão

O ecossistema Spring oferece um conjunto completo de ferramentas para construção de aplicações **Cloud Native**, integrando configuração distribuída, descoberta de serviços, gateways, mecanismos de resiliência e integração com provedores de nuvem.

Dominar esses conceitos é fundamental para arquiteturas modernas, escaláveis e resilientes, especialmente em ambientes baseados em microserviços e infraestrutura distribuída.
