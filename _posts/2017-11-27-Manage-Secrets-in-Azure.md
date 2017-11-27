---
layout: post
title: Centralize secrets in Azure Key Vault
author: Mattias Lögdberg
tags: [Azure Functions, Integration,Security]
categories: [Azure Functions]
image: 
description: 
permalink: /keyvault/centralize-secrets
---

When working with usernames, passwords or api keys these need to be stored in a secure and manageble way. Usually I find that these are added to Application Settings and manually handled in several places, this is **not** a desirable way of working and may look something like this, secrets spread out in all areas with red circle:

![Normal setup](/assets/uploads/2017/11/password-username-keys.png)

The first step is to centralize the Values and there I find that **Azure Key Vault** Vault is a superb place for storage, we got RBAC support for granular security, making it possible for a developer to access the values via code or when deploying via ARM template but not to see or edit the value. So with this we can make sure that passwords/usernames and other secrets are not spread via email's, stored in dropbox or other "safe" places when distributed to the developers to add them to the *App Settings* or store them in *parameter files* for our ARM templates. 
It also adds reusability so if the value is needed in a *Function* and in a *Logic App* we can make sure it's the same value that is used and we can manage it in one place.


So let's have this sample in our KeyVault where sqluser and sqlpassword should be used in both a Function and when creating a Logic App Connector.

![Key Vault sample secrets](/assets/uploads/2017/11/key-vault-secrets.png)

Finding the Key vault Resource id:
![Key Vault resource id](/assets/uploads/2017/11/key-vault-resourceid.png)

## ARM Deployment Logic App Connector
Using **KeyVault** with ARM deployment requires that the KeyVault has enabled access for *Azure Resource Manager for template demployment* and that the principal (AAD Application) that is used when deploying has permissions to access the secrets.


![Enable ARM Deployment with Key Vault](/assets/uploads/2017/11/key-vault-enable-ARM-deployments.png)

After this secrets are accessible via the ARM Template Parameter file, (only from the parameter file or from a Wrapping ARM Template) here is a sample how it would look like in a parameter file:
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "name": {
      "value": "INT001-SQL"
    },
    "location": {
      "value": "West Europe"
    }
    "sqlusername" :{
      "reference": {
        "keyVault": {
          "id": "/subscriptions/fake-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/testenvironment/providers/Microsoft.KeyVault/vaults/ibizmalotest"
        },
        "secretName": "sqluser"
      }
    },
	"sqlpassword" :{
      "reference": {
        "keyVault": {
          "id": "/subscriptions/fake-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/testenvironment/providers/Microsoft.KeyVault/vaults/ibizmalotest"
        },
        "secretName": "sqlpassword"
      }
    }
  }
}
```

When doing a *Resource Group Deployment* the value is collected from Key Vault during deployment time and then stored in the API Connection. (Remember deployment time means that if the value is changed a new deployment is needed to reflect the change to the API Connection)

Now secrets are stored in Key Vault and the developer setting this up don't know the actual values in test/prod just what the secret's name is. The deployment flow looks as follows below where during deployment the secrets is collected from *KeyVault* (1) and used when creating/updating the *API Connection* (2) ) this also means that no secrets are stored in the ARM template parameters file and that is great!

![ARM Deployment with Key Vault](/assets/uploads/2017/11/key-vault-deployment-ARM-layout.png)

**IMPORTANT** if the value in *Key Vault* is changed a new deployment is required to **update** the *API Connection*.


## Accessing Secrets from Functions/Web Apps or other c# applications

Any c# application can easily use the *KeyVault SDK* to retrieve secrets from *Key Vault* during runtime, but we do need some things in order to make this possible, from code we need an AAD Application in order to grant access to the secrets, I will not go through how to create one here but [read more here](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications)

When it's created we need to get the *Application ID* (clientId) and one *Key* (clientSecret).

![Get AAD Application information](/assets/uploads/2017/11/functions-key-vault-get-AAD-Application-values.png)

Now that we have collected the information needed we also need to make sure that our AAD Application has access to the secrets, that is something we are adding to the *Key Vault* under the "Access Policies":

![Set Access Policies](/assets/uploads/2017/11/functions-key-vault-set-access-policies.png)

Press the "Add new", select the principal (AAD Application) in my case the *KeyVaultApp*, in the *Secret Permissions* we only want and need **Get** and no more privileges should be given to the Policy.

![Set Access Policies Settings](/assets/uploads/2017/11/functions-key-vault-set-access-policies-settings.png)


Now to start coding we need some nuget packages:
*Microsoft.Azure.KeyVault;
*Microsoft.IdentityModel.Clients.ActiveDirectory

And a bit of code, sample bellow, for more indept about how to use in *Functions* read more at [Jeff Holan's blog post:](https://medium.com/@jeffhollan/getting-key-vault-secrets-in-azure-functions-37620fd20a0b) 

Code to retrieve a secret thesecrets 
```
public static class KeyVaultFunction
{

    private static string clientID = Environment.GetEnvironmentVariable("clientId", EnvironmentVariableTarget.Process);
    private static string clientSecret = Environment.GetEnvironmentVariable("clientSecret", EnvironmentVariableTarget.Process);       
    private static string keyvaultname = Environment.GetEnvironmentVariable("keyvaultname", EnvironmentVariableTarget.Process);    

    [FunctionName("KeyVaultFunction")]
    public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
    {
        log.Info("C# HTTP trigger function processed a request.");

        string secretUri = $"https://{keyvaultname}.vault.azure.net/secrets/";

        var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));
        log.Info("Token aquired");
        var sqlusernameSecret = await kv.GetSecretAsync(secretUri + "sqluser");
		var sqlpasswordSecret = await kv.GetSecretAsync(secretUri + "sqlpassword");
		
		//do some code here

        return req.CreateResponse(HttpStatusCode.OK, "SQL credentials collected ok");
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
string secretUri = $"https://{keyvaultname}.vault.azure.net/secrets/";
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));
var sqlusernameSecret = await kv.GetSecretAsync(secretUri + "sqluser");
```

After this setup is completed we now are collecting the secrets from Key Vault during runtime and therefore there are no more secrets stored in application settings or in code and we have reused the same secrets in diffrent areas.

![Secrets accessed from KeyVault in code](/assets/uploads/2017/11/key-vault-runtime-code.png)


**Summary:**

I really like to remove username and passowrds from code and settings since it's always a mess manage and handling them, also there are a few security aspects in it aswell.
Making it possible to get username/password or other secrets from KeyVault inside your Function, Web App and at the same time use them when deploying ARM templates like with Logic Apps API Connections or API Management is a great addon, since we now only need to manage these at one place. 

![Secrets centralised in KeyVault](/assets/uploads/2017/11/key-vault-deployment-arm)


**OBS!** Make note that if we access Key vault via code an update is reflected emediatly while used via ARM a new deployment is needed in order for the new value to be deployed to i.e. password for a API Connection, making it vital to have good Build and Release processes).