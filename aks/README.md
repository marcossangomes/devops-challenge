# Requerimentos para contrução do cluster AKS usando Terraform e Azure DevOps

## Instalação do CLI do Azure

PS C:\Windows\system32> Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi

PS C:\Windows\system32> az login
You have logged in. Now let us find all the subscriptions to which you have access...
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "e942b696-35aa-42db-aff5-158d31fcc434",
    "id": "d2b8c7ca-fc1a-4c03-aeea-9740237b2bb4",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Visual Studio Enterprise",
    "state": "Enabled",
    "tenantId": "e942b696-35aa-42db-aff5-158d31fcc434",
    "user": {
      "name": "edson@contoso.com",
      "type": "user"
    }
  }
]

## Região

eastus2

## Criar um "Resource Group" para os recursos do Cluster AKS

az group create -n AksTerraform-RG -l eastus2

## Criar uma "Storage Account" para o cluster do AKS

az storage account create -n saaksterraform -g AksTerraform-RG -l eastus2

## Criar um container "tfstate" para o cluster do AKS

az storage container create -n tfstate --account-name saaksterraform

## Criar um "KeyVault" para o cluster do AKS

az keyvault create -n kvaksterraform -g AksTerraform-RG -l eastus2

## Criando um token SAS para a conta de armazenamento, armazenando no KeyVault

az storage container generate-sas --account-name saaksterraform --expiry 2021-01-01 --name tfstate --permissions dlrw -o json | xargs az keyvault secret set --vault-name kvaksterraform --name TerraformSASToken --value

## Criando uma entidade de serviço para AKS e Azure DevOps

az ad sp create-for-rbac -n "AksTerraformSPN"

## Criando uma chave ssh

ssh-keygen  -f ~/.ssh/id_rsa_terraform

## Armazenar a chave pública no Azure KeyVault

az keyvault secret set --vault-name kvaksterraform --name LinuxSSHPubKey -f ~/.ssh/id_rsa_terraform.pub > /dev/null

## Armazenar a entidade de serviço no Azure KeyVault

az keyvault secret set --vault-name kvaksterraform --name spn-id --value dfafd341-3f43-4b85-8944-21b014dbcfea > /dev/null

## Armazenar a entidade de serviço no Azure KeyVault

az keyvault secret set --vault-name kvaksterraform --name spn-secret --value 008e522b-f1c5-45cd-8f9a-c609bbb504c6 > /dev/null

## Infelizmente, o recurso Terraform Backend não suporta variáveis, precisaremos passar o parâmetro no método init para configurar corretamente o back-end em nosso pipeline de CI / CD

    terraform init \
    -backend-config="resource_group_name= AksTerraform-RG" \
    -backend-config="storage_account_name=saaksterraform" \
    -backend-config="container_name=tfstate"