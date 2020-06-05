---
layout: post
title: Setup Auth0 with API Management 
author: Mattias Lögdberg
tags: [API Management, Integration,Serverless, Auth0, IDP, OAuth]
categories: [API Management]
image: 
description: 
permalink: /blog/2020/05/setupauth0withapim
---

API Management is an awesome API gateway with functionality to really excell in exposing API's to consumers. When it comes to security there are several options and today we will look in to the OAuth. In order to do this we need an IDP (Identity Provider) that we can configure a trust releationship with.

> An Identity provider is a service that a user/application is signing in to (just like Azure AD) and this provider has functionality to provide needed information and grant access to requested resources that the IDP is handling. Just like the fact that you have access to a resource or resource group inside your subscription in Azure.

In API Management a trust to an IDP and creation of a validation of the JWT provided from the IDP is done easily via the restrict policy called [validate-jwt](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT)


Let's go thur how the setup looks like, we will need to set up a Trust between your API Management instance and your Auht0 instance.

![Scenario image](/assets/uploads/2020/05/apim-Auth0-trust.png)

I've created a video that will go thru all of this a link is provided bellow.

[![Azure API Management and OAuth with Auth0](http://img.youtube.com/vi/n2AJsRx3W7U/0.jpg)](https://youtu.be/n2AJsRx3W7U "Setup trust with APIM and Auth0 - Click to watch")

Links used in the video:
* [Validate JWT Token](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#validate-jwt
)
* Auth0 Openid Configuration url: https://YOUR_AUTH0_DOMAIN/.well-known/openid-configuration


## Summary:
Adding a second security layer like this increases security and as you will see later on flexibility. It's an awesome start in order to build a nice consumer experience for your API's. In API Management it's very easy to attach any IDP so you can pick and choose your favourite and the setup will be somehwat similar.