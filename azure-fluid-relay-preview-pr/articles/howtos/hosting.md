---
title: Hosting Fluid Example
description: Detailed explanation about how fluid examples can be hosted on Azure App Service and Static Web App
---

# Overview
The `FluidHelloWorld` repository is framework agnostic and there are three views: react, vue and vanilaJS.

# Connecting to Azure Relay Service
You can connect to `Azure Relay Service` by providing the tenant ID and key that is uniquely generated for you when onboarding to the service. The `tokenProvider` can be generated using two methods: `InsecureTokenProvider` and `FrsAzFunctionTokenProvider`. You can make use of `InsecureTokenProvider` for development purpose because using it exposes the tenant key secret in your client-side code bundle. 

```typescript
import { FrsClient, FrsConnectionConfig } from "@fluidframework/frs-client";

const config: FrsConnectionConfig = {
    tenantId: "myTenantId",
    tokenProvider: new InsecureTokenProvider("myTenantKey", { id: "123", name: "Test User" }),
    orderer: "https://myOrdererUrl",
    storage: "https://myStorageUrl",
};
const client = new FrsClient(config);
```

We make use of [FrsAzFunctionTokenProvider](https://github.com/microsoft/FluidFramework/blob/main/packages/framework/azure-client/src/AzureFunctionTokenProvider.ts) for token generation that fetches the token from your own backend service that is responsible for signing it with the tenant key. The `FrsAzFunctionTokenProvider` is an implemention that fulfills the `ITokenProvider` interface without exposing the tenant key secret in client-side code.

```typescript
import { FrsClient, FrsConnectionConfig } from "@fluidframework/frs-client";

const config: FrsConnectionConfig = {
    tenantId: "myTenantId",
    tokenProvider: new FrsAzFunctionTokenProvider("myAzureFunctionUrl"+"/api/GetFrsToken", { userId: "test-user",userName: "Test User" }),
    orderer: "https://myOrdererUrl",
    storage: "https://myStorageUrl",
};
const client = new FrsClient(config);
```

# Interaction with all the three views
You can switch between three views: react, vue and vanilaJS. The views can be modified by updating the imports in [FluidHelloWorld](https://github.com/microsoft/FluidHelloWorld/blob/main/src/app.ts).

```typescript
import { jsRenderView as renderView } from "./view";
```
```typescript
import { reactRenderView as renderView } from "./view";
```
```typescript
import { vueRenderView as renderView } from "./view";
```

# Setup Visual Studio Code
1. Add the following extensions to Visual Studio Code: 
    - [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)
    - [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice)
    - [Azure Static Web App](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps)

2. Run the following command to clone the repository:

```shell
git clone https://github.com/microsoft/FluidHelloWorld
```

# Deployment using Azure App Services
Run the `npm run build` command to generate the build folder which will be used for deploying to our App Service.`FluidHelloWorld` being framework agnostic, a `dist` folder would be generated. 

### Steps to create an Azure App Service
1. In order to create an app service on Azure, you are required to have either an active Azure subscription or a free account that you can obtain from the [Microsoft Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program).
2. Once logged in, Select `App Services`. Select your `Subscription`, `Resource Group`, `Runtime Stack` as `Node 14 LTS`, `OS` as `Windows` and later select `Create`. 

[!NOTE]
While creating the App service, select `Windows` as the OS. You can select the OS while app creation. Once the app is created, the OS cannot be changed.

### Steps to deploy to Azure App Service
1. Click on the Azure Extension (Ctrl + Shift + A) in Visual Studio Code. In `App Service` section, under your `Subscription`, you could see the App Service you have created through the Azure Portal previously. 
2. Right click on your `App Service` and select `Deploy to Web App`. You would get an option to browse the file explorer. Select the `dist` folder and later select `Deploy`. 
3. After the deployment is completed, you can browse the website by again going to your `Subscription` in your App Service. Later, right click on your App Service and select `Browse Website`. 

# Deployment using Azure Static Web Apps

Static Web App is cost-effective and can be used for development purpose.

### Steps to create an Azure Static Web Apps

Static Web Apps can be created through the [Azure Portal](https://docs.microsoft.com/en-us/azure/static-web-apps/publish-devops) or through [Visual Studio Code](https://docs.microsoft.com/en-us/azure/static-web-apps/getting-started?tabs=vanilla-javascript)
