# 29 — Aplicações de Linha de Comando (Spring Shell e Spring Boot CLI)

O ecossistema Spring oferece suporte robusto para **aplicações de linha de comando (CLI)**, tanto para a criação de ferramentas interativas quanto para automação, DevOps e operações administrativas.

Os principais recursos incluem:

- **Spring Shell** — para criação de CLIs interativas e extensíveis.
- **Spring Boot CLI** — ferramenta oficial para prototipagem rápida e execução de aplicações Spring Boot via scripts.

---

## 📌 Sumário

- [1. O papel das aplicações CLI no ecossistema Spring](#1-o-papel-das-aplicações-cli-no-ecossistema-spring)
- [2. Spring Shell](#2-spring-shell)
- [3. Criação de comandos com Spring Shell](#3-criação-de-comandos-com-spring-shell)
- [4. Parâmetros, opções e validação](#4-parâmetros-opções-e-validação)
- [5. Comandos condicionais e disponibilidade](#5-comandos-condicionais-e-disponibilidade)
- [6. Auto-complete, help e UX](#6-auto-complete-help-e-ux)
- [7. Spring Boot CLI](#7-spring-boot-cli)
- [8. Execução de scripts com Spring Boot CLI](#8-execução-de-scripts-com-spring-boot-cli)
- [9. Integração com pipelines e automação](#9-integração-com-pipelines-e-automação)
- [10. Testes de aplicações CLI](#10-testes-de-aplicações-cli)
- [11. Casos de uso corporativos](#11-casos-de-uso-corporativos)
- [12. Boas práticas](#12-boas-práticas)
- [13. Limitações e considerações](#13-limitações-e-considerações)
- [14. Conclusão](#14-conclusão)

---

## 1. O papel das aplicações CLI no ecossistema Spring

Aplicações CLI são fundamentais para:

- Automação de tarefas administrativas
- Ferramentas internas de suporte
- Scripts de migração e manutenção
- Operações de infraestrutura
- Prototipagem rápida de serviços

No contexto corporativo, CLIs reduzem a dependência de interfaces gráficas e permitem integração direta com pipelines de CI/CD.

---

## 2. Spring Shell

O **Spring Shell** é um framework para criação de shells interativos baseados em Spring Boot, oferecendo:

- Registro automático de comandos via anotações
- Auto-complete
- Help automático
- Integração com IoC, AOP, segurança e observabilidade

---

## 3. Criação de comandos com Spring Shell

### ✔ Exemplo básico

```java
@ShellComponent
public class PedidoCommands {

    @ShellMethod(key = "buscar-pedido", value = "Busca um pedido pelo ID.")
    public String buscarPedido(@ShellOption(help = "ID do pedido") Long id) {
        return "Pedido encontrado: " + id;
    }
}
```

Ao iniciar a aplicação:

```bash
buscar-pedido --id 10
```

---

## 4. Parâmetros, opções e validação

```java
@ShellMethod("Cria um novo usuário.")
public String criarUsuario(
    @ShellOption String nome,
    @ShellOption(defaultValue = "USER") String role,
    @ShellOption(help = "Email do usuário") @Email String email) {
    return "Usuário criado: " + nome;
}
```

### Recursos disponíveis

- Suporte a Bean Validation
- Valores padrão (defaultValue)
- Conversão automática de tipos
- Mensagens de erro padronizadas

---

## 5. Comandos condicionais e disponibilidade

```java
@ShellMethod("Executa tarefa crítica.")
@ShellMethodAvailability("isDisponivel")
public String tarefaCritica() {
    return "Executando...";
}

public Availability isDisponivel() {
    return sistemaAtivo
        ? Availability.available()
        : Availability.unavailable("Sistema em manutenção.");
}
```

Permite ativar ou desativar comandos dinamicamente com base no estado da aplicação.

---

## 6. Auto-complete, help e UX

O Spring Shell fornece:

- Auto-complete de comandos e opções
- Help contextual
- Sugestões dinâmicas
- Suporte a cores e formatação personalizada

Exemplos:

```bash
help
help buscar-pedido
```

---

## 7. Spring Boot CLI

O **Spring Boot CLI** é uma ferramenta voltada para:

- Executar aplicações Spring Boot via scripts Groovy
- Prototipar rapidamente serviços REST
- Gerenciar dependências automaticamente

Ele é ideal para experimentação, prototipagem e scripts rápidos.

---

## 8. Execução de scripts com Spring Boot CLI

### ✔ Exemplo simples

```groovy
@RestController
class App {
    @RequestMapping("/")
    String home() {
        "Olá, Spring Boot CLI!"
    }
}
```

Executar:

```bash
spring run app.groovy
```

Características:

- Sem build explícito
- Sem configuração manual de dependências
- Dependências inferidas automaticamente

---

## 9. Integração com pipelines e automação

Aplicações CLI Spring são ideais para:

- Scripts de migração de banco
- Provisionamento de infraestrutura
- Orquestração de processos
- Execução em pipelines CI/CD
- Operações de manutenção

Elas podem:

- Utilizar perfis (`spring.profiles.active`)
- Ler propriedades externas
- Integrar com Vault, AWS, GCP, Azure, etc.

---

## 10. Testes de aplicações CLI

### ✔ Teste de comandos Spring Shell

```java
@SpringBootTest
class PedidoCommandsTest {

    @Autowired
    Shell shell;

    @Test
    void deveBuscarPedido() {
        Object result = shell.evaluate(() -> "buscar-pedido --id 10");
        assertEquals("Pedido encontrado: 10", result);
    }
}
```

Benefícios:

- Testes automatizados de comandos
- Integração com o contexto Spring
- Validação de comportamento e UX

---

## 11. Casos de uso corporativos

- Ferramentas internas de suporte técnico
- Administração de sistemas distribuídos
- Gestão de usuários e permissões
- Automação de tarefas de negócio
- Operações financeiras, fiscais ou regulatórias

---

## 12. Boas práticas

- Modele comandos como contratos públicos
- Versione comandos e opções
- Utilize validação consistente
- Registre logs estruturados
- Integre com observabilidade
- Garanta idempotência quando necessário
- Forneça help claro e exemplos de uso

---

## 13. Limitações e considerações

- Interfaces CLI não substituem interfaces gráficas completas
- Exigem documentação clara
- A experiência do usuário depende da qualidade do design dos comandos
- Segurança deve ser rigorosa (credenciais, tokens, permissões)

---

## 14. Conclusão

O Spring oferece um ecossistema robusto para aplicações CLI, desde shells interativos com Spring Shell até execução de scripts com Spring Boot CLI.

Essas ferramentas ampliam o alcance do Spring além de APIs web, tornando-o uma plataforma completa para automação, operações, DevOps e desenvolvimento produtivo.

---

<p align="center">
<b>Finalizada a Aplicações de Linha de Comando (Spring Shell e Spring Boot CLI)! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="30-migracao-evolucao.md">Migração e Evolução (Spring Boot, Java e Jakarta EE)</a>
</p>
