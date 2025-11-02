# 📚 Guia Completo de Serviços AWS

> Documentação detalhada sobre AWS Lambda, ECS, EKS, SNS, SQS e Step Functions

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)](https://aws.amazon.com/)

## 📑 Índice

- [AWS Lambda](#-aws-lambda)
- [Amazon ECS e EKS](#-amazon-ecs-e-eks)
- [Amazon SNS e SQS](#-amazon-sns-e-sqs)
- [AWS Step Functions](#-aws-step-functions)

---

## 🚀 AWS Lambda

### O que é AWS Lambda?

AWS Lambda é um serviço de computação **serverless** que permite executar código sem provisionar ou gerenciar servidores. Você paga apenas pelo tempo de computação que utiliza.

### Características Principais

- ⚡ **Execução sob demanda**: Código é executado apenas quando necessário
- 💰 **Modelo de cobrança**: Pay-per-use (por milissegundo de execução)
- 🔄 **Escalabilidade automática**: Escala automaticamente conforme a demanda
- 🛠️ **Suporte a múltiplas linguagens**: Python, Node.js, Java, Go, Ruby, .NET

### Como Funciona?

1. **Upload do código**: Você faz upload do seu código ou implementa diretamente no console
2. **Definição de trigger**: Configura eventos que disparam a função (API Gateway, S3, DynamoDB, etc.)
3. **Execução automática**: Lambda executa o código quando o evento ocorre
4. **Gerenciamento automático**: AWS cuida de toda infraestrutura

### Casos de Uso

```
✅ APIs REST e GraphQL
✅ Processamento de arquivos (imagens, vídeos, documentos)
✅ Processamento de streams de dados
✅ Backends para aplicações web e mobile
✅ Automação de tarefas e ETL
✅ Chatbots e assistentes virtuais
```

### Limites Importantes

| Recurso | Limite |
|---------|--------|
| Timeout máximo | 15 minutos |
| Memória | 128 MB - 10 GB |
| Tamanho do pacote | 50 MB (zipado), 250 MB (extraído) |
| Concorrência padrão | 1000 execuções simultâneas |

### Exemplo Básico (Python)

```python
import json

def lambda_handler(event, context):
    # Seu código aqui
    name = event.get('name', 'World')
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Hello, {name}!')
    }
```

### Boas Práticas

- 🎯 Mantenha funções pequenas e focadas (Single Responsibility)
- ♻️ Reutilize conexões entre invocações
- 📦 Minimize dependências externas
- 🔐 Use variáveis de ambiente para configurações sensíveis
- 📊 Implemente logging adequado com CloudWatch
- ⚠️ Trate erros apropriadamente

---

## 🐳 Amazon ECS e EKS

### Amazon ECS (Elastic Container Service)

#### O que é?

Serviço de orquestração de containers **totalmente gerenciado** pela AWS, que facilita executar, parar e gerenciar containers Docker.

#### Componentes Principais

- **Cluster**: Agrupamento lógico de tasks ou serviços
- **Task Definition**: Blueprint que descreve como um container deve ser executado
- **Service**: Mantém e escala um número específico de tasks
- **Task**: Instância de uma task definition rodando em um cluster

#### Modos de Lançamento

**1. EC2 Launch Type**
```
- Você gerencia as instâncias EC2
- Maior controle sobre infraestrutura
- Requer configuração de Auto Scaling
```

**2. Fargate Launch Type**
```
- Serverless (AWS gerencia infraestrutura)
- Você define CPU e memória necessárias
- Mais simples, sem gerenciamento de servidores
```

#### Quando Usar ECS?

✅ Forte integração com ecossistema AWS  
✅ Menor curva de aprendizado  
✅ Aplicações containerizadas simples a médias  
✅ Quer simplicidade sem Kubernetes  

### Amazon EKS (Elastic Kubernetes Service)

#### O que é?

Serviço gerenciado que facilita executar **Kubernetes** na AWS sem necessidade de instalar e operar seu próprio control plane.

#### Arquitetura

```
┌─────────────────────────────────────┐
│     EKS Control Plane (Gerenciado)  │
│  - API Server                       │
│  - Scheduler                        │
│  - Controller Manager               │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│     Worker Nodes (Você gerencia)    │
│  - EC2 Instances ou Fargate         │
│  - Pods rodando containers          │
└─────────────────────────────────────┘
```

#### Componentes Kubernetes

- **Pod**: Menor unidade deployável (1 ou mais containers)
- **Deployment**: Gerencia réplicas de Pods
- **Service**: Expõe Pods para rede
- **Namespace**: Isolamento lógico de recursos
- **ConfigMap/Secrets**: Configurações e dados sensíveis

#### Quando Usar EKS?

✅ Já usa Kubernetes on-premises  
✅ Precisa de portabilidade multi-cloud  
✅ Aplicações complexas com microsserviços  
✅ Requer recursos avançados do Kubernetes  

### ECS vs EKS - Comparação

| Aspecto | ECS | EKS |
|---------|-----|-----|
| **Complexidade** | Mais simples | Mais complexo |
| **Curva de aprendizado** | Menor | Maior |
| **Portabilidade** | Específico AWS | Multi-cloud |
| **Ecossistema** | AWS nativo | Kubernetes nativo |
| **Custo** | Sem custo extra | ~$0.10/hora por cluster |
| **Casos de uso** | Apps AWS-first | Apps que precisam K8s |

### Exemplo Task Definition (ECS)

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "my-container",
      "image": "my-app:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ]
    }
  ]
}
```

---

## 📨 Amazon SNS e SQS

### Comunicação Assíncrona na AWS

Ambos os serviços permitem **desacoplar componentes** de aplicações distribuídas, mas funcionam de maneiras diferentes.

### Amazon SNS (Simple Notification Service)

#### O que é?

Serviço de **mensageria pub/sub** (publish-subscribe) totalmente gerenciado para mensagens de aplicação-para-aplicação (A2A) e aplicação-para-pessoa (A2P).

#### Modelo Pub/Sub

```
        ┌─────────┐
        │Publisher│
        └────┬────┘
             │
             ▼
        ┌────────┐
        │  SNS   │
        │ Topic  │
        └───┬────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
  [Sub1]  [Sub2]  [Sub3]
  Lambda   SQS    Email
```

#### Características

- 📢 **Fan-out**: Uma mensagem para múltiplos subscribers
- ⚡ **Push**: Entrega proativa de mensagens
- 🎯 **Filtros**: Subscribers recebem apenas mensagens relevantes
- 📱 **Múltiplos protocolos**: HTTP/S, Email, SMS, Lambda, SQS

#### Casos de Uso

```
✅ Notificações em tempo real
✅ Alertas de sistema e monitoramento
✅ Broadcast de eventos para múltiplos serviços
✅ Notificações push mobile
✅ Campanhas de email/SMS em massa
```

### Amazon SQS (Simple Queue Service)

#### O que é?

Serviço de **filas de mensagens** totalmente gerenciado que permite desacoplar e escalar microsserviços, sistemas distribuídos e aplicações serverless.

#### Modelo de Fila

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│Producer 1│────▶│   SQS   │────▶│Consumer 1│
└──────────┘     │  Queue  │     └──────────┘
┌──────────┐     │         │     ┌──────────┐
│Producer 2│────▶│         │────▶│Consumer 2│
└──────────┘     └─────────┘     └──────────┘
```

#### Tipos de Fila

**Standard Queue**
- ⚡ Throughput ilimitado
- 🔄 Entrega "at-least-once" (pode duplicar)
- 🔀 Ordem best-effort (não garantida)

**FIFO Queue**
- 📊 300 mensagens/segundo (3000 com batching)
- ✅ Entrega "exactly-once"
- 🔢 Ordem estritamente preservada

#### Características Importantes

- 🕐 **Visibility Timeout**: Tempo que mensagem fica invisível após ser lida
- ⏰ **Message Retention**: 1 minuto a 14 dias (padrão: 4 dias)
- 📦 **Tamanho máximo**: 256 KB por mensagem
- ⏱️ **Delay Queues**: Atrasa entrega de mensagens
- 💀 **Dead Letter Queue**: Fila para mensagens não processadas

#### Casos de Uso

```
✅ Processamento assíncrono de tarefas
✅ Buffer entre componentes
✅ Gerenciamento de carga de trabalho
✅ Processamento em lote (batch)
✅ Desacoplamento de microsserviços
```

### SNS + SQS: Fan-out Pattern

Um padrão comum é usar SNS para distribuir para múltiplas filas SQS:

```
          ┌────────┐
          │   SNS  │
          │  Topic │
          └───┬────┘
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
   [SQS 1] [SQS 2] [SQS 3]
   Orders  Billing  Analytics
      │       │       │
      ▼       ▼       ▼
  Lambda  Lambda  Lambda
```

**Vantagens:**
- Processamento independente e escalável
- Retry isolado por serviço
- Diferentes velocidades de processamento

### SNS vs SQS - Comparação

| Aspecto | SNS | SQS |
|---------|-----|-----|
| **Modelo** | Pub/Sub | Point-to-Point |
| **Entrega** | Push (proativa) | Pull (consumer busca) |
| **Consumidores** | Múltiplos | Geralmente único por mensagem |
| **Persistência** | Não persiste | Persiste até ser deletada |
| **Caso de uso** | Notificações broadcast | Processamento assíncrono |

### Exemplo: Publicar no SNS (Python/Boto3)

```python
import boto3

sns = boto3.client('sns')

response = sns.publish(
    TopicArn='arn:aws:sns:us-east-1:123456789:my-topic',
    Message='Hello from SNS!',
    Subject='Test Message'
)
```

### Exemplo: Consumir do SQS (Python/Boto3)

```python
import boto3

sqs = boto3.client('sqs')
queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789/my-queue'

# Receber mensagens
response = sqs.receive_message(
    QueueUrl=queue_url,
    MaxNumberOfMessages=10,
    WaitTimeSeconds=20  # Long polling
)

for message in response.get('Messages', []):
    # Processar mensagem
    print(message['Body'])
    
    # Deletar mensagem após processar
    sqs.delete_message(
        QueueUrl=queue_url,
        ReceiptHandle=message['ReceiptHandle']
    )
```

---

## 🔄 AWS Step Functions

### O que é?

AWS Step Functions é um serviço de **orquestração serverless** que permite coordenar múltiplos serviços AWS em workflows visuais, facilitando a construção de aplicações distribuídas e pipelines de dados.

### Conceitos Principais

#### State Machine
Definição do workflow usando **Amazon States Language** (JSON). Cada state machine é composta de estados que realizam trabalho, tomam decisões ou aguardam.

#### Estados (States)

| Tipo de Estado | Descrição | Ícone |
|----------------|-----------|-------|
| **Task** | Executa trabalho (Lambda, ECS, etc) | 🔧 |
| **Choice** | Decisão condicional (if/else) | 🔀 |
| **Parallel** | Executa branches em paralelo | ⚡ |
| **Wait** | Aguarda por tempo fixo ou timestamp | ⏰ |
| **Pass** | Passa input para output (teste/debug) | ➡️ |
| **Succeed** | Termina execução com sucesso | ✅ |
| **Fail** | Termina execução com erro | ❌ |
| **Map** | Itera sobre array de itens | 🔁 |

### Tipos de Workflow

**Standard Workflows**
```
⏱️  Duração: Até 1 ano
💰 Cobrança: Por transição de estado
🔄 Execuções: Exactly-once
📊 Histórico: Completo
🎯 Ideal para: Processos longos e complexos
```

**Express Workflows**
```
⏱️  Duração: Até 5 minutos
💰 Cobrança: Por execuções e duração
🔄 Execuções: At-least-once
📊 Histórico: CloudWatch Logs
🎯 Ideal para: Alta taxa de eventos
```

### Estrutura Básica (Amazon States Language)

```json
{
  "Comment": "Exemplo de State Machine",
  "StartAt": "ProcessarPedido",
  "States": {
    "ProcessarPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:ProcessOrder",
      "Next": "VerificarEstoque"
    },
    "VerificarEstoque": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:CheckStock",
      "Next": "TemEstoque?"
    },
    "TemEstoque?": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.estoque",
          "BooleanEquals": true,
          "Next": "CobrarCartao"
        }
      ],
      "Default": "EstoqueInsuficiente"
    },
    "CobrarCartao": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:ChargeCard",
      "Next": "EnviarPedido"
    },
    "EnviarPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:ShipOrder",
      "End": true
    },
    "EstoqueInsuficiente": {
      "Type": "Fail",
      "Error": "OutOfStock",
      "Cause": "Produto indisponível em estoque"
    }
  }
}
```

### Tratamento de Erros

Step Functions oferece mecanismos robustos de error handling:

#### Retry
```json
"Retry": [
  {
    "ErrorEquals": ["States.Timeout"],
    "IntervalSeconds": 3,
    "MaxAttempts": 2,
    "BackoffRate": 2.0
  }
]
```

#### Catch
```json
"Catch": [
  {
    "ErrorEquals": ["States.ALL"],
    "Next": "TratarErro"
  }
]
```

### Integrações Diretas

Step Functions se integra diretamente com mais de 220 serviços AWS:

```
✅ AWS Lambda (executar funções)
✅ Amazon ECS/Fargate (executar containers)
✅ AWS Batch (jobs em batch)
✅ Amazon DynamoDB (operações de banco)
✅ Amazon SNS/SQS (envio de mensagens)
✅ AWS Glue (ETL jobs)
✅ Amazon SageMaker (ML training/inference)
✅ Amazon EMR (big data processing)
```

### Casos de Uso Comuns

#### 1. Pipeline de Processamento de Dados
```
Extract → Transform → Validate → Load → Notify
```

#### 2. Workflow de Aprovação
```
Submit → Manager Approval → Director Approval → Execute
```

#### 3. E-commerce Order Processing
```
Validate Order → Check Inventory → Process Payment → Ship → Send Notification
```

#### 4. Machine Learning Pipeline
```
Prepare Data → Train Model → Evaluate → Deploy if Good → Monitor
```

### Exemplo: Workflow de Processamento de Imagem

```json
{
  "Comment": "Pipeline de processamento de imagem",
  "StartAt": "ValidarImagem",
  "States": {
    "ValidarImagem": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ValidateImage",
      "Next": "ProcessamentoParalelo"
    },
    "ProcessamentoParalelo": {
      "Type": "Parallel",
      "Next": "SalvarResultados",
      "Branches": [
        {
          "StartAt": "GerarThumbnail",
          "States": {
            "GerarThumbnail": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:...:function:CreateThumbnail",
              "End": true
            }
          }
        },
        {
          "StartAt": "ExtrairMetadata",
          "States": {
            "ExtrairMetadata": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:...:function:ExtractMetadata",
              "End": true
            }
          }
        },
        {
          "StartAt": "DetectarObjetos",
          "States": {
            "DetectarObjetos": {
              "Type": "Task",
              "Resource": "arn:aws:states:::aws-sdk:rekognition:detectLabels",
              "End": true
            }
          }
        }
      ]
    },
    "SalvarResultados": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:putItem",
      "End": true
    }
  }
}
```

### Vantagens do Step Functions

✅ **Visual**: Interface gráfica para visualizar workflows  
✅ **Serverless**: Sem servidores para gerenciar  
✅ **Escalável**: Automaticamente escala com demanda  
✅ **Confiável**: Retry automático e tratamento de erros  
✅ **Auditável**: Histórico completo de execuções  
✅ **Integrações**: Conecta facilmente serviços AWS  

### Boas Práticas

- 🎯 Use workflows Express para alta frequência e baixa latência
- 🔄 Implemente retry logic apropriado
- 📊 Use Parallel state para operações independentes
- 🔐 Aplique princípio de least privilege nas IAM roles
- 📝 Adicione logging adequado em cada estado
- ⚠️ Use Catch para tratamento de erros gracioso
- 💾 Mantenha payloads pequenos (< 256 KB)

### Monitoramento

Step Functions se integra com:
- **CloudWatch**: Métricas e logs
- **X-Ray**: Tracing distribuído
- **EventBridge**: Eventos de execução

---

## 📚 Recursos Adicionais

### Documentação Oficial
- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [Amazon ECS](https://docs.aws.amazon.com/ecs/)
- [Amazon EKS](https://docs.aws.amazon.com/eks/)
- [Amazon SNS](https://docs.aws.amazon.com/sns/)
- [Amazon SQS](https://docs.aws.amazon.com/sqs/)
- [AWS Step Functions](https://docs.aws.amazon.com/step-functions/)

### Certificações AWS Relevantes
- ☁️ AWS Certified Solutions Architect
- 🔧 AWS Certified Developer
- 🏗️ AWS Certified SysOps Administrator

### Ferramentas Úteis
- **AWS CLI**: Interface de linha de comando
- **AWS SAM**: Framework para aplicações serverless
- **Terraform**: Infrastructure as Code
- **LocalStack**: Simula serviços AWS localmente

---

<div align="center">

⭐ Se este guia foi útil, considere dar uma estrela!

[⬆ Voltar ao topo](#-guia-completo-de-serviços-aws)

</div>
