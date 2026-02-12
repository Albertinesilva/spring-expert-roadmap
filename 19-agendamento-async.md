# 19 — Agendamento de Tarefas e Execução Assíncrona no Spring

## 🎯 Objetivo

Este capítulo apresenta como o Spring Boot permite:

- Agendar tarefas automaticamente
- Executar métodos de forma assíncrona
- Melhorar a performance e escalabilidade de aplicações
- Integrar com schedulers avançados quando necessário (Quartz, Cron, etc.)

A combinação de **agendamento** e **execução assíncrona** é essencial em sistemas distribuídos ou com alto volume de requisições.

---

# ⏱️ Agendamento de Tarefas

## 1️⃣ Ativando o Agendamento

Adicione a anotação `@EnableScheduling` em uma classe de configuração ou na classe principal:

```java
@SpringBootApplication
@EnableScheduling
public class MeuApp {

    public static void main(String[] args) {
        SpringApplication.run(MeuApp.class, args);
    }
}
```

---

## 2️⃣ Utilizando `@Scheduled`

A anotação `@Scheduled` permite configurar tarefas agendadas automaticamente.

### 🔹 Execução em Intervalo Fixo (fixedRate)

```java
@Scheduled(fixedRate = 5000)
public void tarefaFixa() {
    System.out.println("Executando a cada 5 segundos");
}
```

- `fixedRate`: intervalo entre o **início** de cada execução
- Pode iniciar nova execução mesmo se a anterior ainda estiver em andamento

---

### 🔹 Execução com Atraso (fixedDelay)

```java
@Scheduled(fixedDelay = 5000)
public void tarefaComDelay() {
    System.out.println("Executando 5s após terminar a execução anterior");
}
```

- `fixedDelay`: intervalo contado após o **término** da execução anterior

---

### 🔹 Expressão Cron

```java
@Scheduled(cron = "0 0 12 * * ?")
public void tarefaDiaria() {
    System.out.println("Executa todos os dias ao meio-dia");
}
```

Formato padrão:

```
segundo minuto hora dia-do-mês mês dia-da-semana
```

#### Exemplos comuns:

| Expressão       | Frequência          |
| --------------- | ------------------- |
| `0 0 * * * *`   | A cada hora cheia   |
| `0 */5 * * * *` | A cada 5 minutos    |
| `0 0 0 1 * *`   | Primeiro dia do mês |

---

## 3️⃣ Parametrizando via Propriedades

### 📄 application.yml

```yaml
app:
  task:
    fixedRate: 5000
```

### 🧩 Uso no código

```java
@Scheduled(fixedRateString = "${app.task.fixedRate}")
public void tarefaParametrizada() {
    System.out.println("Executando com valor configurável");
}
```

Permite alterar a frequência **sem recompilar a aplicação**.

---

# ⚡ Execução Assíncrona

## 1️⃣ Ativando o Suporte Async

```java
@SpringBootApplication
@EnableAsync
public class MeuApp {
}
```

---

## 2️⃣ Utilizando `@Async`

```java
@Async
public void processarRelatorio() {
    System.out.println("Processando relatório em background...");
}
```

### ⚠️ Importante

- Métodos `@Async` devem ser chamados de **outro bean**
- Self-invocation não funciona (não passa pelo proxy)
- Métodos não podem ser `private`

---

## 3️⃣ Retorno Assíncrono com `CompletableFuture`

```java
@Async
public CompletableFuture<String> calcular() {
    return CompletableFuture.completedFuture("Resultado");
}
```

Permite composição:

```java
calcular()
    .thenApply(valor -> valor.toUpperCase())
    .thenAccept(System.out::println);
```

Ideal para workflows paralelos e encadeamento de tarefas.

---

# 🧵 Executor Personalizado

Por padrão, o Spring usa um executor simples. Para maior controle:

```java
@Bean("taskExecutor")
public Executor taskExecutor() {

    ThreadPoolTaskExecutor executor =
        new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(5);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(25);
    executor.setThreadNamePrefix("Async-");
    executor.initialize();

    return executor;
}
```

Utilizando:

```java
@Async("taskExecutor")
public void tarefaComExecutorCustomizado() {
    System.out.println("Executando com pool customizado");
}
```

Permite controle de:

- Número de threads
- Fila de execução
- Performance
- Consumo de recursos

---

# 🔄 Combinando `@Scheduled` e `@Async`

```java
@Scheduled(fixedRate = 5000)
@Async
public void tarefaAgendadaAssincrona() {
    System.out.println("Executando em paralelo a cada 5 segundos");
}
```

Útil para tarefas longas que não devem bloquear o scheduler principal.

---

# 🧠 Schedulers Avançados

Em ambientes distribuídos ou com múltiplas instâncias:

## 🔹 Quartz

- Suporte a persistência
- Jobs distribuídos
- Reexecução após falhas

## 🔹 ShedLock

Evita execução duplicada de tarefas agendadas em clusters.

Exemplo de uso comum:

```java
@SchedulerLock(name = "minhaTarefa")
@Scheduled(cron = "0 0 * * * *")
public void tarefaDistribuida() {
}
```

Ideal para aplicações em Kubernetes ou múltiplas instâncias.

---

# ⚠️ Limitações e Considerações

- `@Async` não funciona com self-invocation
- Métodos `private` não são interceptados
- Tarefas devem ser **idempotentes**
- Em cluster, pode ocorrer execução duplicada sem controle distribuído
- Monitorar uso de threads é essencial

---

# 🧰 Boas Práticas

✔️ Use `@Scheduled` para tarefas simples e internas  
✔️ Externalize frequência no `application.yml`  
✔️ Configure executor customizado em produção  
✔️ Combine `@Async` com `CompletableFuture` para fluxos complexos  
✔️ Em ambientes distribuídos, utilize Quartz ou ShedLock  
✔️ Monitore métricas e consumo de recursos  
✔️ Teste cenários de falha e reinício

---

# 📊 Monitoramento

Recomenda-se integrar com:

- Spring Boot Actuator
- Micrometer
- Logs estruturados
- Monitoramento de ThreadPool

Isso garante controle sobre filas, threads ativas e tempo de execução.

---

# 📚 Referências

- https://docs.spring.io/spring-framework/docs/current/reference/html/scheduling.html
- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.scheduling
- https://www.baeldung.com/spring-scheduled-tasks
- https://www.baeldung.com/spring-async

---

# ✅ Conclusão

O Spring oferece mecanismos robustos para **agendamento de tarefas** e **execução assíncrona**, permitindo melhor uso de recursos e maior escalabilidade.

Quando combinados corretamente — com configuração adequada de executores, controle distribuído e monitoramento — esses recursos tornam aplicações mais eficientes, resilientes e preparadas para ambientes modernos de produção.

---

<p align="center">
<b>Finalizada a Agendamento de Tarefas e Execução Assíncrona no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="20-observabilidade.md">Observabilidade (Metrics, Tracing e Logging) no Spring</a>
</p>
