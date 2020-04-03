# Devops-Challenge

# Infraestrutura-Utilizada

 - Solução de Cloud = Microsoft Azure
 - CI/CD = AzureDevOps
 - Orquestrador = Kubernetes
 - Provisionador = Terraform

## Task 1 - Criação de uma conta no Microsoft Azure

### Primeiramente, crie sua conta gratuita no Microsoft Azure conforme o artigo:

    https://azure.microsoft.com/pt-br/free/

### Após a criação e login no ambiente, execute a instalação do Azure CLI em seu dispositivo:

### Sistema Windows

    Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi

### Sistema Mac

    brew update && brew install azure-cli

### Sistema Linux

    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

    curl -sL https://packages.microsoft.com/keys/microsoft.asc |
    gpg --dearmor |
    sudo tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null

    AZ_REPO=$(lsb_release -cs)
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list

    sudo apt-get update
    sudo apt-get install azure-cli

### Para validar se o Azure CLI foi instalado corretamente em seu dispositivo, execute o comando abaixo:

    az login

## Task 2 - Preparação do ambiente Microsoft Azure para receber as configurações

## Requisitos

### > Definir a região de hospedagem

    eastus2

### > Criar um "Resource Group" para os recursos do Cluster AKS

    az group create -n AksTerraform-RG -l eastus2

### > Criar uma "Storage Account" para o cluster do AKS

    az storage account create -n saaksterraform -g AksTerraform-RG -l eastus2

### > Criar um container "tfstate" para o cluster do AKS

    az storage container create -n tfstate --account-name saaksterraform

### > Criar um "KeyVault" para o cluster do AKS

    az keyvault create -n kvaksterraform -g AksTerraform-RG -l eastus2

### > Criar um token SAS para a conta de armazenamento, armazenando no KeyVault

    az storage container generate-sas --account-name saaksterraform --expiry 2021-01-01 --name tfstate --permissions dlrw -o json | xargs az keyvault secret set --vault-name kvaksterraform --name TerraformSASToken --value

### > Criar uma entidade de serviço para AKS e Azure DevOps

    az ad sp create-for-rbac -n "AksTerraformSPN"

### > Criar uma chave ssh

    ssh-keygen  -f ~/.ssh/id_rsa_terraform

### > Armazenar a chave pública no Azure KeyVault

    az keyvault secret set --vault-name kvaksterraform --name LinuxSSHPubKey -f ~/.ssh/id_rsa_terraform.pub > /dev/null

### > Armazenar a entidade de serviço no Azure KeyVault

    az keyvault secret set --vault-name kvaksterraform --name spn-id --value dfafd341-3f43-4b85-8944-21b014dbcfea > /dev/null

### > Armazenar a entidade de serviço no Azure KeyVault

    az keyvault secret set --vault-name kvaksterraform --name spn-secret --value 008e522b-f1c5-45cd-8f9a-c609bbb504c6 > /dev/null

### > Infelizmente, o recurso Terraform Backend não suporta variáveis, então precisaremos passar o parâmetro no método init para configurar corretamente o back-end em nosso pipeline de CI/CD

    terraform init \
    -backend-config="resource_group_name= AksTerraform-RG" \
    -backend-config="storage_account_name=saaksterraform" \
    -backend-config="container_name=tfstate"


## Task 3 - Criação do Pipeline CI/CD no AzureDevOps para Terraform

## Requisitos

### > Ambiente do Microsoft Azure preparados na Task 2


![node-to-node](img\img1.png) 
![node-to-node](img\img2.png) 
![node-to-node](img\img3.png) 


### > Projeto criado no AzureDevOps:

    https://azure.microsoft.com/pt-br/services/devops/

![node-to-node](img\img4.png)

### > Service Connections criadas no Projeto do AzureDevOps

    https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml

![node-to-node](img\img5.png)

## Task 3 - Deploy do ambiente usando CI/CD no AzureDevOps

## Requisitos

### > Repositório do projeto criado no AzureDevOps

![node-to-node](img\img6.png)

### > Arquivos do Terraform

![node-to-node](img\img7.png)

### > Arquivos da Aplicação:

![node-to-node](img\img8.png)

### > Construção dos Pipelines

- O pipeline de deploy do Cluster AKS foi feito utilizando arquivos declarativos do Terraform e um YAML file do AzurePipelines conforme abaixo:

![node-to-node](img\img9.png)

- O pipeline de deploy da aplicação foi feito via GUI para exemplificar as duas formas de se construir ambientes pelo AzureDevOps

![node-to-node](img\img10.png)

### > Log de execução do pipeline de Terraform

[terraform_log](logs/terraform_log.txt)

### > Log de execução do pipeline de Aplicação

[votacao-app_log](logs/votacao-app_log.txt)

## Task 4 - Validação da Aplicação em funcionamento:

### > Para conectar no Cluster AKS Criado

    az aks get-credentials --resource-group AKSCluster-RG --name AKSTerraform

![node-to-node](img\img11.png)

### > Validando como aplicação foi exposta

![node-to-node](img\img12.png)

### > Acessando pelo browser

![node-to-node](img\img13.png)

# Arquitetura-Final

![node-to-node](img\img14.png)

### > AKS Cluster

![node-to-node](img\img15.png)

### > AKS Networking

![node-to-node](img\img16.png)

### > AKS Secrets

![node-to-node](img\img17.png)

