---
layout: post
title: Logic Apps concurrency control
author: Mattias Lögdberg
tags: [API Management, Integration]
categories: [API Management]
image: 
description: 
permalink: /apimanagement/concurrency-control
---
As I was mentioning in the last post with concurrency control for Logic App, it's important in the distributed world today to make sure to not flood our backend services since there are more restrictions in the shared and distributed landscape than it has been in the *good old days* OnPrem where we had full control over all our services. 
That means that we need to have control over our integration solutions so they don't spin away like crazy and causes unwanted flooding to our backend systems.

Historically we could control the request towards our backends system with *Rate-Call-Limit* and *Qutoas* to make sure that a maximum of request was made by a consumers during a time period, that could be minutes,hours,days,months.

But in the [latest release the API Management](https://blogs.msdn.microsoft.com/apimanagement/2017/08/23/release-notes-august-23-2017/) the team added amongst other concurrency control.
This means that we now also can control/restrict the number of concurrent calls to our backend service.

I.e. we have a SaaS service that allows 20 concurrent calls, all calls without that limit will be declined. This was hard to control from API Management and all the responsibility was left to the consumers.
We had a case where the consumer push 1000 calls in a few miliseconds, distributed world is lovely! But the backend service couldnt handle it and most of the failed and a new batch was fired and yes the story repeated itself.
It ended with the consumer needing to change their solution (yes I agree that was the best way) but we had no chance off protecting the backend from the flood except *Rate-Limit* but that was not an optimal solution.

Now whit *Concurrency control* we can garantée that the backend system don't have more than 20 concurrent calls and all calls after are queued up to a maximum limit, and processed right away as one of the 20 concurrent calls are finished. Making sure to not waste resoruces in retrys and idle time.
Using the Concurrency control is simple here is the policy: [read more here](https://docs.microsoft.com/en-us/azure/api-management/api-management-advanced-policies#LimitConcurrency)
```
<limit-concurrency key="expression" max-count="number" timeout="in seconds" max-queue-length="number">
        <!— nested policy statements -->  
</limit-concurrency>

```


So let's show a sample, I've created a Logic App to act as a backend service, it hasen't mutch logic but a delay to simulate really heavy work (20 seconds)

[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend.png)](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend.png)

And now let's see how this looks like when consumed by another Logic App in a foreach loop (without any concurrency restrictions).

[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-nocontrol-logicapp.png)](/assets/uploads/2017/09/apim-concurrency-control-logic-app-nocontrol-logicapp.png)

As assumed the execution takes abit longer than 20 seconds, almost 30:
[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-nocontrol-logicapp-run.png)](/assets/uploads/2017/09/apim-concurrency-control-logic-app-nocontrol-logicapp-run.png)

In order to protect my backend (the logic app with 20 seconds delay) I've imported the logic App to my API Management instance and added the *Concurrency Control* policy:

[![](/assets/uploads/2017/09/apim-concurrency-control-apim-api.png)](/assets/uploads/2017/09/apim-concurrency-control-apim-api.png)

And the policy looks like:

```
<policies>
	<inbound>
		<base />
		<rewrite-uri id="apim-generated-policy" template="?api-version=2016-06-01&amp;sp=/triggers/request/run&amp;{{request59ad4615791ea086137dcd00}}" />
		<set-backend-service id="apim-generated-policy" base-url="https://prod-40.westeurope.logic.azure.com/workflows/24b778e6805344e686536a203ee47bce/triggers/request/paths/invoke" />
		<set-header name="Ocp-Apim-Subscription-Key" exists-action="delete" />
	</inbound>
	<backend>
		<limit-concurrency key="constantstring" max-count="2" timeout="120" max-queue-length="100">
			<forward-request timeout="120" />
		</limit-concurrency>
		<!--{ "azureResource": { "type": "logicapp", "id": "/subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/Concurrency/providers/Microsoft.Logic/workflows/INT0002-SingelInstance/triggers/request" } }-->
	</backend>
	<outbound>
		<base />
	</outbound>
	<on-error>
		<base />
	</on-error>
</policies>
```

To demonstrat the functionallity a new Logic App is now used , same logic as before but using the API Management to access the backend Logic App.

[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend.png](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend.png)


Executing the Logic App will now take 4 minutes! since the concurrency is set to 2.

[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-run.png](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-run.png)


So that is great now we have prevented to many concurrent calls to our backend system, but is it all good? Let's look in to the log of our "backend system"


[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-cotrolled-runs.png](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-cotrolled-runs.png)

We can see that there are 2 failed runs and 22 in total runs, but as you remember the list had 20 records so we only wanted 20 request to our "backend system".

[![](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-cotrolled-failed-run.png](/assets/uploads/2017/09/apim-concurrency-control-logic-app-backend-cotrolled-failed-run.png)

The failed ones has failed due to timeout and that will also make the Logic App executing a retry, ending with a total of 22 calls to the backend.


**Summary:**
This is truly a nice feature that we will use alot but we also need to understand the nature of the API and how to configure it correctly to prevent unwanted behavior. In a flow where
resending the information is not allowed, the queue lenght should be set as low as possible to prevent the retry behavior of clients resending the information even if it's not wanted.
Or we might want to use other techniques for that kind of flows. 

Anyway a great feature and wery usefull one, but as allways make sure to understand the feature and the requirements of the protected API when configuring it. 

