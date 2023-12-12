# How to deploy a .NET 7 Web API in Azure Container Instance

## 1. Create an Azure Container Registry

```csharp
using Azure.Core;
using Azure;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.ContainerRegistry;
using Azure.ResourceManager.ContainerRegistry.Models;
using System;
using System.Threading.Tasks;
using Azure.ResourceManager.Resources;

var resourceGroupName = "myRG";
var registryName = "mywebapicontainer";
var location = "westeurope"; // e.g., "westus"

var credential = new DefaultAzureCredential();

// Authenticate with Azure
var armClient = new ArmClient(credential);
var subscription = armClient.GetDefaultSubscription();

// Check if the resource group exists, and create it if it doesn't
var resourceGroupExists = await subscription.GetResourceGroups().ExistsAsync(resourceGroupName);
ResourceGroupResource resourceGroup;

if (!resourceGroupExists.Value)
{
    Console.WriteLine($"Creating resource group: {resourceGroupName}");
    var resourceGroupData = new ResourceGroupData(new AzureLocation(location));
    var operation = await subscription.GetResourceGroups().CreateOrUpdateAsync(WaitUntil.Completed, resourceGroupName, resourceGroupData);
    resourceGroup = operation.Value; // Get the ResourceGroupResource from the operation
}
else
{
    Console.WriteLine($"Resource group {resourceGroupName} already exists.");
    resourceGroup = await subscription.GetResourceGroups().GetAsync(resourceGroupName);
}

// Create a new Azure Container Registry
var containerRegistryData = new ContainerRegistryData(new AzureLocation(location), new ContainerRegistrySku(ContainerRegistrySkuName.Basic))
{
    // Additional properties can be set here if needed
};

// Create or update the container registry
var containerRegistryOperation = await resourceGroup.GetContainerRegistries().CreateOrUpdateAsync(WaitUntil.Completed, registryName, containerRegistryData);
var containerRegistry = containerRegistryOperation.Value; // Get the ContainerRegistryResource from the operation

Console.WriteLine($"Created container registry: {containerRegistry.Data.Id}");
```

## 2. Create a .NET 7 Web API in Visual Studio 2022 Community Edition

Open Visual Studio 2022 Community Edition and we create a new .NET 7 API




## 3. Create the docker image and we push in Azure ACR

**IMPORTANT**! We first have to run Docker Desktop

We login in Azure CLI

```
az login
```

We can activate the admin user with this command:

```
az acr update --name mywebapicontainer --resource-group myRG --admin-enabled true
```

Then we login in the Azure ACR and set the username and password in the admin user page

```
az acr login --name mywebapicontainer
```


