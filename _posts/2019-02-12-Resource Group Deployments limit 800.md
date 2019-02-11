---
layout: post
title: Resource Group Deployments 800 Limit fix
author: Mattias Lögdberg
tags: [Azure, ARM,Azure Resource Manager, Powershell, Automation Account]
categories: [ARM]
image: 
description: 
permalink: /2019/02/arm/resourcegroupdeploymentslimit800
---

There is a [limit](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#resource-group-limits) of how many historical deployments that are saved for a resource group in Azure. Currently the limit is 800 and yet there are no out of box solution to start deleting these historical deployments when the limit is closing in.
So what happens is that the next deployment, the 801:st is failing error code is **DeploymentQuotaExceeded** and message as follows:


```
Creating the deployment 'cra3000_website-20190131-150417-7d5f' would exceed the quota of '800'. The current deployment count is '800', please delete some deployments before creating a new one. Please see https://aka.ms/arm-deploy for usage details.
```

So to solve this we manually entered the resource group and deleted some deployments, but that isent what we want to do in the long run. So I looked around and found some powershell scripts I could reuse for this purpose and created a runbbook in an automation account.


I'm not no pro on automation account but I can set em up and use them, there might bsome of you who would give me a tip or 2 on this subject.

So when creating the account we need to use the **Azure Run As** account, since this will create an account i AAD and this will be used for access, by default the account will have contributor access on the whole subscription (if you have the permissions to create it) otherwise you need someone who can.

![Create Automation aCcount](/assets/uploads/2019/02/create_automation_Account.PNG)

If we then look at the resource group we can find this account with contributor access, wich also means that if you only want it to be for sertain resource groups all you need to do is to restrict access for this user and when the script runs *Get-AzureRmResourceGroup* only the resource groups where this user has access to is returned. (by default the whole subscription)

![Automation account contributor](/assets/uploads/2019/02/useraccount_iam.PNG)

So the script is simple and there might be some improvements with parallel actions needed for big environments but after running it on schedule for a while it shouldnt be a big issue. 

All in all it collects all resource groups that the user has access to and start collecting deployments, in this script I always skip to run the script if there is less than 100 deployments made since it's no pain in keeping these, it's just painfull if we get over 800.
But if we have more the script starts deleting deployments older than 90 days (by degault) to keep some history there, this can easily be changed in the script. This is run via a Powershell Workbook.


```PowerShell
$daysBackToKeep = 90

$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "AzureRunAsConnection "
    $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName  

    "Logging in to Azure..."
    (Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint ) | out-null

    "Obtaining Subscription information..."
    $context = Get-AzureRmContext
    Write-Output ("Subscription Name: '{0}'" -f $context.SubscriptionName)
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else {
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

$ResourceGroups = Get-AzureRmResourceGroup

if ($ResourceGroupName)
{
    $ResourceGroups = $ResourceGroups | where { $_.ResourceGroupName -EQ $ResourceGroupName }
}

foreach( $resouregroup in $ResourceGroups) {
    $rgname = $resouregroup.ResourceGroupName
    Write-Output ("Resource Group: '{0}'" -f $rgname)

    $deployments = Get-AzureRmResourceGroupDeployment -ResourceGroupName $rgname
    Write-Output ("Deployments: '{0}'" -f $deployments.Count)

    if($deployments.Count -gt 100 ) {
        $deploymentsToDelete = $deployments | where { $_.Timestamp -lt ((get-date).AddDays($daysBackToKeep)) }
        Write-Output ("Deloyments to delete: '{0}'" -f $deploymentsToDelete.Count )
        foreach ($deployment in $deploymentsToDelete) { 

            Remove-AzureRmResourceGroupDeployment -ResourceGroupName $rgname -DeploymentName $deployment.DeploymentName -Force
            Write-Output ("Deloyments deleted: '{0}'" -f $deployment.DeploymentName )
        }
        Write-Output ("Deployments deleted")
    }
    else {
        Write-Output ("Less than 100 deployments to deleted, only {0}" -f $deployments.Count)
    }

}
```

Running it will start deleting deployments, so here is a pre run deployments history

![Deployments history pre clear](/assets/uploads/2019/02/preclear.PNG)

And after we can see that history is deleted (for this run I just removed the 100 check)

![Deployments history post clear](/assets/uploads/2019/02/postclear.PNG)

After running it the logs will look something like this:

![Runbook logs](/assets/uploads/2019/02/runbook_logs.png


## Summary:
This limit is annoying and as I understand something that should be fixed over time, but until then we need to have these kind of helping scripts in environments with alot of deployments. So hopefully this helps out in the meantime.


Read more on the limits:

[https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#resource-group-limits](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#resource-group-limits)