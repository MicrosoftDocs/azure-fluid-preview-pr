---
title: Quickstart to learn how to use Azure Fluid Relay
description: A quickstart for using Azure Fluid Relay to create a dice roller
author: SamBroner
ms.service: azure-fluid
ms.devlang: typescript
ms.topic: quickstart
ms.custom: devx-track-typescript
ms.date: 01/21/2021
ms.author: sabroner 
---

# Quick Start

In this Quick Start, we will be getting a 'dice roller' Fluid application up and running against the Azure Fluid Relay
service.

The starting code for this sample is at <https://github.com/microsoft/FluidHelloWorld>.

## Set up your development environment

To get started, you need the following installed.

- [Node.js](https://nodejs.org/en/download) -- version 12.17 or higher
- Code Editor -- We recommend [Visual Studio Code](https://code.visualstudio.com/).
- [Git](https://git-scm.com/downloads)

## Getting Started Locally

Open a new command window and navigate to the folder you where you want to install the project, and then clone the
[FluidHelloWorld repo](https://github.com/microsoft/FluidHelloWorld) with the following commands. The cloning process
will create a subfolder named FluidHelloWorld with the project files in it.

```bash
git clone https://github.com/microsoft/FluidHelloWorld.git
```

Navigate to the newly created folder and install required dependencies.

```bash
cd FluidHelloWorld
npm install
```

Start both the client and a local server.

```bash
npm start
```

A new browser tab will open to <http://localhost:8080> and you will see the dice roller appear! To see collaboration in
action copy the full URL in the browser, including the ID, into a new window or even a different browser. This opens a
second client for your dice roller application. With both windows open, click the **Roll** button in either and note
that the state of the dice changes in both clients.

## Running against the Azure Fluid Relay service

To run against the Azure Fluid Relay service, you'll make a code change to `app.ts`. The app is currently configured to
use a local in-memory service called Tinylicious, which runs on port 7070 by default.

### Add the FRS-Client package

Install `@fluidframework/azure-client` and import the client in `app.ts`.

```sh
npm i @fluidframework/azure-client
```

```typescript
import { AzureClient, InsecureTokenProvider } from "@fluidframework/azure-client";
```

### Replace the Tinylicious client with the Azure client

To use an Azure Fluid Relay instance instead, replace the configuration values with your Azure Fluid Relay tenant ID,
orderer, and storage URLs that were provided as part of the onboarding process. Then pass that configuration object into
the `AzureClient` constructor.

Remove the following line:

```typescript
const client = new TinyliciousClient();
```

Replace the removed code with the AzureClient constructor and your Azure Fluid Relay service configuration.

```typescript
const azureUser = {
  userId: "Test User",
    userName: "test-user"
}

// This configures the AzureClient to use a remote Azure Fluid Service instance.
const prodConfig = {
    tenantId: "[REPLACE WITH YOUR TENANT GUID]",
    tokenProvider: new InsecureTokenProvider("", azureUser),
    orderer: "[REPLACE WITH YOUR ORDERER URL]",
    storage: "[REPLACE WITH YOUR STORAGE URL]",
};

const client = new AzureClient(prodConfig);
```

### TokenProvider

The Azure Fluid Relay resource creation process provides you with a secret key for your tenant. You can use
`InsecureTokenProvider` to generate and sign authentication tokens such that the Azure Fluid Relay service will accept
it. **To ensure that the secret doesn't get exposed, this should be replaced with another implementation of
ITokenProvider that fetches the token from a secure, developer-provided backend service prior to releasing to
production.**

### Build and run the client only

Now that you've updated the `AzureClient` configuration, you can run just the client to test it. You no longer need to
run a local Fluid service, because you're using the remote Azure Fluid Relay service instance!

```bash
npm run start:client
```

🥳**Congratulations**🎉 You have successfully taken the first step towards unlocking the world of Fluid collaboration.
