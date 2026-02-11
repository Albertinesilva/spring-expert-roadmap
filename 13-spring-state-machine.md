# 🔄 Máquinas de Estado com Spring State Machine

Processos de negócio raramente são lineares. Eles envolvem **estados**, **transições**, **eventos**, **regras** e **condições**. O **Spring State Machine** fornece uma infraestrutura robusta para modelar, executar e monitorar esses fluxos de forma explícita, segura e observável.

Este capítulo explora como projetar máquinas de estado confiáveis, escaláveis e integradas ao ecossistema Spring.

---

# 🧠 Conceitos Fundamentais

## 🔹 Estado (State)

Representa uma condição ou fase do processo.

Exemplos:

- `CRIADO`
- `PAGO`
- `ENVIADO`
- `CANCELADO`

---

## 🔹 Evento (Event)

Ação ou ocorrência que dispara uma transição.

Exemplos:

- `PAGAR`
- `ENVIAR`
- `CANCELAR`

---

## 🔹 Transição (Transition)

Mudança de um estado para outro, acionada por um evento.

---

## 🔹 Guard (Guarda)

Condição booleana que deve ser verdadeira para permitir a transição.

---

## 🔹 Action (Ação)

Lógica executada durante uma transição ou ao entrar/sair de um estado.

---

## 🔹 Estado Inicial e Estados Finais

Definem o início e o término do fluxo.

---

# 🧱 Estrutura Básica

```
           (PAGAR)
[CRIADO] ------------> [PAGO] ------------> [ENVIADO]
    |                      |
    | (CANCELAR)           | (CANCELAR)
    ↓                      ↓
[CANCELADO]          [CANCELADO]
```

---

# 🧩 Configuração Básica

## 🔸 Definindo Estados e Eventos

```java
public enum EstadosPedido {
    CRIADO,
    PAGO,
    ENVIADO,
    CANCELADO
}

public enum EventosPedido {
    PAGAR,
    ENVIAR,
    CANCELAR
}
```

---

## 🔸 Configurando a Máquina de Estados

```java
@Configuration
@EnableStateMachine
public class PedidoStateMachineConfig 
        extends StateMachineConfigurerAdapter<EstadosPedido, EventosPedido> {

    @Override
    public void configure(
        StateMachineStateConfigurer<EstadosPedido, EventosPedido> states
    ) throws Exception {

        states
            .withStates()
            .initial(EstadosPedido.CRIADO)
            .state(EstadosPedido.PAGO)
            .state(EstadosPedido.ENVIADO)
            .end(EstadosPedido.CANCELADO);
    }

    @Override
    public void configure(
        StateMachineTransitionConfigurer<EstadosPedido, EventosPedido> transitions
    ) throws Exception {

        transitions
            .withExternal()
                .source(EstadosPedido.CRIADO)
                .target(EstadosPedido.PAGO)
                .event(EventosPedido.PAGAR)
            .and()
            .withExternal()
                .source(EstadosPedido.PAGO)
                .target(EstadosPedido.ENVIADO)
                .event(EventosPedido.ENVIAR)
            .and()
            .withExternal()
                .source(EstadosPedido.CRIADO)
                .target(EstadosPedido.CANCELADO)
                .event(EventosPedido.CANCELAR)
            .and()
            .withExternal()
                .source(EstadosPedido.PAGO)
                .target(EstadosPedido.CANCELADO)
                .event(EventosPedido.CANCELAR);
    }
}
```

---

# 🚀 Inicializando e Usando a Máquina

```java
@Autowired
private StateMachine<EstadosPedido, EventosPedido> stateMachine;

public void pagarPedido() {
    stateMachine.start();
    stateMachine.sendEvent(EventosPedido.PAGAR);
}
```

---

# 🔐 Guards (Condições)

Guards validam regras antes de permitir uma transição.

```java
@Bean
public Guard<EstadosPedido, EventosPedido> pagamentoAutorizado() {
    return context -> {
        Pedido pedido = (Pedido) context
            .getExtendedState()
            .getVariables()
            .get("pedido");

        return pedido != null && pedido.isPagamentoAutorizado();
    };
}
```

### 🔸 Aplicando Guard na Transição

```java
.withExternal()
    .source(EstadosPedido.CRIADO)
    .target(EstadosPedido.PAGO)
    .event(EventosPedido.PAGAR)
    .guard(pagamentoAutorizado());
```

---

# ⚙️ Actions (Ações)

Executam lógica durante transições ou entrada/saída de estados.

## 🔸 Ação de Transição

```java
@Bean
public Action<EstadosPedido, EventosPedido> registrarPagamento() {
    return context -> 
        System.out.println("Pagamento registrado para o pedido.");
}
```

Aplicando na transição:

```java
.withExternal()
    .source(EstadosPedido.CRIADO)
    .target(EstadosPedido.PAGO)
    .event(EventosPedido.PAGAR)
    .action(registrarPagamento());
```

---

## 🔸 Ações de Entrada

```java
@Override
public void configure(
    StateMachineStateConfigurer<EstadosPedido, EventosPedido> states
) throws Exception {

    states
        .withStates()
        .initial(EstadosPedido.CRIADO)
        .state(EstadosPedido.PAGO, entradaPago(), null)
        .state(EstadosPedido.ENVIADO, entradaEnviado(), null)
        .end(EstadosPedido.CANCELADO);
}

@Bean
public Action<EstadosPedido, EventosPedido> entradaPago() {
    return context -> 
        System.out.println("Pedido entrou no estado PAGO");
}
```

---

# 🧬 Extended State

Permite armazenar dados de contexto durante o ciclo da máquina.

```java
stateMachine.getExtendedState()
    .getVariables()
    .put("pedido", pedido);
```

---

# 🔄 Estados Hierárquicos (Substates)

```java
public enum EstadosPagamento {
    INICIO,
    PROCESSANDO,
    CONCLUIDO,
    FALHA
}
```

```java
.withStates()
    .initial(EstadosPedido.CRIADO)
    .state(EstadosPedido.PAGO)
    .and()
        .withStates()
        .parent(EstadosPedido.PAGO)
        .initial(EstadosPagamento.INICIO)
        .state(EstadosPagamento.PROCESSANDO)
        .state(EstadosPagamento.CONCLUIDO)
        .state(EstadosPagamento.FALHA);
```

---

# 🔁 Estados Paralelos (Fork/Join)

Permitem execução simultânea de múltiplos fluxos.

```java
.withFork()
    .source(EstadosPedido.PAGO)
    .target(EstadosEnvio.EM_TRANSPORTE)
    .target(EstadosFaturamento.EM_PROCESSAMENTO);

.withJoin()
    .source(EstadosEnvio.CONCLUIDO)
    .source(EstadosFaturamento.CONCLUIDO)
    .target(EstadosPedido.ENVIADO);
```

---

# 🧩 Choice (Escolha Dinâmica)

```java
.withChoice()
    .source(EstadosPedido.PAGO)
    .first(EstadosPedido.ENVIADO, envioDisponivel())
    .last(EstadosPedido.CANCELADO);
```

---

# 💾 Persistência de Estado

Permite salvar e restaurar o estado da máquina.

## 🔸 Configuração de Persistência

```java
@Bean
public StateMachinePersister<EstadosPedido, EventosPedido, String> persister(
        StateMachinePersist<EstadosPedido, EventosPedido, String> persist) {
    return new DefaultStateMachinePersister<>(persist);
}
```

---

## 🔸 Implementação do Persist

```java
@Component
public class PedidoStateMachinePersist
    implements StateMachinePersist<EstadosPedido, EventosPedido, String> {

    @Override
    public void write(
        StateMachineContext<EstadosPedido, EventosPedido> context,
        String pedidoId
    ) {
        // salvar estado e variáveis no banco
    }

    @Override
    public StateMachineContext<EstadosPedido, EventosPedido, String> read(
        String pedidoId
    ) {
        // restaurar estado e variáveis
        return null;
    }
}
```

---

# 🔍 Observabilidade e Monitoramento

## 🔸 Listener da Máquina

```java
@Bean
public StateMachineListener<EstadosPedido, EventosPedido> listener() {

    return new StateMachineListenerAdapter<>() {

        @Override
        public void stateChanged(
                State<EstadosPedido, EventosPedido> from,
                State<EstadosPedido, EventosPedido> to) {

            System.out.println(
                "Estado alterado: " +
                (from != null ? from.getId() : "NONE") +
                " → " + to.getId()
            );
        }

        @Override
        public void eventNotAccepted(
                Message<EventosPedido> event) {

            System.err.println(
                "Evento não aceito: " + event.getPayload()
            );
        }
    };
}
```

---

# 🧪 Testes de Máquina de Estado

## 🔸 Teste Básico

```java
@SpringBootTest
class PedidoStateMachineTest {

    @Autowired
    private StateMachine<EstadosPedido, EventosPedido> stateMachine;

    @Test
    void deveTransitarParaPago() {

        stateMachine.start();
        stateMachine.sendEvent(EventosPedido.PAGAR);

        assertEquals(
            EstadosPedido.PAGO,
            stateMachine.getState().getId()
        );
    }
}
```

---

## 🔸 Testando Guards

```java
@Test
void naoDevePagarSemAutorizacao() {

    stateMachine.getExtendedState()
        .getVariables()
        .put("pedido", new Pedido(false));

    stateMachine.start();

    boolean resultado =
        stateMachine.sendEvent(EventosPedido.PAGAR);

    assertFalse(resultado);
}
```

---

# 🔐 Boas Práticas

- Modele estados de forma explícita.
- Evite lógica de negócio pesada dentro da máquina.
- Use guards para validações simples.
- Separe ações de efeitos colaterais (e-mail, integrações).
- Utilize persistência para processos longos.
- Monitore transições críticas.
- Documente estados e eventos.
- Prefira múltiplas máquinas menores em vez de uma gigante.

---

# 🧱 Casos de Uso Comuns

- Fluxos de pedidos (e-commerce, logística, faturamento)
- Processos de aprovação
- Orquestração de pagamentos
- Workflow de documentos
- Gerenciamento de contratos
- Orquestração de microsserviços (sagas)

---

# 🧠 Conclusão do Capítulo

O Spring State Machine fornece uma base sólida para modelar processos complexos de forma explícita, previsível e observável. Ao transformar regras de negócio em estados e transições bem definidas, você reduz complexidade acidental, melhora a manutenibilidade e aumenta a confiabilidade do sistema.

Dominar máquinas de estado é dominar o controle do fluxo de negócio em arquiteturas modernas.
