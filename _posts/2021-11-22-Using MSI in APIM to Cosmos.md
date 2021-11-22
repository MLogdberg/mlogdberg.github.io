---
layout: post
title: Using MSI in APIM to Cosmos
author: Mattias Lögdberg
tags: [Azure AD, Api Management, MSI]
categories: [Development]
image: 
description: 
permalink: /blog/2021/22/usingmsifromapimtocosmos
---

Quite recently Microsoft finally gave us RBAC roles for data plane access to Cosmos, this means that we now can use MSI to access data in Cosmos.


So what's it all about? Well the best part is that we now can grant access to resources rather than using keys. This increases security and accessability since we don't need to manage the key any more and only specific resources can access our data. When we have switched to RBAC as the authentication method we can switch of the possibility to access data with primary/secondary keys,[read more here.](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac#disable-local-auth)

So how do we do this? [Here is a guide from Microsoft](https://docs.microsoft.com/en-us/azure/cosmos-db/managed-identity-based-authentication) on how to use MSI with an Azure Function. And we will in this post use the same technique but from API Mangement to show the potential.

**Prerequisites:**
- API Management instance
- Identity created on your API Management instance
- Cosmos DB
- Azure CLI

First we need to grant the specific roles to our API Management identity. The role we want to assing is the [Cosmos DB Built-in Data Reader](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac#built-in-role-definitions). This role only have read access and that matches our *Least Previliges* thinking, when we only want to expose read we should also only have read access.

Adding this role is **not** as straight forward as I would like it, they are not possible to set via the Azure Portal yet. So we need to do some *PowerShell* scripting. Fortunally all scripts needed can be found at the Microsoft docs on [How to setup RBAC for Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac). I would recomend looking in there since they also provide how to create your own [custom role definitions](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac#role-definitions).


Anyway let's get started and we need this script:
```PowerShell
$resourceGroupName = "<myResourceGroup>"
$accountName = "<myCosmosAccount>"
$readOnlyRoleDefinitionId = "<roleDefinitionId>" # as fetched above
$principalId = "<aadPrincipalId>"
New-AzCosmosDBSqlRoleAssignment -AccountName $accountName `
    -ResourceGroupName $resourceGroupName `
    -RoleDefinitionId $readOnlyRoleDefinitionId `
    -Scope "/" `
    -PrincipalId $principalId
```
 
Let's get the parameters, first we get the cosmos values, the resource group and the account name of the Cosmos instance (*$resourceGroupName*,*$accountName*) both marked in circles.
![Get Cosmos resourcegroup and account name](/assets/uploads/2021/11/msitocosmos/get_cosmo_data.PNG)

Then as we saw at the [Built in role definitions](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac#built-in-role-definitions) the role with name **Cosmos DB Built-in Data Reader** has Id **00000000-0000-0000-0000-000000000001**

Last we need the principal id of our Api Management instance (this could be any identity created for any resource). We go to the Managed Identity in our API Management instance. There we can collect the *Object (principal) ID*.
![Get APIM Managed Identity Prinipal Id](/assets/uploads/2021/11/msitocosmos/get_apim_managed_identity_principalid.png)

So the script ends up like this: (**Make sure to run the latest az version**)
```PowerShell
$resourceGroupName = "microservice"
$accountName = "mlogdberg"
$principalId = "c7b89cc3-1b27-474b-9575-cdcbc51dffcc"
$readOnlyRoleDefinitionId = "00000000-0000-0000-0000-000000000001" # as fetched from definition list
az cosmosdb sql role assignment create --account-name $accountName --resource-group $resourceGroupName --scope "/" --principal-id $principalId --role-definition-id $readOnlyRoleDefinitionId
```

After this we are ready to start quering, I did a simple request that collects all documents in a specific partion. This can ofcourse be expanded but this is bare minimum what is needed in the API Managemenet Policy.

Some good to know things:
- **authentication-managed-identity** is the step to get the token, make sure to have the patht to your cosmos instance.
- **x-ms-documentdb-partitionkey** is what partion the query is targeting
- **rewrite-uri** this is the patht to your database and collection */dbs/{dbname}/colls/{collectionname}/docs/*
- The url is set on the api to the path for my cosmos instance (*https://mlogdberg.documents.azure.com*).


```XML
    <inbound>
        <base />
        <set-variable name="requestDateString" value="@(DateTime.UtcNow.ToString("r"))" />
        <authentication-managed-identity resource="https://mlogdberg.documents.azure.com" output-token-variable-name="msi-access-token" ignore-error="false" />
        <set-header name="Authorization" exists-action="override">
            <value>@("type=aad&ver=1.0&sig=" + context.Variables["msi-access-token"])</value>
        </set-header>
        <set-header name="x-ms-date" exists-action="override">
            <value>@(context.Variables.GetValueOrDefault<string>("requestDateString"))</value>
        </set-header>
        <set-header name="x-ms-version" exists-action="override">
            <value>2018-12-31</value>
        </set-header>
        <set-header name="x-ms-documentdb-query-enablecrosspartition" exists-action="override">
            <value>false</value>
        </set-header>
        <set-header name="x-ms-documentdb-partitionkey" exists-action="override">
            <value>["1"]</value>
        </set-header>
        <rewrite-uri template="/dbs/staging/colls/order/docs/" copy-unmatched-params="false" />
    </inbound>
```

Now we shall be able to send a request,

![Result request](/assets/uploads/2021/11/msitocosmos/cosmos_result_request.png)

### Troubelshooting tips:
#### Request blocked by Auth
Before the setup is completed you will get this kind of respose trying to call the cosmos rest api.
```R
HTTP/1.1 401 Unauthorized
content-location: https://mlogdberg.documents.azure.com/dbs/staging/colls/order/docs
content-type: application/json
    {
    "code": "Unauthorized",
    "message": "Request blocked by Auth mlogdberg : No AAD tenant is trusted by this database account. Please create atleast one Role Assignment prior to making an AAD-based request.\r\nActivityId: 31a968bf-7652-4434-abd9-5113ea0655ae, Microsoft.Azure.Documents.Common/2.14.0"
}
```
#### Partition key 1 is invalid
The header **x-ms-documentdb-partitionkey** is important and when badly configured these kind of errors is sent back, the expected format is a json array of strings, if your partion is "1" the header should have value *["1"]* (like my example).
You can find your partionkey from browsing your collection.

![Browsing collection to find partionkey](/assets/uploads/2021/11/msitocosmos/cosmos_partionkey.png)

```R
HTTP/1.1 400 Bad Request
content-location: https://mlogdberg.documents.azure.com/dbs/staging/colls/order/docs
content-type: application/json
    {
    "code": "BadRequest",
    "message": "Partition key 1 is invalid.\r\nActivityId: 6bccbf25-657b-4a71-af92-b3e998f0490a, \r\nRequestStartTime: 2021-11-21T18:24:18.9692549Z, RequestEndTime: No response recorded; Current Time: 2021-11-21T18:24:18.9792812Z,  Number of regions attempted:1\r\n{\"systemHistory\":[{\"dateUtc\":\"2021-11-21T18:23:09.7385423Z\",\"cpu\":8.902,\"memory\":58190802944.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.012,\"availableThreads\":32761,\"minThreads\":40,\"maxThreads\":32767}},{\"dateUtc\":\"2021-11-21T18:23:19.7586326Z\",\"cpu\":7.062,\"memory\":56609513472.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.0161,\"availableThreads\":32764,\"minThreads\":40,\"maxThreads\":32767}},{\"dateUtc\":\"2021-11-21T18:23:29.7687307Z\",\"cpu\":5.752,\"memory\":56766377984.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.0173,\"availableThreads\":32765,\"minThreads\":40,\"maxThreads\":32767}},{\"dateUtc\":\"2021-11-21T18:23:49.7689519Z\",\"cpu\":5.634,\"memory\":57133617152.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.0145,\"availableThreads\":32765,\"minThreads\":40,\"maxThreads\":32767}},{\"dateUtc\":\"2021-11-21T18:23:59.7790729Z\",\"cpu\":5.298,\"memory\":57189658624.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.0125,\"availableThreads\":32765,\"minThreads\":40,\"maxThreads\":32767}},{\"dateUtc\":\"2021-11-21T18:24:09.7891858Z\",\"cpu\":8.835,\"memory\":56394321920.000,\"threadInfo\":{\"isThreadStarving\":\"False\",\"threadWaitIntervalInMs\":0.0193,\"availableThreads\":32764,\"minThreads\":40,\"maxThreads\":32767}}]}\r\n, Microsoft.Azure.Documents.Common/2.14.0"
}
```


## Summary:
The usage of **MSI** (Microsoft Service Identity) is a great feature on Cosmos, no need for storage keys. It's a win win thing, increased simplicity and also increased security. No credentials are stored and due to that they cannot be leaked. Forcing components to use **MSI** when connecting to the Cosmos could be done via setting *disableLocalAuth* to *true*: ```"disableLocalAuth": true``` ![read more](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-setup-rbac#disable-local-auth)