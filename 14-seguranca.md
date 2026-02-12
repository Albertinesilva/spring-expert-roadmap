# 🔐 Segurança e Identidade com Spring

A segurança é um pilar central em aplicações modernas. No ecossistema Spring, ela não é tratada como um recurso adicional, mas como uma **infraestrutura transversal**, aplicada por meio de filtros, proxies, interceptores, contexto de segurança e políticas declarativas.

Este capítulo aborda como projetar sistemas seguros, escaláveis e alinhados a padrões modernos como **OAuth 2.1**, **OpenID Connect**, **Zero Trust** e arquiteturas orientadas a identidade.

---

## 🧠 Fundamentos de Segurança no Spring

O **Spring Security** é construído sobre:

- **Filtros HTTP** organizados em uma cadeia (`SecurityFilterChain`)
- **Contexto de Segurança** (`SecurityContextHolder`)
- **Autenticação** (`Authentication`)
- **Autorização** (`GrantedAuthority`)
- **AOP e Proxies** para aplicar segurança declarativa

A segurança é aplicada de forma **declarativa**, mas executada de forma **imperativa** internamente.

---

## 🔐 Spring Security

### 🔹 Arquitetura Interna

```text
HTTP Request
      ↓
Security Filter Chain
      ↓
Authentication Manager
      ↓
Authentication Provider
      ↓
SecurityContextHolder
      ↓
Controller / Service
```

Cada requisição passa por uma cadeia de filtros responsáveis por autenticação, autorização, CSRF, CORS, sessão, logout, entre outros.

---

## ⚙️ Configuração Básica (Spring Boot 3+)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
}
```

---

## 🔑 Autenticação

### 🔸 In-Memory

```java
@Bean
public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
    UserDetails user = User.withUsername("user")
        .password(passwordEncoder.encode("123"))
        .roles("USER")
        .build();

    return new InMemoryUserDetailsManager(user);
}
```

---

### 🔸 JDBC

```java
@Bean
public JdbcUserDetailsManager users(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
}
```

---

### 🔸 JPA Customizado

```java
@Service
public class UsuarioDetailsService implements UserDetailsService {

    private final UsuarioRepository repository;

    public UsuarioDetailsService(UsuarioRepository repository) {
        this.repository = repository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) {
        return repository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado"));
    }
}
```

---

## 🔐 Codificação de Senhas

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

O uso de algoritmos adaptativos como **BCrypt**, **Argon2** ou **PBKDF2** é obrigatório em produção.

---

# 🔑 OAuth2, OpenID Connect e JWT

## 🔹 Conceitos-Chave

- **OAuth 2.1** → Protocolo de autorização
- **OpenID Connect (OIDC)** → Camada de identidade sobre OAuth2
- **JWT (JSON Web Token)** → Token auto-contido para autenticação/autorização

---

## 🔹 Resource Server (API protegida)

```java
@Bean
public SecurityFilterChain apiSecurity(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .oauth2ResourceServer(oauth2 -> oauth2.jwt());

    return http.build();
}
```

---

## 🔹 OAuth2 Client

```java
@Bean
public SecurityFilterChain clientSecurity(HttpSecurity http) throws Exception {
    http
        .oauth2Login(Customizer.withDefaults())
        .oauth2Client(Customizer.withDefaults());

    return http.build();
}
```

---

# 🧭 Spring Authorization Server

Implementação oficial do Spring para servidores OAuth2/OIDC.

## 🔹 Casos de Uso

- Identity Provider (IdP) corporativo
- Autenticação centralizada
- Emissão de tokens JWT
- Gerenciamento de clientes, escopos e consentimento

## 🔹 Configuração Básica

```java
@Bean
public SecurityFilterChain authServerSecurity(HttpSecurity http) throws Exception {
    OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
    return http.build();
}
```

---

# 🛡️ Autorização Declarativa

## 🔹 Com Anotações

```java
@PreAuthorize("hasRole('ADMIN')")
public void metodoAdmin() { }

@PostAuthorize("returnObject.owner == authentication.name")
public Documento buscar(Long id) { }
```

---

## 🔹 SpEL Avançado

```java
@PreAuthorize("hasAuthority('PEDIDO_LEITURA') and #id == principal.id")
public Pedido buscar(Long id) { }
```

---

## 🔹 ABAC (Attribute-Based Access Control)

```java
@PreAuthorize("@verificadorDePermissao.podeAcessar(#pedido, authentication)")
public Pedido buscar(Pedido pedido) { }
```

---

# 🌐 Segurança em APIs e Microserviços

## 🔹 Padrões Arquiteturais

- API Gateway como ponto único de entrada
- OAuth2 + JWT para comunicação entre serviços
- mTLS para tráfego interno
- Zero Trust (nenhuma confiança implícita)

---

## 🔹 Spring Cloud Gateway + Security

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http
        .authorizeExchange(exchanges -> exchanges
            .pathMatchers("/public/**").permitAll()
            .anyExchange().authenticated()
        )
        .oauth2ResourceServer(ServerHttpSecurity.OAuth2ResourceServerSpec::jwt);

    return http.build();
}
```

---

# 🔐 Sessões, CSRF e CORS

## 🔹 CSRF

- Ativado por padrão em aplicações baseadas em sessão
- Desativado em APIs stateless com JWT

```java
http.csrf(csrf -> csrf.disable());
```

---

## 🔹 CORS

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://frontend.app"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);

    return source;
}
```

---

# 🏢 Spring LDAP e Active Directory

## 🔹 Autenticação LDAP

```java
@Bean
public LdapAuthenticationProvider ldapAuthProvider() {
    BindAuthenticator authenticator = new BindAuthenticator(contextSource());
    authenticator.setUserDnPatterns(new String[]{"uid={0},ou=people"});
    return new LdapAuthenticationProvider(authenticator);
}
```

---

## 🔹 Integração com Active Directory

```java
@Bean
public ActiveDirectoryLdapAuthenticationProvider adProvider() {
    return new ActiveDirectoryLdapAuthenticationProvider(
        "dominio.local",
        "ldap://ad.local"
    );
}
```

---

# 🔑 Gestão de Segredos

## 🔹 Spring Cloud Vault

```yaml
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      authentication: TOKEN
      token: ${VAULT_TOKEN}
```

---

## 🔹 Secret Managers Suportados

- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager
- Kubernetes Secrets

---

# 🧪 Testes de Segurança

## 🔹 Mock User

```java
@WithMockUser(username = "admin", roles = {"ADMIN"})
@Test
void acessoAdminPermitido() {
}
```

---

## 🔹 Testando API Protegida

```java
@Test
void deveNegarAcessoSemToken() throws Exception {
    mockMvc.perform(get("/api/protegido"))
        .andExpect(status().isUnauthorized());
}
```

---

## 🔹 Testando Regra de Domínio

```java
@Test
@WithMockUser(username = "user")
void deveNegarAcessoAoPedidoDeOutroUsuario() {
}
```

---

# ⚠️ Armadilhas Comuns

- Métodos internos da mesma classe não passam pelo proxy (self-invocation)
- Uso inconsistente de `ROLE_`
- Exposição de endpoints sensíveis sem autenticação
- Armazenar segredos em código ou repositório
- Não validar corretamente escopos e claims

---

# 🧱 Boas Práticas

- Sempre use **HTTPS**
- Prefira autenticação baseada em **tokens**
- Use escopos, roles e claims de forma consistente
- Separe autenticação de autorização
- Centralize identidade quando possível
- Audite eventos de segurança
- Atualize dependências regularmente
- Adote princípios de **Zero Trust**

---

# 🧠 Conclusão

O **Spring Security** fornece uma das infraestruturas de segurança mais completas do ecossistema Java, permitindo desde autenticação simples até arquiteturas corporativas distribuídas baseadas em identidade.

Dominar segurança no Spring é compreender **filtros, proxies, tokens, políticas, contexto e identidade** — não apenas anotações.

Segurança não é um recurso. É uma arquitetura.

---

<p align="center">
<b>Finalizada a Segurança e Identidade com Spring! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="15-testes.md">Testes no Ecossistema Spring</a>
</p>
