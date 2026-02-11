# 27 — Spring Modulith (Monólitos Modulares)

O **Spring Modulith** é uma iniciativa do ecossistema Spring que fornece suporte arquitetural, ferramentas e validações para a construção de **monólitos modulares bem estruturados**, promovendo alto acoplamento interno dentro dos módulos e baixo acoplamento entre eles, com governança explícita.

Ele combina os benefícios do monólito (simplicidade operacional, consistência transacional e menor latência) com princípios de modularidade, clareza de domínio e isolamento arquitetural.

---

## 📌 Sumário

- [1. O que é Spring Modulith](#1-o-que-é-spring-modulith)
- [2. Por que monólitos modulares](#2-por-que-monólitos-modulares)
- [3. Conceitos Fundamentais](#3-conceitos-fundamentais)
- [4. Estruturação de Módulos](#4-estruturação-de-módulos)
- [5. Comunicação entre Módulos](#5-comunicação-entre-módulos)
- [6. Eventos de Domínio entre Módulos](#6-eventos-de-domínio-entre-módulos)
- [7. Restrições Arquiteturais e Governança](#7-restrições-arquiteturais-e-governança)
- [8. Testes de Módulos](#8-testes-de-módulos)
- [9. Observabilidade e Documentação Arquitetural](#9-observabilidade-e-documentação-arquitetural)
- [10. Modular Monolith vs Microservices](#10-modular-monolith-vs-microservices)
- [11. Boas Práticas](#11-boas-práticas)
- [12. Conclusão](#12-conclusão)

---

## 1. O que é Spring Modulith

Spring Modulith é um conjunto de bibliotecas que permite:

- Declarar módulos explicitamente
- Validar dependências entre módulos
- Publicar e consumir eventos entre módulos
- Testar módulos isoladamente
- Gerar documentação arquitetural

Ele não é um novo framework, mas uma camada de governança arquitetural sobre aplicações Spring Boot tradicionais.

---

## 2. Por que monólitos modulares

### Problemas comuns de monólitos tradicionais

- Alto acoplamento
- Fronteiras de domínio implícitas
- Dificuldade de manutenção
- Evolução desorganizada

### Vantagens do monólito modular

- Clareza de domínio
- Evolução controlada
- Facilidade de deploy
- Transações locais
- Base sólida para futura migração para microsserviços

---

## 3. Conceitos Fundamentais

### 🔹 Módulo

Um módulo representa um **Bounded Context** ou subdomínio do sistema.

### 🔹 Fronteiras explícitas

Cada módulo:

- Possui um pacote raiz
- Expõe apenas contratos públicos
- Oculta sua implementação interna

### 🔹 Governança arquitetural

O framework valida dependências ilegais entre módulos em tempo de build ou durante testes automatizados.

---

## 4. Estruturação de Módulos

### ✔ Estrutura por pacotes

```text
com.exemplo.app
├── vendas
│   ├── internal
│   ├── api
│   └── VendasConfiguration.java
├── faturamento
│   ├── internal
│   ├── api
│   └── FaturamentoConfiguration.java
└── Application.java
```

Cada módulo:

- Define um pacote raiz
- Possui uma classe de configuração
- Expõe apenas APIs públicas no pacote `api`
- Mantém a implementação no pacote `internal`

---

## 5. Comunicação entre Módulos

### ❌ Comunicação direta via classes internas (evitar)

```java
@Autowired
ClasseInternaOutroModulo classe;
```

### ✔ Comunicação via APIs públicas

```java
@Autowired
VendasService vendasService;
```

As APIs públicas devem residir no pacote `api` de cada módulo.

---

## 6. Eventos de Domínio entre Módulos

### ✔ Publicação de eventos

```java
public class PedidoService {

    private final ApplicationEventPublisher publisher;

    public void concluirPedido(Pedido pedido) {
        publisher.publishEvent(new PedidoConcluidoEvent(pedido.getId()));
    }
}
```

### ✔ Consumo em outro módulo

```java
@Component
public class FaturamentoListener {

    @EventListener
    public void on(PedidoConcluidoEvent event) {
        // lógica de faturamento
    }
}
```

Eventos promovem:

- Desacoplamento
- Separação de responsabilidades
- Consistência eventual

---

## 7. Restrições Arquiteturais e Governança

Declaração de módulo:

```java
@ApplicationModule
public class VendasConfiguration {
}
```

Teste de módulo:

```java
@ApplicationModuleTest
class VendasModuleTest {
}
```

Dependências proibidas resultam em falha de build ou teste, garantindo disciplina arquitetural contínua.

---

## 8. Testes de Módulos

### ✔ Teste isolado de módulo

```java
@ApplicationModuleTest
class FaturamentoModuleTest {

    @Autowired
    FaturamentoService faturamentoService;

    @Test
    void deveGerarFatura() {
        // teste
    }
}
```

Benefícios:

- Isolamento real
- Maior confiabilidade
- Evolução segura

---

## 9. Observabilidade e Documentação Arquitetural

O Spring Modulith permite:

- Gerar diagramas de dependência entre módulos
- Inspecionar eventos publicados e consumidos
- Auditar comunicação interna

Isso facilita:

- Revisões arquiteturais
- Onboarding técnico
- Auditorias técnicas

---

## 10. Modular Monolith vs Microservices

| Critério        | Monólito Modular | Microserviços |
| --------------- | ---------------- | ------------- |
| Deploy          | Único            | Múltiplos     |
| Comunicação     | In-process       | Rede          |
| Transações      | Locais           | Distribuídas  |
| Complexidade    | Menor            | Maior         |
| Escalabilidade  | Vertical         | Horizontal    |
| Observabilidade | Mais simples     | Mais complexa |

➡️ O Spring Modulith é ideal para a maioria dos sistemas corporativos que buscam equilíbrio entre simplicidade e robustez arquitetural.

---

## 11. Boas Práticas

- Modele módulos por domínio, não por camada técnica
- Utilize eventos para comunicação entre módulos
- Evite dependências cíclicas
- Defina contratos públicos claros
- Teste módulos isoladamente
- Utilize validações arquiteturais automatizadas

---

## 12. Conclusão

O Spring Modulith oferece um caminho pragmático e robusto para construir sistemas:

- Bem estruturados
- Evolutivos
- Governáveis
- Preparados para escalar técnica e organizacionalmente

Ele representa uma abordagem moderna para arquitetura corporativa Java, unindo disciplina arquitetural e simplicidade operacional.
