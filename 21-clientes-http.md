# 21 — Clientes HTTP e Integrações no Spring

## 🎯 Objetivo

Este capítulo aborda as diversas formas de realizar **requisições HTTP** e integrar sistemas externos utilizando Spring, incluindo:

- RestTemplate (legado)
- WebClient (reativo)
- RestClient (Spring 6.1+)
- Clientes declarativos com `@HttpExchange`

O objetivo é permitir que aplicações Spring se comuniquem de forma **eficiente, segura e escalável** com APIs externas ou microserviços internos.

---

## 🧠 Conceitos Fundamentais

A comunicação HTTP é essencial em arquiteturas **microservices**, **cloud-native** e em cenários de **integração de sistemas**.

As opções de cliente no Spring evoluíram para oferecer:

- Suporte **síncrono** e **reativo**
- Tratamento de erros e timeouts
- Serialização e desserialização automática (JSON, XML)
- Configuração de autenticação e headers customizados
- Integração com métricas e tracing (Micrometer / Observabilidade)

---

# ⚙️ RestTemplate (Legado)

O `RestTemplate` é o cliente HTTP clássico do Spring.  
Atualmente está em modo de manutenção e **não é recomendado para novos projetos**.

## 1. Exemplo básico

```java
@Autowired
private RestTemplate restTemplate;

public String consumirApi() {
    String url = "https://api.exemplo.com/pedidos/1";
    return restTemplate.getForObject(url, String.class);
}
```

## 2. Configuração do Bean

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

## 3. Limitações

- Cliente **bloqueante (síncrono)**
- Não recomendado para aplicações reativas modernas
- Configuração de timeouts e interceptadores menos flexível
- Evolução descontinuada em favor do `RestClient`

---

# 🔄 WebClient (Reativo)

O `WebClient` substitui o `RestTemplate` em aplicações reativas e de alta performance.  
Faz parte do **Spring WebFlux**.

## 1. Criação do Bean

```java
@Bean
public WebClient webClient(WebClient.Builder builder) {
    return builder
            .baseUrl("https://api.exemplo.com")
            .build();
}
```

## 2. Uso básico

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

- `Mono` → 0 ou 1 resultado
- `Flux` → múltiplos resultados
- Suporte nativo a programação reativa

## 3. Headers e autenticação

```java
webClient.get()
        .uri("/pedidos")
        .header("Authorization", "Bearer " + token)
        .retrieve()
        .bodyToMono(List.class);
```

Também é possível configurar:

- Filtros globais (logging, tracing)
- Timeouts
- Retry
- Circuit breaker

---

# ⚡ RestClient (Spring 6.1+)

O `RestClient` é o cliente HTTP moderno introduzido no Spring Framework 6.1.  
É a evolução do `RestTemplate`, com API mais fluida e integração nativa com observabilidade.

## Exemplo básico

```java
RestClient client = RestClient.create();

String resposta = client.get()
        .uri("https://api.exemplo.com/pedidos/1")
        .retrieve()
        .body(String.class);
```

## Com configuração como Bean

```java
@Bean
public RestClient restClient() {
    return RestClient.builder()
            .baseUrl("https://api.exemplo.com")
            .build();
}
```

### Vantagens

- API fluente e moderna
- Integração com Micrometer e tracing
- Substituto recomendado para `RestTemplate`
- Suporte síncrono com melhor configuração

---

# 🧩 Clientes Declarativos com `@HttpExchange`

O Spring permite criar **clientes HTTP declarativos**, reduzindo boilerplate.

## 1. Interface declarativa

```java
@HttpExchange("/pedidos")
public interface PedidoClient {

    @GetExchange("/{id}")
    Pedido buscarPedido(@PathVariable Long id);
}
```

## 2. Registro do cliente

```java
@Bean
public PedidoClient pedidoClient(RestClient.Builder builder) {
    return HttpServiceProxyFactory
            .builderFor(RestClientAdapter.create(builder.build()))
            .build()
            .createClient(PedidoClient.class);
}
```

### Benefícios

- Código mais limpo
- Redução de repetição
- Integração com DI
- Ideal para APIs estáveis e bem definidas

---

# ⚙️ Tratamento de Erros e Timeouts

## WebClient — Tratamento de erro

```java
webClient.get()
        .uri("/pedidos/{id}", id)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError,
                response -> Mono.error(new RuntimeException("Erro 4xx")))
        .bodyToMono(Pedido.class);
```

## Configuração de Timeout (WebClient)

```java
HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(3));

WebClient client = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

## Estratégias recomendadas

- Retry com `retryWhen` (Reactor)
- Circuit breaker com **Resilience4j**
- Fallbacks para alta disponibilidade

---

# 🧰 Boas Práticas

- Prefira **WebClient** ou **RestClient** em novos projetos
- Configure timeouts curtos e retries controlados
- Evite chamadas bloqueantes em aplicações reativas
- Use clientes declarativos para APIs conhecidas
- Habilite métricas e tracing para monitoramento
- Centralize configuração de autenticação e interceptadores
- Proteja chamadas externas com circuit breaker

---

# ⚠️ Armadilhas Comuns

- Não configurar timeout (pode travar threads)
- Fazer `.block()` em aplicações reativas sem necessidade
- Excesso de retries sem controle
- Ignorar tratamento adequado de erros HTTP
- Não monitorar chamadas externas

---

# 📚 Referências

- https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.client
- https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client
- https://spring.io/blog/2022/07/25/restclient-introducing-a-modern-spring-http-client
- https://www.baeldung.com/spring-5-webclient

---

# ✅ Conclusão

O Spring oferece múltiplas formas de integração HTTP, desde o legado `RestTemplate` até clientes modernos como `WebClient`, `RestClient` e clientes declarativos com `@HttpExchange`.

A escolha depende do contexto:

- Aplicações reativas → **WebClient**
- Aplicações síncronas modernas → **RestClient**
- Código mais declarativo → **HttpExchange**

Selecionar corretamente a ferramenta garante melhor desempenho, escalabilidade e observabilidade em sistemas distribuídos.

---

<p align="center">
<b>Finalizada a Clientes HTTP e Integrações no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="22-webflux.md">Programação Reativa com Spring WebFlux</a>
</p>
