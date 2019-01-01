---
layout: post
title: Logic App Template Creator Updates Januari 2019 
author: Mattias Lögdberg
tags: [LogicApps, Integration,ARM Templates, ARM]
categories: [API Management]
image: 
description: 
permalink: /logicapps/template-creator-updates-Jan-2019
---

Updates on the [Logic Apps Template Creator](https://github.com/jeffhollan/LogicAppTemplateCreator) has been published:

A minor thing for usage but great for consistency and quality in the generator is that there is now a build and release pipeline setup in DevOps. 

* Added Build & Release via DevOps to increase quality in merged sprints and automate release to PowerShell Gallery [LogicAppTemplate](https://www.powershellgallery.com/packages/LogicAppTemplate)
* Improved support for Connectors to Storage Tables and Queues.
* Added Commandlet to generate ARM Template for Integration Account Maps

## Power Shell Gallery
Now the LogicAppTemplate is updated more frequqntly since I've added Build and Release setup to publish new releases to PowerShell Gallery.

```powershell
PS> Install-Module -Name LogicAppTemplate
```

Or update to the newest version
```powershell
PS> Update-Module -Name LogicAppTemplate
```

## Storage Connector Tables and Queues
Now generated on the same way as Storage Blob Connector, meaning that the key will be collected duirng the deployment time based on the storage account name instead of needed to be provided as parameters. This will make it simpler and more neat to do deployments.


There are 3 parameters added
```
"azureblob_name": {
      "type": "string",
      "defaultValue": "azureblob"
    },
    "azureblob_displayName": {
      "type": "string",
      "defaultValue": "myDisplayName"
    },
    "azureblob_accountName": {
      "type": "string",
      "defaultValue": "myStorageAccountName",
      "metadata": {
        "description": "Name of the storage account the connector should use."
      }
    }
```

And they are later used in the connection to get the accountKey automatically during deployment.
```
 {
      "type": "Microsoft.Web/connections",
      "apiVersion": "2016-06-01",
      "location": "[parameters('logicAppLocation')]",
      "name": "[parameters('azureblob_name')]",
      "properties": {
        "api": {
          "id": "[concat('/subscriptions/',subscription().subscriptionId,'/providers/Microsoft.Web/locations/',parameters('logicAppLocation'),'/managedApis/azureblob')]"
        },
        "displayName": "[parameters('azureblob_displayName')]",
        "parameterValues": {
          "accountName": "[parameters('azureblob_accountName')]",
          "accessKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('azureblob_accountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value]"
        }
      }
    }
```

All magic is in this ListKeys part, so this tells **ARM** to collect the key based on a reference to the storage account. (this also means that the account doing the Resource Group Deployment also needs access to the storage account).
```
[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('azureblob_accountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value]
```

## New CommandLet Get-IntegrationAccountMapTemplate
So another improvement is that we now can extract a map from an Integration Account in to a directly deployable ARM template. It work similar to the other extractions and bellow is a sample of how to get the Integration Account map as an ARM template.

```powershell
Get-IntegrationAccountMapTemplate -ArtifactName 'mapname' -IntegrationAccount 'ianame' -ResourceGroup 'myresourcegroup' -SubscriptionId 'guid()' -TenantName 'mattiaslogdberg.onmicrosoft.com'
```


## Summary:
Exporting Logic Apps via the [Logic App Template Extractor](https://github.com/jeffhollan/LogicAppTemplateCreator) simplifies the CI/CD area of using Logic Apps. Without needing to manually add extra work, making the development inside the portal and then just extracting the work and setup the import to the next environment .
