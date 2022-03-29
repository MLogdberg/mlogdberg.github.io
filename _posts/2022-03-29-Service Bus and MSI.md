---
layout: post
title: MSI with Service Bus
author: Mattias Lögdberg
tags: [Azure, Service Bus, MSI]
categories: [Development]
image: 
description: 
permalink: /blog/2022/03/servicebusmsi
---

Using MSI with Service Bus is a great addition to the SaaS key that is the standard authentication mechanism.

What is MSI (Managed Service Identity)? [Read more in previous post](/blog/2022/01/msiwhatandwhy).
In a short summary it's a way of giving your resources (can be a Logic App, a Function App etc.) an identity and assing permissions.


In the case of using MSI with **Service Bus** it can be used with our internal services accessing the **Service Bus** **queues**, **topics** or **topic subscriptions**.


The Access Roles that are available on Service Bus are the following:
![Azure Service Bus Roles](/assets/uploads/2022/03/servicebusroles.png)

* **Azure Service Bus Data Owner**: Grants access to both send and recieve
* **Azure Service Bus Data Reciever**: Grants access to read messages from queues or topic subscriptions
* **Azure Service Bus Data Sender**: Grants access to send message on queues or topics.


When assigning roles there are some things to think about:
* What are the current service suppose to be able to access? All queues or specific queues?
* If you use Functions or Logic Apps as Reciever  of messages from either queue or topic subscriptions you must have assigned the **Azure Service Bus Data Reciever** on the specific queue or topic subscription. It will not work to assign it on the whole namespace.

Let's take an example of reading messages from a queue with a Logic App.
We are creating a consumption Logic App and the first step we need to do is assignt an identity to the Logic App.
![Assign Identity to Logic Apps](/assets/uploads/2022/03/logicappassignidentity.png)

Now we  need to Assign Roles to the Logic Apps Identity to the Queue, remember that it has to be on the Queue in order to work with recieving messages.

Go to the **Service Bus** namespace and to the **Queue**, in my case *Messages* and add the role assingment **Azure Service Bus Reciever** to our Logic App.

![Assign Permissions to Logic Apps](/assets/uploads/2022/03/SBAssignRole.png)

Pick the role and then select the Managed Identity that is connected to your Logic App

![Pick identity of Logic Apps](/assets/uploads/2022/03/SBassignRolepickidentity.png)

Now we can create a trigger in the Logic Apps Designer as follows:

![Create Trigger in Logic Apps](/assets/uploads/2022/03/designerSBTrigger.png)

If you have assigned the Role on the queue you are now ready to start recieving messages.

### Removing the access SaaS key
There is an option to remove access with the SaaS key. This is preferable if the Service Bus namespace can be used with  resources who can use Managed Idenity or Service Principal when authenticating to the Service Bus Namespace. 
If the SaaS key access is removed the whole namepsace is only accessible with a valid Aure user. This is a much securer way and moves the access control to Azure AD rather than a Service Bus specific authentication mechanism.

To close the Access Key go to the namespace and press the Local Authentication: *Enabled* link and change to *Disabled*.
After this only access via Azure AD Roles will be working.

![Disable Local Auth](/assets/uploads/2022/03/localauth.png)

### Troubelshooting
If no messages are recieved check the triggerhistory and verify that there are no errors.

![Trigger History](/assets/uploads/2022/03/triggerhistory.png)


## Summary
Using Managed Identity or Servcei Principal with Service Bus is a great feature, it's much easier and secure. No static keys that can wander of and no unkown users. Developers accessing the namespace have to have valid Azure AD accounts that have assigned permissions.
I know it can be tricker in the start but in the long run it will help out alot.