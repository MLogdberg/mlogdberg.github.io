---
layout: post
title: Fixing Function v2 add extension issues in portal
author: Mattias Lögdberg
tags: [Azure Functions, Integration,Serverless]
categories: [Azure Functions]
image: 
description: 
permalink: /azurefunctions/functionsv2addextensionissuesportal
---

I'm out doing educations and it's both fun and a great chance for me to learn new stuffs since it's always something that just for no reason going south.


So today I was out on one of these missions and we where doing some labs developing functions, when we added an extra input of from a table storage the Function App just chrashed and then we just got errors that the site was offline.
The message that is shown is following:

```HTML
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
    <title>Host Offline</title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }

        a {
            text-decoration: underline;
            color: #FF381A;
        }

        a:hover {
            text-decoration: none;
        }

        body {
            line-height: 1.75em;
            background: #fff;
            font-size: 11.5pt;
            color: #5A6466;
        }

        body,input {
            font-family: Kreon, serif;
        }

        br.clearfix {
            clear: both;
        }

        h1,h2,h3,h4 {
            text-transform: uppercase;
            font-weight: normal;
        }

        h2 {
            font-size: 1.5em;
        }

        h2,h3,h4 {
            font-family: "Open Sans", sans-serif;
            color: #2A3436;
            margin-bottom: 1em;
        }

        h3 {
            font-size: 1.25em;
        }

        h4 {
            font-size: 1em;
        }

        img.alignleft {
            float: left;
            margin: 5px 30px 20px 0;
        }

        img.aligntop {
            margin: 5px 0 20px 0;
        }

        p {
            margin-bottom: 1.5em;
        }

        a {
            color: #2A3436;
        }

        .box {
            margin: 0 0 250px 0;
        }

        img.alignleft {
            float: left;
            margin: 5px 30px 30px 0;
            border-radius: 6px;
        }

        img.aligntop {
            margin: 5px 0 20px 0;
            border-radius: 6px;
        }

        #content {
            padding: 0;
            width: 815px;
            margin: 0 0 0 0px;
        }


        #page {
            margin: 0;
            position: relative;
            width: 900px;
            padding: 20px 40px 0 40px;
        }


        #wrapper {
            width: 978px;
            position: relative;
            background: #FFF;
            margin: 0 auto 0 auto;
            box-shadow: 0px 0px 300px 0px rgba(0,0,0,0.15);
            border: solid 1px #82A7AD;
            
        }

        
    </style>
</head>
<body>
    <div>&nbsp;</div>
    <div id="wrapper">
        
        <div id="page">
            <div id="content">

                <div class="box">
                    <h2>This Azure Functions app is down for maintenance</h2>
                    <p>
                        Host is offline
                    </p>
                </div>
            </div>
        </div>
    </div>
     
</body>
</html>
```

What I did to get here?

I had a default HTTP Trigger function and wanted to add an input from **Table Storage**. 

Here is my default http function:
![Defualt Http Trigger](/assets/uploads/2019/01/functions-defualt-http-trigger.png)

Then I went to the *Integrate* pane and started to add the input from my **Table Storage** 

![Integrate Pane](/assets/uploads/2019/01/functions-adding-table-storage-input.png)

After selecting the Table storage we get an warning that the extension is not installed, let's install it!

![Install Storage Extension](/assets/uploads/2019/01/functions-install-storage-extension.png)

It's installing and we proceed as told in the message:

![Install Storage Extension Message](/assets/uploads/2019/01/functions-install-storage-extension-install-msg.png)

Fill in the other details and press Save.

Going back to my function to add the input, now I just updated the signature from:
```c#
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
```
To:
```c#
public static async Task<IActionResult> Run(HttpRequest req,Newtonsoft.Json.Linq.JArray inputTable, ILogger log)
```
To get the input from the table.

But now when I try to execute the function it will start failing giving me a http error code **503 Service Unavailable** and in the body I can see the text "Host Offline".

![503 Service Unavailable Host Offline](/assets/uploads/2019/01/functions-install-storage-host-offline.png)

Now we need to start checking out what is going wrong, first check that the extension got added properly, follow this guide: [https://github.com/Azure/azure-functions-host/wiki/Updating-your-function-app-extensions](https://github.com/Azure/azure-functions-host/wiki/Updating-your-function-app-extensions)
And make sure that the extension file exists and contains atleast **"Microsoft.Azure.WebJobs.Extensions.Storage** if not create and/or add **"Microsoft.Azure.WebJobs.Extensions.Storage** to the file.
Sample bellow:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <WarningsAsErrors />
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.WebJobs.Extensions.Storage" Version="3.0.0" />
    <PackageReference Include="Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator" Version="1.0.*" />
  </ItemGroup>
</Project>  
```
If you are missing the file it can easily be created in the command prompt witb a simple script:
```cmd
echo.>extensions.csproj
```

Run the build command (this takes a while to run so be patient):
```cmd
dotnet build extensions.csproj -o bin --no-incremental --packages D:\home\.nuget
```

![Restore extension build complete](/assets/uploads/2019/01/functions-install-storage-complete-restore.png)

So now the extensions is installed, let's verify if this solves the issue completly? Start the Function App and test the function again. If you are as unlucky as me and the function is  **not** working we need to do one more thing.
Let's go back to Kudos and the CMD Prompt again, the problem is the  **app_offline.htm** file:

![Restore extension build complete](/assets/uploads/2019/01/functions-install-extension-offline.png)

Delete it and go back and test agian, it works again!

![Restore extension build complete](/assets/uploads/2019/01/functions-working.png)


## Summary:
The problem is that the extension is not properly installed and when we add the extension in the configuration without the extension installed the Function App chrashes as produces the **app_offline.htm** and if it's present in the folder the response is defaulted to 503 and Host Offline, by removing the file the Function App starts executing as normal and if we have fixed the extensions no errors comes up. This works for all extensions.

By showing the problem and the reproduce scenario we can help out improving the products! So I hope this helps anyone and leads to a fix from the prodcut team.

