# 🏆 Desafio DIO: Consolidando Workflows Automatizados com AWS Step Functions

Este repositório é a entrega do desafio de consolidação de workflows com AWS Step Functions da DIO. O objetivo foi **construir, executar e documentar** um fluxo de trabalho serverless para demonstrar a aplicação dos conceitos aprendidos e consolidar o perfil técnico.

---

## 💡 1. Objetivo e Caso de Uso do Workflow

### Cenário Abordado: Validação e Enriquecimento de Dados

O workflow implementado tem como objetivo orquestrar um fluxo de **processamento condicional de dados**.

1.  **Entrada:** Recebe um payload JSON (dados brutos).
2.  **Validação (`Choice`):** Verifica se os dados são válidos usando uma função AWS Lambda.
3.  **Fluxo de Sucesso:** Se validado, os dados são **enriquecidos** por outra Lambda e o fluxo é concluído.
4.  **Fluxo de Falha:** Se a validação falhar, o processo é encerrado com falha (`Fail State`), impedindo o processamento de dados inválidos.

---

## ⚙️ 2. Arquitetura e Componentes AWS

O projeto integra os seguintes serviços:

| Serviço | Função no Workflow | Tipo de State (ASL) |
| :--- | :--- | :--- |
| **AWS Step Functions** | Orquestrador principal e controle de estado. | State Machine |
| **AWS Lambda** | Executa a lógica de negócio (Validação e Enriquecimento). | Task State |
| **AWS IAM** | Gerencia as permissões para o Step Function invocar as Lambdas. | N/A |

---

## 💻 3. Implementação da State Machine (ASL)

Abaixo está a definição da Máquina de Estados em **Amazon States Language (ASL)**. Este código define a lógica e a transição entre os estados.

```json
{
  "Comment": "Workflow de Validação e Enriquecimento de Dados.",
  "StartAt": "Verificar Dados",
  "States": {

    "Verificar Dados": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:[SUA_LAMBDA_DE_VALIDACAO]"
      },
      "Catch": [
        {
          "ErrorEquals": [ "States.ALL" ],
          "Next": "Notificar Erro Inesperado"
        }
      ],
      "Next": "Decidir Proximo Passo"
    },

    "Decidir Proximo Passo": {
      "Type": "Choice",
      "Choices": [
        {
          // Condição que verifica o resultado retornado pela Lambda de 'Verificar Dados'
          "Variable": "$.Payload.status", 
          "StringEquals": "VALIDADO",
          "Next": "Enriquecer Dados"
        }
      ],
      // Se a condição acima não for atendida, o fluxo segue para falha
      "Default": "Notificar Falha de Validacao"
    },

    "Enriquecer Dados": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:[SUA_LAMBDA_DE_ENRIQUECIMENTO]"
      },
      "End": true
    },

    "Notificar Falha de Validacao": {
      "Type": "Fail",
      "Cause": "Dados de entrada não passaram na validação de formato. (Erro Tratado)",
      "Error": "ValidationFailed"
    },
    
    "Notificar Erro Inesperado": {
      "Type": "Fail",
      "Cause": "Ocorreu um erro não previsto durante a execução de uma Task. (Erro Catch)",
      "Error": "UnexpectedExecutionError"
    }
  }
}
```

## 🔍 4. Anotações e Insights Adquiridos (A Prova de Conceito)

Esta seção documenta a **experiência prática** e os **aprendizados técnicos** obtidos durante a construção e execução do Workflow, conforme solicitado no desafio, demonstrando a consolidação dos conceitos.

### 🧠 Aplicações de Conceitos Chave

Aqui estão os principais conceitos aplicados e o valor que trouxeram ao projeto:

* **Controle de Fluxo com `Choice State`:** O estado `Decidir Proximo Passo` exemplifica o poder do Step Functions para implementar **lógica condicional (if/else)**. Ele toma a decisão analisando o `output` (`$.Payload.status`) da função Lambda anterior, mantendo a **lógica de orquestração** claramente separada da **lógica de negócio**.
* **Robustez com `Catch`:** A utilização do bloco `Catch` no estado `Verificar Dados` foi crucial para demonstrar o **tratamento de erros (resiliência)**. Ele captura falhas de execução genéricas (`States.ALL`) e as direciona para um estado de `Fail` controlado, garantindo que o *workflow* não trave de forma abrupta.
* **Orquestração Visual:** Percebi que o Step Functions atua como uma **"cola" serverless**. É muito mais eficiente e observável orquestrar a sequência de eventos e gerenciar o estado do *workflow* com o ASL do que tentar gerenciar tudo isso dentro de uma única (e complexa) função Lambda.

### 🚧 Desafios Encontrados e Soluções

| Desafio Prático | Solução / Insight Adquirido |
| :--- | :--- |
| **Passagem de Dados (Payload)** | A maior dificuldade foi garantir o fluxo de dados (JSON) correto entre os *States*. O insight foi entender a importância de usar **`ResultPath`** para que o resultado de uma Lambda fosse inserido no payload, permitindo que o `Choice State` tomasse a decisão correta na etapa seguinte. |
| **Configuração de Permissões (IAM)** | Foi necessário configurar uma **IAM Role** de execução específica para o Step Function, concedendo permissão (`lambda:InvokeFunction`) para chamar cada uma das funções Lambda. **Permissões de Least Privilege** são essenciais. |
| **Debug e ASL** | A sintaxe da **Amazon States Language (ASL)** é sensível. Utilizar o **Workflow Studio** no console da AWS facilitou a visualização e a validação do código JSON, acelerando o *debug* e a construção do fluxo. |

### 🔑 Conclusão do Aprendizado

O AWS Step Functions se mostrou a ferramenta ideal para **coordenar microsserviços**, oferecendo um fluxo de trabalho claro, auditável e resiliente. O principal aprendizado é que a **complexidade de orquestração** deve ser movida do código da aplicação (Lambda) para o serviço de orquestração (Step Functions), resultando em Lambdas menores, mais simples e *workflows* mais robustos.

---
