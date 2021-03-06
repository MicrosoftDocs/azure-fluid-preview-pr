---
author: sharmakhushboo
description: Better understand the authorization flow in Azure Fluid Relay
title: Authorization Flow in Azure Fluid Relay
ms.author: sharmak
ms.date: 02/09/2021
ms.service: app-service
ms.topic: reference
---

# Authorization Flow in Azure Fluid Relay

Once you create an Azure Fluid Relay resource, you will be given a tenantId and a tenant key. This information is used in your application to connect the clients with the Azure Fluid Relay service. Your application is responsible for authenticating and authorizing the end users. Once authorized, the end users will interact with your application. When requests are submitted via your application, Azure Fluid Relay will only authorize your application. 

Requests sent to Azure Fluid Relay should contain a JWT token. This token should be signed by the tenant key (obtained during provisioning) and should abide by Azure Fluid Relay Token Contract. Azure Fluid Relay authorizes every request via JWT token validation. 

This article talks about how Azure Fluid Relay will authorize different requests. 

<br/>

## Socket Connection Requests

When your application submits a request to Azure Fluid Relay to establish a socket.io connection, a JWT token should be sent as a parameter to the connect_document message event. The token is validated against the Azure Fluid Relay Token Contract. The important criterias for token validation are: 
* Token expiry. 
* Signed with the correct tenant key. 
* Valid tenantId is used. 
* Valid documentId 
If the validation is successful, the socket connection is established and connect_document response is returned. Else is it rejected with connect_document_error:Invalid token response. 

<br/>

## Https Requests

When Https requests are submitted to Azure Fluid Relay, a JWT token should be sent in the authorization header of the request. The token validation criteria is similar to the socket connection requests. If the token validation is successful, the request is processed, and the corresponding response is returned. Else a Http 401 (Unauthorized) response is returned.
