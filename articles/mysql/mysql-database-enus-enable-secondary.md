<properties
    linkid=""
    urlDisplayName=""
    pageTitle="Enable a readable standby database – Microsoft Azure Cloud"
    metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQs,readable standby database,Backup database instances,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ"
    description="This article explains how to implement a MySQL PaaS readable standby database."
    metaCanonical=""
    services="MySQL"
    documentationCenter="Services"
    authors="v-chenyh"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="mysql_en"
    ms.author="v-chenyh"
    ms.topic="article"
    ms.date="05/02/2017"
    wacn.date="05/02/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-enable-secondary/)
- [English](/documentation/articles/mysql-database-enus-enable-secondary/)

# <a name=""></a>Enable a readable standby database


**Enabling the readable standby database** function helps you to further improve the availability of a MySQL Database on Azure instance. After the readable standby database function has been activated for a MySQL Database on Azure instance, Azure will automatically create a standby instance. Data is synchronized between the primary and standby instances using replication technology. You can use the standby instance as a read-only instance, in order to meet the high level of read demand on the app. If the master instance fails, Azure will automatically upgrade the standby instance to become the new master instance and create a new standby instance. After the standby database function is enabled, the failure recovery time is generally in the minute range (less than 60 seconds). There will be no data loss after a failover.

## <a name=""></a>Limitations on enabling the standby database function

This function currently has the following limitations:

- Only supports MySQL version 5.6 and above
- Enabling the standby database function is not supported on read-only instances or external subordinate instances
- Restoring to the current server is not supported
- Directly creating readable standby databases is not supported, as you can only enable or disable the readable standby database function for existing instances.

## <a name="powershell"></a>Enabling the readable standby database function using PowerShell

Enter the following commands in PowerShell to enable the readable standby database function:

    Set-AzureRMResource -ResourceType "Microsoft.MySql/servers" -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast -PropertyObject @{enableSecondary=$true} -UsePatchSemantics

## <a name="powershell"></a>Disabling the readable standby database function using PowerShell

Enter the following commands in PowerShell to disable the readable standby database function:

    Set-AzureRMResource -ResourceType "Microsoft.MySql/servers" -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast -PropertyObject @{enableSecondary=$false} -UsePatchSemantics

## <a name="powershell"></a>View the standby instance using PowerShell

A command for viewing the enabled readable standby database function and the corresponding outputs are given below:

    Get-AzureRMResource -ResourceType "Microsoft.MySql/servers"   -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast

The outputs are shown below:

*Name              : mpswitch00*

*ResourceId        : /…/mpswitch00*

*ResourceName      : mpswitch00*

*ResourceType      : Microsoft.MySql/servers*

*ResourceGroupName : Default-MySql-ChinaEast*

*Location          : ChinaEast*

*SubscriptionId    : …*

*Properties        : @{AllowAzureServices=True; Version=5.7; ProvisioningState=Succeeded; EnableSecondary=True; ServiceEndPoint=mpswitch00.mysqldb.chinacloudapi.cn:3306; RunningState=Running}*

*ETag              : W/"datetime'2017-03-22T14%3A27%3A14.8784432Z'"*

*Sku               : @{Name=MP1}*

## <a name=""></a>Connect to the standby instance

The name of the standby instance is *[Master Instance name]-sec*. Change the master instance name in the database account to the name of the standby instance to connect to the standby instance. Master and slave instances on the same database account have the same password.

Let’s assume that you have the database instance *mpswitch*. The server address for this instance is *mpswitch.mysqldb.chinacloudapi.cn*. It has the database account *mpswitch%user*. If you enable the readable standby database function for this instance, the server address of the instance will be *mpswitch-sec.mysqldb.chinacloudapi.cn* and the database account for the standby instance will be *mpswitch-sec%user*.

## <a name=""></a>Testing instance crashes

You can use a *select crash()* fault injection query to force the master instance to crash, thereby triggering failover, so you can test fault recovery.

<!--HONumber=May17_HO3-->