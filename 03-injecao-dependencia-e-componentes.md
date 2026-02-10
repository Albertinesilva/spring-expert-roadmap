# 🧩 Injeção de Dependência e Componentes

A **Injeção de Dependência (Dependency Injection — DI)** é um dos pilares centrais do Spring Framework. Ela é a principal manifestação prática do princípio de **Inversão de Controle (IoC)**, no qual a criação, configuração e gerenciamento de objetos deixam de ser responsabilidade do código da aplicação e passam a ser controlados pelo contêiner.

No Spring, a DI não é apenas um recurso de conveniência, mas uma base arquitetural que possibilita:

- Baixo acoplamento entre componentes.
- Alta testabilidade.
- Modularidade.
- Substituição transparente de implementações.
- Aplicação transversal de comportamentos como transações, segurança e cache via proxies.

---

## 🧠 Inversão de Controle (IoC) no Spring

Em arquiteturas tradicionais, um objeto é responsável por instanciar suas dependências. No Spring, esse controle é invertido:

- O contêiner cria os objetos (beans).
- O contêiner resolve e injeta suas dependências.
- O contêiner gerencia o ciclo de vida completo desses objetos.

Isso resulta em código mais limpo, mais flexível e menos acoplado a implementações concretas.

---

## 🔧 Formas de Injeção de Dependência

O Spring suporta três formas principais de injeção:

### 1️⃣ Injeção por Construtor (Recomendada)

```java
@Service
public class PedidoService {

    private final PagamentoService pagamentoService;

    public PedidoService(PagamentoService pagamentoService) {
        this.pagamentoService = pagamentoService;
    }
}
```

Vantagens:

Garante imutabilidade das dependências.

Permite validação explícita no construtor.

Facilita testes unitários.

Torna as dependências obrigatórias explícitas.

É a forma recomendada pela documentação oficial do Spring.

2️⃣ Injeção por Setter

```java
@Component
public class NotificacaoService {

    private EmailService emailService;

    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Uso típico:

Dependências opcionais.

Cenários em que a dependência pode ser alterada após a construção.

3️⃣ Injeção por Campo (Desencorajada)

```java
@Component
public class RelatorioService {

    @Autowired
    private ExportadorService exportadorService;
}
```

Desvantagens:

Dificulta testes unitários.

Oculta dependências reais da classe.

Impede imutabilidade.

Aumenta o acoplamento ao framework.

Embora funcional, essa abordagem é desencorajada em aplicações profissionais.

🧩 Resolução de Dependências

O Spring resolve dependências com base em:

Tipo (Type-based resolution).

Nome (Name-based resolution).

Qualificadores (@Qualifier).

Anotações personalizadas.

Primazia (@Primary).

🔸 Exemplo com múltiplas implementações

```java
public interface PagamentoService {
    void pagar();
}

@Service
@Qualifier("cartao")
public class PagamentoCartaoService implements PagamentoService { ... }

@Service
@Qualifier("pix")
public class PagamentoPixService implements PagamentoService { ... }

@Service
public class PedidoService {

    private final PagamentoService pagamentoService;

    public PedidoService(@Qualifier("pix") PagamentoService pagamentoService) {
        this.pagamentoService = pagamentoService;
    }
}
```

Ou:

```java
@Primary
@Service
public class PagamentoPadraoService implements PagamentoService { ... }
```

🧩 Componentes e Estereótipos

O Spring fornece anotações de estereótipo para indicar o papel de um componente:

@Component — componente genérico.

@Service — camada de serviço / lógica de negócio.

@Repository — camada de persistência.

@Controller / @RestController — camada web.

Essas anotações são semanticamente equivalentes do ponto de vista técnico, mas possuem valor arquitetural e semântico, além de permitir comportamentos adicionais (ex.: tradução de exceções em @Repository).

🧭 Component Scan e Detecção de Beans

O Spring detecta automaticamente classes anotadas através do component scanning, iniciado por:

```java
@ComponentScan
```

Por padrão, o Spring Boot escaneia o pacote onde está localizada a classe principal (@SpringBootApplication) e todos os seus subpacotes.

🔸 Boas práticas

Posicionar a classe principal no pacote raiz do projeto.

Organizar pacotes por domínio funcional (ex.: pedido, pagamento, cliente), e não apenas por camadas técnicas.

🧬 Beans e seus Escopos

Cada bean no Spring possui um escopo, que define sua estratégia de instânciação.

Escopos principais:

singleton (padrão) — uma única instância por contexto.

prototype — uma nova instância a cada injeção.

request — uma instância por requisição HTTP.

session — uma instância por sessão HTTP.

application — uma instância por aplicação web.

websocket — uma instância por sessão WebSocket.

🔸 Observação importante

Beans prototype não têm seu ciclo de destruição gerenciado pelo Spring, diferentemente dos singleton.

🧪 Ciclo de Vida dos Beans

O ciclo de vida de um bean envolve:

Instanciação.

Injeção de dependências.

Execução de callbacks:

@PostConstruct

InitializingBean.afterPropertiesSet()

Bean pronto para uso.

Destruição:

@PreDestroy

Métodos de destruição configurados.

Exemplo:

```java
@Component
public class CacheManager {

    @PostConstruct
    public void init() {
        // Inicialização
    }

    @PreDestroy
    public void destroy() {
        // Liberação de recursos
    }
}
```

🧠 Beans, Proxies e AOP

Um aspecto fundamental é que muitos beans não são instâncias diretas da classe original, mas proxies criados pelo Spring para aplicar comportamentos transversais, como:

Transações (@Transactional)

Segurança (@PreAuthorize, @Secured)

Cache (@Cacheable)

Observabilidade

Logs e métricas

Esses proxies interceptam chamadas aos métodos e aplicam lógica adicional antes, depois ou ao redor da execução real.

🔍 Implicações práticas

Chamadas internas entre métodos da mesma classe não passam pelo proxy.

Comportamentos como transações ou segurança podem não ser aplicados em cenários de self-invocation.

O tipo do proxy (JDK dynamic proxy ou CGLIB) influencia compatibilidade e design.

Esses temas serão aprofundados nos capítulos de AOP e Transações.

🧱 Configuração Baseada em Java (@Configuration)

Além de estereótipos, o Spring permite definir beans explicitamente via classes de configuração:

```java
@Configuration
public class AppConfig {

    @Bean
    public PagamentoService pagamentoService() {
        return new PagamentoPixService();
    }
}
```

Diferença entre @Configuration e @Component

@Configuration utiliza proxy de método para garantir que cada @Bean seja tratado como singleton, mesmo quando métodos se chamam internamente.

Classes anotadas apenas com @Component não possuem esse comportamento.

⚠️ Armadilhas Comuns

Uso excessivo de injeção por campo.

Beans com múltiplas responsabilidades.

Ambiguidade de dependências sem uso de @Qualifier ou @Primary.

Dependência circular.

Expectativa de comportamento transacional em chamadas internas.

🧩 Conclusão do Capítulo

A injeção de dependência no Spring não é apenas um recurso sintático, mas uma base arquitetural que sustenta todo o ecossistema. Compreender profundamente como o contêiner cria, gerencia, injeta e envolve os beans é essencial para:

Projetar sistemas escaláveis e testáveis.

Evitar armadilhas relacionadas a proxies, escopos e ciclo de vida.

Explorar de forma consciente recursos avançados como AOP, transações, cache e segurança.

Este capítulo estabelece os fundamentos para compreender como o Spring organiza e orquestra os componentes de uma aplicação moderna.
