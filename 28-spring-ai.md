# 28 — Spring AI

O **Spring AI** é o projeto do ecossistema Spring que fornece abstrações, integrações e padrões para consumir modelos de linguagem (LLMs), embeddings, vetores, ferramentas e fluxos de IA generativa de forma segura, testável e produtiva dentro de aplicações Spring Boot.

Ele aplica os princípios clássicos do Spring — **Inversão de Controle (IoC), auto-configuração, observabilidade e portabilidade** — ao domínio da Inteligência Artificial.

---

## 📌 Sumário

- [1. O que é Spring AI](#1-o-que-é-spring-ai)
- [2. Objetivos do Spring AI](#2-objetivos-do-spring-ai)
- [3. Arquitetura Conceitual](#3-arquitetura-conceitual)
- [4. Modelos de Linguagem (LLMs)](#4-modelos-de-linguagem-llms)
- [5. Prompt Engineering com Spring](#5-prompt-engineering-com-spring)
- [6. Embeddings e Vetores](#6-embeddings-e-vetores)
- [7. Armazenamento Vetorial (Vector Stores)](#7-armazenamento-vetorial-vector-stores)
- [8. Retrieval-Augmented Generation (RAG)](#8-retrieval-augmented-generation-rag)
- [9. Ferramentas (Function Calling / Tools)](#9-ferramentas-function-calling--tools)
- [10. Observabilidade e Monitoramento](#10-observabilidade-e-monitoramento)
- [11. Segurança e Governança](#11-segurança-e-governança)
- [12. Casos de Uso Corporativos](#12-casos-de-uso-corporativos)
- [13. Boas Práticas](#13-boas-práticas)
- [14. Limitações e Desafios](#14-limitações-e-desafios)
- [15. Conclusão](#15-conclusão)

---

## 1. O que é Spring AI

Spring AI é uma camada de abstração para trabalhar com:

- Modelos de linguagem (OpenAI, Azure OpenAI, Anthropic, Google, etc.)
- Modelos de embeddings
- Bancos vetoriais
- Retrieval-Augmented Generation (RAG)
- Ferramentas (Function/Tool Calling)
- Observabilidade, testes e governança

Ele elimina dependências diretas de SDKs proprietários, promovendo portabilidade e boas práticas arquiteturais.

---

## 2. Objetivos do Spring AI

- Padronizar o acesso a LLMs
- Reduzir acoplamento com fornecedores
- Facilitar testes e mocks
- Integrar IA ao ecossistema Spring
- Oferecer observabilidade, métricas e tracing
- Garantir segurança e controle de dados

---

## 3. Arquitetura Conceitual

```
Aplicação Spring
        │
Spring AI (Abstrações)
        │
Model Providers (OpenAI, Anthropic, Google, etc.)
        │
Vector Stores (Pinecone, Redis, PGVector, etc.)
```

### Componentes principais

- **ChatModel / ChatClient** – Interação textual com LLMs
- **EmbeddingModel / EmbeddingClient** – Geração de vetores
- **VectorStore** – Armazenamento vetorial
- **Prompt** – Representação estruturada de entrada
- **Tool / Function** – Extensão do modelo com funções Java

---

## 4. Modelos de Linguagem (LLMs)

### ✔ Exemplo básico

```java
@Autowired
ChatClient chatClient;

String resposta = chatClient.call("Explique o padrão Factory.");
```

O Spring AI oferece abstrações unificadas, independentemente do provedor utilizado.

---

## 5. Prompt Engineering com Spring

### ✔ Prompt estruturado

```java
Prompt prompt = new Prompt(
    List.of(
        new SystemMessage("Você é um assistente técnico."),
        new UserMessage("Explique o padrão Observer.")
    )
);

ChatResponse response = chatClient.call(prompt);
```

### Benefícios

- Prompts versionáveis
- Reutilização
- Testabilidade
- Observabilidade

---

## 6. Embeddings e Vetores

```java
@Autowired
EmbeddingClient embeddingClient;

List<Double> vector = embeddingClient.embed("Spring Boot é um framework Java.");
```

Embeddings transformam texto em vetores numéricos usados para:

- Busca semântica
- Clusterização
- Sistemas de recomendação
- Implementação de RAG

---

## 7. Armazenamento Vetorial (Vector Stores)

Spring AI suporta diversos mecanismos de armazenamento vetorial:

- Redis
- PostgreSQL (pgvector)
- Pinecone
- Weaviate
- Elasticsearch
- ChromaDB

### ✔ Exemplo

```java
vectorStore.add(List.of(
    new Document("Spring é uma plataforma Java."),
    new Document("JPA é uma API de persistência.")
));
```

---

## 8. Retrieval-Augmented Generation (RAG)

RAG combina:

1. Recuperação de contexto relevante
2. Geração de resposta com LLM

### ✔ Fluxo típico

```java
List<Document> docs = vectorStore.similaritySearch("O que é Spring Security?");

String resposta = chatClient.call(
    new Prompt("Responda com base nos seguintes documentos:\n" + docs)
);
```

### Benefícios

- Redução de alucinações
- Atualização dinâmica de conhecimento
- Controle da fonte de dados
- Respostas contextualizadas

---

## 9. Ferramentas (Function Calling / Tools)

Permite que o modelo invoque funções Java diretamente.

```java
@Tool
public String buscarPedidoPorId(Long id) {
    return pedidoService.buscar(id).toString();
}
```

O modelo decide quando chamar a função com base no contexto da conversa.

### Casos de uso

- Integração com sistemas internos
- Execução de comandos
- Orquestração de fluxos de negócio
- Automação de processos

---

## 10. Observabilidade e Monitoramento

Spring AI integra-se com:

- Micrometer
- OpenTelemetry
- Spring Boot Actuator

Permite monitorar:

- Métricas de latência
- Consumo de tokens
- Taxa de erros
- Tracing distribuído

---

## 11. Segurança e Governança

Boas práticas recomendadas:

- Gerenciamento seguro de chaves (Vault, Secrets Manager)
- Controle de dados sensíveis
- Logging estruturado
- Auditoria de prompts e respostas
- Rate limiting e controle de quotas
- Redação automática de dados pessoais

---

## 12. Casos de Uso Corporativos

- Assistentes internos (DevOps, suporte, jurídico)
- Chatbots corporativos
- Análise e classificação de documentos
- Geração automática de relatórios
- Sumarização de textos
- Pesquisa semântica
- Automação inteligente de processos

---

## 13. Boas Práticas

- Versione prompts como código
- Implemente fallback entre provedores
- Utilize RAG para dados internos
- Monitore custos e consumo de tokens
- Teste prompts sistematicamente
- Proteja dados sensíveis
- Combine IA com lógica determinística

---

## 14. Limitações e Desafios

- Alucinações ainda podem ocorrer
- Custo variável e potencialmente imprevisível
- Dependência de provedores externos
- Governança de dados é crítica
- Testes determinísticos podem ser complexos

---

## 15. Conclusão

O Spring AI representa a convergência entre:

- Arquitetura corporativa Java
- Inteligência Artificial generativa
- Observabilidade, segurança e governança

Ele permite que equipes construam aplicações de IA robustas, seguras, escaláveis e alinhadas às boas práticas do ecossistema Spring — transformando IA de experimento em infraestrutura estratégica.
