# 🗂️ APIs REST Maturas

APIs REST modernas vão além de simplesmente expor endpoints HTTP. Elas devem ser **autodescritivas**, **versionadas**, **seguras**, **observáveis**, **consistentes** e **evolutivas**. No ecossistema Spring, isso é alcançado por meio de boas práticas arquiteturais, padrões de contrato e ferramentas como Spring MVC, Spring HATEOAS, Spring Data REST e SpringDoc/OpenAPI.

Este capítulo apresenta os fundamentos técnicos para projetar, implementar e evoluir APIs REST maduras em ambientes corporativos e distribuídos.

---

## 🧠 O que caracteriza uma API REST madura?

Uma API REST madura:

- Segue corretamente os verbos HTTP (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`).
- Utiliza códigos de status semânticos.
- Possui contratos bem definidos e documentados.
- Suporta versionamento e evolução sem quebrar clientes.
- É segura, observável e resiliente.
- Usa hipermídia quando apropriado (HATEOAS).
- Possui padronização de erros (RFC 7807 – Problem Details).

---

## 🔹 Spring HATEOAS

### 🧩 Conceito

HATEOAS (*Hypermedia as the Engine of Application State*) é um princípio REST que afirma que a API deve fornecer links que orientam o cliente sobre quais ações são possíveis a partir do estado atual.

No Spring, isso é suportado via **Spring HATEOAS**.

---

### 🔧 Exemplo básico

```java
@GetMapping("/usuarios/{id}")
public EntityModel<Usuario> buscar(@PathVariable Long id) {
    Usuario usuario = service.buscarPorId(id);

    return EntityModel.of(usuario,
        linkTo(methodOn(UsuarioController.class).buscar(id)).withSelfRel(),
        linkTo(methodOn(UsuarioController.class).listar()).withRel("usuarios")
    );
}
```

### 📥 Resposta JSON

```json
{
  "id": 1,
  "nome": "Albert",
  "email": "albert@email.com",
  "_links": {
    "self": { "href": "/usuarios/1" },
    "usuarios": { "href": "/usuarios" }
  }
}
```

---

### 🎯 Quando usar HATEOAS?

- Em APIs públicas e navegáveis.
- Em sistemas com múltiplos clientes e fluxos dinâmicos.
- Quando a API deve guiar o comportamento do consumidor.

⚠️ Não é obrigatório para toda API REST, mas é valioso em domínios complexos.

---

## 🔹 Spring Data REST

### 🧩 Conceito

O Spring Data REST expõe automaticamente repositórios JPA como endpoints REST, com suporte a:

- CRUD completo.
- Paginação e ordenação.
- Filtros via query parameters.
- HATEOAS automático.
- HAL como formato padrão.

---

### 🔧 Exemplo

```java
@RepositoryRestResource(path = "usuarios")
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
}
```

### 📌 Endpoints gerados automaticamente

- `GET /usuarios`
- `POST /usuarios`
- `GET /usuarios/{id}`
- `PUT /usuarios/{id}`
- `DELETE /usuarios/{id}`

---

### ⚠️ Quando NÃO usar Spring Data REST?

- Quando a API exige forte controle de contratos.
- Quando há lógica de negócio complexa.
- Quando DTOs e camadas de serviço são essenciais.
- Em APIs públicas de alto impacto.

Spring Data REST é mais adequado para administração interna, protótipos ou backoffice.

---

## 🔹 Documentação de API (SpringDoc / OpenAPI)

### 🧩 Conceito

A especificação OpenAPI (antiga Swagger) descreve formalmente contratos REST. No Spring Boot moderno, o padrão é usar SpringDoc.

---

### 🔧 Dependência Maven

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.x.x</version>
</dependency>
```

---

### 🔧 Acesso à documentação

- `/swagger-ui.html`
- `/v3/api-docs`

---

### 🔧 Anotações comuns

```java
@Operation(summary = "Buscar usuário por ID", description = "Retorna os dados de um usuário existente")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Usuário encontrado"),
    @ApiResponse(responseCode = "404", description = "Usuário não encontrado")
})
@GetMapping("/{id}")
public UsuarioDTO buscar(@PathVariable Long id) {
    ...
}
```

---

### 🎯 Boas práticas de documentação

- Documentar todos os endpoints públicos.
- Descrever códigos de erro e payloads.
- Versionar documentação junto com a API.
- Manter contratos atualizados automaticamente.

---

## 🔹 Versionamento, Padrões de Resposta e Contratos

### 🧭 Versionamento de APIs

#### 🔸 Estratégias comuns

**Via URL**
```
/api/v1/usuarios
/api/v2/usuarios
```

**Via Header**
```
Accept: application/vnd.minhaapi.v1+json
```

**Via Parâmetro**
```
/usuarios?version=1
```

✅ Recomendação: versionamento por URL ou media type.

---

## 📦 Padrões de Resposta

### 🔸 Envelope de resposta (opcional)

```json
{
  "data": { ... },
  "meta": { "timestamp": "...", "version": "v1" },
  "links": { ... }
}
```

⚠️ Em APIs REST modernas, muitas vezes é preferível retornar diretamente o recurso, exceto quando há necessidade de metadados.

---

## 🚨 Padrão de Erros – RFC 7807 (Problem Details)

```json
{
  "type": "https://api.exemplo.com/erros/recurso-nao-encontrado",
  "title": "Recurso não encontrado",
  "status": 404,
  "detail": "Usuário com ID 99 não foi encontrado",
  "instance": "/usuarios/99"
}
```

---

### 🔧 Implementação no Spring

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public ProblemDetail handle(RecursoNaoEncontradoException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        problem.setTitle("Recurso não encontrado");
        problem.setDetail(ex.getMessage());
        return problem;
    }
}
```

---

## 📐 Contratos, DTOs e Isolamento de Camadas

### 🔸 Por que usar DTOs?

- Evitar exposição de entidades.
- Controlar contratos externos.
- Reduzir acoplamento.
- Permitir evolução sem quebrar clientes.

```java
public record UsuarioDTO(Long id, String nome, String email) {}
```

---

### 🔸 Mapeamento com MapStruct

```java
@Mapper(componentModel = "spring")
public interface UsuarioMapper {
    UsuarioDTO toDTO(Usuario usuario);
    Usuario toEntity(UsuarioDTO dto);
}
```

---

## 🧪 Testes de Contrato

APIs maduras devem possuir testes de contrato, garantindo que consumidores e provedores permaneçam compatíveis.

Ferramenta recomendada: **Spring Cloud Contract**.

---

## 🛡️ Segurança em APIs REST

Boas APIs REST:

- Não confiam em sessão (stateless).
- Utilizam JWT, OAuth2 ou mTLS.
- Validam entrada rigorosamente.
- Possuem controle de acesso por escopo/role.
- Registram tentativas suspeitas.

Esses temas são detalhados no capítulo de Segurança.

---

## 📊 Observabilidade

APIs maduras devem ser:

- Monitoráveis (métricas).
- Rastreáveis (tracing distribuído).
- Logáveis (structured logging).
- Testáveis em produção (health checks).

Esses temas são aprofundados no capítulo de Observabilidade.

---

## 🧱 Conclusão do Capítulo

Projetar APIs REST maduras exige muito mais do que criar controladores e expor endpoints. É necessário compreender contratos, versionamento, erros, segurança, observabilidade, hipermídia e evolução contínua.

O Spring fornece um ecossistema completo para isso — mas cabe ao desenvolvedor dominar seus mecanismos e aplicar boas práticas arquiteturais para construir APIs resilientes, escaláveis e sustentáveis ao longo do tempo.
