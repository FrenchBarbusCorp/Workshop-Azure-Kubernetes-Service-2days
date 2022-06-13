# Azure Kubernetes Service Cluster deployment with Terraform
      
=== Still work in progress, need to adapt this TF Code to some change of AzureRM 3.0 provider === 
=== version up to date June 2022 ===

= (not) Tested (yet) with success with 
Terraform v1.2.2
on linux_amd64 (WSL2)
+ provider registry.terraform.io/hashicorp/azurerm v3.1.0
+ provider registry.terraform.io/hashicorp/helm v2.5.0
+ provider registry.terraform.io/hashicorp/kubernetes v2.10.0
+ provider registry.terraform.io/hashicorp/random v3.1.0
+ provider registry.terraform.io/providers/hashicorp/time v0.7.2

--------------------------------------------------------------------------------------------------------

This is a set of Terraform files used to deploy an Azure Kubernetes Cluster with some cool features :

- Nodes will be dispatched in different Availability Zones (AZ)
- Node pools will support Autoscaling
- pool1 is a linux node pool (it is mandatory because of kube system pods)
- pool2 (optional) is a windows server 2019 node pool with a taint
- System Managed Identities are used instead of Service Principal
- Choice of SKU (Free or Paid) for Azure Kubernetes Service (Control Plane)

These Terraform files can be used to deploy the following Azure components :

- An Azure Resource Group
- An Azure Kubernetes Services Cluster with 1 node pool running Linux 
- An additionnal node pool (pool2) with Windows Server 2019 nodes (optional)
- An Azure Load Balancer Standard SKU
- A Virtual Network with it Subnets (subnet for AKS Pods, subnets for AzureBastion and AzureFirewall/NVA if needed, Azure Application Gateway)
- Azure Application Gateway + Application Gateway Ingress Controller AKS add-on
- An Azure Log Analytics Workspace (used for Azure Monitor Container Insight)

On Kubernetes, these Terraform files will :

- Deploy Grafana using Bitnami Helm Chart and exposed Grafana Dashboard using Ingress (and AGIC)
- Create a pod, a service and an ingress (the file associated is renamed in .old because of issue during first terraform plan) 

__Prerequisites :__

- An Azure Subscription with enough privileges (create RG, AKS...)
- Azure CLI 2.37 or >: <https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest>
   And you need to activate features that are still in preview and add extension aks-preview to azure CLI (az extension add --name aks-preview)
- Terraform CLI 1.2.2 or > : <https://www.terraform.io/downloads.html>
- Helm CLI 3.9.0 or > : <https://helm.sh/docs/intro/install/> if you need to test Helm charts

__To deploy this infrastructure :__

1. Log to your Azure subscription (az login)
2. Create an Azure Key Vault and create all secrets defined in datasource.tf
3. Define the value of each variable in .tf and/or .tfvars files
4. Initialize your terraform deployment : `terraform init`
5. Plan your terraform deployment : `terraform plan --var-file=myconf.tfvars`
6. Apply your terraform deployment : `terraform apply --var-file=myconf.tfvars`



After deployment is succeeded, you can check your cluster using portal or better with azure cli and the following command: 
```bash
az aks show --resource-group NAMEOFYOURRESOURCEGROUP --name NAMEOFYOURAKSCLUSTER -o jsonc
```

Get your kubeconfig using :

```bash
az aks get-credentials --resource-group NAMEOFYOURRESOURCEGROUP --name NAMEOFYOURAKSCLUSTER --admin
```
