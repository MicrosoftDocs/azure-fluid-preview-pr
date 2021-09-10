---
title: Architecture
menuPosition: 1
---

> [!NOTE]
> Outstanding issues for this article:
> - Need a post-name change terminology review of the service and each component 
> - Each component (loader, containers, service) needs a link to the corresponding detail article
> - capitalization (e.g. is Loader capitalized in Fluid Loader?)

There are three primary concepts to understand when building an application with Fluid.

- Service
- Container
- Shared objects

### Service

Fluid clients require a centralized service that all connected clients use to send and receive operations. When using
Fluid in an application you must use the correct package that corresponds to the underlying service you are connecting
to.

For the Azure Fluid Relay service this package is `@fluidframework/azure-client`. This package helps create and load
Fluid containers

### Container

The container is the primary unit of encapsulation in Fluid. It consists of a collection of shared objects and supporting APIs to manage the lifecycle of the container and the objects within it.

Creating new containers is a client-driven action and container lifetimes are bound to the data stored on the supporting server. When getting existing containers it's important to consider the previous state of the container.

For more about containers see [Containers](./containers.md).

### Shared objects

A shared object is an object type that powers collaborative data by exposing a specific API. Many shared objects can exist within the context of a container and they can be created either statically or dynamically. Distributed Data Structures(DDSes) and DataObjects are both types of shared objects.

For more information see [Data modeling](./data-modeling.md).

## Package structure

There are two primary packages you'll use when building with Fluid. The `fluid-framework` package
and a service-specific client package like `azure-client`.

For more information see [Packages](https://fluidframework.com/docs/build/packages/).

### The `fluid-framework` package

The `fluid-framework` package is a collection of core Fluid APIs that make it easy to build and use applications. This package contains all the common type definitions as well as all the primitive shared objects.

### Service-specific client packages

Fluid works with multiple service implementations. Each service has a corresponding service-specific client package. These packages contain a common API structure but also support functionality unique to each service.

The Tinylicious service is a local Fluid service. This documentation uses `@fluidframework/tinylicious-client` (or simply `client`). For specifics about each service-specific client implementation see their corresponding documentation.

- `@fluidframework/tinylicious-client` -- a client dedicated to the [Tinylicious]({{< relref "Tinylicious" >}}) service.
- `@fluidframework/azure-client` -- a client for the [Azure Fluid Relay service]({{< relref "azure-frs.md" >}}). This client can also use Tinylicious for local testing.
