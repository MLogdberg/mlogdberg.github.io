---
layout: post
title: CI with Logic App Custom Connector
author: Mattias Lögdberg
tags: [Logic Apps, ARM Template,Logic Apps Custom Connector]
categories: [LogicApps]
image: 
description: 
permalink: /logicapps/ci-with-logic-apps-custom-connector
---

Just had some fortune to work with some projects using the **Logic App Custom Connector** it's alot of fun now with all new capabilites on prem connectivity and all and as allways we end upp with setting up CI deployments via TFS. 

That is when I found that the *On Prem Data Gateway* **ARM template** is poorly documented and the generated template from *automation script* are missing some important properties.
So I thought I'll gather the information needed here, provide som pitfalls and so on.

The schema for the ARM template can be found at GitHub: [https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2016-06-01/Microsoft.Web.json](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2016-06-01/Microsoft.Web.json). 

It's inside the Web schema and not that *"easy"* to get that you are in the correct definition but go down to row 119  and you find the **customApis** part, this is good for reference. 


So  let's see how a ARM template for the **On Premise Data Gateway** looks like when complete: ( I'm actually adding a fully working connector for [https://billogram.com/](https://billogram.com/))
The full represenatation of the swagger can also be found at my GitHub repository [https://github.com/MLogdberg/LogicAppCustomConnectors/tree/master/Billogram](https://github.com/MLogdberg/LogicAppCustomConnectors/tree/master/Billogram)
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "customApis_name": {
      "defaultValue": "Billogram",
      "type": "String"
    },
    "location": {
      "defaultValue": "westeurope",
      "type": "String"
    },
    "serviceUrl": {
      "defaultValue": "https://sandbox.billogram.com/api/v2",
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Web/customApis",
      "name": "[parameters('customApis_name')]",
      "apiVersion": "2016-06-01",
      "location": "[parameters('location')]",
      "properties": {
        "connectionParameters": {
          "username": {
            "type": "securestring",
            "uiDefinition": {
              "displayName": "API USer",
              "description": "The API User for this api",
              "tooltip": "Provide the API User",
              "constraints": {
                "tabIndex": 2,
                "clearText": true,
                "required": "true"
              }
            }
          },
          "password": {
            "type": "securestring",
            "uiDefinition": {
              "displayName": "API Password",
              "description": "The API Password for this api",
              "tooltip": "Provide the API Apssword",
              "constraints": {
                "tabIndex": 3,
                "clearText": false,
                "required": "true"
              }
            }
          }
        },
        "backendService": {
          "serviceUrl": "[parameters('serviceUrl')]"
        },
        "swagger": {
          "swagger": "2.0",
          "info": {
            "description": "The Billogram API is built according to RESTful principles, which means it uses HTTP as an application protocol rather than just as a transport layer for a custom protocol, like SOAP does. In other words, HTTP features such as the various request types (GET, PUT, POST, DELETE), response codes (403 Forbidden, 404 Not Found, 500 Internal Server Error) and standard headers (Accept, Authorization, Cache-Control) are a part of the API.",
            "version": "1.0.0",
            "title": "Swagger Invoice/Billogram",
            "termsOfService": "http://swagger.io/terms/",
            "contact": {
              "email": "billogram@billogram.se"
            }
          },
          "host": "sandbox.billogram.com",
          "basePath": "/api/v2",
          "schemes": [
            "https"
          ],
          "consumes": [],
          "produces": [],
          "paths": {
            ...removed for simplicity display...
              }
            }
          },
          "parameters": {},
          "responses": {},
          "securityDefinitions": {
            "basic_auth": {
              "type": "basic"
            }
          },
          "security": [
            {
              "basic_auth": []
            }
          ],
          "tags": [
            {
              "name": "Billogram Invoice",
              "description": "Handling Billogram/Invoices",
              "externalDocs": {
                "description": "Billogram structure",
                "url": "https://billogram.com/api/documentation#billogram_structure"
              }
            }
          ],
          "externalDocs": {
            "description": "Find out more about Swagger",
            "url": "http://swagger.io"
          }
        },
        "description": "My ARM deployed Custom Connector",
        "displayName": "[parameters('customApis_name')]",
        "iconUri": "/Content/retail/assets/default-connection-icon.6296199fc1d74aa45ca68e1253762025.2.svg"
      },
      "dependsOn": []
    }
  ]
}
```

Make sure to add both the swagger **and** the **backendService** object since the url specified in the **serviceUrl** is the one that is used in the Custom Connector during runtime, not the one specified in the swagger, this is good for easier management of dev/test/prod environments.

The Custom Connector is a great addon to Logic Apps and by been able to easily manage and deploy them is crusial, so even if this is great to be able to manage my Custom Connector via ARM I'm still missing the possibility to reference my swagger via a url since that would be most suitable way of deploying and kkeping the ARM template and swagger separated.
