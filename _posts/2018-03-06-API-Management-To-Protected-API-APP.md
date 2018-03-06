---
layout: post
title: Access AAD Secured Web API's from API Management
author: Mattias Lögdberg
tags: [API Management, Integration,Security]
categories: [API Management]
image: 
description: 
permalink: /api-management/access-secured-web-apis
---

Protecting Web Apps and Web API's by the built in *Authentication and authorization* in Azure App Service is a great way to protect resources without adding code to handle the authorization. This means that the site or api is fully secured without the need of implementing it a great example of separate of concern .
[Read more on how it works ](https://docs.microsoft.com/en-us/azure/app-service/app-service-authentication-overview)

What we then need is to access the API and in this case we need to be able to access the API via  Management using *service principals*  since the client using our API's in API Management is outside our organization and should not be bothered by our internal security mechanisms.

The scenario:
The API is protected by Azure AD and in order to access it we need a valid JWT token added to the *Auhtorization* header in the request, first(1) we need to get the JWT token and then add it to the (2) request to the API App.

![Scenario Overview](/assets/uploads/2018/03/apim_to_api_app_protected.png)

In this post we have used the standard generated API App from Visual Studio and published it to 2 instances in Azure **Protected** and **Protected2**.

## Enabling the built in security Authentication and Authorization to the API's

In the Azure Portal head to the API App and go in to the settings tab **"Authentication/Authorization"**. Enable the App Service Authentication by pressing the *On* button.

![Enable Authentication/Ahithorization](/assets/uploads/2018/03/Enable_Authentaction_AppService.png)


Now we can enable several diffrent providers and in this demo we will focus on the Azure Active Directory, press the Active Directory,

![Configure Azure AD Athentication](/assets/uploads/2018/03/Enable_Authentaction_AppService_providers.png)

After chosing the Azure Active Directory, we are presented with 2 options to configure the Azure Active Directory setup, Express gives some guidance and possiblity to create or select an existing AD Application meawhile advance requires the AAD Application clientid and the issuer normaly *https://sts.windows.net/{tenantguid}/* followed by the tenat guid URL. But in this setup we will use the Express option and create a new AD Application.

Here we can choose to use an existing AD Application or create a new one, I will create a new one named *ProtectedAppAAD*, press *Done*.

![New Azure AD App](/assets/uploads/2018/03/Enable_Authentaction_AppService_new_aad_app.png)

After this is installed we can now save and test that the API is protected, if you open a new browser or log out from the account you should be forced to login and the site is now secured.

![Login Sign](/assets/uploads/2018/03/Enable_Authentaction_AppService_login_sign.png)

So now the site is securely protected and we can access it when logged in, now we need to be able to access the API via API Managamenet.

## Provide access to the API
So the API is now protected and we need to provide a way for API Management to access it, we will need to use another AAD Application (App Registration) and provide access to the *ProtectedAppAAD*. So we need to create a new Application, named *APIManagementAADApp*, the sign in URL can be *http://localhost*.

![Create a new AAD Application](/assets/uploads/2018/03/create_apimanagement_aadapp.png)


After the AAD application is created we need to provide access to the *ProtectedAppAAD* this is done via permissions to assign permission go to *Settings*, press *Required Permissions* and *Add*.

![AAD Application add permission](/assets/uploads/2018/03/aadapp_assign_premissions.png)

Under *Select an API* we need to search for *ProtectedAppAAD* in order to find it, default is only Microsoft standard API's. So seartch for it, select and press *Select*.

![AAD Application add permission select API](/assets/uploads/2018/03/aadapp_assign_premissions_select_api.png)

Now we need to select the permission and the one we want is Access, select it and press *Select*, finish the setup by press *Done*.

![AAD Application add permission select API](/assets/uploads/2018/03/aadapp_assign_premissions_select_premission.png)

Last step is pressing the **Grant Permissions** to enable the permissions (do not forgett this!).

![AAD Application add permission select API](/assets/uploads/2018/03/aadapp_assign_premissions_grant.png)


## API Management Policy for Aquiring JWT Token
In order to be able to expose this API we need to get a token from AAD using the Application, this will be done inside a policy and lucky for us the API Management team has provided a set of code snippets at GitHub and one of these is exactly doing that,  [get it here.](https://github.com/Azure/api-management-policy-snippets/blob/master/Snippets/Get%20OAuth2%20access%20token%20from%20AAD%20and%20forward%20it%20to%20the%20backend.xml)

There is alot more of them and if you have a great snippet that you want to share you can add a Pull Request and if the team aproves it. it will end up there. Here is the GitHub repository, https://github.com/Azure/api-management-policy-snippets/blob/master/Snippets/.


The Get OAuth2 access token from AAD snipet looks like this.
```xml
<!-- The policy defined in this file provides an example of using OAuth2 for authorization between the gateway and a backend. -->
<!-- It shows how to obtain an access token from AAD and forward it to the backend. -->

<!-- Send request to AAD to obtain a bearer token -->
<!-- Parameters: authorizationServer - format https://login.windows.net/TENANT-GUID/oauth2/token -->
<!-- Parameters: scope - a URI encoded scope value -->
<!-- Parameters: clientId - an id obtained during app registration -->
<!-- Parameters: clientSecret - a URL encoded secret, obtained during app registration -->

<!-- Copy the following snippet into the inbound section. -->

<policies>
  <inbound>
    <base />
      <send-request ignore-error="true" timeout="20" response-variable-name="bearerToken" mode="new">
        <set-url>{{authorizationServer}}</set-url>
        <set-method>POST</set-method>
        <set-header name="Content-Type" exists-action="override">
          <value>application/x-www-form-urlencoded</value>
        </set-header>
        <set-body>
          @{
          return "client_id={{clientId}}&resource={{scope}}&client_secret={{clientSecret}}&grant_type=client_credentials";
          }
        </set-body>
      </send-request>

      <set-header name="Authorization" exists-action="override">
        <value>
          @("Bearer " + (String)((IResponse)context.Variables["bearerToken"]).Body.As<JObject>()["access_token"])
	  </value>
      </set-header>

      <!--  Don't expose APIM subscription key to the backend. -->
      <set-header exists-action="delete" name="Ocp-Apim-Subscription-Key"/>
  </inbound>
  <backend>
    <base />
  </backend>
  <outbound>
    <base />
  </outbound>
  <on-error>
    <base />
  </on-error>
</policies>
```

The way it works is that 2 things happen before we send the request to the backend.
1. First a request in the **send-request** action is made to the AAD tenant and aquiring a token.
2. The **set-header** action is adding the token in the Authroization header, by extracting it from the result from the first request.

But before we can add this snippet to our API we need to add a few values to the **Named Values** section in our API Management instance.

* authorizationServer - the tenant URL
My tenant ID is *1cb87777-3df4-428b-811a-86a0f215cd35* so the URL then is *https://login.microsoftonline.com/1cb87777-3df4-428b-811a-86a0f215cd35/oauth2/token*
* clientId - the Application id of our AAD Application in this case of *APIManagementAADApp*
    * ![AAD Application id APIManagementAADApp](/assets/uploads/2018/03/aadapp_get_Application_id_apim.png)
* clientSecret - the key to our AAD Application in this case of *APIManagementAADApp*
    * Create a new key  ![Create AAD Application key APIManagementAADApp](/assets/uploads/2018/03/aadapp_get_Application_key.png)
* scope - the AAD Application id of the AAD Application that is protecting our resource, in this case the id of *ProtectedAppAAD*
    * ![AAD Application id ProtectedAppAAD](/assets/uploads/2018/03/aadapp_get_Application_id.png)


 After adding these will end up looking like this in the **Named Values** section of our API Managemet. Make sure to add this before adding the policy, since we cannot save the policy until the named values are created.

![AAD Application id ProtectedAppAAD](/assets/uploads/2018/03/named_values.png)


Add the Policy, now we just need to add the policy to our API and we will be able to access the protected API.

![AAD Application id ProtectedAppAAD](/assets/uploads/2018/03/named_values.png)


## Summary:
In todays solutions we often are accessing other resurces and we need em protected in several ways, this is one way of using out of box security built in into the platform. This also makes a separation by concern point since we separate authentication to the API from the code. But the best part is that it can be combined with Basic, Oauth, API keys, Certificates etc. authentications and adding a second leayer of security.

![AAD Application id ProtectedAppAAD](/assets/uploads/2018/03/apim_to_api_app_protected.png)


As a bonus the API Management Snippets is a really nice initiative with pre created advanced snippets so make sure to check this out. 

https://github.com/Azure/api-management-policy-snippets/blob/master/Snippets/
