# Mecânica Hermes - Deploy Kubernetes

Este repositório é responsável por realizar o deploy do projeto.

---

## Arquitetura dos Projetos de Infra

Nossa infraestrutura para disponibilizar um ambiente completo e funcional é composta por três passos sequenciais que são executados em diferentes repositórios:

1. [Mecânica Hermes - Infra Kubernetes](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra)
2. [Mecânica Hermes - Infra Database](https://github.com/fiap-challenge-13soat/mecanica-hermes-db)
3. Mecânica Hermes - Deploy Kubernetes (Este projeto)

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

```bash
kubectl apply -f k8s/overlays/${{ inputs.environment }}/namespace.yaml
```

---

### Criação da Secret

Define ambiente de Homologação ou Produção

```bash
$DB_ENVIRONMENT="prd" ou "hml"
```

Extraia o endpoint do database

```bash
$DB_ENDPOINT=aws rds describe-db-instances --db-instance-identifier mechermes-$DB_ENVIRONMENT-rds --query "DBInstances[0].Endpoint.Address" --output text
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
kubectl create secret generic app-secret --from-literal=ConnectionStrings__DefaultConnection="Host=$DB_ENDPOINT;Database=mechermesdb;Username=postgres;Password=$DB_PASSWORD;Pooling=true" --from-literal=Jwt__Key="THATSmyAw3SomeSecretTokenThatNeedsToCh@angeBeforeDeploy" --from-literal=EmailSenderOptions__Password="xFCsMj6a4NWbZgr5" -n mecanica-hermes-$DB_ENVIRONMENT
```

---

### Deploy das Aplicações

Deploy no ambiente selecionado

```bash
kubectl apply -k k8s/overlays/$DB_ENVIRONMENT
```

---

### Extração do Endpoint da API

```bash
kubectl get svc -n mecanica-hermes-$DB_ENVIRONMENT
```

---

### Observações

> O processo de deploy pode demorar de 5 a 10 minutos.

---

### Correção de Problemas

> Este processo foi amplamente testado, mas caso ocorra algum erro durante a sua execução, sugerimos abrir o ambiente no VSCode e solicitar ajuda ao Co-Pilot.
