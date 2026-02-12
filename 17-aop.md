# 17 — AOP (Programação Orientada a Aspectos) no Spring

## 🎯 Objetivo

Este capítulo apresenta os conceitos, fundamentos e uso prático da **Programação Orientada a Aspectos (AOP)** no Spring, permitindo a implementação de funcionalidades transversais como:

- Logging
- Segurança
- Monitoramento
- Auditoria
- Controle transacional
- Tratamento de exceções

---

# 🧠 O que é AOP?

AOP (Aspect-Oriented Programming) é um paradigma que visa separar **preocupações transversais** (_cross-cutting concerns_) da lógica principal do negócio.

Preocupações transversais são funcionalidades que se repetem em várias partes da aplicação, como:

- Log de métodos
- Verificação de permissões
- Medição de tempo de execução
- Gerenciamento de transações
- Auditoria de ações

Sem AOP, essas responsabilidades acabam se espalhando pelo código, tornando-o mais acoplado e difícil de manter.

---

# 🔍 Conceitos Fundamentais

| Conceito       | Descrição                                            |
| -------------- | ---------------------------------------------------- |
| **Aspect**     | Classe que contém a lógica transversal               |
| **Join Point** | Ponto específico da execução (ex: chamada de método) |
| **Advice**     | Ação executada em um Join Point                      |
| **Pointcut**   | Expressão que define onde o Advice será aplicado     |
| **Weaving**    | Processo de aplicar aspectos ao código               |
| **Proxy**      | Objeto gerado pelo Spring para interceptação         |

---

# ⚙️ Tipos de Advice

| Tipo              | Descrição                                  |
| ----------------- | ------------------------------------------ |
| `@Before`         | Executa antes do método                    |
| `@After`          | Executa após o método (sempre)             |
| `@AfterReturning` | Executa após retorno com sucesso           |
| `@AfterThrowing`  | Executa após exceção                       |
| `@Around`         | Envolve completamente a execução do método |

---

# 📦 Dependência Maven

Para utilizar AOP no Spring Boot:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

---

# 🧩 Criando um Aspect

Exemplo simples de logging:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.exemplo.servico..*(..))")
    public void logAntes(JoinPoint joinPoint) {
        System.out.println("Chamando método: "
            + joinPoint.getSignature().getName());
    }
}
```

---

# 🎯 Expressões Pointcut

As expressões definem onde o aspecto será aplicado.

## 🔹 Por pacote

```java
execution(* com.exemplo.servico..*(..))
```

## 🔹 Por anotação

```java
@annotation(org.springframework.transaction.annotation.Transactional)
```

## 🔹 Por classe

```java
within(com.exemplo.controller..*)
```

## 🔹 Por nome de método

```java
execution(* *Service.salvar*(..))
```

---

# 🔄 Exemplo com `@Around`

O `@Around` permite controlar totalmente a execução do método.

```java
@Around("execution(* com.exemplo.servico..*(..))")
public Object medirTempo(ProceedingJoinPoint pjp) throws Throwable {

    long inicio = System.currentTimeMillis();

    Object retorno = pjp.proceed();

    long fim = System.currentTimeMillis();

    System.out.println("Tempo de execução: "
        + (fim - inicio) + " ms");

    return retorno;
}
```

---

# 🏷️ Criando uma Anotação Customizada

Criar anotações próprias torna o código mais claro e expressivo.

## 🔹 Anotação

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Logavel {
}
```

## 🔹 Aspecto usando a anotação

```java
@Aspect
@Component
public class LogavelAspect {

    @Before("@annotation(Logavel)")
    public void logMetodoAnotado(JoinPoint joinPoint) {
        System.out.println("Método anotado executado: "
            + joinPoint.getSignature().getName());
    }
}
```

---

# 🧪 AOP e Transações

O Spring utiliza AOP internamente para implementar:

```java
@Transactional
public void processarPedido() {
    // lógica com rollback automático
}
```

A transação é aplicada via proxy e aspecto, sem necessidade de código explícito de controle transacional.

---

# ⚠️ Limitações do AOP no Spring

- Apenas métodos **públicos** são interceptados por padrão
- Chamadas internas (_self-invocation_) não passam pelo proxy
- Não funciona em métodos `final`, `private` ou `static`
- Atua apenas em beans gerenciados pelo Spring

Essas limitações existem porque o Spring AOP utiliza proxies baseados em JDK ou CGLIB.

---

# 🔐 Ordem de Execução de Aspectos

Quando múltiplos aspectos são aplicados, pode-se definir prioridade:

```java
@Aspect
@Order(1)
@Component
public class PrimeiroAspect {
}
```

```java
@Aspect
@Order(2)
@Component
public class SegundoAspect {
}
```

Menor valor → maior prioridade de execução.

---

# 🧩 Exemplo Prático Completo

## 🔹 Serviço

```java
@Service
public class PedidoService {

    @Logavel
    public void criarPedido() {
        System.out.println("Pedido criado!");
    }
}
```

## 🔹 Aspecto

```java
@Aspect
@Component
public class AuditoriaAspect {

    @Before("@annotation(Logavel)")
    public void auditar(JoinPoint jp) {
        System.out.println("Auditando método: "
            + jp.getSignature().getName());
    }
}
```

---

# 🧰 Boas Práticas

✔️ Use AOP apenas para preocupações transversais  
✔️ Evite colocar lógica de negócio dentro de aspectos  
✔️ Prefira anotações customizadas para maior clareza  
✔️ Documente bem seus pointcuts  
✔️ Teste cenários com e sem proxy  
✔️ Utilize `@Order` quando múltiplos aspectos forem aplicados  
✔️ Evite aspectos excessivamente genéricos

---

# 📚 Referências

- https://docs.spring.io/spring-framework/reference/core/aop.html
- https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-aop
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/AspectJExpressionPointcut.html

---

# ✅ Conclusão

A AOP no Spring é uma ferramenta poderosa para separar responsabilidades transversais da lógica principal, promovendo código mais limpo, reutilizável e de fácil manutenção.

Seu uso consciente melhora significativamente a qualidade arquitetural da aplicação e contribui para sistemas mais organizados e escaláveis.

---

<p align="center">
<b>Finalizada a AOP (Programação Orientada a Aspectos) no Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="18-caching.md">Caching no Spring</a>
</p>
