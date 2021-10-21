---
layout: post
title: AAD Personal Token in requests with VS Code
author: Mattias Lögdberg
tags: [Azure AD, Oauth,VS Code,Token generation]
categories: [Development]
image: 
description: 
permalink: /blog/2021/10/personaltokengeneration
---

Two weeks ago I did a post around how I use Postman to generate AAD Oauth tokens for *Service Principals*. Soon after released I started to get questions, **THANKS ALOT** to all of you who reads and comments. One thing lead to another and an interesting discussion arised with one of you, from that dicussion I had two questions ringing in my head. First is it a good idea to give all developers *client credentials* to access services? It takes time to create and these credentials is possible to share amongst others, even outside organisations. And second can't we as a developer just use our own credentials and access?


If we could use our own account instead of a service principal we would increae security, starting time and flexibility all at once.

So I started to look around and I found a good option for this, and that is what I'm going to share today.

I found that **VS Code** has a nice extension that I will use in this case the [**Rest Client**](https://marketplace.visualstudio.com/items?itemName=humao.rest-client).

So first install the extension in VS code.

![VsCode Rest Client Extension](/assets/uploads/2021/10/personaltoken/vscode_extension_rest_client.png)

After this, let's create a new Workspace and start testing this out. As a standard I create a folder and add the requests there, it's easier to reuse and modify for new requests.

I've create a folder called *D365* and added a file *GetQuotes.http*. the .http ending tells the Rest Client extension to VS Code it's a http request and therefore adds a button on the top **Send Request** that will trigger the request.

Structure and semantics in this file is very easy, tactic is *just write the request in raw format and the press send*.

Let's go thru some small exampels, simple **GET** example get comment 1, write the Method URL and HTTP protocol to use, so for a *GET* to *https://example.com/comments/1* using standard *HTTP/1.1* it will be written as:
```
GET https://example.com/comments/1 HTTP/1.1
```

A more advanced exampel, **POST** example, to create a new comment,*POST* to *https://example.com/comments* using standard *HTTP/1.1* and we add the json body raw and don't forgett the **content-type** header:
```
POST https://example.com/comments HTTP/1.1
content-type: application/json

{
    "name": "sample",
    "time": "Wed, 21 Oct 2015 18:27:50 GMT"
}
```

As you can see we just write what we want to send, but enough of the basics let's get going.

When we want to use our own credentials in order to do calls to an AAD proteced resource like Dynamics D365.
We can then use the built in variable **$aadToken** in the **Rest Client**, read more [here](https://marketplace.visualstudio.com/items?itemName=humao.rest-client#system-variables).

That is simple to use we just need to add it to our Authorization header. Let's look at an example to get *Sales Quotation Headers* from D365.
```
GET https://{{yourD365Instance}}.cloudax.dynamics.com/data/SalesQuotationHeaders?cross-company=true&$top=3 HTTP/1.1
Authorization: {{$aadToken}}
```

When we then press the **Send Request** as shown in the image:
![VS Code http file](/assets/uploads/2021/10/personaltoken/requestsamplebeforesend.png)


A login prompt will poput like the one bellow, press signin.

![VS Code prompt](/assets/uploads/2021/10/personaltoken/loginprompt.png)

A browser will open and you need to paste the code from the previous promt in the code box (it's in your clipboard at this point so just paste it)

![Browser Login](/assets/uploads/2021/10/personaltoken/loginpromptbrowser.png)

Follow the login procedure and get back to VS Code to see the request been sent and you can see the data.

![Result in VS Code](/assets/uploads/2021/10/personaltoken/requestsample.png)


This request is then sent to D365 with a token generated for you. As with Postman or other tools you can now start building your own collections of requests in **VS Code** to share, check in with code or just for your own use. Read more in the [**Rest Client**](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) page to get details on advanced request building.


## Summary:
As a developer we often need to query systems like D365. When using the *"normal"* way with service principals we are using credentials anyone can use. We can share them amongst collegues and they will be stored on developers computers. But with the described approach we can stop that and start pushing developers to use their own credentials. Increasing security and control over who has access to our systems and data.  
So not only is it awesome for a developer to get started more quicker and easier without the need of a service principal. It's also a better practice security wise. 

There are probobly many more ways to solve this, but this is one way and my thoughts on it.