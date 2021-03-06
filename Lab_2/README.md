# Lab 2 : création d'un cluster plus avancé via Azure CLI, connexion et utilisation basique de kubectl
## Objectif:
L'objectif de ce Lab 2, c'est de déployer un cluster AKS en Az CLI avec une configuration de type de "NAT Gateway", c'est à dire, d'avoir le contrôle de l'IP publique sortantes des "Node Pool" (Remarque: Il n'est pas possible de le faire dans la console). L'objectif de ce Lab 2, c'est également de manipuler avec KUBECTL (Kubernetes command-line tool)<br> 


1. Prérequis:<br>
- Authentification `az login` (pas nécessaire avec Azure Cloud Shell)
- Checkez votre abonnement:<br> `az account list -o table`
- Option: Pour se mettre dans son abonnement <br> `az account set --subscription 'mon_abonnement'`
- Checkez les providers: Microsoft.OperationsManagement & Microsoft.OperationalInsights<br>
`az provider show -n Microsoft.OperationsManagement -o table`<br>
`az provider show -n Microsoft.OperationalInsights -o table`<br>
- Option: Pour enregistrer les providers<br>
`az provider register --namespace Microsoft.OperationsManagement`<br>
`az provider register --namespace Microsoft.OperationalInsights`<br>
- Pour faire ce Lab, mettre ses propres paramètres dans les "double cote" (ex: "RG-AKS-CLI" -> "mon-resource-group") <br>
- Vous pouvez lancer les commandes une par une ou faite un script <br>

2. Création d'un "resource group"<br>
```
az group create \
    --location "eastus2" \
    --resource-group "RG-AKS-CLI"
```
3. Création d'une "Public Ip" <br>
```
az network public-ip create \
    --resource-group "RG-AKS-CLI" \
    --name natGatewaypIpAks \
    --location "eastus2" \
    --sku standard  
```
4. Création d'une "Azure nat Gateway" <br>
```
az network nat gateway create \
    --resource-group "RG-AKS-CLI" \
    --name natGatewayAks \
    --location "eastus2" \
    --public-ip-addresses natGatewaypIpAks
```
5. Création d'un "Virtual Network" <br>
```
az network vnet create \
    --resource-group "RG-AKS-CLI" \
    --name AKSvnet \
    --location "eastus2" \
    --address-prefixes 172.16.0.0/20
```
6. Création d'un "subnet avec le paramétrage de la nat Gateway" <br>
```
SUBNET_ID=$(az network vnet subnet create \
    --resource-group "RG-AKS-CLI" \
    --vnet-name AKSvnet \
    --name natclusterAKS \
    --address-prefixes 172.16.0.0/22 \
    --nat-gateway natGatewayAks \
    --query id \
    --output tsv)
```
7. Création d'une "Managed Identity" <br>
```
IDENTITY_ID=$(az identity create \
    --resource-group "RG-AKS-CLI" \
    --name idAks \
    --location "eastus2" \
    --query id \
    --output tsv)
```
8. Création du "cluster AKS" <br>
```
az aks create \
    --resource-group "RG-AKS-CLI" \
    --name "AKS-CLI" \
    --location "eastus2" \
    --network-plugin azure \
    --generate-ssh-keys \
    --node-count 2 \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --vnet-subnet-id $SUBNET_ID \
    --outbound-type userAssignedNATGateway \
    --enable-managed-identity \
    --assign-identity $IDENTITY_ID \
    --yes

```
9. Test du Cluster AKS <br>
- Connexion au cluster <br>
`az aks get-credentials --resource-group "RG-AKS-CLI" --name "AKS-CLI" ` <br>
- Liste des nodes du cluster <br>
`kubectl get nodes` <br>
- Liste les dépoiements dans tous les namespaces <br>
`kubectl get deployments --all-namespaces=true`

10. Installation d'une application avec les fichiers Manifests<br>
- Dans le répertoire ./Manifest il y a cinq fichiers: <br>
`Namespace.yml` (création du namespace) <br>
`Deployment-redis.yml` (déploiement d'une base Redis) <br>
`Service-redis.yml` (déploiement du service Redis) <br>
`Deployment-front.yml` (déploiement du Front) <br>
`Service-front.yml` (déploiement du service Front) <br>
- Création du Namespace <br>
`kubectl apply -f ./Namespace.yml` <br>
liste les Namespace <br>
`kubectl get namespace`
- Déploiement du Redis (azure-vote-back) <br>
`kubectl apply -f ./Deployment-redis.yml --namespace=azure-vote`<br>
`kubectl apply -f ./Service-redis.yml --namespace=azure-vote`<br>
Vérification:<br>
`kubectl get deployments --all-namespaces=true`<br>
`kubectl get service --namespace=azure-vote`<br>
- Déploiement du Front (azure-vote-front)<br>
`kubectl apply -f ./Deployment-front.yml --namespace=azure-vote`<br>
`kubectl apply -f ./Service-front.yml --namespace=azure-vote`<br>
Vérification:<br>
`kubectl get deployments --all-namespaces=true`<br>
`kubectl get service --namespace=azure-vote --watch`<br>
Attendre la récupération de "l'EXTERNAL-IP" & crt+c <br>
Aller avec votre navigateur sur "l'EXTERNAL-IP"<br>
Voir les logs "d'azure-vote-front" `kubectl logs -l app=azure-vote-front --namespace azure-vote`

11. Fin du Lab 2
az group delete --name "RG-AKS-CLI"











