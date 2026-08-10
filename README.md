#  AWS Lambda — Word Count com S3 e SNS

##  Sobre o projeto

Este projeto foi desenvolvido como um **desafio prático de AWS Lambda**, com o objetivo de criar uma arquitetura orientada a eventos capaz de receber arquivos de texto no Amazon S3, processá-los automaticamente com uma função AWS Lambda e enviar o resultado da contagem de palavras por e-mail utilizando o Amazon SNS.

O laboratório demonstra, na prática, a integração entre serviços AWS e o desenvolvimento de uma aplicação **serverless**, sem necessidade de manter servidores para executar o processamento.

---

##  Objetivos

* Criar uma função **AWS Lambda** utilizando Python.
* Processar arquivos `.txt` armazenados no **Amazon S3**.
* Acionar automaticamente a Lambda quando um arquivo for enviado ao bucket.
* Contar a quantidade de palavras presentes no arquivo.
* Publicar o resultado em um tópico do **Amazon SNS**.
* Enviar o resultado da contagem por e-mail.
* Validar a solução por meio de testes com diferentes arquivos.

---

##  Arquitetura da solução

```text
                    ┌─────────────────┐
                    │   Arquivo .TXT  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Amazon S3      │
                    │                 │
                    │ Upload do arquivo│
                    └────────┬────────┘
                             │
                     Evento de criação
                             │
                             ▼
                    ┌─────────────────┐
                    │   AWS Lambda    │
                    │ WordCountFunction│
                    │                 │
                    │ Python          │
                    │                 │
                    │ Conta palavras  │
                    └────────┬────────┘
                             │
                         Publicação
                             │
                             ▼
                    ┌─────────────────┐
                    │   Amazon SNS    │
                    │ WordCountTopic  │
                    └────────┬────────┘
                             │
                             ▼
                         📧 E-mail
```

---

##  Serviços AWS utilizados

| **Serviço** | **Função no projeto** |                                              |
| --------------------- | -------------------------------------------------------------- |
| **AWS Lambda**        | Executa o processamento do arquivo sem necessidade de servidor |
| **Amazon S3**         | Armazena os arquivos `.txt` utilizados como entrada            |
| **Amazon SNS**        | Publica e distribui o resultado da contagem                    |
| **Amazon CloudWatch** | Permite monitorar e registrar a execução da Lambda             |
| **AWS IAM**           | Controla as permissões utilizadas pela função Lambda           |

---

##  Configuração da solução

### AWS Lambda

**Nome da função:**

```text
WordCountFunction
```

**Runtime:**

```text
Python 3.15
```

**Arquitetura:**

```text
x86_64
```

**IAM Role:**

```text
LambdaAccessRole
```

A função utiliza permissões para acessar os serviços necessários durante o processamento.

---

### Amazon S3

Foi criado um bucket S3 na mesma região da função Lambda.

O bucket funciona como ponto de entrada dos arquivos de texto.

Foi configurado um gatilho para que a Lambda seja executada automaticamente quando um objeto for criado.

**Filtro utilizado:**

```text
Sufixo: .txt
```

Dessa forma, arquivos de texto enviados ao bucket acionam automaticamente a função.

---

### Amazon SNS

Foi criado o tópico:

```text
WordCountTopic
```

Uma assinatura de e-mail foi configurada e confirmada para receber os resultados processados pela Lambda.

**Assunto das mensagens:**

```text
Word Count Result
```

**  Formato da mensagem:**

```text
O número de palavras no arquivo <nome-do-arquivo> é <quantidade>.
```

---

##  Código da Lambda

A função utiliza **Python** e os SDKs da AWS para acessar o S3 e o SNS.

Principais etapas executadas pelo código:

1. Recebe o evento gerado pelo S3.
2. Identifica o bucket.
3. Identifica o arquivo enviado.
4. Faz a leitura do objeto armazenado no S3.
5. Converte o conteúdo para texto.
6. Conta as palavras utilizando `split()`.
7. Publica o resultado no Amazon SNS.
8. Retorna o resultado da execução.

Trecho principal da lógica:

```python
content = response['Body'].read().decode('utf-8')

word_count = len(content.split())

message = f"O número de palavras no arquivo {file_key} é {word_count}."
```

Publicação no SNS:

```python
sns.publish(
    TopicArn=SNS_TOPIC_ARN,
    Subject="Word Count Result",
    Message=message
)
```

---

##  Testes realizados

Foram realizados testes utilizando arquivos `.txt` com diferentes quantidades de palavras.

## Testes realizados

Foram realizados testes utilizando arquivos `.txt` com diferentes quantidades de palavras.

### Teste de validação

Arquivo:

`teste10.txt`

### Teste  — Validação

Arquivo:

```text
teste10.txt
```

Conteúdo:

```text
um dois tres quatro cinco seis sete oito nove dez
```

Quantidade esperada:

```text
10 palavras
```

Resultado recebido por e-mail:

```text
O número de palavras no arquivo teste10.txt é 10.
```

 **  Resultado validado com sucesso.**

---

##  Segurança e permissões

O projeto utilizou uma função IAM disponibilizada pelo ambiente do laboratório:

```text
LambdaAccessRole
```

Essa função fornece as permissões necessárias para que a Lambda interaja com os serviços utilizados no exercício.

O bucket S3 foi configurado com:

* ACLs desabilitadas;
* Bloqueio de acesso público;
* Criptografia SSE-S3;
* Sem necessidade de acesso público aos objetos.

---

##  Conceitos praticados

Durante o laboratório foram praticados conceitos importantes de Cloud Computing e AWS:

* Serverless Computing
* Event-Driven Architecture
* AWS Lambda
* Amazon S3
* Amazon SNS
* AWS IAM
* Amazon CloudWatch
* Integração entre serviços AWS
* Triggers/Event Notifications
* Processamento automático de arquivos
* Python com Boto3
* Controle de permissões
* Monitoramento de funções serverless

---

##  O que aprendi

Este laboratório permitiu compreender, na prática, como diferentes serviços da AWS podem ser integrados para construir uma solução automatizada e orientada a eventos.

Um dos principais aprendizados foi entender que uma aplicação serverless pode reagir automaticamente a eventos. Nesse projeto, o upload de um arquivo no S3 funciona como gatilho para iniciar todo o processamento.

Também foi possível praticar o uso do **Boto3**, leitura de objetos no S3, publicação de mensagens no SNS e gerenciamento de permissões através do IAM.

---

##  Possíveis melhorias

A solução pode ser evoluída futuramente com:

* Tratamento de erros e exceções;
* Validação do tipo do arquivo;
* Logs estruturados no CloudWatch;
* Dead Letter Queue (DLQ);
* Amazon SQS;
* Infraestrutura como código utilizando AWS CloudFormation ou Terraform;
* Monitoramento e métricas personalizadas;
* Processamento de arquivos maiores;
* Armazenamento dos resultados em banco de dados;
* Dashboard de monitoramento;
* Pipeline CI/CD para implantação da Lambda.

---

##  Evidências

### Função AWS Lambda

> Adicione aqui uma captura de tela da `WordCountFunction`.

```text
docs/images/lambda-function.png
```

### Arquitetura

> Adicione aqui uma captura de tela mostrando a integração S3 → Lambda.

```text
docs/images/aws-architecture.png
```

### Resultado do teste

> Adicione aqui uma captura do e-mail recebido pelo Amazon SNS.

```text
docs/images/sns-result.png
```

---

##  Contexto do laboratório

Projeto desenvolvido como parte da formação prática em **AWS Cloud Computing**, com foco em serviços serverless, integração entre serviços e desenvolvimento de soluções orientadas a eventos.

**Formação:** AWS re/Start — Escola da Nuvem

---

##  Autor

**Marcelo Gomes**

 Análise e Desenvolvimento de Sistemas
 AWS Cloud Computing
 Python
 Git/GitHub
 Data & Cloud

### 🔗 Contato

* LinkedIn: [linkedin.com/in/marcelogsouza](https://www.linkedin.com/in/marcelogsouza/)

* GitHub: [github.com/marcelogomestech](https://github.com/marcelogomestech)

---


⭐ **Se este projeto foi útil para você, considere deixar uma estrela no repositório!**
