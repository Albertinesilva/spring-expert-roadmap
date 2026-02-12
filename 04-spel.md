# 🔍 Spring Expression Language (SpEL)

A **Spring Expression Language (SpEL)** é uma linguagem de expressões poderosa, dinâmica e fortemente integrada ao ecossistema Spring. Ela permite consultar, manipular e avaliar objetos em tempo de execução, sendo amplamente utilizada em recursos como:

- Segurança (`@PreAuthorize`, `@PostAuthorize`)
- Cache (`@Cacheable`, `@CacheEvict`)
- Configuração (`@Value`)
- Eventos
- Integração
- AOP

Mais do que uma linguagem de conveniência, o SpEL é um mecanismo essencial para tornar o comportamento da aplicação **declarativo, dinâmico e orientado a contexto**.

---

## 🧠 Propósito e Papel Arquitetural

O SpEL foi projetado para:

- Permitir decisões dinâmicas baseadas em estado de execução.
- Desacoplar lógica condicional do código imperativo.
- Viabilizar configurações avançadas sem alterar o código-fonte.
- Integrar metadados, contexto de segurança e parâmetros de método em expressões.

Ao contrário de linguagens de template, o SpEL atua diretamente sobre objetos Java, métodos, coleções, mapas e até contextos externos, como o `SecurityContext`.

---

## 🔧 Sintaxe Básica

Uma expressão SpEL é normalmente delimitada por:

```text
#{ ... }
```

Exemplo simples:

```java
@Value("#{2 + 2}")
private int resultado; // 4
```

---

## 🧩 Acesso a Propriedades e Métodos

```java
@Value("#{usuario.nome}")
private String nome;

@Value("#{usuario.getIdade()}")
private int idade;
```

Ou em anotações como:

```java
@Cacheable(value = "produtos", key = "#id")
public Produto buscarPorId(Long id) { ... }
```

---

## 📦 Operadores

### 🔸 Operadores aritméticos

`+, -, *, /, %`

```text
#{10 * 2} → 20
#{10 / 2} → 5
```

---

### 🔸 Operadores relacionais

`==, !=, <, >, <=, >=`

Alternativas textuais: `eq, ne, lt, gt, le, ge`

```text
#{#idade >= 18}
```

---

### 🔸 Operadores lógicos

`and, or, not`

```text
#{#ativo and #verificado}
#{not #expirado}
```

---

## 🧭 Operador Ternário

```text
#{#idade >= 18 ? 'Maior' : 'Menor'}
```

---

## 📚 Acesso a Coleções, Mapas e Arrays

### 🔸 Índices

```text
#{lista[0]}
#{mapa['chave']}
```

Resultado: lista contendo apenas os nomes.

---

### 🔸 Seleção

```text
#{usuarios.?[idade >= 18]}
```

Resultado: sublista com usuários maiores de idade.

---

## 🧠 Variáveis de Contexto

No contexto de métodos interceptados, como em cache ou segurança, o SpEL permite acessar:

- Parâmetros de método:  
  `#id`, `#usuario`
- Retorno:  
  `#result`
- Exceções:  
  `#exception`

Exemplo:

```java
@CachePut(value = "usuarios", key = "#result.id")
public Usuario salvar(Usuario usuario) { ... }
```

---

## 🔐 SpEL e Spring Security

O SpEL é amplamente utilizado em anotações de segurança:

```java
@PreAuthorize("hasRole('ADMIN') and #id == authentication.principal.id")
public Usuario buscar(Long id) { ... }
```

Elementos disponíveis:

- `authentication`
- `principal`
- `hasRole()`, `hasAuthority()`
- Beans registrados no contexto (`@beanName`)

---

## 🚀 SpEL e Cache

Exemplo típico:

```java
@Cacheable(value = "produtos", key = "#categoria + ':' + #pagina")
public List<Produto> listar(String categoria, int pagina) { ... }
```

Ou:

```java
@CacheEvict(value = "produtos", allEntries = true, condition = "#resultado > 0")
public void atualizarEstoque(int resultado) { ... }
```

---

## ⚙️ SpEL em Configurações (`@Value`)

```java
@Value("#{systemProperties['user.home']}")
private String home;

@Value("#{T(java.lang.Math).random() * 100}")
private double valorAleatorio;
```

### 🔸 Acesso a tipos estáticos

```java
#{T(java.lang.Math).PI}
```

---

## 🧬 Avaliação Programática de Expressões

Além do uso declarativo, o SpEL pode ser avaliado programaticamente:

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("1 + 2");
int valor = exp.getValue(Integer.class); // 3
```

Com contexto:

```java
StandardEvaluationContext context = new StandardEvaluationContext(objeto);
Expression exp = parser.parseExpression("nome");
String nome = exp.getValue(context, String.class);
```

---

## ⚠️ Considerações de Segurança

Como o SpEL permite execução de métodos e acesso a tipos, ele deve ser utilizado com cautela, especialmente quando:

- Expressões são construídas a partir de entradas externas.
- O contexto contém objetos sensíveis.

O Spring fornece mecanismos de restrição por meio de:

- Contextos de avaliação controlados.
- Limitação de resolvers e acessos permitidos.

---

## 🧱 Boas Práticas

- Utilize SpEL apenas quando a lógica realmente precisar ser dinâmica.
- Prefira expressões simples e legíveis.
- Evite expressões complexas que dificultem manutenção.
- Documente expressões relevantes em código crítico.
- Não exponha o SpEL a entradas de usuários sem sanitização.

---

## 🧩 Conclusão do Capítulo

O Spring Expression Language é um dos recursos mais poderosos — e muitas vezes subestimados — do ecossistema Spring. Ele permite:

- Tornar configurações declarativas mais expressivas.
- Aplicar regras dinâmicas em segurança, cache, eventos e integração.
- Reduzir acoplamento entre lógica de negócio e infraestrutura.

Dominar o SpEL é compreender como o Spring transforma metadados em comportamento dinâmico, mantendo o código limpo, modular e altamente configurável.

---

<p align="center">
<b>Finalizada a Spring Expression Language (SpEL)! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="05-validacao.md">Validação (Spring Validation / Jakarta Validation)</a>
</p>
