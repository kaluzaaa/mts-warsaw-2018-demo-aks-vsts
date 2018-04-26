rg=shared
rg_aks=aks
location=eastus
epoch=`date +%s`
keyvault_name="kv$epoch"
cluster_name="aks$epoch"
acr_name="acr$epoch"
sp_name="sp-aks-$epoch"
az group create --name $rg --location $location
az group create --name $rg_aks --location $location
az keyvault create --resource-group $rg --name $keyvault_name
read sp_password < <(az ad sp create-for-rbac --name $sp_name --output tsv | cut -f4)
read sp_appid < <(az ad app list -o tsv | grep $sp_name | cut -f2)
az acr create --resource-group $rg_aks --name $acr_name --admin-enabled --sku Standard
az aks create --resource-group $rg_aks --name $cluster_name --node-count 1 --node-vm-size Standard_DS3_v2 --kubernetes-version 1.9.6 --service-principal $sp_appid --client-secret $sp_password --generate-ssh-keys
echo "KeyVault:\t$keyvault_name"
echo "AKS:\t$cluster_name"
echo "ACR:\t$acr_name"
echo "SP:\t$sp_name"