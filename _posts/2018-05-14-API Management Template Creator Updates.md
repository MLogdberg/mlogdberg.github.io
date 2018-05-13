---
layout: post
title: API Management Template Creator Updates May 2018 
author: Mattias Lögdberg
tags: [API Management, Integration,ARM Templates, ARM]
categories: [API Management]
image: 
description: 
permalink: /api-management/template-creator-updates-May-2018
---

Updates on the [API Management Template Creator](https://github.com/MLogdberg/APIManagementARMTemplateCreator) has been dragging but now I'm pleased to have fixed the two most urgent ones.

* Support for Versions, with version sets
* Schemas

## Support for Versions, with Version Sets
If an API that is extracted has enabled the current versionset is also extracted and provided inside the ARM Template.
The version set states the version that is used and is needed byt the API to be able to support versions, it contains the information about how the verions is handled for the specific API.
The sample bellow is using a *HTTP header* named **API-Version** to set the version of the API, the version later is then set on the API itself.


Read more in the documentation: [https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/service/api-version-sets ](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/service/api-version-sets)
```
{
      "comments": "Generated for resource /subscriptions/fake439-770d-43f3-9e4a-8b910457a10c/resourceGroups/API/providers/Microsoft.ApiManagement/service/dev/api-version-sets/5ae6f90fc96f5f1090700732",
      "type": "Microsoft.ApiManagement/service/api-version-sets",
      "name": "[concat(parameters('service_dev_name'), '/' ,parameters('versionset_5ae6f90fc96f5f1090700732_name'))]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "Arkivet",
        "description": null,
        "versioningScheme": "Header",
        "versionQueryName": null,
        "versionHeaderName": "API-Version"
      },
      "resources": [],
      "dependsOn": []
    }
```


## Support for Schema
If an API has schemas added to oeprations these schemas will be extracted and added to the ARM template, all operations will also have a "dependsOn" to the specific schema to prevent errors when executing the ARM template.
The sample bellow is a simple schema that is added in the ARM template and referenced in the Operations section.

Read more in the documentation: [https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/service/apis/schemas ](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/service/apis/schemas)
```
{
          "comments": "Generated for resource /subscriptions/fake439-770d-43f3-9e4a-8b910457a10c/resourceGroups/API/providers/Microsoft.ApiManagement/service/dev/apis/arkivet/schemas/5af038365b73730dd01453ad",
          "type": "Microsoft.ApiManagement/service/apis/schemas",
          "name": "[concat(parameters('service_dev_name'),'/',parameters('api_arkivet_name'), '/5af038365b73730dd01453ad')]",
          "apiVersion": "2017-03-01",
          "properties": {
            "contentType": "application/vnd.ms-azure-apim.swagger.definitions+json",
            "document": {
              "definitions": {
                "Body": {
                  "type": "object",
                  "properties": {
                    "No": {
                      "type": "string"
                    },
                    "ReportedDate": {
                      "type": "string",
                      "format": "date-time"
                    }
                  }
                },
                "BodyExists": {
                  "type": "object",
                  "properties": {
                    "CivicNo": {
                      "type": "string"
                    }
                  }
                },
                "Post400ApplicationJsonResponse": {
                  "type": "object",
                  "properties": {
                    "message": {
                      "type": "string"
                    },
                    "description": {
                      "type": "string"
                    },
                    "errors": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "properties": {
                          "field": {
                            "type": "string"
                          },
                          "message": {
                            "type": "string"
                          }
                        }
                      }
                    },
                    "hasErrors": {
                      "type": "boolean"
                    }
                  }
                },
                "ExistsPost200ApplicationJsonResponse": {
                  "type": "object",
                  "properties": {
                    "arkivet": {
                      "type": "string"
                    }
                  }
                }
              }
            }
          },
          "resources": [],
          "dependsOn": []
        }
```


## Summary:
Exporting API's ia the [API Management Template Extractor](https://github.com/MLogdberg/APIManagementARMTemplateCreator) simplifies the CI area of using API Management, here we can select a specific API and export only that API, with Operations, version sets, schemas, properties etc. Without needing to manually add extra work, making the development inside the portal and then just extracting the work and setup the import to the next environment .
