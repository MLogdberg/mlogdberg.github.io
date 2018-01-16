---
layout: post
title: Januari update Logic App Template Creator
author: Mattias Lögdberg
tags: [Logic Apps, ARM Template,Logic Apps Template Generator]
categories: [LogicApps]
image: 
description: 
permalink: /logicapps/template-extractur-update-januari
---

A new year comes with new updates, as you might now there is now a Custom Connector option for Logic  Apps and earlier this had some changes in the reource id handling in comparision with the managed api's. So this has now been updated and fixed.

Also there where some issues regarding names that could not contain . (dot) and ohter special characters that has been fixed, a new way of handlign reource id now prevent problems related to naming restrictions.

The last part added was an option to be able to save the incoming body of the requested resources, this was done to be able to easier to debugg when developing, thanks to this and a Mock resource collector class  that can replay the saved files we  now can create full test runs that are replaying the whole collection of resources.

The github repository is found here: [https://github.com/jeffhollan/LogicAppTemplateCreator](https://github.com/jeffhollan/LogicAppTemplateCreator)

## Updates
* Now supporting extraction of Logic Apps containg Custom Connectors and generating working importable Connection resources with the Logic App.
* Updates to resource id handling, now able to handle resource groups, resouirce names with .(dot) and other special characters that previously was not supported
* Added a resource handler to be able to print out the incoming requests, to be able to build complete test suites with full replay of a extraction of a Logic App