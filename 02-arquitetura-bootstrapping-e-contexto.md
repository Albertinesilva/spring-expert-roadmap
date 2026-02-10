# ⚙️ Arquitetura, Bootstrapping e Contexto

A arquitetura do Spring Boot é construída sobre os fundamentos do Spring Framework, especialmente o conceito de **Inversão de Controle (IoC)** e do **ApplicationContext**. Compreender o processo de inicialização (*bootstrapping*) é essencial para entender como a aplicação é configurada, como os componentes são registrados e como o ciclo de vida dos objetos é gerenciado.

Mais do que um simples ponto de entrada (`main`), o bootstrapping define:

- Como o contexto de aplicação é criado.
- Como os beans são descobertos, instanciados e conectados.
- Como as configurações são aplicadas condicionalmente.
- Como o ambiente (profiles, propriedades, variáveis) influencia o comportamento da aplicação.
- Como o ciclo de vida da aplicação é gerenciado do início ao encerramento.

---

## 🔹 Ciclo de Vida do ApplicationContext

O `ApplicationContext` é o contêiner central do Spring. Ele estende a funcionalidade básica da `BeanFactory`, oferecendo suporte a:

- Eventos de aplicação.
- Internacionalização (i18n).
- Resolução de recursos.
- Integração com AOP, transações, segurança, etc.

O ciclo de vida típico de um contexto Spring Boot envolve as seguintes fases:

### 1. Criação do contexto

O ponto de entrada é:

```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}
```

A classe `SpringApplication` é responsável por:

- Determinar o tipo de aplicação (Servlet, Reactive, CLI).
- Criar o `ApplicationContext` apropriado.
- Preparar o ambiente (`Environment`).
- Aplicar listeners e inicializadores.

---

### 2. Preparação do ambiente

Antes da criação dos beans, o Spring:

- Carrega propriedades de múltiplas fontes:
  - `application.properties` / `application.yml`
  - Variáveis de ambiente
  - Argumentos de linha de comando
  - Configurações externas (ex.: Config Server)
- Resolve profiles ativos.
- Aplica conversão de tipos e precedência entre fontes.

---

### 3. Registro de definições de beans

As definições de beans são obtidas a partir de:

- Classes anotadas (`@Configuration`, `@Component`, etc.).
- Classes de autoconfiguração do Spring Boot.
- Configurações explícitas via Java, XML ou outras fontes.

> Nesse momento, ainda não há instâncias criadas — apenas metadados.

---

### 4. Processamento de BeanFactoryPostProcessors

Antes da criação dos beans, o Spring executa:

- `BeanFactoryPostProcessor`
- `BeanDefinitionRegistryPostProcessor`

Esses componentes podem:

- Modificar definições de beans.
- Registrar novos beans dinamicamente.
- Alterar escopos, dependências e propriedades.

Exemplo típico: suporte a `@ConfigurationProperties`.

---

### 5. Criação e inicialização dos beans

O contêiner:

- Resolve dependências.
- Aplica injeção (construtor, setter, campo).
- Executa:
  - Métodos `@PostConstruct`
  - `InitializingBean.afterPropertiesSet()`
  - Métodos customizados de inicialização
- Envolve beans com proxies quando necessário (AOP, transações, segurança, cache).

---

### 6. Publicação de eventos do ciclo de vida

Durante o bootstrapping, eventos como:

- `ApplicationStartingEvent`
- `ApplicationEnvironmentPreparedEvent`
- `ApplicationContextInitializedEvent`
- `ApplicationPreparedEvent`
- `ApplicationStartedEvent`
- `ApplicationReadyEvent`

São publicados, permitindo que componentes reajam a momentos específicos da inicialização.

---

### 7. Encerramento do contexto

Ao finalizar a aplicação, o Spring:

- Publica `ContextClosedEvent`.
- Executa métodos `@PreDestroy`.
- Executa callbacks de destruição configurados.
- Libera recursos (threads, conexões, caches, etc.).

---

## 🔹 Auto-configuração e Conditional Annotations

A autoconfiguração é um dos pilares do Spring Boot. Ela permite que a aplicação seja configurada automaticamente com base no classpath, nas propriedades e no ambiente de execução.

### 🔸 Ativação da autoconfiguração

A anotação:

```java
@SpringBootApplication
```

É composta por:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

O `@EnableAutoConfiguration` instrui o Spring a carregar automaticamente classes de configuração presentes nos starters.

---

### 🔸 Como funciona a autoconfiguração

O Spring Boot utiliza:

- Arquivos `META-INF/spring.factories` (Spring Boot ≤ 2.x)  
ou  
- Arquivos `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Spring Boot 3.x)

Para localizar classes de autoconfiguração.

Cada autoconfiguração é fortemente baseada em anotações condicionais, como:

- `@ConditionalOnClass` — ativa se uma classe existir no classpath.
- `@ConditionalOnMissingBean` — ativa se não existir um bean do mesmo tipo.
- `@ConditionalOnProperty` — ativa se uma propriedade estiver definida.
- `@ConditionalOnWebApplication` — ativa apenas em aplicações web.
- `@ConditionalOnSingleCandidate` — ativa se houver exatamente um bean candidato.

---

### 🔸 Exemplo conceitual

Uma configuração de `DataSource` só será aplicada se:

- Houver uma dependência JDBC no classpath.
- Não existir outro `DataSource` definido manualmente.
- As propriedades necessárias estiverem disponíveis.

Isso garante configuração automática sem sobrescrever configurações explícitas.

---

## 🔹 Spring Boot DevTools e Experiência do Desenvolvedor

O módulo Spring Boot DevTools foi projetado para melhorar a produtividade durante o desenvolvimento local.

### Funcionalidades principais

#### Restart automático

- Reinicia o contexto quando classes do projeto são modificadas.
- Mantém caches internos fora do classloader reiniciado, acelerando o ciclo.

#### LiveReload

- Atualiza automaticamente o navegador quando recursos front-end mudam.

#### Configurações específicas para desenvolvimento

- Desativa caches de templates.
- Habilita mensagens de erro mais detalhadas.
- Altera comportamentos padrão que não são ideais para produção.

> O DevTools é automaticamente desativado em ambientes de produção, garantindo que seus efeitos sejam limitados ao desenvolvimento.

---

## 🔹 Docker Compose Support e Development Mode

A partir do Spring Boot 3.1, foi introduzido suporte nativo ao Docker Compose como parte da experiência de desenvolvimento.

### Principais benefícios

- Inicialização automática de serviços auxiliares (banco, cache, mensageria).
- Injeção automática de propriedades com base nos containers em execução.
- Redução de configuração manual de ambientes locais.

### Exemplo de uso

Com um arquivo `docker-compose.yml` no projeto, o Spring Boot:

- Detecta os serviços.
- Inicializa containers necessários.
- Conecta automaticamente a aplicação a esses serviços durante o desenvolvimento.

Isso promove um ambiente local mais próximo da produção, sem complexidade operacional adicional.

---

## 🔹 Graceful Shutdown e Lifecycle Management

O Spring Boot oferece suporte a encerramento gracioso, garantindo que requisições em andamento sejam finalizadas antes da aplicação ser encerrada.

### Funcionalidades principais

Suporte a:

- HTTP servers (Tomcat, Jetty, Undertow).
- Pools de threads.
- Conexões de banco de dados.
- Mensageria e filas.

Configuração via propriedades:

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

### Benefícios

- Evita perda de requisições.
- Garante integridade transacional.
- Promove estabilidade em ambientes distribuídos e orquestrados (ex.: Kubernetes).

---

## 🧩 Conclusão do Capítulo

Compreender o processo de bootstrapping e o funcionamento do `ApplicationContext` é fundamental para:

- Diagnosticar problemas de configuração.
- Entender comportamentos implícitos do framework.
- Projetar arquiteturas mais previsíveis, performáticas e seguras.
- Explorar de forma consciente recursos como AOP, transações, cache, segurança e observabilidade.

Este capítulo estabelece a base conceitual para todos os demais, pois praticamente todos os módulos do Spring se integram e se manifestam por meio do ciclo de vida do contexto.
