# Mecânica Hermes - Deploy Kubernetes

Este repositório é responsável por realizar o deploy do projeto.

---

## Tecnologias Utilizadas

---

![Kubernetes](https://img.shields.io/badge/Kubernetes-Orquestra%C3%A7%C3%A3o-326CE5?logo=kubernetes&logoColor=white)
![Kustomize](https://img.shields.io/badge/Kustomize-Config%20Management-4A86CF?logo=kustomize&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Cloud-232F3E?logo=amazonaws&logoColor=white)
![Amazon EKS](https://img.shields.io/badge/Amazon%20EKS-Kubernetes-FF9900?logo=amazon-eks&logoColor=white)
![Amazon RDS](https://img.shields.io/badge/Amazon%20RDS-PostgreSQL-527FFF?logo=amazonrds&logoColor=white)
![Kubectl](https://img.shields.io/badge/kubectl-CLI-326CE5?logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI/CD-2088FF?logo=githubactions&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Images-2496ED?logo=docker&logoColor=white)

- **Kubernetes** para orquestração das aplicações e recursos de cluster.
- **Kustomize** para sobreposição de ambientes (hml/prd) e padronização de manifests.
- **AWS** como provedor de nuvem.
- **Amazon EKS** como cluster Kubernetes gerenciado.
- **Amazon RDS (PostgreSQL)** como banco de dados consumido pela API.
- **kubectl** para aplicação e gerenciamento dos manifests.
- **GitHub Actions** para execução da pipeline de deploy/destroy.
- **Docker** para distribuição das imagens das aplicações.

---

## Arquitetura dos Projetos de Infra

Nossa infraestrutura para disponibilizar um ambiente completo e funcional é composta por três passos sequenciais que são executados em diferentes repositórios:

1. [Mecânica Hermes - Infra Kubernetes](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra)
2. [Mecânica Hermes - Infra Database](https://github.com/fiap-challenge-13soat/mecanica-hermes-db)
3. Mecânica Hermes - Deploy Kubernetes (Este projeto)

---

```mermaid
flowchart TB
  subgraph Step1["Repo: Infra Kubernetes"]
   TF1["Terraform"]
   VPC["VPC: mechermes-vpc"]
   SG["Security Group: mechermes-sg"]
   Subnet["Subnets privadas/públicas"]
   EKS["EKS Cluster (privado)"]
   Nodes["Node Groups"]
  end

  subgraph Step2["Repo: Infra Database"]
   TF2["Terraform"]
   Data["Data sources: VPC/SG/Subnets"]
   RDS["RDS PostgreSQL (privado)"]
  end

  subgraph Step3["Conta Externa: Terraform State & Lock"]
   S3["S3: tfstate"]
   DDB["DynamoDB: tf locks"]
  end

  subgraph Step4["Repo: Deploy Kubernetes (este projeto)"]
   Kustomize["Kustomize overlays: hml/prd"]
   Namespace["Namespace: mecanica-hermes-<env>"]
   Secret["Secret: app-secret"]
   CMApi["ConfigMap: mecanica-hermes-api-config"]
   ApiDeploy["Deployment: mecanica-hermes-api"]
   HPA["HPA: mecanica-hermes-api"]
   ApiSvc["Service: mecanica-hermes-api (LoadBalancer)"]
  end

  TF1 --> VPC
  TF1 --> SG
  TF1 --> Subnet
  TF1 --> EKS
  TF1 --> Nodes
  TF1 --> S3
  TF1 --> DDB

  TF2 --> Data --> RDS
  VPC --> Data
  SG --> Data
  Subnet --> Data

  EKS --> Kustomize
  Kustomize --> Namespace
  Namespace --> ApiDeploy
  Secret --> ApiDeploy
  CMApi --> ApiDeploy
  ApiDeploy --> HPA
  ApiDeploy --> ApiSvc

  ApiSvc --> Clients["Clientes externos"]
  ApiDeploy --> RDS
```

## Execução pela Pipeline

**Configure as Secrets abaixo no repositório.**

---

| Variável              |
| --------------------- |
| AWS_ACCESS_KEY_ID     |
| AWS_SECRET_ACCESS_KEY |
| AWS_SESSION_TOKEN     |
| AWS_DEFAULT_REGION    |
| JWT_KEY               |
| EMAIL_PASSWORD        |

---

Para gerar nova Key e Token use o comando:

```bash
aws sts get-session-token
```

---

### Deploy Environment

---
Dispare a Action Trigger manual para realizar o deploy do ambiente:

- `Kubernetes Deploy`.

---

### Destroy Environment

---
Dispare a Action Trigger manual para realizar o deploy do ambiente:

- `Kubernetes Destroy`.

---

> **Atenção!**
>
> Caso não tenha sido realizado o deploy do ambiente de infraestrutura de clusters e do database, será impossível concluir este deploy.
---

## Execução Local

Abordaremos os passos para realizar o deploy do ambiente em infraestrutura AWS partindo do seu computador.

---

### Configuração da AWS

Realize o comando abaixo e forneça suas credenciais.

```bash
aws configure
```

---

### Configuração do Kubectl

Configure o seu AWS com o Kubectl.

```bash
aws eks update-kubeconfig --name mechermes-eks
```

### Criação do Namespace

Define ambiente de Homologação ou Produção

```bash
$ENVIRONMENT="prd" ou "hml"
```

```bash
kubectl apply -f k8s/overlays/$ENVIRONMENT/namespace.yaml
```

---

### Criação da Secret

Extraia o endpoint do database

```bash
$DB_ENDPOINT=aws rds describe-db-instances --db-instance-identifier mechermes-$ENVIRONMENT-rds --query "DBInstances[0].Endpoint.Address" --output text
```

Extraia o ARN do database

```bash
$DB_ARN=aws rds describe-db-instances --db-instance-identifier mechermes-rds --query "DBInstances[0].MasterUserSecret.SecretArn" --output text
```

Extraia a senha do database

```bash
$DB_PASSWORD=aws secretsmanager get-secret-value --secret-id $DB_ARN --query SecretString --output text | ConvertFrom-Json | Select-Object -ExpandProperty password
```

Crie a Secret no Cluster

```bash
kubectl create secret generic app-secret --from-literal=ConnectionStrings__DefaultConnection="Host=$DB_ENDPOINT;Database=mechermesdb;Username=postgres;Password=$DB_PASSWORD;Pooling=true" --from-literal=Jwt__Key="THATSmyAw3SomeSecretTokenThatNeedsToCh@angeBeforeDeploy" --from-literal=EmailSender__SenderInformation__Name="xFCsMj6a4NWbZgr5" -n mecanica-hermes-$ENVIRONMENT
```

---

### Deploy das Aplicações

Deploy no ambiente selecionado

```bash
kubectl apply -k k8s/overlays/$ENVIRONMENT
```

---

### Extração do Endpoint da API

```bash
kubectl get svc -n mecanica-hermes-$ENVIRONMENT
```

---

### Observações

> O processo de deploy pode demorar de 5 a 10 minutos.

---

### Correção de Problemas

> Este processo foi amplamente testado, mas caso ocorra algum erro durante a sua execução, sugerimos abrir o ambiente no VSCode e solicitar ajuda ao Co-Pilot.
