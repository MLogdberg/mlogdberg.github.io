---
layout: post
title: August 2019 update Logic App Template Creator
author: Mattias Lögdberg
tags: [Logic Apps, ARM Template,Logic Apps Template Generator]
categories: [LogicApps]
image: 
description: 
permalink: /2019/08/logicapps/template-extractor-update
---

There has been some time since last updates where announced on the Logic App Template Creator, but work has been ongoing thanks to contributors so time to form out the new updates.

There are some small changes and bug fixes as usual and fixed some issues reported but here is some new functionality.

The github repository is found here: [https://github.com/jeffhollan/LogicAppTemplateCreator](https://github.com/jeffhollan/LogicAppTemplateCreator)

## Updates
* ISE support, you can now extract Logic Apps in ISE for deployments
* Extract custom Connector, you can now extract an ARM template for Custom Connectors
* Extract Integration Account Schemas, you can now extract an ARM template for Integration Account Schemas.
* Get the Deployed Logic App's URL as an output parameter from the ARM template deploy (usable when URL is needed in nested ARM templates)
* Improved Get-Help method to provide more valuable help and information
* Extract Logic App that will be in disabled mode when deployed
* Possibility to improved security around parameters and passwords generated in the ARM template and paramter files


## Thanks to contributors!
* [Splaxi](https://github.com/Splaxi)
* [Geronius](https://github.com/Geronius)
* [bhoang](https://github.com/bhoang)
* [svenskfisk](https://github.com/svenskfisk)
* [lfalck](https://github.com/lfalck)
