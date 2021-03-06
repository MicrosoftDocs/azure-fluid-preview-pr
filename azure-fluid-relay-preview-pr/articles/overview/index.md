---
author: hickeys
description: Better understand the typical use cases and key benefits of the Azure Fluid Relay Framework
title: Azure Fluid Relay overview
ms.author: sabroner
ms.date: 01/22/2021
ms.service: app-service
ms.topic: overview
---

> [!NOTE]
> I updated the product name in the descrption, but it may be incorrect. If this is an incorrect use of the Azure Fluid Relay name, please change it back.

# Azure Fluid Relay overview

The Azure Fluid Relay service is a managed offering for the Fluid Framework that helps developers build real-time collaborative
experiences and replicate state across connected JavaScript clients in real-time.

## What is the Fluid Framework?

Fluid Framework is a collection of client libraries for distributing and synchronizing shared state. These libraries
allow multiple clients to simultaneously create and operate on shared data structures using coding
patterns similar to those used to work with local data.

More documentation on the [FluidFramework.com](https://fluidframework.com).

## Why Fluid?

Because building low-latency, collaborative experiences is hard!

Fluid Framework offers:

* Client-centric application model with data persistence requiring no custom server code.
* Distributed data structures with familiar programming patterns.
* Very low latency.

The developers at Microsoft have built collaboration into many applications, but many required application specific
server-side logic to manage the collaborative experience. The Fluid Framework is the result of Microsoft's investment
in reducing the complexity of creating collaborative applications.

What if you didn't have to invest in server code at all? Imagine if you could use a general purpose server
that was designed to be lightweight and low cost. Imagine if all your development was focused on the client
experience and data sync was handled for you. That is the promise of Fluid.

## Focused on the client developer

Applications built with Fluid Framework require zero custom code on the server to enable sophisticated data sync
scenarios such as real-time typing across text editors. Client developers can focus on customer experiences while
letting Fluid do the work of keeping data in sync.

Fluid Framework works with your application framework of choice. Whether you prefer straight JavaScript or
a framework like React, Angular, or Vue, Fluid Framework makes building collaborative experiences simple and
flexible.

## How Fluid works

Fluid was designed to deliver collaborative experiences with blazing performance. To achieve this goal, the team kept
the server logic as simple and lightweight as possible. This approach helped ensure virtually instant syncing across
clients with low server costs.

To keep the server simple, each Fluid client is responsible for its own state. While previous systems keep a source of
truth on the server, the Fluid service is responsible for taking in data operations, sequencing the operations, and
returning the sequenced operations to the clients. Each client is able to use that sequence to independently and
accurately produce the current state regardless of the order it receives operations.

The following steps are a typical flow.

* Client code changes data locally.
* Fluid runtime sends that change to the Fluid service.
* Fluid service sequences that operation and broadcasts it to all clients.
* Fluid runtime incorporates that operation into local data and raises a "valueChanged" event.
* Client code handles that event (updates view, runs business logic).

## Getting to version 1.0

The core technology powering Fluid Framework is mature and stable. However, the layers built on top of that
foundation are still a work in progress. Over the coming months we will be evolving APIs, adding new features,
and working to further simplify using the framework. These changes are driven by Microsoft's use of
Fluid internally and by requirements we are gathering from developers currently building on Fluid.

Fluid Framework is not ready to power production-quality solutions yet. But we are excited to open source it now
to give developers an opportunity to explore, learn, and contribute both through feedback and through direct
participation.
