# How to deploy a .NET 7 Web API in Azure Container Instance

## 1. Create an Azure Container Registry with Azure SDK for .NET

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

We enable the Docker support in the application for automatically creating a dockerfile

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

## 4. We build the Docker Image

```
docker build -t mywebapicontainer.azurecr.io/mywebapicontainer:v1 .
```

## 5. We push the Docker image to the Azure ACR

```
docker push mywebapicontainer.azurecr.io/mywebapicontainer:v1
```

## 6. We get the ACI credentials

We copy the password for creating the ACI and deploy it in the next section

```
az acr credential show --name mycontainerinstance
```

## 7. We create the Azure Container Instance (ACI) and we deploy it

```
az container create --resource-group myRG --name mycontainerinstance --image mywebapicontainer.azurecr.io/mywebapicontainer:v1 --cpu 1 --memory 1.5 --registry-login-server mywebapicontainer.azurecr.io --registry-username mywebapicontainer --registry-password tk5N+2tBFnNxImB0ByTt58Nt+HLvCwLWMA8bNn1lwY+ACRAeOtn/ --dns-name-label mywebapidns7788 --ports 80 --location westeurope
```
