# 25 — GraalVM e Spring Native

O **GraalVM** é uma máquina virtual de alto desempenho que permite executar aplicações Java com maior eficiência e gerar _native images_, resultando em inicialização ultrarrápida, menor consumo de memória e melhor adequação para arquiteturas cloud-native, serverless e microsserviços.

Com o **Spring Boot 3+**, o suporte a compilação nativa passou a ser oficial e integrado ao próprio ecossistema Spring (substituindo o antigo projeto _Spring Native_), permitindo gerar binários nativos com suporte completo da plataforma.

---

## 📌 Sumário

- [1. O que é o GraalVM](#1-o-que-é-o-graalvm)
- [2. Benefícios do Native Image](#2-benefícios-do-native-image)
- [3. GraalVM e Spring Boot](#3-graalvm-e-spring-boot)
- [4. Requisitos e Instalação](#4-requisitos-e-instalação)
- [5. Compilando Aplicações Spring em Native Image](#5-compilando-aplicações-spring-em-native-image)
- [6. Configurações de Reflexão, Recursos e Proxy](#6-configurações-de-reflexão-recursos-e-proxy)
- [7. Limitações e Cuidados](#7-limitações-e-cuidados)
- [8. Desempenho: JVM vs Native](#8-desempenho-jvm-vs-native)
- [9. Testes em Aplicações Nativas](#9-testes-em-aplicações-nativas)
- [10. Casos de Uso Ideais](#10-casos-de-uso-ideais)
- [11. Integração com Docker e Cloud](#11-integração-com-docker-e-cloud)
- [12. Boas Práticas](#12-boas-práticas)
- [13. Conclusão](#13-conclusão)

---

## 1. O que é o GraalVM

O **GraalVM** é uma plataforma de execução multi-linguagem (Java, Kotlin, Scala, JavaScript, Python, Ruby, R, entre outras), baseada na JVM e desenvolvida pela Oracle, que oferece:

- Compilação _Ahead-of-Time (AOT)_ para geração de binários nativos
- Melhor tempo de inicialização
- Menor consumo de memória
- Execução poliglota

O recurso mais relevante para aplicações Spring é o **Native Image**, que permite compilar aplicações Java diretamente em executáveis nativos.

---

## 2. Benefícios do Native Image

Ao compilar uma aplicação Java em um binário nativo:

| Característica          | JVM Tradicional | Native Image             |
| ----------------------- | --------------- | ------------------------ |
| Tempo de inicialização  | Segundos        | Milissegundos            |
| Consumo de memória      | Mais alto       | Significativamente menor |
| Tamanho do artefato     | Médio           | Maior (binário único)    |
| Escalabilidade elástica | Boa             | Excelente                |
| Uso em serverless       | Limitado        | Ideal                    |

---

## 3. GraalVM e Spring Boot

Desde o **Spring Boot 3**, o suporte ao GraalVM é:

- Oficial e integrado ao framework
- Baseado em processamento AOT
- Compatível com Spring Web, Data, Security, Batch, entre outros

O Spring realiza análises em tempo de build para gerar automaticamente grande parte das configurações necessárias para execução nativa.

---

## 4. Requisitos e Instalação

### ✔ Requisitos

- Java 17 ou superior
- GraalVM instalado
- Maven ou Gradle atualizados
- Docker (opcional, para build containerizado)

### ✔ Instalação via SDKMAN

```bash
sdk install java 23-graalce
sdk use java 23-graalce
```

### ✔ Verificação

```bash
java -version
native-image --version
```

---

## 5. Compilando Aplicações Spring em Native Image

### 🔹 Usando Spring Boot Maven Plugin

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <image>
          <builder>paketobuildpacks/builder-jammy-base</builder>
        </image>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### 🔹 Build da imagem nativa

```bash
mvn -Pnative spring-boot:build-image
```

Ou:

```bash
mvn -Pnative native:compile
```

O processo pode levar mais tempo que um build tradicional devido à compilação AOT.

---

## 6. Configurações de Reflexão, Recursos e Proxy

Aplicações nativas exigem configuração explícita para:

- Reflexão
- Carregamento de recursos
- Proxies dinâmicos

Exemplo de arquivo:

```
src/main/resources/META-INF/native-image/reflect-config.json
```

```json
[
  {
    "name": "com.exemplo.cliente.Cliente",
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true,
    "allDeclaredFields": true
  }
]
```

⚠️ Observação: No Spring Boot 3+, grande parte dessas configurações é gerada automaticamente durante o build AOT.

---

## 7. Limitações e Cuidados

Algumas limitações importantes:

❌ Uso intenso de reflexão dinâmica não declarada  
❌ Carregamento de classes por nome em tempo de execução  
❌ Bibliotecas incompatíveis com AOT  
❌ Dependência de `Unsafe`, JNI ou bytecode weaving dinâmico

➡️ É altamente recomendável testar exaustivamente o binário nativo antes de uso em produção.

---

## 8. Desempenho: JVM vs Native

### 🔹 JVM

- Melhor desempenho em execução prolongada
- JIT otimiza código ao longo do tempo
- Ideal para aplicações _long-running_

### 🔹 Native Image

- Inicialização extremamente rápida
- Menor consumo de memória
- Ideal para microsserviços, FaaS, CLI e workloads elásticos

A escolha depende do perfil da aplicação e da carga esperada.

---

## 9. Testes em Aplicações Nativas

É possível executar testes no modo nativo:

```bash
mvn -Pnative test
```

Considerações:

- Tempo de build maior
- Ajustes em testes de integração
- Verificação de compatibilidade de dependências externas

---

## 10. Casos de Uso Ideais

✔ Microsserviços cloud-native  
✔ Funções serverless (AWS Lambda, Azure Functions)  
✔ APIs com alta elasticidade  
✔ Ferramentas de linha de comando (CLI)  
✔ Edge computing

---

## 11. Integração com Docker e Cloud

### 🔹 Build de imagem nativa

```bash
mvn -Pnative spring-boot:build-image
```

### 🔹 Execução

```bash
docker run -p 8080:8080 minha-imagem:latest
```

### ✔ Vantagens

- Imagens menores
- Inicialização quase instantânea
- Redução de custos em ambientes escaláveis

---

## 12. Boas Práticas

✔ Utilize Spring Boot 3+  
✔ Evite reflexão dinâmica sem configuração explícita  
✔ Teste o binário nativo em ambientes reais  
✔ Utilize logs e perfis específicos para modo nativo  
✔ Prefira buildpacks para builds reprodutíveis  
✔ Documente dependências incompatíveis

---

## 13. Conclusão

O GraalVM, aliado ao suporte nativo do Spring Boot 3+, transforma aplicações Java em soluções modernas, rápidas e eficientes para ambientes cloud-native.

Embora existam limitações técnicas relacionadas a AOT e reflexão, os ganhos em tempo de inicialização, consumo de memória e escalabilidade tornam essa abordagem altamente recomendada para microsserviços, arquiteturas serverless e aplicações altamente elásticas.

---

<p align="center">
<b>Finalizada a GraalVM e Spring Native! 🏁</b><br>
  <b>Próximo Nível: 👉 </b> <a href="26-virtual-threads.md">Virtual Threads (Java 21+) no Ecossistema Spring</a>
</p>
