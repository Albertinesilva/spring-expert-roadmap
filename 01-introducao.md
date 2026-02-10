# 📘 Introdução

O Spring não é apenas um framework facilitador — ele é uma plataforma completa de desenvolvimento, arquitetura e operação de aplicações Java modernas. Embora sua proposta inicial tenha sido simplificar o desenvolvimento corporativo, hoje o Spring constitui um ecossistema robusto que exige compreensão profunda de seus mecanismos internos para ser utilizado com excelência.

Mais do que “fazer funcionar”, dominar Spring significa entender:

- Como o Container IoC gerencia ciclos de vida de objetos.
- Como o AOP é implementado por meio de proxies dinâmicos.
- Como transações, segurança, cache e eventos são aplicados sem código explícito.
- Como o Spring se comporta em ambientes Cloud Native, distribuídos e altamente concorrentes.

Este material tem como objetivo ir além da superfície, explorando o que acontece **por baixo do capô (under the hood)**, para formar uma base sólida tanto para:

- Desenvolvimento de aplicações monolíticas, modulares e distribuídas.
- Atuação em ambientes de produção, nuvem e microsserviços.
- Preparação para entrevistas técnicas de nível pleno, sênior e especialista.

---

## 🎯 Objetivos deste Material

Este repositório foi criado para:

- Servir como base de estudo contínuo sobre o ecossistema Spring.
- Funcionar como guia técnico de referência para projetos reais.
- Consolidar conhecimento avançado em arquitetura, performance, segurança e observabilidade.
- Documentar tanto os conceitos clássicos quanto as atualizações mais recentes do Spring Boot e Spring Framework.

---

## 🧠 Filosofia: Entender > Memorizar

Aqui não buscamos apenas decorar anotações como `@Autowired`, `@Transactional` ou `@RestController`, mas compreender:

- Por que elas funcionam.
- Quando elas falham.
- Como o Spring resolve esses problemas internamente.
- Quais trade-offs arquiteturais cada escolha implica.

### 🔍 Exemplo clássico

Um método anotado com `@Transactional` pode não funcionar quando chamado por outro método da mesma classe. Esse comportamento só se torna compreensível ao entender:

- O uso de proxies pelo Spring.
- O conceito de _self-invocation_.
- A separação entre chamadas externas ao proxy e chamadas internas ao objeto.

---

## 🧱 O Spring como Plataforma

O Spring hoje vai muito além de um framework web:

- 🧩 **Core & IoC** – Gerenciamento avançado de dependências e ciclo de vida.
- 🌐 **Web** – Spring MVC, WebFlux, REST, GraphQL.
- 🗄️ **Data** – JPA, JDBC, R2DBC, MongoDB, Redis, Neo4j.
- 🔐 **Security** – OAuth2, JWT, LDAP, Active Directory, Zero Trust.
- ⚙️ **Cloud Native** – Config Server, Service Discovery, Circuit Breakers, Kubernetes.
- 📊 **Observabilidade** – Micrometer, OpenTelemetry, tracing e métricas.
- 🧪 **Testes** – Test Slices, Testcontainers, testes de contrato.
- 🧠 **AI** – Integração com LLMs e serviços de IA via Spring AI.
- 🚀 **Performance** – GraalVM, CDS, Virtual Threads, otimização de startup.

Este material está organizado por módulos e domínios técnicos, refletindo essa visão de plataforma.

---

## 🔍 Público-Alvo

Este conteúdo é voltado para:

- Desenvolvedores Java que desejam sair do uso superficial do Spring.
- Profissionais que atuam ou desejam atuar com sistemas críticos, distribuídos ou de grande escala.
- Estudantes e arquitetos que buscam uma base técnica sólida, moderna e alinhada ao mercado.

---

## 🏁 Como Usar Este Material

- Utilize o menu principal para navegar entre os módulos.
- Cada seção é independente, mas o entendimento completo vem da visão integrada do ecossistema.
- Sempre que possível, os tópicos apresentarão:
  - Conceito
  - Funcionamento interno
  - Casos reais
  - Boas práticas
  - Armadilhas comuns
  - Exemplos práticos

---
