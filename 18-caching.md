# 18 — Caching no Spring

## 🎯 Objetivo

Este capítulo aborda o suporte a **caching** no Spring Framework, incluindo:

- Conceitos de cache
- Configuração de caches
- Anotações do Spring Cache
- Estratégias avançadas
- Boas práticas e armadilhas

O objetivo é melhorar **performance**, reduzir **acesso repetitivo a recursos** e aumentar a **escalabilidade** da aplicação.

---

# 🧠 Conceitos Fundamentais

**Cache** é um mecanismo que armazena temporariamente dados ou resultados de operações para evitar cálculos ou consultas repetitivas.

## ✅ Benefícios

- Redução de latência
- Menor carga em banco de dados e serviços externos
- Melhoria na experiência do usuário
- Maior escalabilidade

## ⚖️ Trade-offs

- Possível inconsistência temporária (dados desatualizados)
- Consumo adicional de memória
- Complexidade na estratégia de invalidação

Caching deve ser aplicado com critério e estratégia clara de expiração.

---

# ⚙️ Configuração Básica

## 📦 Dependência Maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

## 🚀 Ativando o Cache

```java
@SpringBootApplication
@EnableCaching
public class MeuApp {

    public static void main(String[] args) {
        SpringApplication.run(MeuApp.class, args);
    }
}
```

A anotação `@EnableCaching` habilita o suporte a cache baseado em proxies.

---

# 🧩 Anotações do Spring Cache

## 🔹 `@Cacheable`

Armazena o resultado de um método no cache.

```java
@Cacheable("pedidos")
public Pedido buscarPedido(Long id) {
    simulateSlowService();
    return pedidoRepository.findById(id)
        .orElseThrow();
}
```

- `"pedidos"` é o nome do cache
- Se o método for chamado novamente com o mesmo argumento, o resultado será retornado diretamente do cache
- O método **não será executado novamente** se o valor já estiver armazenado

---

## 🔹 `@CachePut`

Atualiza o cache **sem impedir a execução do método**.

```java
@CachePut(value = "pedidos", key = "#pedido.id")
public Pedido atualizarPedido(Pedido pedido) {
    return pedidoRepository.save(pedido);
}
```

Ideal para manter o cache sincronizado após atualizações.

---

## 🔹 `@CacheEvict`

Remove dados do cache.

```java
@CacheEvict(value = "pedidos", key = "#id")
public void removerPedido(Long id) {
    pedidoRepository.deleteById(id);
}
```

Para limpar todo o cache:

```java
@CacheEvict(value = "pedidos", allEntries = true)
```

---

## 🔹 `@Caching`

Permite múltiplas operações de cache no mesmo método.

```java
@Caching(
    put = {
        @CachePut(value = "pedidos", key = "#pedido.id")
    },
    evict = {
        @CacheEvict(value = "relatorios", allEntries = true)
    }
)
public Pedido atualizarPedido(Pedido pedido) {
    return pedidoRepository.save(pedido);
}
```

---

# 🗄️ Backends de Cache Suportados

O Spring abstrai o cache por meio da interface `CacheManager`.

## Principais implementações:

- `ConcurrentMapCache` (em memória — ideal para testes)
- **Caffeine**
- **Redis**
- EhCache
- Hazelcast
- Infinispan

---

# ☕ Exemplo com Caffeine

```java
@Bean
public CacheManager cacheManager() {

    CaffeineCacheManager cacheManager =
        new CaffeineCacheManager("pedidos");

    cacheManager.setCaffeine(
        Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(100)
    );

    return cacheManager;
}
```

Recursos importantes:

- TTL (`expireAfterWrite`)
- Limite de tamanho (`maximumSize`)
- Alta performance em memória

---

# 🔴 Exemplo com Redis

## 📄 application.yml

```yaml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 600000
```

## 🔧 Configuração Java

```java
@Bean
public RedisCacheManager cacheManager(
        RedisConnectionFactory connectionFactory) {

    return RedisCacheManager.builder(connectionFactory)
        .cacheDefaults(
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
        )
        .build();
}
```

Redis é ideal para ambientes distribuídos e múltiplas instâncias.

---

# 🔄 Estratégias de Invalidação

- ⏳ Expiração por tempo (TTL)
- 🧹 Remoção manual (`@CacheEvict`)
- 🔑 Invalidação por chave específica
- 🗑️ Limpeza total (`allEntries = true`)

Estratégia de invalidação é tão importante quanto o cache em si.

---

# 🧩 Configuração Avançada de Chaves

Por padrão, todos os parâmetros do método compõem a chave.

É possível personalizar usando **SpEL**:

```java
@Cacheable(value = "pedidos", key = "#id + '-' + #status")
```

Expressões suportadas:

- `#root.methodName`
- `#root.args`
- `#result`
- Parâmetros nomeados (`#id`, `#pedido.id`)

---

# ⚠️ Armadilhas Comuns

- ❌ _Self-invocation_: chamadas internas não passam pelo proxy
- ❌ Métodos `private`, `final` ou `static` não são interceptados
- ❌ Cache sem TTL pode crescer indefinidamente
- ❌ Falta de limite de tamanho pode gerar consumo excessivo de memória
- ❌ Dados sensíveis armazenados sem proteção

---

# 📊 Monitoramento

Spring Boot Actuator expõe métricas de cache automaticamente.

## 📄 application.yml

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics
```

Com **Micrometer**, é possível monitorar:

- Cache hits
- Cache misses
- Tempo médio de acesso
- Taxa de acerto

Monitoramento é essencial para validar se o cache está realmente trazendo benefício.

---

# 🧰 Boas Práticas

✔️ Cacheie apenas operações custosas ou muito acessadas  
✔️ Configure TTL e tamanho máximo  
✔️ Prefira Redis ou Caffeine em produção  
✔️ Evite cachear dados extremamente voláteis  
✔️ Combine cache com monitoramento  
✔️ Teste cenários de expiração e invalidação  
✔️ Não utilize cache como substituto de consistência transacional

---

# 📚 Referências

- https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#cache
- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching
- https://www.baeldung.com/spring-cache-tutorial
- https://www.baeldung.com/guide-to-spring-cache

---

# ✅ Conclusão

O Spring Cache permite desacoplar a lógica de caching do código de negócio, oferecendo uma abstração poderosa para múltiplos backends.

Quando bem configurado — com TTL, limites, monitoramento e estratégia de invalidação — o cache melhora significativamente a performance e a escalabilidade da aplicação.

Caching não é apenas otimização. É uma decisão arquitetural.

---

<p align="center">
<b>Finalizada a Caching no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="19-agendamento-async.md">Agendamento de Tarefas e Execução Assíncrona no Spring</a>
</p>
