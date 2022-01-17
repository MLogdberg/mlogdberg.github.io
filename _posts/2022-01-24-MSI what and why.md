---
layout: post
title: MSI what and why?
author: Mattias Lögdberg
tags: [Azure AD, MSI]
categories: [Development]
image: 
description: 
permalink: /blog/2022/01/msiwhatandwhy
---

MSI or Managed Service Identity that it stands for has been aroud for some time but quite recently a new set of resources has been integrated in to this and we need to start thinking of using MSI alot more. But most of Microsoft's documentation around this is still for Virtual Machines and I want to enlighten this alot more for IPaaS services like Logic Apps, Functions, API Management, Web Apps, DataFactory Service Bus and Event Grid.

So MSI is an identity of a resource just like any other user in Azure AD. The beuty of this is that with this we can assign access rights to that identity and the resource now authenticates by itself (with credentials no one else nows) and with those credentials it can use the resource.
I usually like to discribe at as a virtualization of the service and used for authorization purposes, in the image bellow we can see that an Azure Function are doing requests to a Cosmos. This is allowed since we have assigned the role **Cosmos DB Built-in Data Contributor** to the identity of the function in a scope 

![Result in VS Code](/assets/uploads/2022/01/drawing-ad-virtualization.png)

What Resources can use MSI: (there are more but these are Integration focused)
* Logic Apps, 
* Azure Functions
* API Management 
* Web Apps 
* DataFactory 
* Event Grid
Read the full list [here](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities).


What resources can we use authenticate to with MSI.
* Azure SQL
* Cosmos
* Service Bus
* Event Hub
* Azure Functions (if you use Azure AD as provider in *Authentication*)
* Web Site (if you use Azure AD as provider in *Authentication*)
* Logic Apps (If you use Oauth Authorization)
  * In Logic Apps we can use MSI to connect to resources like Event Grid for creating subscriptions.


###Isen't this what we accomplished before?
No before we created a SQL account, took keys from Cosmos, Service Bus etc, create a service principal or a service user in AD to accomplish other acesses. All of these could potentiaally be simplified and changed to only give the resource the appropiate access rights. Note not all resources is supporting this yet, but this is an area where Microsoft has done some great gains last 6 months.

###But isent this making my development process a pain?
No on the contrary, you as a developer don't need to gather all these credentials either, development and testing can be used with your own account. Making it easier to get started and another great benefit is that you canno't accidently keep credentials and access after an completed assignement or project. With good documentation it's also easier to get a new team mate up to speed. Another good thing is that we know that credentials are not wondering away when members leave the team.

A tip here is to create a Azure AD Group and add all developers to it that need specific access, remove them when they don't need it. An extra step would be creating a PIM group for it.

###Read more:
* [What are managed identities for Azure resources? by Microsoft](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview))
* [What is Azure AD Privileged Identity Management? by Microsoft](https://docs.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-configure)
* [List of resources that support Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities).
