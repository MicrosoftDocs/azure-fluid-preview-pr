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

In this Quick Start, we will be getting a 'dice roller' Fluid application up and running against the Azure Fluid Relay service. The code for this sample is currently living in a special branch of our Hello World repository. You can find it at [github.com/microsoft/FluidHelloWorld/tree/new-hello-world.](https://github.com/microsoft/FluidHelloWorld/tree/new-hello-world)

> This demo is current as of version ```^0.41.0```

## Set up your development environment

To get started, you need the following installed.

- [Node.js](https://nodejs.org/en/download) -  V12.17+
- Code Editor - We recommend [Visual Studio Code](https://code.visualstudio.com/).
- [Git](https://git-scm.com/downloads)

## Getting Started Locally

Open a new command window and navigate to the folder you where you want to install the project, and then clone the "new-hello-world"
branch of the [FluidHelloWorld repo](https://github.com/microsoft/FluidHelloWorld/tree/new-hello-world) with the following commands. 
The cloning process will create a subfolder named FluidHelloWorld with the project files in it.

```bash 
git clone https://github.com/microsoft/FluidHelloWorld.git
cd FluidHelloWorld
git checkout --track origin/new-hello-world
```

<blockquote>
The `new-hello-world` branch has our latest & greatest thinking about how the Fluid Framework API surface should look. These instructions applicable to all samples that use our API changes. Check out <a href="https://github.com/microsoft/FluidExamples">Github.com/Microsoft/FluidExamples</a>
</blockquote>

Navigate to the newly created folder and install required dependencies.

```bash
npm install
```

Start both the client and server.

```bash
npm start
```

A new browser tab will open to <http://localhost:8080> and you will see the dice roller appear! To see collaboration in
action copy the full URL in the browser, including the ID, into a new window or even a different browser. This opens a
second client for your dice roller application. With both windows open, click the **Roll** button in either and note
that the state of the dice changes in both clients.

Now, let's run the same sample against the Azure Fluid Relay service.

## Running against the Azure Fluid Relay service

To run against the Azure Fluid Relay service, we'll have to make a simple code change to ```app.ts```.

We're going to replace Tinylicious, our local test service, with Routerlicious, our internal name for the Azure Fluid Relay Service.

### Add the FRS-Client package
Install ```@fluid-experimental/frs-client``` and import the client and config in ```app.ts```

```sh
npm i @fluid-experimental/frs-client
```

```typescript
import { FrsConnectionConfig, FrsClient } from '@fluid-experimental/frs-client';
```

### Replace the Tinylicious client with the FRS client

Remove the following line
```typescript
TinyliciousClient.init();
```

and replace it with the RouterliciousService constructor and your Azure Fluid Relay Service configuration.

```typescript
const connectionConfig: FrsConnectionConfig = {
  type: "key",
  tenantId: "",
  key: "",
  orderer: "",
  storage: "",
};

FrsClient.init(connectionConfig);
```

Finally, fetch the container from FRS instead of tinylicious using your newly initialized client.

```typescript
const [fluidContainer] = isNew
    ? (await FrsClient.createContainer({ id }, containerSchema))
    : (await FrsClient.getContainer({ id }, containerSchema));
```

🥳**Congratulations**🎉 You have successfully taken the first step towards unlocking the world of Fluid collaboration.

----
