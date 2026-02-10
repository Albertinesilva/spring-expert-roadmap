# 🔁 Transações

Transações são fundamentais para garantir **consistência, integridade e confiabilidade** em sistemas que manipulam dados. No ecossistema Spring, o suporte transacional é profundo, extensível e altamente integrado, permitindo gerenciar transações de forma declarativa, programática, distribuída e reativa.

Este capítulo explora desde os conceitos fundamentais até o funcionamento interno do mecanismo transacional do Spring, incluindo propagação, isolamento, rollback, sincronização e armadilhas comuns.

---

## 🧠 Conceito de Transação

Uma transação é uma unidade lógica de trabalho que deve obedecer às propriedades **ACID**:

- **Atomicidade** — Tudo ou nada.
- **Consistência** — O sistema sai de um estado válido para outro.
- **Isolamento** — Transações concorrentes não interferem indevidamente.
- **Durabilidade** — Uma vez confirmada, a transação persiste mesmo após falhas.

---

## 🧩 Arquitetura de Transações no Spring

Componentes principais:

- **PlatformTransactionManager** — Interface central.
- **TransactionDefinition** — Define regras (propagação, isolamento, timeout, read-only).
- **TransactionStatus** — Representa o estado da transação.
- **TransactionInterceptor** — Aplica transações via AOP.
- **TransactionSynchronizationManager** — Gerencia recursos associados à transação.

---

# 🔧 Gerenciamento Declarativo

## 🔸 `@Transactional`

```java
@Transactional
public void salvarPedido(Pedido pedido) {
    repository.save(pedido);
    estoqueService.reservar(pedido);
}
```

O Spring cria um proxy ao redor do bean e inicia/encerra transações conforme necessário.

### 🔸 Onde aplicar `@Transactional`?

- Preferencialmente na camada de serviço.
- Em métodos públicos.
- Evitar uso direto em controladores.

---

# 🔹 Propagation

Define como métodos transacionais se comportam quando chamados dentro de outra transação.

## 🔧 Principais modos

| Propagation      | Comportamento |
|------------------|--------------|
| REQUIRED         | Usa a transação existente ou cria uma nova |
| REQUIRES_NEW     | Suspende a atual e cria uma nova |
| SUPPORTS         | Usa se existir, senão executa sem |
| MANDATORY        | Exige transação existente |
| NOT_SUPPORTED    | Executa sem transação |
| NEVER            | Falha se houver transação |
| NESTED           | Cria subtransação (se suportado) |

### 🔸 Exemplo

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void registrarAuditoria(Auditoria auditoria) {
    auditoriaRepository.save(auditoria);
}
```

---

# 🔹 Isolation

Define o nível de isolamento entre transações concorrentes.

## 🔧 Níveis

| Isolation        | Garante evitar |
|------------------|---------------|
| READ_UNCOMMITTED | Nenhum |
| READ_COMMITTED   | Dirty reads |
| REPEATABLE_READ  | Non-repeatable reads |
| SERIALIZABLE     | Phantom reads |

### 🔸 Exemplo

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void processarPagamento(Pagamento pagamento) {
    // lógica crítica
}
```

---

# 🔹 Rollback Rules

## 🔧 Regras padrão

Rollback ocorre automaticamente em:

- `RuntimeException`
- `Error`

Não ocorre por padrão em:

- `Exception` (checked)

### 🔸 Configuração explícita

```java
@Transactional(rollbackFor = Exception.class)
public void processarPedido(Pedido pedido) throws Exception {
    // lógica
}
```

```java
@Transactional(noRollbackFor = NegocioException.class)
public void validarPedido(Pedido pedido) {
    // lógica
}
```

---

# 🔁 Transaction Synchronization

Permite executar lógica antes ou após eventos transacionais.

## 🔧 Exemplo Programático

```java
TransactionSynchronizationManager.registerSynchronization(
    new TransactionSynchronizationAdapter() {
        @Override
        public void afterCommit() {
            enviarEmailConfirmacao();
        }
    }
);
```

## 🔧 Uso moderno com eventos

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void aoConfirmarPedido(PedidoConfirmadoEvent event) {
    enviarEmail(event);
}
```

---

# 🧠 Funcionamento Interno: Proxies e AOP

Spring usa proxies (JDK ou CGLIB) para interceptar chamadas a métodos anotados com `@Transactional`.

## 🔄 Fluxo simplificado

1. Cliente chama método público do bean.
2. Proxy intercepta.
3. Inicia transação via `TransactionManager`.
4. Executa método.
5. Commit ou rollback conforme exceção.
6. Libera recursos.

---

## ⚠️ Self-invocation Issue

Chamadas internas dentro da mesma classe não passam pelo proxy:

```java
public class PedidoService {

    @Transactional
    public void metodoA() {
        metodoB(); // @Transactional em metodoB será ignorado
    }

    @Transactional
    public void metodoB() {
        // lógica
    }
}
```

### ✅ Soluções

- Mover métodos para outro bean.
- Injetar o próprio proxy.
- Usar AspectJ weaving.

---

# 🔁 Transações Programáticas

## 🔧 Com `PlatformTransactionManager`

```java
TransactionStatus status = transactionManager.getTransaction(
    new DefaultTransactionDefinition()
);

try {
    repository.save(pedido);
    transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
    throw e;
}
```

---

## 🔧 Com `TransactionTemplate`

```java
transactionTemplate.execute(status -> {
    repository.save(pedido);
    estoqueService.reservar(pedido);
    return null;
});
```

---

# 🔹 Transações Read-Only

```java
@Transactional(readOnly = true)
public List<Usuario> listarUsuarios() {
    return repository.findAll();
}
```

### 🎯 Benefícios

- Otimização de performance.
- Previne alterações acidentais.
- Pode ativar otimizações no banco e no ORM.

---

# 🔄 Transações em Sistemas Distribuídos

## 🧠 Problema

Transações distribuídas (2PC/XA) são caras, complexas e frágeis em ambientes modernos.

## 🔸 Alternativas modernas

- Saga Pattern
- Eventual Consistency
- Compensating Transactions
- Outbox Pattern
- Event-Driven Architecture

Spring suporta esses padrões por meio de:

- Spring Events
- Spring Cloud Stream
- Mensageria
- Transações locais + mensagens confiáveis

---

# 🔁 Transações Reativas

Em aplicações reativas (WebFlux + R2DBC), transações são tratadas de forma diferente.

```java
@Transactional
public Mono<Void> processarPedido(Pedido pedido) {
    return repository.save(pedido)
        .then(estoqueService.reservar(pedido));
}
```

⚠️ Requer `ReactiveTransactionManager` e operadores como `TransactionalOperator`.

---

# 🧪 Testes com Transações

## 🔸 Testes com rollback automático

```java
@SpringBootTest
@Transactional
class PedidoServiceTest {

    @Autowired
    private PedidoService service;

    @Test
    void deveSalvarPedido() {
        service.salvarPedido(new Pedido());
    }
}
```

Por padrão, a transação será revertida ao final do teste.

---

## 🔸 Desabilitando rollback

```java
@Rollback(false)
@Test
void devePersistirDados() {
    // lógica
}
```

---

# 🛡️ Boas Práticas

- Coloque `@Transactional` na camada de serviço.
- Evite transações longas.
- Use `readOnly = true` quando aplicável.
- Evite lógica de negócio em controladores.
- Compreenda propagação antes de usar `REQUIRES_NEW`.
- Monitore deadlocks e contenção.
- Evite transações distribuídas sempre que possível.
- Documente políticas transacionais.
- Teste cenários de falha.

---

# 🧱 Conclusão do Capítulo

O gerenciamento de transações no Spring é poderoso, flexível e profundamente integrado ao ecossistema. Contudo, seu uso eficaz exige compreensão dos mecanismos internos, especialmente proxies, propagação, isolamento e rollback.

Dominar transações é essencial para construir sistemas consistentes, resilientes e escaláveis, especialmente em ambientes distribuídos e altamente concorrentes.
