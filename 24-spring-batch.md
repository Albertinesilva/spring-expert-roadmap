# 24 — Spring Batch

O **Spring Batch** é um framework do ecossistema Spring voltado para processamento em lote (_batch processing_), ideal para tarefas como importação/exportação de dados, geração de relatórios, integração entre sistemas, migração de bases e processamento massivo de registros com confiabilidade, escalabilidade e controle transacional.

---

## 📌 Sumário

- [1. O que é Processamento em Lote](#1-o-que-é-processamento-em-lote)
- [2. Arquitetura do Spring Batch](#2-arquitetura-do-spring-batch)
- [3. Componentes Principais](#3-componentes-principais)
- [4. Estrutura de um Job](#4-estrutura-de-um-job)
- [5. ItemReader, ItemProcessor e ItemWriter](#5-itemreader-itemprocessor-e-itemwriter)
- [6. Configuração Básica](#6-configuração-básica)
- [7. Jobs com Múltiplos Steps](#7-jobs-com-múltiplos-steps)
- [8. Controle Transacional e Commit Interval](#8-controle-transacional-e-commit-interval)
- [9. Execução, Reinício e Parâmetros](#9-execução-reinício-e-parâmetros)
- [10. Tratamento de Erros e Retentativas](#10-tratamento-de-erros-e-retentativas)
- [11. Leitura e Escrita de Arquivos](#11-leitura-e-escrita-de-arquivos)
- [12. Processamento Paralelo e Particionado](#12-processamento-paralelo-e-particionado)
- [13. Monitoramento e Metadados](#13-monitoramento-e-metadados)
- [14. Integração com Spring Boot](#14-integração-com-spring-boot)
- [15. Testes em Spring Batch](#15-testes-em-spring-batch)
- [16. Casos de Uso Comuns](#16-casos-de-uso-comuns)
- [17. Boas Práticas](#17-boas-práticas)
- [18. Conclusão](#18-conclusão)

---

## 1. O que é Processamento em Lote

Processamento em lote é a execução de grandes volumes de dados de forma não interativa, geralmente:

- Fora do horário de pico
- De forma automatizada
- Com garantias de integridade e reprocessamento

### Exemplos

- Importação de dados de CSV
- Processamento de folha de pagamento
- Geração de faturas
- Sincronização entre bancos de dados

---

## 2. Arquitetura do Spring Batch

A arquitetura do Spring Batch é baseada em:

- **Job** → Unidade lógica de trabalho
- **Step** → Fase dentro de um Job
- **Processamento orientado a itens (Item-Oriented Processing)** → Leitura, processamento e escrita em blocos (_chunks_)
- **Repositório de metadados** → Armazena status, execuções e falhas

### Diagrama Conceitual

```
Job
├── Step 1
│   ├── ItemReader
│   ├── ItemProcessor
│   └── ItemWriter
└── Step 2
    ├── ItemReader
    ├── ItemProcessor
    └── ItemWriter
```

---

## 3. Componentes Principais

### 🔹 Job

Representa uma execução batch completa.

### 🔹 Step

Fase do Job, normalmente baseada em processamento orientado a itens.

### 🔹 ItemReader

Responsável por ler dados da origem (arquivo, banco, API, etc.).

### 🔹 ItemProcessor

Aplica transformação, validação ou regra de negócio.

### 🔹 ItemWriter

Responsável por persistir, exportar ou enviar os dados processados.

### 🔹 JobRepository

Armazena metadados de execução (status, parâmetros, falhas).

### 🔹 JobLauncher

Responsável por iniciar Jobs.

---

## 4. Estrutura de um Job

Exemplo básico:

```java
@Bean
public Job job(JobRepository jobRepository, Step step) {
    return new JobBuilder("meuJob", jobRepository)
            .start(step)
            .build();
}
```

---

## 5. ItemReader, ItemProcessor e ItemWriter

### 📥 ItemReader

```java
@Bean
public FlatFileItemReader<Cliente> reader() {
    FlatFileItemReader<Cliente> reader = new FlatFileItemReader<>();
    reader.setResource(new ClassPathResource("clientes.csv"));
    reader.setLineMapper(lineMapper());
    return reader;
}
```

### 🔄 ItemProcessor

```java
@Bean
public ItemProcessor<Cliente, Cliente> processor() {
    return cliente -> {
        cliente.setNome(cliente.getNome().toUpperCase());
        return cliente;
    };
}
```

### 📤 ItemWriter

```java
@Bean
public JdbcBatchItemWriter<Cliente> writer(DataSource dataSource) {
    return new JdbcBatchItemWriterBuilder<Cliente>()
            .dataSource(dataSource)
            .sql("INSERT INTO cliente (nome, email) VALUES (:nome, :email)")
            .beanMapped()
            .build();
}
```

---

## 6. Configuração Básica

### Dependência Maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

### Classe de Configuração

```java
@Configuration
@EnableBatchProcessing
public class BatchConfig {
}
```

---

## 7. Jobs com Múltiplos Steps

```java
@Bean
public Job job(JobRepository jobRepository, Step step1, Step step2) {
    return new JobBuilder("jobComDoisSteps", jobRepository)
            .start(step1)
            .next(step2)
            .build();
}
```

---

## 8. Controle Transacional e Commit Interval

O processamento é feito em blocos (_chunks_).

```java
@Bean
public Step step(JobRepository jobRepository,
                 PlatformTransactionManager transactionManager,
                 ItemReader<Cliente> reader,
                 ItemProcessor<Cliente, Cliente> processor,
                 ItemWriter<Cliente> writer) {

    return new StepBuilder("stepClientes", jobRepository)
            .<Cliente, Cliente>chunk(10, transactionManager)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
}
```

➡️ A cada 10 registros processados, ocorre um **commit** da transação.

---

## 9. Execução, Reinício e Parâmetros

### Parâmetros de Job

Execução via linha de comando:

```bash
java -jar app.jar nomeArquivo=clientes.csv data=2026-02-11
```

Acesso aos parâmetros:

```java
@Value("#{jobParameters['nomeArquivo']}")
private String nomeArquivo;
```

### Reinício Automático

Se um `Step` falhar, o Spring Batch pode reiniciá-lo a partir do ponto de falha, desde que o Job seja configurado como reiniciável.

---

## 10. Tratamento de Erros e Retentativas

### Retry e Skip

```java
@Bean
public Step step(JobRepository jobRepository,
                 PlatformTransactionManager transactionManager) {

    return new StepBuilder("stepComRetry", jobRepository)
            .<Cliente, Cliente>chunk(10, transactionManager)
            .reader(reader())
            .processor(processor())
            .writer(writer())
            .faultTolerant()
            .retry(Exception.class)
            .retryLimit(3)
            .skip(ValidationException.class)
            .skipLimit(5)
            .build();
}
```

- `retryLimit(3)` → Tenta novamente até 3 vezes.
- `skipLimit(5)` → Permite ignorar até 5 erros específicos.

---

## 11. Leitura e Escrita de Arquivos

### Leitura de CSV

```java
@Bean
public FlatFileItemReader<Cliente> reader() {
    FlatFileItemReader<Cliente> reader = new FlatFileItemReader<>();
    reader.setResource(new FileSystemResource("clientes.csv"));
    reader.setLinesToSkip(1);
    reader.setLineMapper(lineMapper());
    return reader;
}
```

### Escrita em CSV

```java
@Bean
public FlatFileItemWriter<Cliente> writer() {
    FlatFileItemWriter<Cliente> writer = new FlatFileItemWriter<>();
    writer.setResource(new FileSystemResource("saida.csv"));
    writer.setLineAggregator(cliente ->
        cliente.getNome() + ";" + cliente.getEmail()
    );
    return writer;
}
```

---

## 12. Processamento Paralelo e Particionado

### Multithreaded Step

```java
@Bean
public Step stepParalelo(JobRepository jobRepository,
                         PlatformTransactionManager transactionManager,
                         TaskExecutor taskExecutor) {

    return new StepBuilder("stepParalelo", jobRepository)
            .<Cliente, Cliente>chunk(10, transactionManager)
            .reader(reader())
            .processor(processor())
            .writer(writer())
            .taskExecutor(taskExecutor)
            .throttleLimit(5)
            .build();
}
```

### Particionamento (Partitioning)

Divide os dados em múltiplas partições para processamento paralelo ou distribuído, melhorando escalabilidade em grandes volumes.

---

## 13. Monitoramento e Metadados

O Spring Batch mantém tabelas como:

- `BATCH_JOB_INSTANCE`
- `BATCH_JOB_EXECUTION`
- `BATCH_STEP_EXECUTION`
- `BATCH_JOB_EXECUTION_PARAMS`

Essas tabelas permitem:

- Auditoria
- Reexecução
- Monitoramento
- Diagnóstico de falhas

---

## 14. Integração com Spring Boot

Com Spring Boot:

- Jobs podem ser iniciados automaticamente na inicialização.
- Ou manualmente via `JobLauncher`.

### Desabilitar execução automática

```properties
spring.batch.job.enabled=false
```

### Executar via código

```java
jobLauncher.run(job, new JobParametersBuilder()
        .addString("data", LocalDate.now().toString())
        .toJobParameters());
```

---

## 15. Testes em Spring Batch

### Dependência

```xml
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Teste de Job

```java
@SpringBatchTest
@SpringBootTest
class JobTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    void deveExecutarJobComSucesso() throws Exception {
        JobExecution execution = jobLauncherTestUtils.launchJob();
        assertEquals(BatchStatus.COMPLETED, execution.getStatus());
    }
}
```

---

## 16. Casos de Uso Comuns

- Importação de arquivos CSV/Excel
- Migração de bancos de dados
- Geração de relatórios em massa
- Processamento de logs
- Consolidação de dados
- Integração entre sistemas legados

---

## 17. Boas Práticas

✔ Utilize _chunks_ adequados para evitar estouro de memória  
✔ Implemente retry e skip para tolerância a falhas  
✔ Use parâmetros para criar Jobs reutilizáveis  
✔ Separe Jobs por responsabilidade  
✔ Monitore execuções via banco ou dashboards  
✔ Evite lógica excessivamente complexa no `ItemProcessor`  
✔ Teste Steps individualmente

---

## 18. Conclusão

O Spring Batch é a solução padrão do ecossistema Spring para processamento em lote robusto, confiável e escalável. Ele fornece:

- Controle transacional
- Reprocessamento automático
- Monitoramento detalhado
- Integração simples com bancos, arquivos, APIs e mensageria

É especialmente indicado para sistemas corporativos, governamentais e acadêmicos que demandam processamento massivo, confiável e auditável.
