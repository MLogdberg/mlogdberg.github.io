---
layout: post
title: API Management and Auth0 Consumer Friendly API's 
author: Mattias Lögdberg
tags: [API Management, Integration,Serverless, Auth0, IDP, OAuth]
categories: [API Management]
image: 
description: 
permalink: /blog/2020/06/apimconsumerfriendlyapis
---

Today we will extend our setup with API Management even further. Focusing on consumer friendly API's and in this setup that is knowing our user more. In this example we will add the customer id to the consumer representation in Auth0 and use this id to give the consumer an operation called **My orders**. No need for knowing or using or adding an id in the request. The id will be added to the token from Auth0 and then extracted in API Management and sent downstreams to filter the result set.

> An Identity provider is a service that a user/application is signing in to (just like Azure AD) and this provider has functionality to provide needed information and grant access to requested resources that the IDP is handling. Just like the fact that you have access to a resource or resource group inside your subscription in Azure.

In API Management we have a trust setup as of the previus post [Setup Auth0 with API Management](/blog/2020/05/setupauth0withapim). Then we added RBAC permission in the second post [RBAC with API Management](/blog/2020/06/rbacwithapim) where we added functionality to provide granular control over who has access to what operation. Now we will continue this and focus on extending this setup with adding a customerid to our consumer representation i Auth0 and then extract that id in API Management to enforce that they only can access these customers orders. Providing an operation to get just my orders without any additional parameters. This is achieved via the  [validate-jwt](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) policy and then reading the JWT token custom claim and store it in a variable for later use.

```xml
<set-variable name="customerid" value="@(((Jwt)context.Variables["token"]).Claims.GetValueOrDefault("https://mlogdberg.com/customerid","-1"))" />
```

The end scenario will look like the illustration bellow where we can extract the customerid from the token and use it later down the path.

![Scenario image](/assets/uploads/2020/06/extend-apim-auth0.png)

I've created a video that will go thru all of this a link is provided bellow.

[![Azure API Management and OAuth with Auth0](http://youtu.be/RiYBKqk9N3A/0.jpg)](https://youtu.be/RiYBKqk9N3A "RBAC in Azure API Management - Click to watch")

Links used in the video:
* [Validate JWT Token](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#validate-jwt
)


## Summary:
Beeing a great API publisher is not as easy at it first looks like. Always striding for better and easier to use API's is a good thing and here we show how to move more logic inside the setup rather than at the consumer. If we can prevent the consumer from needing to filter data and/or manage identifiers and so forth we can make it easier for the app builders that consume our api's and provide speed and agility to those teams. At the same time we can create more secure and robust API's with more automated testing. There are more ways to use this and interested to see what you will find most usefull.