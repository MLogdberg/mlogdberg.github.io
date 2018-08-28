---
layout: post
title: API Management Template Creator Updates August 2018 
author: Mattias Lögdberg
tags: [API Management, Integration,ARM Templates, ARM]
categories: [API Management]
image: 
description: 
permalink: /api-management/template-creator-updates-Aug-2018
---

Updates on the [API Management Template Creator](https://github.com/MLogdberg/APIManagementARMTemplateCreator) has been published a few weeks back:

A minor thing for usage but great for consistency and quality in the generator is that there are alot of tests added. 

* Functions support added
* Updated Logic App Support 
* Removed some name parameter generation

## Functions support added
Finally the support for API's that are generated from Functions is live, now an API generated from Functions get the appropiate automatic generation of function keys via ARM functions.
Meaning that you don't need to supply the function kode in the parameter files they are collected automatically via Listsecrets function in ARM.

**Now:**
```
{
      "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/properties/5b41934c6d0f59440d20c5ee",
      "type": "Microsoft.ApiManagement/service/properties",
      "name": "[concat(parameters('service_ibizmalo_name'),'/','5b41934c6d0f59440d20c5ee')]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "maloapimtest_HttpTriggerAdminKey_query_5b41934c6d0f59440d20c5ee",
        "value": "[listsecrets(resourceId(parameters('FunctionApp_maloapimtest_resourceGroup'),'Microsoft.Web/sites/functions', parameters('FunctionApp_maloapimtest_siteName'), parameters('operations_api-HttpTriggerAdminKey-post_name')),'2015-08-01').key]",
        "tags": [],
        "secret": true
      },
      "resources": [],
      "dependsOn": []
    }
```

**Previously:**
```
parameters: {
    "INT2051functionkey_value": {
      "type": "securestring",
      "defaultValue": "Aze/1W8wCwItuP8JacdyHa2vDw8YScrOkbq6uHcTXiOasb2wi3kZoQ=="
    }
 }
	. . . 
 {
      "comments": "Generated for resource /subscriptions/1fake145-d7f4-4d0f-b406-7394a2b64fb4/resourceGroups/Api-Default-West-Europe/providers/Microsoft.ApiManagement/service/apidev/properties/int2051functionkey",
      "type": "Microsoft.ApiManagement/service/properties",
      "name": "[concat(parameters('service_cpidev_name'), '/' ,parameters('property_INT2051functionkey_name'))]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "INT2051functionkey",
        "value": "[parameters('INT2051functionkey_value')]",
        "tags": null,
        "secret": true
      },
      "resources": [],
      "dependsOn": []
}
```

## Updated Logic App Support
Two bigger things has been targeted to improve the export and later deployment experience for APIM API's that uses Logic Apps.

### Trigger name fix
A Logic App Trigger can now have any name, previous it was hardcoded to 'Manual'. There is now logic added to retrieve the information at extraction time, the extractor fetches the Logic App definition and finds the HTTP trigger to get the correct name.

Trigger name is extracted from the Logic App and added to the ARM functions (**customtriggername** in below sample):

![Logic App Trigger name](/assets/uploads/2018/08/logicappimagecustomtrigger.png)

Will result in (note the **'customtriggername'**):

```
"[listCallbackUrl(resourceId(parameters('LogicApp_customtrigger_resourceGroup'), 'Microsoft.Logic/workflows/triggers', parameters('LogicApp_customtrigger_logicAppName'), 'customtriggername'), '2017-07-01').queries.sig]"
```

### Add Logic App as operation to exisitng API fix
There has been diffrent behavior between API's generated from a Logic App imported as a new operation to an existing API. The main diffrent has been that the sig has been added as a "normal" parameter and no ARM functionality added to generate the *sig*. This is now adressed and the appropiate ARM functions will be added and generated automatically for both types.

Properties will get value from the *[ListCallBackUrl]* function:
```
{
      "comments": "Generated for resource /subscriptions/1fake145-d7f4-4d0f-b406-7394a2b64fb4/resourceGroups/Api-Default-West-Europe/providers/Microsoft.ApiManagement/service/apidev/properties/int7100-payment-to-o-test_manual-invoke_5b349f2a42974a989226cf33",
      "type": "Microsoft.ApiManagement/service/properties",
      "name": "[concat(parameters('service_apidev_name'), '/' ,parameters('property_int7100-payment-to-o-dev_manual-invoke_5b349f2a42974a989226cf33_name'))]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "int7100-payment-to-o-dev_manual-invoke_5b349f2a42974a989226cf33",
        "value": "[listCallbackUrl(resourceId(parameters('LogicApp_INT7100-Payment-To-o-DEV_resourceGroup'), 'Microsoft.Logic/workflows/triggers', parameters('LogicApp_INT7100-Payment-To-o-DEV_logicAppName'), 'request'),'2017-07-01').queries.sig]",
        "tags": [],
        "secret": true
      },
      "resources": [],
      "dependsOn": []
    }
```

## Removed some name parameters
Parameters generated for name of resources like, Policy, version sets, operations etc has no real use case or benefits associcated with changing the name and is therefore removed to simplify the ARM templates.

**Sample of parameters that are now removed:**
```
    "versionset_5af9817ca656c6952416b779_name": {
      "type": "string",
      "defaultValue": "5af9817ca656c6952416b779"
    },
    "operations_GetSalesOrderById_name": {
      "type": "string",
      "defaultValue": "GetSalesOrderById"
    },
    "operations_policy_name": {
      "type": "string",
      "defaultValue": "policy"
    }
```


## Summary:
Exporting API's ia the [API Management Template Extractor](https://github.com/MLogdberg/APIManagementARMTemplateCreator) simplifies the CI area of using API Management, here we can select a specific API and export only that API, with Operations, version sets, schemas, properties etc. Without needing to manually add extra work, making the development inside the portal and then just extracting the work and setup the import to the next environment .
