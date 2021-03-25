---
title: Quickstart to learn how to use Azure Fluid
description: A quickstart for using Azure Fluid to create a dice roller
author: SamBroner
ms.service: azure-fluid
ms.devlang: typescript
ms.topic: quickstart
ms.custom: devx-track-typescript
ms.date: 01/21/2021
ms.author: sabroner 
---

# Quick Start

In this Quick Start, we will be getting a 'dice roller' Fluid application up and running against the Azure Fluid service. The code for this sample is currently living in a special branch of our Hello World repository. You can find it at [github.com/microsoft/FluidHelloWorld/tree/new-hello-world.](https://github.com/microsoft/FluidHelloWorld/tree/new-hello-world)

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

Now, let's run the same sample against the Azure Fluid service.

## Running against the Azure Fluid service

To run against the Azure Fluid service, we'll have to make a simple code change to our utils.

<blockquote>
We expect the FluidStatic API surface to experience significant changes over the next months. The rest of the API surface should be much more stable.
</blockquote>

Navigate to ```src/utils/FluidStatic.ts```. It will include the following.

```typescript
export class Fluid {
    public static async createContainer(docId: string, factories: any[]): Promise<FluidDocument> {
        const container = await getTinyliciousContainer(
            docId,
            getRuntimeFactory(factories),
            true /* createNew */
        );
        const document = new FluidDocument(container, true /* createNew */);
        return document;
    }
    public static async getContainer(docId: string, factories: any[]): Promise<FluidDocument> {
        const container = await getTinyliciousContainer(
            docId,
            getRuntimeFactory(factories),
            false /* createNew */
        );
        const document = new FluidDocument(container, false /* createNew */);
        return document;
    }
}
```

We'll be replacing the getTinyliciousContainer method with the getRouterliciousContainer method. Routerlicious is our name for the driver set used by the Azure Fluid service. Tinylicious is the name of drivers for the local test service.

<blockquote>
The <b>getRouterliciousContainer</b> package has not yet been published to NPM. 

You'll have to include the following file manually for now: https://github.com/microsoft/FluidFramework/blob/main/examples/utils/get-tinylicious-container/src/getRouterliciousContainer.ts

Instructions at bottom.
</blockquote>

```typescript
const config: IRouterliciousConfig = {
    tenantId: "example tenantId",
    key: "example key",
    orderer: "example orderer url",
    storage: "example storage url",
}

// eslint-disable-next-line @typescript-eslint/no-extraneous-class
export class Fluid {
    public static async createContainer(docId: string, factories: any[]): Promise<FluidDocument> {
        const container = await getRouterliciousContainer(
            docId,
            getRuntimeFactory(factories),
            true /* createNew */,
            config
        );
        const document = new FluidDocument(container, true /* createNew */);
        return document;
    }
    public static async getContainer(docId: string, factories: any[]): Promise<FluidDocument> {
        const container = await getRouterliciousContainer(
            docId,
            getRuntimeFactory(factories),
            false /* createNew */,
            config
        );
        const document = new FluidDocument(container, false /* createNew */);
        return document;
    }
}

```


🥳**Congratulations**🎉 You have successfully taken the first step towards unlocking the world of Fluid collaboration.

----

## Appendix: How do I include getRouterliciousContainer in my project?

Add the following dependencies to your package.json
```json
"@fluidframework/container-definitions": "^0.34.0-14942",
"@fluidframework/core-interfaces": "^0.34.0-14942",
"@fluidframework/driver-definitions": "^0.34.0-14942",
"@fluidframework/protocol-definitions": "^0.1019.0-0",
"@fluidframework/routerlicious-driver": "^0.34.0-14942",
"@fluidframework/test-runtime-utils": "^0.34.0-14942",
```

Install them
```bash
npm i
```

Add file ```getRouterliciousContainer.ts``` to ```src/utils```.

Paste the following into ```getRouterliciousContainer.ts```
```typescript
import { IRuntimeFactory } from "@fluidframework/container-definitions";
import { IRequest } from "@fluidframework/core-interfaces";
import { IFluidResolvedUrl, IResolvedUrl, IUrlResolver } from "@fluidframework/driver-definitions";
import { RouterliciousDocumentServiceFactory } from "@fluidframework/routerlicious-driver";
import { InsecureTokenProvider } from "@fluidframework/test-runtime-utils";
import { IUser } from "@fluidframework/protocol-definitions";
import { getContainer } from "@fluidframework/get-tinylicious-container";
// @ts-ignore
import jwt from "jsonwebtoken";

export interface IRouterliciousConfig {
    orderer: string,
    storage: string,
    tenantId: string,
    key: string,
}

class SimpleUrlResolver implements IUrlResolver {
    private readonly token: string;

    constructor(
        readonly containerId: string,
        private readonly config: IRouterliciousConfig,
        readonly user: IUser,
    ) {
        this.token = jwt.sign(
            {
                user,
                documentId: containerId,
                tenantId: config.tenantId,
                scopes: ["doc:read", "doc:write", "summary:write"],
            },
            config.key);
    }

    public async resolve(request: IRequest): Promise<IFluidResolvedUrl> {
        const documentUrl = `${this.config.orderer}/${this.config.tenantId}/${request.url}`;
        return Promise.resolve({
            endpoints: {
                deltaStorageUrl: `${this.config.orderer}/deltas/${this.config.tenantId}/${request.url}`,
                ordererUrl: `${this.config.orderer}`,
                storageUrl: `${this.config.storage}/repos/${this.config.tenantId}`,
            },
            tokens: { jwt: this.token },
            type: "fluid",
            url: documentUrl,
        });
    }
    public async getAbsoluteUrl(resolvedUrl: IResolvedUrl, relativeUrl: string): Promise<string> {
        if (resolvedUrl.type !== "fluid") {
            throw Error("Invalid Resolved Url");
        }
        return `${resolvedUrl.url}/${relativeUrl}`;
    }
}

/**
 * Connect to an implementation of the Routerlicious service and retrieve a Container with
 * the given ID running the given code.
 *
 * @param containerId - The document id to retrieve or create
 * @param containerRuntimeFactory - The container factory to be loaded in the container
 * @param createNew - Is this a new container
 * @param config
 */
export async function getRouterliciousContainer(
    containerId: string,
    containerRuntimeFactory: IRuntimeFactory,
    createNew: boolean,
    config: IRouterliciousConfig,
) {
    const user = {
        id: "unique-id",
        name: "Unique Idee",
    };

    const tokenProvider = new InsecureTokenProvider(config.key, user);
    const documentServiceFactory = new RouterliciousDocumentServiceFactory(tokenProvider);

    const urlResolver = new SimpleUrlResolver(containerId, config, user);

    return getContainer(
        containerId,
        createNew,
        { url: containerId },
        urlResolver,
        documentServiceFactory,
        containerRuntimeFactory,
    );
}


```