---
layout: post
title: Oauth Token generation with Postman
author: Mattias Lögdberg
tags: [Azure AD, Oauth,Postman,Token generation]
categories: [Development]
image: 
description: 
permalink: /blog/2021/10/oauthtokengeneration
---

Beginning to develop solutions where ouath is used as authentication mechanism is often easy due to alot of frameworks sorting out the heavy lifting. But I prefer to go in and see the raw data and make small changes/new requests to the apis to get data in order to understand what's going on. In those cases I use Postman and build up collections to easyily get the data I want and need. And the last few days I've gotten questions from multiple people around how I do it so I felt I should share a blog post about it since more might be interested.

First of all this the way I'm doing it so if you have a better please fill in the chat since there are more than one way to solve this problem.
I create a new Workspace "Token generation demo" in [Postman](https://www.postman.com/) (Workspace is a container for i.e. a project where I need to gather all requests and settings I need when working in this project).

![Postman Workspace start](/assets/uploads/2021/10/tokengeneration/newworkspace.png)

Let's go thru the basics needed, if you never have been working with Postman these are the areas we will use.
1) Collections, here is all our collections that contains requests that we want to group togheter, like "Sales Orders" or "D365"
2) Request area, here we will work with our requests
3) Environmnet variables, these will be used to safely store credentials and also quickly change between environments

![Postman Workspace info](/assets/uploads/2021/10/tokengeneration/newworkspace_withnumbers.png)

There are some built in mechanism but I'm always finding myself in some area of trouble and spend alot of time troubelshooting and just come out confused, so the approach I'm using feels easier and you get more control over what's going on, call me old school if you want :).

So let's start of by creating a collection, I name it **Token Sample** after that we are ready to add our first request, it's the one for creating the token, we are doing this towards Azure AD.
1) Press the new button and pick **'Http Request'**
2) Add the url and to Azure it's always *https://login.microsoftonline.com/{{tenantId}}/oauth2/token* only switch the {{tennantId}} to your tenant id.
3) Add the body, since this is where we will send in credentials and all other information needed.
```
grant_type:client_credentials
client_id:{{client_id}}
client_secret:{{client_secret}}
resource:{{audience}}
```
What type to use, since we use client credentials we pick *client_credentials* and then the id and secret of your Azure AD app. And last the audience we are targeting, in this sample it's the Azure Management api but ut could be any resource that you have in your AD.

4) Name the request so you can find it easily
5) Save it in your collection 

![New Request](/assets/uploads/2021/10/tokengeneration/postman_new_gettoken_request.png)

Now we have added a bunch of variable, each place that looks like this {{variable}} is avariable with the name that's inside the curly brackets. Let's add an environment to add these variables too, clikc on the **eye** and then the **Add** button for environment. Start populating the variables we need:
* tenantId - your tenant ID can be found in the tageted AD
* client_id - the client id of the application you are authenticating with
* client_secret - the client secret of the application you are authenticating with
* audience - the targeted audience, meaning where do I want to have access, in my case to the Managemet Api.

![New Environment](/assets/uploads/2021/10/tokengeneration/postman_new_environment.png)

If we now go back to the request we can generate a token, make sure to pick the environment at top right (see image):

![Test Get Token Request](/assets/uploads/2021/10/tokengeneration/postman_test_get_token.png)

Now we need that token when we do request's there are several ways to do this but my favourit one is to add a variable for it, and get it updated automatically. Under the tab **Tests** we can add scripts, [read more on scripts](https://learning.postman.com/docs/writing-scripts/test-scripts/). We will then add a simple script that will verify that we get a json body back, and pick out the *bearer token* in the response and update the environment variable named *bearerToken*.

```javascript
var json = JSON.parse(responseBody);
tests["Get Azure AD Token"] = !json.error && responseBody !== '' && responseBody !== '{}' && json.access_token !== '';
postman.setEnvironmentVariable("bearerToken", json.access_token);
```

![Postman Test's tab](/assets/uploads/2021/10/tokengeneration/postman_teststab.png)

And after the run we now have the token in the variable *bearerToken* as we can see under the environment by clicking on the **eye**.

![Token stored to Environment variables](/assets/uploads/2021/10/tokengeneration/postman_teststab_with_environment.png)

Now we can start using it and it's very simple to start using it, let's start of by scanning what subscriptions I have access to with this service principal from the Azure management Api.

1) Create a new request
2) Name it *Get Subscriptions*
3) Set operation to **GET** and url to ```https://management.azure.com/subscriptions?api-version=2014-04-01-preview```
4) Add the Authorization header with the *Bearer* type and the token from the variable.
5) Send the Request

![Test the setup](/assets/uploads/2021/10/tokengeneration/postman_request_copmpleted.png)

As we can see there is a list of Subscriptions returned, and we can now easily setup new environments and reuse the same requests just change the enivornment and the results will be based on the new credentials. This will quickly get you up and running with wathere fun things you are doing.

## Summary:
We often need the flexibility that running our own requests gives us, this is how you can use [Postman](https://www.postman.com/). I often end up building request collections in Postman and distribute them over the team or use them for testing and retesting things. As I usually do I prepared a collection for you if you want to speed up the implementation, [download it here](../assets/uploads/2021/10/tokengeneration/Token%20Sample.postman_collection.json)