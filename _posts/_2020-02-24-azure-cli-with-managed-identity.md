---
layout: post
title: "Azure CLI with Managed Identity"
categories: azure-cli
author: "Venura Athukorala"
meta: "Azure, Azure CLI, ARM, Azure Resource Manager"
---


Azure Managed Identitiy, formerly known as Managed Service Identitiy (MSI) has come a long way from being an unstable toggle to ensuring a solid mutual trust between azure resources using Azure Active Directory as the Identity Providor. 

> There are two types of managed identities:
>
> A **system-assigned managed identity** is enabled directly on an Azure service instance. When the identity is enabled, Azure creates an identity for the instance in the Azure AD tenant that's trusted by the subscription of the instance. After the identity is created, the credentials are provisioned onto the instance. The lifecycle of a system-assigned identity is directly tied to the Azure service instance that it's enabled on. If the instance is deleted, Azure automatically cleans up the credentials and the identity in Azure AD.
>
> A **user-assigned managed identity** is created as a standalone Azure resource. Through a create process, Azure creates an identity in the Azure AD tenant that's trusted by the subscription in use. After the identity is created, the identity can be assigned to one or more Azure service instances. The lifecycle of a user-assigned identity is managed separately from the lifecycle of the Azure service instances to which it's assigned.
>
> Ref: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/


In this blog post, I'm going to explain how you can achieve a unique state, allowing to run Azure CLI from a VM itself using System-Assigned Managed Identity. This allows for scenarios such as performing CRUD operations on azure through a VM. 
E.g. Self Attach/Detach IP addresses. 

Also, this method does not require passing explicit secrets to authenicate as the secrets are all managed by azure itself. 


1. Let's login to Azure and select a subscription which you want the resources to be created
```bash
az login
_subscription_id='00000000-0000-0000-0000-000000000000' # Insert your subscription id
az account set --subscription $_subscription_id
```

2. Get our resource group created in Sydney. 
```bash
az group create --name myResourceGroup --location australiaeast
```

3. Create our linux VM with system assigned managed identity enabled, scope & role also defined. 
```bash
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys --assign-identity --location australiaeast --scope "/subscriptions/${_subscription_id}/resourcegroups/myResourceGroup" --role contributor
```

4. Install Azure CLI on the VM (via Custom Script Extension)
```bash
az vm extension set --resource-group myResourceGroup --vm-name myVM --name customscript --publisher Microsoft.Azure.Extensions --settings '{"commandToExecute": "curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash"}'
```

5. Get the current Public IP
```bash
_public_ip=$(az vm list-ip-addresses -g myResourceGroup -n myVM --query "[].virtualMachine.network.publicIpAddresses[0].ipAddress" --output tsv)
```

6. Login to the VM
```bash
ssh azureuser@${_public_ip}
```

7. From the VM just run below to login with Managed Identity (It's always possible to use the custom script extension to run more commands at VM creation)
```bash
az login --identity
```
<!-- ![Login via Managed Identity on Azure VM](https://venura9.github.io/assets/managed-identity-login-azure-vm.png) -->
<img src="https://venura9.github.io/assets/managed-identity-login-azure-vm.png" width="480" alt="Login via Managed Identity on Azure VM">

8. Clean up after we're done. 
```bash
az group delete --name myResourceGroup --yes
```


Need to learn more about attaching Managed Identities to VMs? 
More information about Managed Identities can be found at: https://aka.ms/azure-msi-docs

Need to learn more about Azure CLI?
More information about Azure CLI can be found at: https://aka.ms/cli
