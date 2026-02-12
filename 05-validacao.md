# 🛡️ Validação (Spring Validation / Jakarta Validation)

A validação de dados é um elemento essencial na construção de aplicações confiáveis, seguras e consistentes. No ecossistema Spring, a validação é baseada principalmente na especificação **Jakarta Bean Validation** (antiga JSR 303/380), com suporte nativo e integração profunda ao Spring Framework por meio do módulo **Spring Validation**.

Mais do que verificar campos, a validação no Spring:

- Garante integridade de dados antes da persistência.
- Protege o sistema contra entradas inválidas ou maliciosas.
- Centraliza regras de negócio estruturais.
- Integra-se de forma transparente com camadas web, de serviço e de persistência.

---

## 🧠 Arquitetura da Validação no Spring

A arquitetura de validação envolve três componentes principais:

1. **Anotações de restrição** (constraints), como `@NotNull`, `@Size`, `@Email`.
2. **Validadores**, responsáveis por aplicar a lógica das restrições.
3. **Motor de validação** (Bean Validation provider), como Hibernate Validator.

O Spring atua como integrador, acionando automaticamente a validação em pontos estratégicos, como:

- Binding de requisições HTTP.
- Execução de métodos anotados com `@Validated`.
- Persistência via JPA/Hibernate.

---

## 📦 Dependências e Ativação

No Spring Boot, a validação é ativada automaticamente ao incluir o starter:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Esse starter inclui o Hibernate Validator como implementação padrão da especificação Jakarta Validation.

---

## 🧩 Anotações de Validação Comuns

### 🔸 Restrições básicas

- `@NotNull` — não permite valor nulo.
- `@NotBlank` — não permite valor nulo ou string vazia.
- `@NotEmpty` — não permite valor nulo ou coleção vazia.
- `@Size(min, max)` — tamanho mínimo e máximo.
- `@Min`, `@Max` — valores mínimos e máximos.
- `@Positive`, `@Negative`, `@PositiveOrZero`, `@NegativeOrZero`.
- `@Email` — valida formato de e-mail.
- `@Pattern(regexp = "...")` — valida por expressão regular.
- `@Past`, `@PastOrPresent`, `@Future`, `@FutureOrPresent`.

---

### 🔸 Exemplo

```java
public class UsuarioDTO {

    @NotBlank
    private String nome;

    @Email
    private String email;

    @Min(18)
    private int idade;

    // getters e setters
}
```

---

## 🌐 Validação em Controllers (Spring MVC / WebFlux)

A validação é automaticamente acionada quando se utiliza:

- `@Valid` (Jakarta Validation)
- `@Validated` (Spring Validation)

### 🔸 Exemplo com `@Valid`

```java
@PostMapping("/usuarios")
public ResponseEntity<Void> criar(@RequestBody @Valid UsuarioDTO dto) {
    // lógica de criação
    return ResponseEntity.ok().build();
}
```

### 🔸 Tratamento de erros

Quando a validação falha, o Spring lança exceções como:

- `MethodArgumentNotValidException`
- `ConstraintViolationException`

Essas exceções podem ser tratadas globalmente via `@ControllerAdvice`.

---

## 🧩 Validação em Métodos de Serviço

Além da camada web, é possível aplicar validação diretamente em métodos de serviço:

```java
@Service
@Validated
public class UsuarioService {

    public void salvar(@Valid UsuarioDTO dto) {
        // lógica de persistência
    }
}
```

Nesse caso, a validação é aplicada por meio de proxies, utilizando AOP, e é executada antes da lógica do método.

---

## 🔐 Validação em Entidades JPA

As anotações de validação também podem ser aplicadas diretamente em entidades:

```java
@Entity
public class Usuario {

    @Id
    @GeneratedValue
    private Long id;

    @NotBlank
    private String nome;

    @Email
    private String email;
}
```

Quando integradas ao JPA/Hibernate, essas validações podem ser executadas automaticamente:

- Antes de `persist()`
- Antes de `merge()`

---

## 🧪 Validação Condicional e Grupos de Validação

### 🔸 Grupos de validação

É possível agrupar validações para cenários distintos:

```java
public interface Criacao {}
public interface Atualizacao {}

public class UsuarioDTO {

    @NotNull(groups = Atualizacao.class)
    private Long id;

    @NotBlank(groups = {Criacao.class, Atualizacao.class})
    private String nome;
}
```

Uso:

```java
@PostMapping
public void criar(@Validated(Criacao.class) @RequestBody UsuarioDTO dto) { }

@PutMapping
public void atualizar(@Validated(Atualizacao.class) @RequestBody UsuarioDTO dto) { }
```

---

## 🧩 Validações Customizadas

Quando as restrições padrão não são suficientes, é possível criar validações personalizadas.

### 🔸 Definição da anotação

```java
@Constraint(validatedBy = DocumentoValidator.class)
@Target({ FIELD, METHOD, PARAMETER, ANNOTATION_TYPE })
@Retention(RUNTIME)
public @interface DocumentoValido {
    String message() default "Documento inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

### 🔸 Implementação do validador

```java
public class DocumentoValidator implements ConstraintValidator<DocumentoValido, String> {

    @Override
    public boolean isValid(String valor, ConstraintValidatorContext context) {
        if (valor == null) return false;
        // lógica de validação
        return valor.matches("\\d{11}");
    }
}
```

---

## 🧠 Validação em Cascata (`@Valid` em Relacionamentos)

É possível validar objetos aninhados:

```java
public class PedidoDTO {

    @Valid
    private ClienteDTO cliente;

    @Valid
    private List<ItemPedidoDTO> itens;
}
```

Nesse caso, a validação percorre toda a árvore de objetos.

---

## ⚠️ Validação e Proxies

Quando a validação é aplicada em métodos (`@Validated`), ela depende de:

- Proxies Spring.
- Interceptores AOP.

Isso implica:

- Validações não são aplicadas em chamadas internas (self-invocation).
- Métodos `private` não são interceptados.
- A validação ocorre apenas em chamadas externas ao proxy.

---

## 🧱 Boas Práticas

- Centralize validações estruturais no nível de DTOs e entidades.
- Use validações customizadas para regras de negócio reutilizáveis.
- Evite lógica de validação complexa diretamente em controllers.
- Utilize grupos de validação para cenários distintos (criação vs atualização).
- Trate erros de validação de forma padronizada via `@ControllerAdvice`.

---

## 🧩 Conclusão do Capítulo

A validação no Spring é um mecanismo essencial para garantir qualidade, segurança e consistência dos dados. Integrada de forma profunda ao ciclo de vida das requisições, métodos e entidades, ela permite:

- Declarar regras de forma clara e reutilizável.
- Reduzir erros em produção.
- Aumentar a confiabilidade do sistema.

Dominar a validação no Spring é compreender como o framework transforma metadados declarativos em garantias concretas de integridade e robustez.

---

<p align="center">
<b>Finalizada a Validação (Spring Validation / Jakarta Validation)! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="06-web-e-spring-mvc.md">Web e Spring MVC</a>
</p>
