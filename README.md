# 📚 Guia Completo — Ecossistema Spring

![Java](https://img.shields.io/badge/Java-17%2B-blue)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-Em%20Desenvolvimento-yellow)

Um guia técnico, acadêmico e didático sobre o **Spring Framework e seu ecossistema**, projetado para desenvolvedores que desejam ir além do uso superficial e compreender o funcionamento interno, arquitetura, performance e operação de aplicações Java modernas.

Este repositório apresenta uma construção modular e técnica do ecossistema Spring, documentando seus principais componentes, arquiteturas e práticas, com foco em funcionamento interno, uso avançado e aplicações reais.

---

## 🎯 Objetivo

Este repositório tem como propósito:

- Construir uma visão modular, técnica e aprofundada do ecossistema Spring, documentando seus principais componentes e arquiteturas.
- Servir como base sólida de estudo contínuo sobre o Spring.
- Atuar como referência técnica para projetos reais.
- Consolidar conhecimento avançado em arquitetura, performance, segurança e observabilidade.
- Preparar profissionais para entrevistas técnicas de nível pleno, sênior e especialista.

---

## 🧠 Filosofia

> **Entender > Memorizar**

Aqui, o foco não é apenas aprender anotações como `@Autowired`, `@Transactional` ou `@RestController`, mas compreender:

- Por que funcionam.
- Quando falham.
- Como o Spring as implementa internamente.
- Quais trade-offs arquiteturais estão envolvidos.

---

## 🧭 Navegação Rápida

### 🔰 Fundamentos

| Nº  | Tema                                     | Link                                                                                     |
| --- | ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| 00  | 📚 Sumário Geral                         | [00-sumario.md](00-sumario.md)                                                           |
| 01  | 📘 Introdução                            | [01-introducao.md](01-introducao.md)                                                     |
| 02  | ⚙️ Arquitetura, Bootstrapping e Contexto | [02-arquitetura-bootstrapping-e-contexto.md](02-arquitetura-bootstrapping-e-contexto.md) |
| 03  | 🧩 Injeção de Dependência e Componentes  | [03-injecao-dependencia-e-componentes.md](03-injecao-dependencia-e-componentes.md)       |
| 04  | 🔍 Spring Expression Language (SpEL)     | [04-spel.md](04-spel.md)                                                                 |
| 05  | 🛡️ Validação                             | [05-validacao.md](05-validacao.md)                                                       |

---

### 🌐 Web, APIs e Integrações

| Nº  | Tema                           | Link                                                 |
| --- | ------------------------------ | ---------------------------------------------------- |
| 06  | 🌐 Web e Spring MVC            | [06-web-e-spring-mvc.md](06-web-e-spring-mvc.md)     |
| 07  | 🗂️ APIs REST Maturas           | [07-apis-rest-maturas.md](07-apis-rest-maturas.md)   |
| 08  | 📦 Spring GraphQL              | [08-spring-graphql.md](08-spring-graphql.md)         |
| 12  | 🔗 Spring Integration          | [12-spring-integration.md](12-spring-integration.md) |
| 21  | 🌍 Clientes HTTP e Integrações | [21-clientes-http.md](21-clientes-http.md)           |

---

### 🗄️ Dados, Transações e Consistência

| Nº  | Tema                                    | Link                                                             |
| --- | --------------------------------------- | ---------------------------------------------------------------- |
| 09  | 🗄️ Persistência e Exploração de Dados   | [09-persistencia-spring-data.md](09-persistencia-spring-data.md) |
| 10  | 🔁 Transações                           | [10-transacoes.md](10-transacoes.md)                             |
| 24  | 📦 Processamento em Lote (Spring Batch) | [24-spring-batch.md](24-spring-batch.md)                         |

---

### 🔐 Segurança, Testes e Qualidade

| Nº  | Tema                                 | Link                               |
| --- | ------------------------------------ | ---------------------------------- |
| 14  | 🔐 Segurança e Identidade            | [14-seguranca.md](14-seguranca.md) |
| 15  | 🧪 Testes                            | [15-testes.md](15-testes.md)       |
| 17  | 🧵 AOP (Aspect-Oriented Programming) | [17-aop.md](17-aop.md)             |
| 18  | 🚀 Caching                           | [18-caching.md](18-caching.md)     |

---

### ⚡ Concorrência, Performance e Observabilidade

| Nº  | Tema                                           | Link                                               |
| --- | ---------------------------------------------- | -------------------------------------------------- |
| 19  | ⏱️ Agendamento e Execução Assíncrona           | [19-agendamento-async.md](19-agendamento-async.md) |
| 20  | 📊 Observabilidade (Metrics, Tracing, Logging) | [20-observabilidade.md](20-observabilidade.md)     |
| 22  | ⚡ Programação Reativa (WebFlux)               | [22-webflux.md](22-webflux.md)                     |
| 25  | 🧠 GraalVM e Compilação Nativa                 | [25-graalvm.md](25-graalvm.md)                     |
| 26  | 🧵 Virtual Threads (Java 21+)                  | [26-virtual-threads.md](26-virtual-threads.md)     |

---

### ☁️ Cloud Native, Modularidade e Evolução

| Nº  | Tema                                          | Link                                                 |
| --- | --------------------------------------------- | ---------------------------------------------------- |
| 23  | ☁️ Cloud Native e Gateways                    | [23-cloud-native.md](23-cloud-native.md)             |
| 27  | 🧱 Spring Modulith                            | [27-spring-modulith.md](27-spring-modulith.md)       |
| 30  | 🔄 Migração e Evolução                        | [30-migracao-evolucao.md](30-migracao-evolucao.md)   |
| 31  | 🆕 Anotações Modernas e Atualizações Recentes | [31-anotacoes-modernas.md](31-anotacoes-modernas.md) |
| 32  | 🏗️ Boas Práticas Arquiteturais                | [32-boas-praticas.md](32-boas-praticas.md)           |

---

### 🤖 Ferramentas, IA e Interfaces

| Nº  | Tema                              | Link                               |
| --- | --------------------------------- | ---------------------------------- |
| 28  | 🤖 Spring AI                      | [28-spring-ai.md](28-spring-ai.md) |
| 29  | 🖥️ Aplicações de Linha de Comando | [29-cli.md](29-cli.md)             |

---

### 📎 Referências

| Nº  | Tema                    | Link                                   |
| --- | ----------------------- | -------------------------------------- |
| 33  | 📎 Referências Oficiais | [33-referencias.md](33-referencias.md) |

---

## 🏗️ Estrutura do Repositório

```bash
📦 spring-guia-completo
 ┣ 📜 00-sumario.md
 ┣ 📜 01-introducao.md
 ┣ 📜 02-arquitetura-bootstrapping-e-contexto.md
 ┣ 📜 03-injecao-dependencia-e-componentes.md
 ┣ 📜 04-spel.md
 ┣ 📜 05-validacao.md
 ┣ 📜 06-web-e-spring-mvc.md
 ┣ 📜 07-apis-rest-maturas.md
 ┣ 📜 08-spring-graphql.md
 ┣ 📜 09-persistencia-spring-data.md
 ┣ 📜 10-transacoes.md
 ┣ 📜 11-eventos-mensageria-streaming.md
 ┣ 📜 12-spring-integration.md
 ┣ 📜 13-spring-state-machine.md
 ┣ 📜 14-seguranca.md
 ┣ 📜 15-testes.md
 ┣ 📜 16-configuracao-propriedades-perfis.md
 ┣ 📜 17-aop.md
 ┣ 📜 18-caching.md
 ┣ 📜 19-agendamento-async.md
 ┣ 📜 20-observabilidade.md
 ┣ 📜 21-clientes-http.md
 ┣ 📜 22-webflux.md
 ┣ 📜 23-cloud-native.md
 ┣ 📜 24-spring-batch.md
 ┣ 📜 25-graalvm.md
 ┣ 📜 26-virtual-threads.md
 ┣ 📜 27-spring-modulith.md
 ┣ 📜 28-spring-ai.md
 ┣ 📜 29-cli.md
 ┣ 📜 30-migracao-evolucao.md
 ┣ 📜 31-anotacoes-modernas.md
 ┣ 📜 32-boas-praticas.md
 ┣ 📜 33-referencias.md
 ┣ 📜 CONTRIBUTING.md
 ┣ 📜 LICENSE
 ┣ 📜 README.md
```

---

🤝 Contribuindo

Contribuições são bem-vindas! Você pode:

- Sugerir melhorias.
- Corrigir erros.
- Adicionar exemplos.
- Atualizar conteúdo conforme novas versões do Spring e Java.

Veja o arquivo `CONTRIBUTING.md` (se aplicável) ou abra uma issue.

📜 Licença

Este projeto está licenciado sob a licença MIT. Consulte o arquivo <a href="https://github.com/Albertinesilva/spring-expert-roadmap/tree/main?tab=License-1-ov-file">`LICENSE`</a> para mais detalhes.



