---
layout: post
title: Logic Apps concurrency control
author: Mattias Lögdberg
tags: [Logic Apps, Integration]
categories: [LogicApps]
image: 
description: 
permalink: /logicapps/concurrency-control
---
In a distributed world with increasing amount of moving parts we need to make sure to not flood our services since there are more restrictions in the shared and distributed landscape than it has been in the *good old days* where we had full control over all our services. 
That means that we need to have control over our solutions so they don't spin away like crazy.

On this topic Logic Apps has earlier had problem, when looping over an array in a foreach loop and calling a connector we could choose between 20 concurrent actions or sequentially call it one by one. This mean that if 20 parallel actions was flooding the destination we had to make it to sequantial call the destination and therefore also taking alot longer. 
When working with batches this is not always possible since it might take to long time, so sometimes we had to create two or more parallel flows in Logic Apps to make suer that we could keep the time limit.

Making the flow complex and look *"uggly"*:

[![](/assets/uploads/2017/08/Logic-App-concurrency-multiple-foreach.png)](/assets/uploads/2017/08/Logic-App-concurrency-multiple-foreach.png)


# Newly released concurrency control!
But fortunately there is now a newly released object property on the for each action that will help us, only available in **CodeView** for now but that will work just fine.

The new property looks like:

```
"runtimeConfiguration": {
    "concurrency": {
        "repetitions": 2
    }
},
```
And setting the *repetitions* to 2 will be the more or less the same as above solution in execution but it will be mutch easier to maintain and work with the Logic App now.

Logic App in the designer will now look like:

[![](/assets/uploads/2017/08/Logic-App-concurrency-one-foreach.PNG)](/assets/uploads/2017/08/Logic-App-concurrency-one-foreach.PNG)


# Triggers also have concurent control
We can also set this on triggers, sample bellow is on a recurrent trigger (or polling trigger withou spliton) will look like;

```
"trigger": {
"type":"http",
"recurrence": {},
"runtimeConfiguration": {
    "concurrency" : {
    "runs": 10
	}
}
```


I'm really happy that the product group finally has released this since it's such an important part in our *new* world of ditributed landscapes and our "new" responsibility and interest in making sure our solutions are not flooding the destionations.
We got the power and now we got the possibility to restrain them to the appropiate amount.

