# 🌐 Web e Spring MVC

O **Spring MVC** é o principal módulo do Spring Framework para construção de aplicações web baseadas no padrão **Model-View-Controller (MVC)**. Ele fornece uma arquitetura bem definida, extensível e altamente integrada com os demais módulos do ecossistema Spring, como validação, segurança, injeção de dependência e persistência.

Este capítulo aborda os conceitos fundamentais, a arquitetura interna, o fluxo de requisição e os principais recursos utilizados no desenvolvimento de aplicações web com Spring MVC.

---

## 🧠 Arquitetura do Spring MVC

O Spring MVC é estruturado em torno do **DispatcherServlet**, que atua como um **Front Controller**, responsável por:

- Receber todas as requisições HTTP  
- Delegar o processamento aos controllers apropriados  
- Resolver views  
- Retornar a resposta ao cliente  

### 🔹 Componentes principais

1. **DispatcherServlet** — Ponto de entrada das requisições  
2. **HandlerMapping** — Mapeia URLs para métodos dos controllers  
3. **HandlerAdapter** — Invoca o método correto do controller  
4. **HandlerInterceptor** — Executa lógica antes e depois do controller  
5. **ViewResolver** — Resolve nomes de views para templates  
6. **View** — Renderiza a resposta (HTML, JSON, etc.)  

---

## 🔁 Fluxo de Requisição

O fluxo típico de uma requisição no Spring MVC é:

1. O cliente envia uma requisição HTTP.  
2. O `DispatcherServlet` recebe a requisição.  
3. O `HandlerMapping` localiza o controller/método correspondente.  
4. O `HandlerAdapter` executa o método do controller.  
5. O controller retorna:
   - Um nome de view, ou  
   - Um objeto de resposta, ou  
   - Um `ResponseEntity`.  
6. O `ViewResolver` resolve a view (se necessário).  
7. A resposta é renderizada e enviada ao cliente.  

---

## 🧩 Controllers e Mapeamento de Rotas

### 🔹 `@Controller` e `@RestController`

- `@Controller` — Usado para aplicações MVC tradicionais com views.  
- `@RestController` — Combina `@Controller` + `@ResponseBody`, retornando diretamente o corpo da resposta (JSON/XML).

```java
@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    @GetMapping
    public List<UsuarioDTO> listar() {
        return service.listar();
    }
}
```

---

## 🗺️ Mapeamento de Requisições

### 🔹 Anotações de mapeamento

- `@RequestMapping` — Mapeamento genérico  
- `@GetMapping`  
- `@PostMapping`  
- `@PutMapping`  
- `@DeleteMapping`  
- `@PatchMapping`  

```java
@GetMapping("/{id}")
public UsuarioDTO buscar(@PathVariable Long id) {
    return service.buscarPorId(id);
}
```

---

## 🧾 Binding de Parâmetros

O Spring converte automaticamente dados da requisição para parâmetros do método.

### 🔹 Tipos de binding

- `@PathVariable`  
- `@RequestParam`  
- `@RequestBody`  
- `@RequestHeader`  
- `@CookieValue`  

```java
@PostMapping
public ResponseEntity<Void> criar(@RequestBody @Valid UsuarioDTO dto) {
    service.criar(dto);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}
```

---

## 🧪 Validação Integrada

A validação é acionada automaticamente quando se utiliza:

- `@Valid`  
- `@Validated`  

```java
@PostMapping
public ResponseEntity<Void> criar(@RequestBody @Valid UsuarioDTO dto) {
    return ResponseEntity.ok().build();
}
```

Erros de validação resultam em exceções como `MethodArgumentNotValidException`.

---

## 🧩 Retornos de Controllers

Os métodos dos controllers podem retornar:

- Objetos simples → convertidos automaticamente em JSON/XML (via Jackson)  
- `ResponseEntity<T>` → permite controlar status, headers e corpo  
- `String` → nome da view  
- `ModelAndView` → combina modelo e view  

```java
@GetMapping("/{id}")
public ResponseEntity<UsuarioDTO> buscar(@PathVariable Long id) {
    return ResponseEntity.ok(service.buscarPorId(id));
}
```

---

## 🎨 Views e Template Engines

Em aplicações MVC tradicionais, o Spring suporta mecanismos de template como:

- Thymeleaf (padrão no Spring Boot)  
- JSP  
- FreeMarker  
- Mustache  

### 🔹 Exemplo com Thymeleaf

```java
@GetMapping("/form")
public String formulario(Model model) {
    model.addAttribute("usuario", new UsuarioDTO());
    return "usuario/form";
}
```

---

## 🧱 Interceptadores (HandlerInterceptor)

Interceptadores permitem executar lógica antes e depois da execução do controller.

```java
public class AutenticacaoInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        // lógica antes do controller
        return true;
    }
}
```

### 🔹 Registro

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AutenticacaoInterceptor());
    }
}
```

---

## 🧠 Argument Resolvers

Os `HandlerMethodArgumentResolver` permitem criar parâmetros customizados nos métodos dos controllers.

```java
public class UsuarioAutenticadoArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().equals(Usuario.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) {
        // lógica para resolver usuário autenticado
        return usuario;
    }
}
```

### 🔹 Registro

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new UsuarioAutenticadoArgumentResolver());
    }
}
```

---

## 🛡️ Tratamento Global de Exceções

O Spring MVC permite tratamento centralizado de exceções via `@ControllerAdvice`.

```java
@ControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Object> tratarValidacao(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest().body("Erro de validação");
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Object> tratarErroGeral(Exception ex) {
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Erro interno");
    }
}
```

---

## ⚙️ Configuração com Spring Boot

No Spring Boot, a maior parte da configuração web é automática, baseada em:

- `@SpringBootApplication`  
- Auto-configuração  
- Convenções sobre configuração  

Personalizações podem ser feitas implementando `WebMvcConfigurer`.

---

## 🧱 Boas Práticas

- Separe controllers REST de controllers MVC (views).  
- Utilize DTOs para comunicação externa.  
- Centralize tratamento de erros com `@ControllerAdvice`.  
- Use `ResponseEntity` para controle explícito da resposta.  
- Evite lógica de negócio nos controllers; delegue para serviços.  
- Utilize validação automática com `@Valid`.  

---

## 🧩 Conclusão do Capítulo

O Spring MVC fornece uma arquitetura robusta, extensível e madura para desenvolvimento web. Sua integração com validação, segurança, injeção de dependência e persistência torna-o um dos frameworks web mais utilizados no ecossistema Java.

Compreender o funcionamento interno do fluxo de requisição, o papel do `DispatcherServlet` e o uso adequado de controllers e interceptadores é essencial para construir aplicações web escaláveis, seguras e bem organizadas.
