---
layout: post
title: Innovation enabled by Integration
author: Mattias Lögdberg
tags: [Innovation, Integration,Architecture, Integration Architecture, Power Apps, Flow]
categories: [Innovation, Power Apps, Flow]
image: 
description: 
permalink: /innovation/enable-innovation
---

Normally we focus on technical implementations and problem solving but from time to time we get the opportunity to expand the view and focus on innovative thinking. This is the time where we can show our customers the real **value** of applying **good architecture and principles**. Currently I’m having so much fun doing innovative prototyping work with one of my customers that I need to share it with you. We are using **Power Apps** and **Flow** for our prototyping in order to get prototypes done fast and easy, without spending too much time and effort. 


Power Apps is a platform where you easy can build apps in a point-and-click approach and to multiple device types at the same time, read more at: [https://powerapps.microsoft.com/](https://powerapps.microsoft.com/)


Flow is a lightweight integration platform that runs on top of Logic Apps, it’s designed to make it easy to create your own flows and turn repetitive tasks into automated workflows, read more at [https://emea.flow.microsoft.com/](https://emea.flow.microsoft.com/)


These platforms are very easy to get started with and there are several samples of applications regarding office365 and common applications. But the real power for innovation comes clear if you have a good API landscape that could be used when building these applications. 


We have a customer that we started to work with for almost a year ago and now when all the hard work is done, and we have a good integration landscape we can harvest the benefits of our good architecture. I started with showing the case described below and now they really hunger for more of this kind of innovation both for internal and external use. It’s so wonderful to see how our hard work pays off. Showing that we really are the “integration heroes”, we are the enablers, making this possible.


## Architecture that enables innovation?
In reality it’s quite simple, create a platform that contains API’s and flows that can be reused. Make sure to build domain model-based API’s and function driven API’s and focus on making them simple to use and most important reusable. Here is a Sample Architecture: 

![Integration Architecture Sample](/assets/uploads/2018/08/integration-archtecture-sample.png)


And in our API Management instance we have created a few domain model API’s:

![API List Sample](/assets/uploads/2018/08/api-sample.png)

A few months back we helped in replacing an internal system for product information, it was a system where you could enter a serial number and get basic information about the product. The end solution was a website that used a couple of API’s to retrieve the information needed. The fun part of this was that most of the API’s where already created in other projects for other/similar needs but used from other systems/apps. All we did was fine tune an API, leaving us with time to innovate. On a computer it makes sense to write a serial number but on a mobile phone we expect to be able to scan the serial. Since we had the time we started to investigate the problems around this and if this was solvable. 


We started to look in to this and the first “problem” was that the serial plates on the machines had no barcodes, only pure text:

![Serial Plate Sample](/assets/uploads/2018/08/serial-plate-sample.png)

Luckily this function exists in the Cognitive services in Azure read more at [Azure Cogniative Services](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc)


We started to test this with a Power App that used a Flow to send the image to the Cognitive Service. The Power App solution is very simple, it uses the camera to take a picture and afterwards sends the picture to a Flow that utilizes the cognitive service to identify the serial number from the plate. The result from the Flow is a URL with the serial number that we used to launch the website, making use of the web app. With this we had expanded the functionality with a Serial Plate Scanner and provided basis for the decision on investing in a mobile app.


Here is the flow:

![Microsoft Flow Overview](/assets/uploads/2018/08/microsoft-flow-overview.png)


Here is the Power App in the Power Apps Studio developed in my browser:

![Microsoft Power App](/assets/uploads/2018/08/powerapp-sample.png)

As you can see there is not much going on but a *Camera* item with a *OnSelect* statement that launches the web browser with the URL that comes back from our *Flow* “Get-Path-To-Product”.



The flow is also quite simple:

![Microsoft Flow Solution](/assets/uploads/2018/08/flow-overview.png)

The tricky part was how to send the image to the *Cognitive Services*, in this case we created a *compose* action with a *dataUritoBinary* function: 

![Microsoft Flow Action Compose](/assets/uploads/2018/08/flow-action-compose.png)
***dataUriToBinary(triggerBody()['Skapa_Indata'])**

And in the HTTP action we send the binary data to the OCR service: (in the sample below the OCR service is behind our API Management instance)

![Microsoft Flow Action HTTP](/assets/uploads/2018/08/flow-action-http.png)

All in all, the prototype was easy and fast to build, but most importantly it successfully gave us the power to demo the concept. Providing a visual demo to show the value of having an app that can scan the serial plate. This gave a good understanding in the benefits of the investment on a native app to decision makers without spending too much time and resources.


## Summary:
The app is currently under development and we have more of these innovative prototypes coming up to prototype new or upgraded functionality in a variety of sizes. Best part of using this approach is that we can create a prototype in a day or two providing fast prototyping, giving decision makers a visual and testable solution to review. All thanks to our hard work with API’s, workflows and the amazing tools Power Apps and Flow!

Become the **Integration Hero** and show the business the **innovation power** that is hidden in their business.

![Help Business Evolve](/assets/uploads/2018/08/mlogdberg-help-business-evolve.jpg)
