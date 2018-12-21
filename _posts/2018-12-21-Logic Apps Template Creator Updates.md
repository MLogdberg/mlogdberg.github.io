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

Updates on the [API Management Template Creator](https://github.com/MLogdberg/APIManagementARMTemplateCreator) has been published:

I've got a lot of help this time frome some awesome contributors a big thanks to them! [NilsHedstrom](https://github.com/nilshedstrom), [Geronius](https://github.com/Geronius), [joeyeng](https://github.com/joeyeng), [Lfalk](https://github.com/lfalck).

A minor thing for usage but great for consistency and quality in the generator is that there is now a build and release pipeline setup in DevOps. 

* Improved support for Identity Providers, Products and more
* Added to PowerShell Gallery [APIManagementTemplate](https://www.powershellgallery.com/packages/APIManagementTemplate/)
* Split the Template to a template per resource to follow [API Management DevOps best pratices](https://github.com/Azure/azure-api-management-devops-example)

## Power Shell Gallery
Now it's mutch easier to get started, just install the APIMangementTemplate modul from PowerShell Gallery.

```powershell
PS> Install-Module -Name APIManagementTemplate
```

Or update to the newest version
```powershell
PS> Update-Module -Name APIManagementTemplate
```

## Split the Template to a template per resource
In order to follow the best pratice from [Azure API Management DevOps example](https://github.com/Azure/azure-api-management-devops-example) we now have a new command **Write-APIManagementTemplates** this command will take a template as input and split into a file per resource to make it easy to manage and handle files and create more customized deployment with a linked template.


## Summary:
Exporting API's ia the [API Management Template Extractor](https://github.com/MLogdberg/APIManagementARMTemplateCreator) simplifies the CI area of using API Management, here we can select a specific API and export only that API, with Operations, version sets, schemas, properties etc. Without needing to manually add extra work, making the development inside the portal and then just extracting the work and setup the import to the next environment .
