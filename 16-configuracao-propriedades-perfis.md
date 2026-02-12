# 16 — Configuração, Propriedades e Perfis no Spring Boot

## 🎯 Objetivo

Este capítulo apresenta como configurar aplicações Spring Boot de forma **flexível**, **segura** e **organizada**, utilizando:

- Arquivos de propriedades (`.properties` / `.yml`)
- Variáveis de ambiente
- Perfis (`@Profile`, `spring.profiles.active`)
- Binding com `@ConfigurationProperties`
- Boas práticas de organização e segurança

---

# 📁 Estrutura de Configuração no Spring Boot

Por padrão, o Spring Boot carrega configurações de múltiplas fontes:

- `application.properties`
- `application.yml`
- `application-{perfil}.properties`
- `application-{perfil}.yml`
- Variáveis de ambiente
- Argumentos de linha de comando

O Spring Boot aplica uma **ordem de precedência**, onde fontes mais específicas sobrescrevem as anteriores.

## 🔢 Ordem de Precedência (Simplificada)

1. Argumentos de linha de comando
2. Variáveis de ambiente
3. Arquivos `application-{perfil}.yml`
4. Arquivo `application.yml`

---

# 🧾 Arquivo `application.properties`

Exemplo:

```properties
server.port=8080
spring.application.name=meu-sistema
spring.datasource.url=jdbc:postgresql://localhost:5432/app
spring.datasource.username=postgres
spring.datasource.password=123456
```

---

# 🧾 Arquivo `application.yml`

Exemplo equivalente em YAML:

```yaml
server:
  port: 8080

spring:
  application:
    name: meu-sistema
  datasource:
    url: jdbc:postgresql://localhost:5432/app
    username: postgres
    password: 123456
```

O formato YAML é geralmente preferido por ser mais legível e hierárquico.

---

# 🔄 Perfis no Spring Boot

Perfis permitem alternar configurações conforme o ambiente:

- `dev`
- `test`
- `prod`
- `local`
- etc.

---

## 🚀 Ativando um Perfil

### Via `application.properties`

```properties
spring.profiles.active=dev
```

### Via variável de ambiente

```bash
export SPRING_PROFILES_ACTIVE=prod
```

### Via linha de comando

```bash
java -jar app.jar --spring.profiles.active=prod
```

---

# 📂 Arquivos por Perfil

Exemplo de estrutura:

- `application-dev.yml`
- `application-test.yml`
- `application-prod.yml`

### Exemplo — `application-dev.yml`

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
```

### Exemplo — `application-prod.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/app
```

---

# 🧩 Uso de `@Profile` em Beans

Permite ativar beans apenas em determinados ambientes.

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource dataSourceDev() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    @Bean
    @Profile("prod")
    public DataSource dataSourceProd() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://prod-db:5432/app")
            .username("prod")
            .password("segredo")
            .build();
    }
}
```

---

# 🔗 Injeção de Propriedades com `@Value`

```java
@Value("${spring.application.name}")
private String appName;
```

Com valor padrão:

```java
@Value("${app.timeout:30}")
private int timeout;
```

Apesar de útil, o uso excessivo de `@Value` não é recomendado para múltiplas propriedades relacionadas.

---

# 🧩 Binding com `@ConfigurationProperties`

Forma recomendada para agrupar propriedades relacionadas.

## 🔹 Exemplo de Propriedades

```yaml
app:
  security:
    jwt-secret: minha-chave
    token-expiration: 3600
```

## 🔹 Classe de Configuração

```java
@Component
@ConfigurationProperties(prefix = "app.security")
public class SecurityProperties {

    private String jwtSecret;
    private long tokenExpiration;

    public String getJwtSecret() {
        return jwtSecret;
    }

    public void setJwtSecret(String jwtSecret) {
        this.jwtSecret = jwtSecret;
    }

    public long getTokenExpiration() {
        return tokenExpiration;
    }

    public void setTokenExpiration(long tokenExpiration) {
        this.tokenExpiration = tokenExpiration;
    }
}
```

Em versões mais antigas do Spring Boot, pode ser necessário usar:

```java
@EnableConfigurationProperties(SecurityProperties.class)
```

---

# 🛡️ Configuração Sensível e Segurança

Nunca versionar:

- Senhas reais
- Tokens
- Chaves privadas
- Secrets

## 🔹 Uso de Variáveis de Ambiente

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

A variável `DB_PASSWORD` deve ser definida no ambiente de execução.

---

# 🧪 Perfis para Testes

## 🔹 `application-test.yml`

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop
```

## 🔹 Ativação Automática em Testes

```java
@SpringBootTest
@ActiveProfiles("test")
class MeuServicoTest {
}
```

---

# ⚙️ Sobrescrita por Linha de Comando

```bash
java -jar app.jar --server.port=9090 --spring.datasource.username=admin
```

Parâmetros informados na linha de comando têm prioridade máxima.

---

# 🧰 Configurações Avançadas

## 🔹 Importação de Múltiplos Arquivos

```yaml
spring:
  config:
    import: classpath:datasource.yml,classpath:security.yml
```

Permite modularizar configurações.

---

## 🔹 Propriedades Customizadas por Módulo

### YAML

```yaml
modulo:
  pagamento:
    timeout: 30
    retries: 3
```

### Classe de Binding

```java
@ConfigurationProperties(prefix = "modulo.pagamento")
public class PagamentoProperties {

    private int timeout;
    private int retries;

    public int getTimeout() {
        return timeout;
    }

    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }

    public int getRetries() {
        return retries;
    }

    public void setRetries(int retries) {
        this.retries = retries;
    }
}
```

---

# 🧠 Boas Práticas

✔️ Prefira `application.yml` para melhor organização  
✔️ Separe configurações por perfil  
✔️ Utilize `@ConfigurationProperties` em vez de muitos `@Value`  
✔️ Nunca exponha segredos em repositórios  
✔️ Utilize variáveis de ambiente em produção  
✔️ Documente propriedades customizadas  
✔️ Evite lógica de negócio baseada diretamente em valores hardcoded

---

# 📚 Referências

- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config
- https://docs.spring.io/spring-framework/reference/core/beans/environment.html
- https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html

---

# ✅ Conclusão

O sistema de configuração do Spring Boot é poderoso e flexível, permitindo aplicações **seguras**, **portáveis** e **fáceis de manter** em diferentes ambientes.

O uso correto de perfis, binding estruturado e boas práticas de segurança é essencial para projetos profissionais e ambientes corporativos.

Configuração bem feita é base para arquitetura sustentável.

---

<p align="center">
<b>Finalizada a Configuração, Propriedades e Perfis no Spring Boot! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="17-aop.md">AOP (Programação Orientada a Aspectos) no Spring</a>
</p>
