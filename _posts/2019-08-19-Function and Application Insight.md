---
layout: post
title: Azure Functions logs in Application Insights
author: Mattias Lögdberg
tags: [Azure Functions, Integration,Serverless, Application Insights]
categories: [Azure Functions, Application Insights]
image: 
description: 
permalink: /2019/02/functions/understandhowlogsinai
---


Azure Functions is a great tool in our toolbox and as all our tools they have their strengths and flaws. When it comes to logging and monitoring Functions rely on Application Insight's and later on Azure Monitor.
It's also alot dependent on how the implement of your solution, and there are som **out of box** features that are really amazing.

If you are unfamiliar with Functions and Application Insights, read more here: [https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring](https://docs.microsoft.com/en-us/azure/azure-functions/functions-monitoring)


By just enabling application insights, without any further addons we get all the logs that are sent to the *ILogger* object to application Insights. And we also get some **End-To-End** tracking via supported SDK's and HttpClient requests. This is very useful to get an understanding of what is happening.
So let's look in to how this manifests in a rather complex scenario.

The Scenario is that a message is recieve via API Management, sent to a Azure Function that publishes the message on a topic, a second Azure Function reads the published message and publishes the message on a second topic, a third Azure Function recieves the message and sends the message over http to a Logic App that represents a "3:e party system".

![Scenario image](/assets/uploads/2019/08/functionsAi-scenario.png)

After connecting all functions to the **same** Application Insight instance we are sending a message! Let's see how this can look like in Application Insights, first we go to the Performance tab:

![Performance tab](/assets/uploads/2019/08/functionsAi-performancetab.png)

First let's just point out some small good to know things, the Operation Name in this view is actually the name of the function that you execute, so Function 1 in the list of Operation Names bellow:

![Operation Name](/assets/uploads/2019/08/functionsAi-endtoendoverview-OperationName.png)

Has this signature in the code **FunctionName("Function1")**, this is important as your instances grow make sure the name of the function is unique to easily find em.


```csharp
 [FunctionName("Function1")]
        [return: ServiceBus("topic1", Microsoft.Azure.WebJobs.ServiceBus.EntityType.Topic, Connection = "servicebusConnection")]
        public static async Task<string> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
```

Then we pick one of the requests, press the *samples* and pick one, it loads the end-to-end overivew


![End to end Overview](/assets/uploads/2019/08/functionsAi-endtoendoverview.png)

We can actually see all the steps from the first function to the last function and the call to the Logic App, impressive! The message is beeing sent over Service bus topics but the correlation is all set and we can follow the message all the way!

So this is truly awesome and alot of standard Properties are set out of box, but none specific to our request so it's hard to understand what data is sent so let's look in some special ways of adding custom information to the logs.

## Logs in Application Insights

Out of box logs is sent to Application Insights via the *ILogger* object so let's take a short look at the interface, it contains 3 different operations, *Log* that logs some information, there are different types and extensions on this to make it easier to use.
An isEnabled check to see if the logger is enabled and last a BeginScope operation that will create a scope for our log entries. [Read more](https://docs.microsoft.com/en-us/azure/azure-monitor/app/ilogger)


```csharp
public interface ILogger
{
    //
    // Summary:
    //     Begins a logical operation scope.
    //
    // Parameters:
    //   state:
    //     The identifier for the scope.
    //
    // Returns:
    //     An IDisposable that ends the logical operation scope on dispose.
    IDisposable BeginScope<TState>(TState state);
    //
    // Summary:
    //     Checks if the given logLevel is enabled.
    //
    // Parameters:
    //   logLevel:
    //     level to be checked.
    //
    // Returns:
    //     true if enabled.
    bool IsEnabled(LogLevel logLevel);
    //
    // Summary:
    //     Writes a log entry.
    //
    // Parameters:
    //   logLevel:
    //     Entry will be written on this level.
    //
    //   eventId:
    //     Id of the event.
    //
    //   state:
    //     The entry to be written. Can be also an object.
    //
    //   exception:
    //     The exception related to this entry.
    //
    //   formatter:
    //     Function to create a string message of the state and exception.
    void Log<TState>(LogLevel logLevel, EventId eventId, TState state, Exception exception, Func<TState, Exception, string> formatter);
}
```


Let's look at the simplest one just adding a Log text with *LogInformation* (an extension on Log)some unique data this is to log our productnumber, so we can see what productnumber this request is processing:
```csharp
	log.LogInformation("ProductNumber {productnumber}", productnumber);
```

Let's move to the collected Telemetry to see this log in Azure Portal:

![Scenario image](/assets/uploads/2019/08/functionsAi-viewTelemetry.png)


And now we can find our log row:

![ProductNumber log row](/assets/uploads/2019/08/functionsAi-informationlogproductnumber.png)


So now we know this request is processing the specific product, this is searchable in the Search Menu, and an easy click will take you to the End-to-End tracking. 

![ProductNumber log row](/assets/uploads/2019/08/functionsAi-searchProduct.png)


## Create a log scope with BeginScope
Sometimes we need to add some data that will be added as properties to all upcoming logs, this can be done via the *BeginScope*, with the following line all upcoming logs will automatically have the market and productnumber added as properties, the are added as with a prefix of *prop_*

```csharp
	using (log.BeginScope("Function3 processed message for {market} with productnumber {productnumber}", market, productnumber))
	{
		log.LogInformation("Processing Lines");
	}
```

Upcoming Log entry will have **prop_market** and **prop_productnumber**.

![Properties added with Begin Scope](/assets/uploads/2019/08/functionsAi-viewCustomPeropertiesBeginScope.png)

One good use is to do looping over multiple items, and therefore all entries will have these properties added, also all other properties added with the *{}* format in *log.Information* as **lineNumber** and **guid** in bellow sample will be added in the customDimension with prefix **prop_**

```csharp
	 log.LogInformation("Processing {lineNumber} and random guid {guid}",i.ToString(),Guid.NewGuid().ToString());
```

And if we want to loop over multiple lines we can add an extra scope to make sure we log the specific unique data for each iteration, following we log the *lineNumber* and a random *guid* and as you can see we still have the productnumber and market from before all with the **prop_** prefix.

![Properties added with Begin Scope](/assets/uploads/2019/08/functionsAi-viewCustomPeropertiesBeginScopeLoop.png)

Code sample of the loop

```csharp
	using (log.BeginScope("Function3 processed message for {market} with productnumber {productnumber}", market, productnumber))
    {
	    log.LogInformation("Processing Lines");
	    for (int i = 0; i < 10; i++)
	    {
	        using (log.BeginScope("Function3 Processing line {lineNumber}", i.ToString()))
	        {
	            log.LogInformation("Processing {lineNumber} and random guid {guid}",i.ToString(),Guid.NewGuid().ToString());
	        }
	    }
	}
```

## Add Filterable Property to Request
Let's say that we want to not just find a specific run, we want to understand overall performance or errors on a more wider term, like for a market. Let's consider we want to understand performance or failures for a specific market. This is done via the *Performance* or *Failures* tabs, but in order to narrow down the data presented we need to be able to filter the data like the the image bellow adding a filter on **Market**.

![ProductNumber log row](/assets/uploads/2019/08/functionsAi-performance-expectedsearch.png)

To get this working we need to add an property to our request log. (The request log is the first log event, see the telemetry for more information) the code to achieve this is super simple we just need to add this small snippet of code:

```csharp
System.Diagnostics.Activity.Current?.AddTag("Market", market);
```

The function now looks like:
```csharp
  public static class Function1
    {
        [FunctionName("Function1")]
        [return: ServiceBus("topic1", Microsoft.Azure.WebJobs.ServiceBus.EntityType.Topic, Connection = "servicebusConnection")]
        public static async Task<string> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            string productnumber = (string)data.productnumber;
            string market = (string)data.marketnumber;

            log.LogInformation("ProductNumber {productnumber}", productnumber);
            System.Diagnostics.Activity.Current?.AddTag("Market", market);

           
            return JsonConvert.SerializeObject(data);
        }
    }
```

This will now give us the following result:

In the *Search* section we can pick the **Market** in the filter to build our queries.

![Search with add filter Market](/assets/uploads/2019/08/functionsAi-searchMarket.png)

In the Search section we can pick the **Market** in the filter to build our queries.

![Search with add filter Market](/assets/uploads/2019/08/functionsAi-searchMarket.png)

In the *Performance* and *Failures* sections we need to add a bit more to be able to work with the filter, we need to add **customDimensions** before **Market** making the filter look like **customDimensions.Market**.

![Search with add filter Market](/assets/uploads/2019/08/functionsAi-perfromancecustomdimensionsearch.png)

And as you can see we get the filtered result after pressing the green confirm button.

![Search with add filter Market](/assets/uploads/2019/08/functionsAi-perfromancecustomdimensionsearchafter.png)

Adding more *Tags* will allow more possibliites for search, the tag will be attached only to the request log entry and not to the following traces. But I would recomend logging both a id and a more broader value like market or similar to find errors/performace issues that span a broader range.


## Summary:
I love developing in Azure Functions but in order to have a good Operations experience there is alot of things to consider in the logging strategy to make sure it's easy to understand what is going on and how to take advantage of the statistics provided by Application Insights.

This post is about how things can be done and hopefully this can be a guidance in creating a better logging and searching experience, we want to avoid the developers "let me just attach my debugger" response to what is the problem with the flow running in production.
