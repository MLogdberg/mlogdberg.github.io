---
layout: post
title: OMS and Non Events
author: Mattias Lögdberg
tags: [Logic Apps, OMS, Monitoring, Integration]
categories: [OMS]
image: 
description: 
permalink: /oms/oms-non-events
---
When it comes to monitoring a integration flows, there are several types of monitoring that we need to cover, just the other day I got the requirement to make sure that a Logic App has been run atleast one time during a 24 hour period.

To solve this I started to look at possible solutions and turning my head to the Alert section of Logic Apps Diagnostics since it has the possibility to create a alarm based on input as number of runs, number of failed runs etc. unfortunally for me the max time limit was 6 hours. 
In this solution we are using *Log Analytics* and **OMS** as monitor tool, using the new Logic Apps Gallery it's super nice (guess I need another post on this later on). Anyway there is alarm functionality inside the OMS Portal so I started to look in to that.


Alarms in OMS is easily created based on a Search so first of we need to create a search, this is done in the search area and easiest way is to click your way to a start of the search and then change the last parts manually, I needed to check that a workflow was executed and ended sucessfully.
So my query ended up like this: (easy to reuse, just change the *resource_workflowName_s* to the name of your Logic App.

```
search * | where ( Type == "AzureDiagnostics" ) | where OperationName == "Microsoft.Logic/workflows/workflowRunCompleted" | where status_s == "Succeeded"   | where ( resource_workflowName_s == "INT002_Update_Work_Order2" )  
```

This query will return the results of succesfully runs, and we will be able to use it in our alarm.

Setting up the alarm is rather simple, just *Add Alert Rule* and fill in some information about the alarm, the important parts are the **Search Query** here you shall choose the saved query above, **Time Window** this is how far back we will look, so for our case we look 24 hours back.
**Alert Frequency** is another time slot and it's how often to check this rule, based on our case we wanted to check every hour. But in order for us to not get a new alert every hour (seems unnessisary to get 7 alarm emails during th night we can also specify **Spuress alerts** this will make sure that an alarm will not be sent out more often than every 6 hours.

So in reality we can now be sure that at most we will be alerted within 25 hours from the last run that there has not been any more runs and we will be alerted again every 6 hours until we have sucessfully completed a run.

[![](/assets/uploads/2017/08/OMS-non-event-alarm.png)](/assets/uploads/2017/08/OMS-non-event-alarm.png)

There is possibility to invoke webhooks, Runbook or ITSM actions but in our case an email is good enough so we will use that.
The email sent out is not containing that mutch information but it's enought for us, we now that the flow has not been working and most likely the sending system has failed to send their file.

The email:

[![](/assets/uploads/2017/08/OMS-non-event-alarm-email.png)](/assets/uploads/2017/08/OMS-non-event-alarm-email.png)


Creating reliable flows where we can tell our end users that we will monitor this to make sure that files are sent as promised or if they are not sent we will notify you about the issue is a vital part of building a good and reliable Integration department.
When we can garantee this and users start's to feel confortable with the solution, they can start use that time spent checking files and systems to mae sure the data is sent in doing better and more important things. 

It's often the small things that makes the most in the end.
