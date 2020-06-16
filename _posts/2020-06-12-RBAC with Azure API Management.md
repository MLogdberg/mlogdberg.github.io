---
layout: post
title: RBAC with API Management 
author: Mattias Lögdberg
tags: [API Management, Integration,Serverless, Auth0, IDP, OAuth]
categories: [API Management]
image: 
description: 
permalink: /blog/2020/06/rbacwithapim
---

Extending our setup with API Management and Auth0 to create RBAC control over our operations in our API's. The idea is to make sure that consumers can be granted access to specific operations in an API and not only to the whole API. This can be achieved in a number of ways and in this topic we will use Auth0 as the Identity Provider to help out with this, but you could use any other of your choice.

> An Identity provider is a service that a user/application is signing in to (just like Azure AD) and this provider has functionality to provide needed information and grant access to requested resources that the IDP is handling. Just like the fact that you have access to a resource or resource group inside your subscription in Azure.

In API Management we have a trust setup as of the previus post [Setup Auth0 with API Management](/blog/2020/05/setupauth0withapim). Then we add permissions to our representation of the **Api Management** instance in Auth0 and grant the conusmer only the permissions that we want to give. After the permissions is setup and granted we willenforce these permissions in our API Management instance via the [validate-jwt](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) policy.


The end scenario will look like the illustration bellow where we can grant access to some or all operations in our API.

![Scenario image](/assets/uploads/2020/06/apim-Auth0-RBAC.png)

I've created a video that will go thru all of this a link is provided bellow.

[![Azure API Management and OAuth with Auth0](http://youtu.be/0ItzeSiHl24/0.jpg)](https://youtu.be/0ItzeSiHl24 "RBAC in Azure API Management - Click to watch")

Links used in the video:
* [Validate JWT Token](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#validate-jwt
)


## Summary:
There is alot of times where extra granularity is needed in the API's exposed. It helps out with creating easier to use and more natural API's. I can be a good selpoint if used to show that there are more features for premium customers. Regardless adding RBAC to your API's will increase seurity and eas of use. It will also give you a possibility to move the approving of new customer out of the API Managament instance and become more of a business thing to increas new consumer adoption. The setup used in Auth0 is very powerfull and can be reflected on end users aswell and that can be very helpfull when adding more lightweight consumers with single purposes.