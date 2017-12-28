---
layout: post
title: Logic App Custom Connector via On Prem Gateway
author: Mattias Lögdberg
tags: [Logic Apps, Integration,Hybrid]
categories: [Logic Apps]
image: 
description: 
permalink: /logicapps/custom-connector-via-onpremgateway
---

In an ever growing hybrid world the need of exposing services that are hosted On Premise are growing and so are the limitations and requirments aswell, the newest way of exposing HTTP based services in Logic Apps is via the **Logic App Custom Connector** via the **On Premise Data Gateway** 

So previusly when having the task of calling on-premise firewall protected services from our Logic App's it wasent possible without firewall openings or other resources to help connecting.

![Task Overview](/assets/uploads/2017/12/logica-app-to-custom-webservice.png)

But now it's possible and let's solve it with the newly updated **On Premise Data Gateway** and a  **Logic App Custom Connector**.

## Prerequisites

### Install the Gateway
First of make sure you have the latest version installed, follow the [install instructions found here.](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install) .

Rememeber the name of the installed Gateway if you have many.

### Create the API
In this scenario I will expose a swagger api endpoint, the simple standard *Values* Api, that are created default in a new Web App project with type *Api App* after enabling *Swagger* we can browse the swagger definition, as follows:

![Values Controller Swagger](/assets/uploads/2017/12/values-controller-swagger.png)

Copy the URL to the swagger defintion, shown in the middle bo in the image above ( *http://localhost:51921/swagger/docs/v1*  ) and save the content as a text file. (Simplest way is to browse it and copy the swagger definition to a text file)

## Create Custom Logic App Connector 
Now we are prepared and can start of with creating the **Logic App Custom Connector** based on the swagger file, note that it's also possible to use WSDL files as source.

In the Azure Portal create a **Logic Apps Custom Connector**:

![Azure Portal Create Logic Apps Custom Connector](/assets/uploads/2017/12/azure-portal-create-custom-connector.png)

No definition is added during the creation time, sp we will have to edit the Custom Connector once it's created. Start edit the newly created Custom Connector and let's add the swagger definition:

![Go to Resource](/assets/uploads/2017/12/azure-portal-custom-connector-deployed.png)

Press the **Edit** button.

![Press the edit button](/assets/uploads/2017/12/azure-portal-custom-connector-deployed-edit.png)

Upload the swagger file:
![Upload swagger file](/assets/uploads/2017/12/azure-portal-custom-connector-deployed-upload.png)

Now the API Connection is generated based on the swagger file, scroll down and make changes as you see them neeed **but** dont' fotgett to check the *"Connect via on-premises data gateway"* checkbox and if needed change the URL's to match the URL's to the server exposing the service (I'll just keep em same since it's on the same machine for me)

![Check the checkbox for on-premise data gateway](/assets/uploads/2017/12/azure-portal-custom-connector-deployed-check-onpremgateway.PNG)

Now press Continue and move on with the process, once you are finished make sure to update the *Logic Apps Custom Connector* by pressing the check mark at the top:

![Save the updates](/assets/uploads/2017/12/azure-portal-custom-connector-deployed-save.png)

After the *Logic Apps Custom Connector* is saved we can now use it in our Logic Apps, so lets' create a Logic App and start using our newly created Custom Connector. (Make sure to create the Logic App in the same Location as your on-premise data gateway is installed otherwise it wont be available in the Logic App).
 
Inside the Logic App we can now find our **Custom Connector** among all the others and if I start searching for it I can easily select the appropiate action I want to use:

![Logic App browse the Custom Connector](/assets/uploads/2017/12/azure-portal-custom-connector-logic-app-browse.png)

After I pick my operation, I just took the *Get All* I need to create the API Connection that will be connected to my Custom Connector, here is where I can pcik authentication method and also the Gateway I want to use.

![Logic App create the API Connection for the Custom Connector](/assets/uploads/2017/12/azure-portal-custom-connector-logic-app-create-connection.png)

Pressing create will create the API Connection and the Logic App is completed for this sample, let's save the Logic App and test run it, the result is that we will get the "values" returned as promised:

![Logic App run result](/assets/uploads/2017/12/azure-portal-custom-connector-logic-app-run.png) 
 
 
The flow is now completed and since we have installed an *On Premise Data Gateway* on our webservice we can use the **Logic App Custom Connector** definition in our calls to our on-premise webservice, when creating the Logic App we are creating a *API Connection* based on the  **Logic App Custom Connector** that holds credentials and Gateway information to be used when executing the Logic App run and we can now connect to our firewall protected on-premise webservice!

![Ovreview result](/assets/uploads/2017/12/logica-app-to-custom-webservice-completed.png)  
 
 
## Summary:
I think this is an amasing new feature that we really have wanted for a long time. This enables so mutch simpler exposure of services on-premises and in a secure and reliable way.
No need for fireall openings, vpn's, express routes or reverse proxies in a DMZ, we can just install the gateway and use it as our entry point.

Remember this is still in preview and I've encountered some issues when working with SOAP so there are some fixes needed, but it looks good so far!
