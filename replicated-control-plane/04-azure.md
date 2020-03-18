# Prepare

export RESOURCE_GROUP=connect-azure-sandbox

# Install azure cli

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg

curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null

AZ_REPO=$(lsb_release -cs)

echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list

sudo apt-get update
sudo apt-get install azure-cli
```

# Login to Azure

```bash
az login
```

# Create a kubernetes cluster

```bash
# az aks create --resource-group $RESOURCE_GROUP --name TTT --node-count 1 --enable-addons monitoring --generate-ssh-keys

az aks create --resource-group $RESOURCE_GROUP --name TTT --node-count 1
```

# Use role-based access controls

```bash
AKS_CLUSTER=$(az aks show --resource-group $RESOURCE_GROUP --name TTT --query id -o tsv)        
az account show --query user.name                                                                     

ACCOUNT_UPN=$(az account show --query user.name -o tsv)                                                             
ACCOUNT_ID=$(az ad user show --id $ACCOUNT_UPN --query objectId -o tsv)                                             

az role assignment create --assignee $ACCOUNT_ID --scope $AKS_CLUSTER --role "Azure Kubernetes Service Cluster Admin Role"

az aks get-credentials --resource-group $RESOURCE_GROUP --name TTT --admin
```

