---
layout: post
title: Azure Functions Get Key Vault Secret
author: Mattias Lögdberg
tags: [Azure Functions, Integration,Security]
categories: [Azure Functions]
image: 
description: 
permalink: /azurefunctions/get-azure-key-vault-secret
---

When working with usernames. password or api keys these need's to be stored in a secure and manageble way. Usually I find that these are added to Application Settings and manually handled in several places, this is **not** a desired way of working so this need's to be fixed.

First step is to centralise the Values and there I find that **Azure Key Vault** is a superb place, this way username/passwords/api keys etc are accessible during ARM deployments, from code during runtime and therefore easy to reuse and manage.

How do we do this? I've described in earlier post how to use Key Vault with ARM templates so now I'll try to explain how to use Key Vault from code, in this case a Azure Function. (I've assumed that a Key Vault instance is created)
Let's get started and add a secret to our keyvault, after the value is added the name will be used to retrieve the value. This way we don't need to know the value just what name the secret has.

![Add Secret to KeyVault](/assets/uploads/2017/11/functions-key-vault-add-secret.png)


In order to access the secret from code we need to use an AAD Application, I will not go thru how to create one here but [read more here](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications)

When it's created we need to get the Application ID (clientId) and one Key (clientSecret).

![Get AAD Application information](/assets/uploads/2017/11/functions-key-vault-get-AAD-Application-values.png)

Now we have collected the information needed, now we just need to make sure that our AAD Application has access to the secrets, that is somethign we are adding to the Key Vault under the "Access Policies":

![Set Access Policies](/assets/uploads/2017/11/functions-key-vault-set-access-policies.png)

Press the "Add new", select the principal (AAD Application) in my case the *KeyVaultApp*, in the *Secret Permissions* we only want and need **Get** an no more previliges should be given to the Policy.

![Set Access Policies Settings](/assets/uploads/2017/11/functions-key-vault-set-access-policies-settings.png)


In a Azure Function we can add the following code (complete function):
Following nuget packages are needed:
*Microsoft.Azure.KeyVault;
*Microsoft.IdentityModel.Clients.ActiveDirectory

```
public static class KeyVaultFunction
{

    private static string clientID = Environment.GetEnvironmentVariable("clientId", EnvironmentVariableTarget.Process);
    private static string clientSecret = Environment.GetEnvironmentVariable("clientSecret", EnvironmentVariableTarget.Process);       
    private static string keyvaultname = Environment.GetEnvironmentVariable("keyvaultname", EnvironmentVariableTarget.Process);
    private static string secretname = Environment.GetEnvironmentVariable("secretname", EnvironmentVariableTarget.Process);

    [FunctionName("KeyVaultFunction")]
    public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
    {
        log.Info("C# HTTP trigger function processed a request.");

        //string secretUri = $"https://ibizmalotest.vault.azure.net/secrets/customkeysecretname";
        string secretUri = $"https://{keyvaultname}.vault.azure.net/secrets/{secretname}";

        var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));
        log.Info("Token aquired");
        var sec = await kv.GetSecretAsync(secretUri);


        return req.CreateResponse(HttpStatusCode.OK, "Value: " + sec.Value);
    }

    public static async Task<string> GetToken(string authority, string resource, string scope)
    {
        var authContext = new AuthenticationContext(authority);
        ClientCredential clientCred = new ClientCredential(clientID, clientSecret);
        AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

        if (result == null)
            throw new System.Exception("Failed to obtain the JWT token");

        return result.AccessToken;
    }
}
```

First the settings are added to the Function App Application Settings:
```
private static string clientID = Environment.GetEnvironmentVariable("clientId", EnvironmentVariableTarget.Process);
private static string clientSecret = Environment.GetEnvironmentVariable("clientSecret", EnvironmentVariableTarget.Process);       
private static string keyvaultname = Environment.GetEnvironmentVariable("keyvaultname", EnvironmentVariableTarget.Process);
private static string secretname = Environment.GetEnvironmentVariable("secretname", EnvironmentVariableTarget.Process);
```
![Set Access Policies Settings](/assets/uploads/2017/11/functions-key-vault-application-settings.png)

To handle the authentication we need a method to send i to the *AuthenticationCallback* object that takes 3 parameters, authority is the login authroity i.e: *"https://login.windows.net/1cb87777-3df4-428b-811a-86a0f215cd35"* the resoruce is what resource we are trying to access (KeyVault): *"https://vault.azure.net"* and scope is the scope we are accessing with normaly empty string.

```
public static async Task<string> GetToken(string authority, string resource, string scope)
{
    var authContext = new AuthenticationContext(authority);
    ClientCredential clientCred = new ClientCredential(clientID, clientSecret);
    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

    if (result == null)
        throw new System.Exception("Failed to obtain the JWT token");

    return result.AccessToken;
}
```

Then the code part that is actually collecting the secret is the following, we need the URI to the secret that we compose of the Key Vault name and the secret name, and the **GetToken** function to send in to the *AuthenticationCallback* object.

```
string secretUri = $"https://{keyvaultname}.vault.azure.net/secrets/{secretname}";
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));
var sec = await kv.GetSecretAsync(secretUri);
```

**Summary:**

I really like to remove username and passowrds from code and settings since it's always a mess manage and handling them, also there are a few security aspects in it aswell.
Making it possible to get username/password or other secrets from KeyVault inside your Function, Web App and at the same time use them when deploying ARM templates like with Logic Apps API Connections or API Management is a great addon, since we now only need to manage these at one place. 

**OBS!** Make note that if we access Key vault via code an update is reflected emediatly while used via ARM a new deployment is needed in order for the new value to be deployed to i.e. password for a API Connection, making it vital to have good Build and Release processes).