---
title: Hosting Fluid Example
description: Detailed explanation about how Fluid examples can be hosted on Azure App Service and Static Web App
---

This article demonstrates how to deploy your Fluid examples to Azure App Service and Static Web Apps. [FluidHelloWorld](https://github.com/microsoft/FluidHelloWorld/tree/main-frs) repository is a Fluid example which contains a simple app that enables all connected clients to roll a dice and view the result. 

# Pre-requisites
1. Add the following extensions to Visual Studio Code: 
    - [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account): It is required to sign in to your `Azure Subscription`.

    You can install any one of the following extensions depending on the type of deployment you choose:

    - [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice): It is required for deploying app using `App Services`.
    - [Azure Static Web App](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps): It is required for deploying app using `Static Web Apps`.

These extensions allow the developer to deploy to Azure directly from Visual Studio Code.

2. Run the following command to clone the repository:

```shell
git clone https://github.com/microsoft/FluidHelloWorld
```

# Connecting to Azure Fluid Relay Service
You can connect to `Azure Fluid Relay service` by providing the tenant ID and key that is uniquely generated for you when onboarding to the service. Although you can build your own implementation that fulfills the `ITokenProvider` interface, we provide two in-built options for instantiating a `tokenProvider`:

1. `InsecureTokenProvider`: This type of implementation is only recommended for development purposes as it exposes the tenant key secret in your client-side code bundle. You can add the below `config` in [app.ts](https://github.com/microsoft/FluidHelloWorld/blob/main/src/app.ts), replacing the strings with the values appropriate for your assigned instance.

```typescript
import { AzureClient, AzureConnectionConfig } from "@fluidframework/azure-client";
import { InsecureTokenProvider } from `@fluidframework/test-client-utils`;

const config: AzureConnectionConfig = {
    tenantId: "myTenantId",
    tokenProvider: new InsecureTokenProvider("myTenantKey", { id: "123", name: "Test User" }),
    orderer: "https://myOrdererUrl",
    storage: "https://myStorageUrl",
};
const client = new AzureClient(config);
```

2. `AzureFunctionTokenProvider`: This implementation can be used for fetching the token from your own backend service that is responsible for signing it with the tenant key. It provides you with a secured way to generate the token and pass it back to the client app. `AzureFunctionTokenProvider` is an implementation that fulfills the `ITokenProvider` interface without exposing the tenant key secret in client-side code. Developers would require their own backend `HTTPTrigger Azure Function` for fetching the token, thereby passing the url to `AzureFunctionTokenProvider` in [app.ts](https://github.com/microsoft/FluidHelloWorld/blob/main/src/app.ts). We will discuss setting up this backend endpoint in the next step.

```typescript
import { AzureClient, AzureConnectionConfig, AzureFunctionTokenProvider } from "@fluidframework/azure-client";

const config: AzureConnectionConfig = {
    tenantId: "myTenantId",
    tokenProvider: new AzureFunctionTokenProvider("myAzureFunctionUrl"+"/api/GetAzureToken", { userId: "test-user",userName: "Test User" }),
    orderer: "https://myOrdererUrl",
    storage: "https://myStorageUrl",
};
const client = new AzureClient(config);
```

<br />

> [!NOTE]
> The below implementation can be used for `Static Web Apps` only.

3. `Static Web Apps` allow you to develop a full-stack web site without needing to deal with the server-side configuration of an entire web hosting environment and not exposing the token on the client-side bundle code. You can fetch the token by setting up your own backend `api`. Developers can create their own `HTTPTrigger Azure Function` under an `api` folder in the `FluidHelloWorld` repository itself. The `GetAzureToken` shown below is an `HTTPTrigger Azure Function` which is responsible for generating tokens on your very own backend. 

<img src='https://user-images.githubusercontent.com/46719950/133346937-ca85c926-7282-44aa-a644-fd039a065050.PNG' style="height: 300px"> 

<br />
<br />

The below code snippet can help you with creating an `HTTPTrigger` function for fetching the token by passing in your tenant key.

```typescript
import { AzureFunction, Context, HttpRequest } from "@azure/functions";
import { ScopeType } from "@fluidframework/azure-client";
import { generateToken, generateUser } from "@fluidframework/azure-service-utils";

//Replace "myTenantKey" with your key here.
const key = "myTenantKey";

const httpTrigger: AzureFunction = async function (context: Context, req: HttpRequest): Promise<void> {
    const tenantId = (req.query.tenantId || (req.body && req.body.tenantId)) as string;
    const documentId = (req.query.documentId || (req.body && req.body.documentId)) as string;
    const userId = (req.query.userId || (req.body && req.body.userId)) as string;
    const userName = (req.query.userName || (req.body && req.body.userName)) as string;
    const scopes = (req.query.scopes || (req.body && req.body.scopes)) as ScopeType[];

    if (!tenantId) {
        context.res = {
            status: 400,
            body: "No tenantId provided in query params",
        };
    }

    if (!key) {
        context.res = {
            status: 404,
            body: `No key found for the provided tenantId: ${tenantId}`,
        };
    }

    if (!documentId) {
        context.res = {
            status: 400,
            body: "No documentId provided in query params"
        };
    }

    let user = { name: userName, id: userId };
    if (!userId || !userName) {
        const generatedUser = generateUser() as any;
        user = { name: userName ?? generatedUser.name, id: userId ?? generatedUser.id };
    }

    const token = generateToken(
        tenantId,
        documentId,
        key,
        scopes ?? [ScopeType.DocRead, ScopeType.DocWrite, ScopeType.SummaryWrite],
        user
    );

    context.res = {
        status: 200,
        body: token
    };
};

export default httpTrigger; 
```

<br />

Once the app is deployed, you will have to update the `AzureFunctionTokenProvider` URL to point to your `HTTPTrigger Azure Function` as shown below:

```typescript
import { AzureClient, AzureConnectionConfig } from "@fluidframework/azure-client";

const config: AzureConnectionConfig = {
    tenantId: "myTenantId",
    tokenProvider: new AzureFunctionTokenProvider("myStaticWebAppUrl"+"/api/GetAzureToken", { userId: "test-user",userName: "Test User" }),
    orderer: "https://myOrdererUrl",
    storage: "https://myStorageUrl",
};
const client = new AzureClient(config);
```

The above `api` would be accesible through the `api_location` field in `azure-static-web-apps-xxx-xxx-xxx.yml` deployment file located in the `/.github/workflows` directory once you have configured the `Static Web App` deployment.

# Building app code
Run the `npm run build` command from the root directory to generate a `dist` folder holding the bundle which will be used for deploying to our App Service.

# Deployment using Azure App Services

### Steps to create Azure App Service using Azure Portal
1. In order to create an `App Service` on Azure, you are required to have either an active Azure subscription or a free account that you can obtain from the [Microsoft Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program).
2. Once logged in, Select `App Services`. Select your `Subscription`, `Resource Group`, `Runtime Stack` as `Node 14 LTS`, `OS` as `Windows` and later select `Create`. 

> [!NOTE]
> While creating the App service, select `Windows` as the OS. You can select the OS while app creation. Once the app is 
> created, the OS cannot be changed.

### Steps to deploy to Azure App Service using Visual Studio Code
1. Click on the Azure Extension (Ctrl + Shift + A) in Visual Studio Code. In `App Service` section, under your `Subscription`, you can see the App Service you have created through the Azure Portal previously. 
2. Right click on your `App Service` and select `Deploy to Web App`. You would get an option to browse the file explorer. Select the `dist` folder and later select `Deploy`. 

    <img src='https://user-images.githubusercontent.com/46719950/133347127-8d4f1c64-a391-4e52-bead-e21f5a014b8c.png' style="height: 300px">

3. This will bring a prompt `Select the folder to deploy`. 

<img src='https://user-images.githubusercontent.com/46719950/133347129-75c84893-e2ab-448c-8b8d-22b5fedb2632.png' style="height: 200px">

<br />

Browse to the `dist` folder and select it to deploy to the App Service.

<img src='https://user-images.githubusercontent.com/46719950/133347117-94c514a1-03d3-42e5-9295-2d891ca66f1c.png' style="height: 100px">

You should now see a notification indicating that deployment is commencing and you can view the output in a terminal window.

4. After the deployment is completed, you can browse the website by again going to your `Subscription` in your App Service. Later, right click on your App Service and select `Browse Website`. 

# Deployment using Azure Static Web App

Static Web App is more cost-effective and lightweight than using the App Services. You need to link your `GitHub Account` for this type of deployment.

Once you have updated the url, commit these changes to GitHub.

### Steps to create an Azure Static Web App through Visual Studio Code

1. You are required to sign-in to GitHub in Visual Studio Code. If you are not authenticated, Static Web App extension will prompt you to sign in for the creation process.
2. Under the Static Web App section, right-click on your `Subscription` and select `Create Static Web App`. 
3. The command palette opens up on the very top. There are, in total, eight configurations that need to be set: 
    - Enter the name of the Static Web App

        <img src='https://user-images.githubusercontent.com/46719950/133347130-3d9d3782-ccee-4b69-871c-c5c125e8235b.png' style="height: 100px">

    - Select a resource group/Create a new resource group
    - Select a sku: Free/Standard
    - Choose build preset to configure default project structure as `Custom`

        <img src='https://user-images.githubusercontent.com/46719950/133347131-89224941-adc5-4c4b-91a1-ae838176366b.png' style="height: 200px">

    - Enter the location of your Application Code
        
        <img src='https://user-images.githubusercontent.com/46719950/133347115-99611f21-eed1-4090-b4eb-369d379ee182.png' style="height: 100px">

    - Enter the location of your Azure Function Code
        
        <img src='https://user-images.githubusercontent.com/46719950/133347059-ac595a86-3a9b-44e4-9395-2b7c7f3b0185.png' style="height: 100px">

    - Enter the location of your build output
        
        <img src='https://user-images.githubusercontent.com/46719950/133347119-34c473c6-6a4b-4d50-8650-4b52bb17bd1e.png' style="height: 100px">

    - Select a location for the new resource

Once the app is created, a confirmation notification will be shown in Visual Studio Code. The location of your `Application Code`, `Azure Function` and `build output` is part of your `azure-static-web-apps-xxx-xxx-xxx.yml` deployment file located in the `/.github/workflows` directory. When you finish the creation steps, your repository will have a GitHub Action to build and deploy to your Static Web App.

<img src='https://user-images.githubusercontent.com/46719950/133347124-ea092816-e898-4bb7-8842-df888a69435a.png' style="height: 150px">

Once your build is completed, you can right-click on your `Static Web App` and select `Open in Portal`.

<img src='https://user-images.githubusercontent.com/46719950/133347133-e49cf32f-7f19-4238-86a3-70d1306b9290.png' style="height: 200px">

<br />
<br />

On your Portal, in `Settings` navigate to `Environments`. Here you can see the status of your build and browse the website.

<img src='https://user-images.githubusercontent.com/46719950/133347120-d942e2bb-31dc-4356-b829-911848d2745e.png' style="height: 150px">

<br />
<br />

# Clean up Resources
If you're not going to continue to use this application, you can delete the Azure Static Web Apps instance through the extension.

In the Visual Studio Code Explorer window, return to the Static Web Apps section and right-click on your Static Web App and select `Delete`.

<img src='https://user-images.githubusercontent.com/46719950/133347125-a5a0b60c-94d9-45f8-8aaa-6a8dd3d09cb1.PNG' style="height: 200px">
