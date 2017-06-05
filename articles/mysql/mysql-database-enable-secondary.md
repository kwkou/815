<properties linkid="" urlDisplayName="" pageTitle="启用可读备用库 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,可读备用库,备用库实例,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="本文介绍了MySQL PasS可读备用库的实现方法。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="05/02/2017" wacn.date="05/02/2017" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-enable-secondary/)
- [English](/documentation/articles/mysql-database-enus-enable-secondary/) 

# 启用可读备用库

**启用可读备用库**功能可以帮助用户进一步增强MySQL Database on Azure实例的可用性。 当用户为MySQL Database on Azure实例启用可读备用库功能后，Azure会自动创建一个备用实例。 主备实例之间通过复制技术同步数据。用户可以将备用实例作为只读实例使用，以满足应用程序的大量读需求。当主实例发生故障，Azure会自动提升备用实例为新的主实例，并且创建新的备用实例。启用可读备用库功能后，故障恢复时间通常在分钟级（60秒内）。故障转移后无数据丢失。

## 启用可读备用库功能的限制

目前该功能有以下这些限制：

- 仅支持MySQL 5.6及以上版本
- 不支持在只读实例和外部从属实例上启用可读备用库功能
- 不支持还原到当前服务器
- 不支持直接创建可读备用库，只能对已有实例开启或关闭可读备用库功能。

## 通过PowerShell开启可读备用库功能

在PowerShell中输入以下命令即可启用可读备用库功能：

	Set-AzureRMResource -ResourceType "Microsoft.MySql/servers" -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast -PropertyObject @{enableSecondary=$true} -UsePatchSemantics

## 通过PowerShell关闭可读备用库功能

在PowerShell中输入以下命令即可关闭可读备用库功能：

	Set-AzureRMResource -ResourceType "Microsoft.MySql/servers" -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast -PropertyObject @{enableSecondary=$false} -UsePatchSemantics

## 通过PowerShell查看备用实例

以下给出一个查看已启用可读备用库功能的命令以及相应输出：

	Get-AzureRMResource -ResourceType "Microsoft.MySql/servers"   -ResourceName mpswitch00 -ApiVersion 2015-09-01 -ResourceGroupName Default-MySql-ChinaEast

输出如下：


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

## 连接备用实例

备用实例的名称是*[主实例名称]-sec*。 将数据库帐户中的主实例名称替换为备用实例名称，即可连接该备用实例。对于同一数据库帐户，主备实例上密码相同。

假设客户拥有数据库实例*mpswitch*。该实例的服务器地址为 *mpswitch.mysqldb.chinacloudapi.cn*。它拥有一个数据库帐户*mpswitch%user*。 用户为该实例开启可读备用库功能，则其备用实例的服务器地址为*mpswitch-sec.mysqldb.chinacloudapi.cn*，备用实例的数据库帐户为*mpswitch-sec%user*。

## 测试实例崩溃

可使用*select crash()*错误注入查询强制使主实例发生崩溃，从而触发故障转移，测试故障恢复情况。