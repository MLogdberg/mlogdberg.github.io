---
layout: post
title: Azure Functions npm install via VSTS
author: Mattias Lögdberg
tags: [Azure Functions, Integration, Deployments]
categories: [Azure Functions]
image: 
description: 
permalink: /azurefunctions/vsts-npm-install
---

When setting upp build and release cyckles for diffrent Azure resources we bump in to diffrent challenges, this post will cover the challenge of npm install of node modules in to a Function App when doing deployments.


When we are working with our development enviornment it's okay to go in and run kudo commands and so on to make sure our Function App has the correct packages installed, but as we are adding speed and agility to our processes our **Release** pipelines in **VSTS** has to automate more of these processes.
It's not that it's hard to go in to *Kudo* and run the command but it's a manual step that takes time and knowledge when doing a release, I prefer to have everything prepped so that I know things are working when the Release is installed.


So let's look in to how we can run *Kudo* commands from VSTS, since *Kudo* has a REST API we can use it for these kind of tasks, read more on it here [Kudo Rest API](https://github.com/projectkudu/kudu/wiki/REST-API)

When looking in to the API documentation we easily find a function for executing commands: 
```
POST /api/command
{
    "command": 'echo Hello World',
    "dir": 'site\\repository'
}
```

This means that we could create a message that looked like this for a npm install of the *request* package:
```
{
    "command": "npm install request",
    "dir": "site\\wwwroot"
}
```

So let's start figuring out the rest, how to execute this from VSTS?

As good as it's get this can be executed via PowerShell explained and sampled at the bottom of the API reference (I changed the sample in the following code snippet to execute the command for npm install):
```
$username = "`$website"
$password = "pwd"
# Note that the $username here should look like `SomeUserName`, and **not** `SomeSite\SomeUserName`
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username,$password)))

$userAgent = "powershell/1.0"
$apiUrl = "https://$functionAppName.scm.azurewebsites.net/api/command"
$command = '{"command": "npm install ' + $npmpackage + '","dir": "site\\wwwroot"}'

Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method POST -Body $command -ContentType "application/json"
```

The credentials is now needed and that is somewhat messy, but since it's the same as the deployment credentials we can add a commands to get that information via *AzureRmResourceAction* command:
```
$creds = Invoke-AzureRmResourceAction -ResourceGroupName $resourceGroup -ResourceType Microsoft.Web/sites/config `
            -ResourceName $functionAppName/publishingcredentials -Action list -ApiVersion 2015-08-01 -Force
```

So with this information we can now build a more generic script that will only need three (3) parameters to execute a npm install command on our Function App:
* **functionAppName**: the name of the Function App
* **resourceGroupName**: the name of the resource group that contains the Function App
* **npmpackage**: the npm package to install
```
param([string] $functionAppName, [string] $resourceGroup, [string] $npmpackage)

$creds = Invoke-AzureRmResourceAction -ResourceGroupName $resourceGroup -ResourceType Microsoft.Web/sites/config `
            -ResourceName $functionAppName/publishingcredentials -Action list -ApiVersion 2015-08-01 -Force

$username = $creds.Properties.PublishingUserName
$password = $creds.Properties.PublishingPassword


$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username,$password)))

$userAgent = "powershell/1.0"
$apiUrl = "https://$functionAppName.scm.azurewebsites.net/api/command"
$command = '{"command": "npm install ' + $npmpackage + '","dir": "site\\wwwroot"}'

Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method POST -Body $command -ContentType "application/json"
```

Running this script in a Azure Poershell Task that with a Service Endpoitn connection with the appropiate Role set will install the npm pacakge during the release, image bellow shows the Release setup.

[![](/assets/uploads/2017/11/functions-vsts-release-run-script-png)](/assets/uploads/2017/11/functions-vsts-release-run-script-png)




**Summary:**

I like to automate these tasks since it will give me a "easier" Release and a more reliable Release, but as for now it's hard to verify that the package is installed and if it's installed previously so a verification step that the function is loaded correctly is something we are doing.

