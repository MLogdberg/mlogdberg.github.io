---
layout: post
title: Troubelshooting Azure API Management
author: Mattias Lögdberg
tags: [API Management, Integration,Serverless]
categories: [API Management]
image: 
description: 
permalink: /blog/2021/01/troubelshootingazureapim
---

We often encounter situations where we need to troubelshoot our API's and in this session I will go thru the different options we have. Following is the 3 different places where we can troubelshoot, here is a quick overview of the three options and here is a link to a video using all these options.
[![Troubelshooting in Azure API Management](http://youtu.be/0o763Yy00NM/0.jpg)](https://youtu.be/0o763Yy00NM "Troubelshooting in Azure API Management - Click to watch")


### Built in Analytics
In the built in Analytics we can easily see a good overview and status on our API's. We can see hwo request's are going, succes, failed? and so on we can also see data transfered and get an insight into response times. Filtering can be done on API's, Products and even subscriptions to get an insight into how a specific consumer experiences our API's. Using the Request tab we can even se the requests made, very usefull i you want to find isues like 404 responses or similar.

![Analytics startpage](/assets/uploads/2021/01/apim-analytics-startpage.png)


### Application Insights
In addition we can add other tools to get deeper insight into what is happening, i.e why do the consumer get a 401? Or find the error text with that 500 response that is returned. If our backend resource also uses Applciation Insight's you get a the full end-to-end experience where you can track all relevant information connected to the request like dependencies, logged events and so forth.
In Application Insights we can see all failures under the **Failures** tab.

![App Insigths Failures](/assets/uploads/2021/01/apim-appinsights-failures.PNG)

Clicking you thru we can see the timeline and also the specific error text, in my case there is a **TokenClaimValueMismatch** at the policy section **validate-jwt** and if you watch the video we can see that this is exactly why we get the 401. The token provided is missing a permission.

![App Insigths Failures](/assets/uploads/2021/01/apim-appinsights-endtoend.png)


### Live Debugging in VSCode
If we need to go deeper in to your troubelshooting in your API Management Policy we can use the [Azure API Management VSCode extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). In this extension we can edit our policies for our API Management instance and upload them back when done. 

![VSCode Policy](/assets/uploads/2021/01/apim-vscode-policy.png)

We can start a debugg session where we can send a request and debug the policy step by step! This is awesome and so usefull! (only that request will be captured, so this can safely be done even in production). Right click the Operation and select "Start Policy Debugging".

![VSCode Policy Debugging](/assets/uploads/2021/01/apim-vscode-policydebugging.PNG)

The Request is creted and you can add you settings, payload etc and then press "Send Request".

> When using versioning please verify the URL, there is currently no support for the versioning so in my case the versioning is lost my URL is in the sample bellow *https://{{apimname}}.azure-api.net/Orders/orders/my* but due to versioning should be *https://{{apimname}}.azure-api.net/Orders/v1/orders/my*.

![VSCode Send Request](/assets/uploads/2021/01/apim-vscode-sendrequest.png)

For more details and talk, see the video: [![Troubelshooting in Azure API Management](http://youtu.be/RiYBKqk9N3A/0.jpg)](https://youtu.be/RiYBKqk9N3A "Troubelshooting in Azure API Management - Click to watch")

## Summary:
Beeing a great API publisher is about building robust API's and taking care of our API Consumers and help them to give a great experience on all fronts, specially when there is issues. We want to be able to find em, fix em and report back to the consumer with either help on what to do to fix it on the consumer side or a "we fixed it" from the API publisher side. 