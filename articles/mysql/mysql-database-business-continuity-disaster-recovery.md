<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,区域性灾难,业务连续性方案,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="v-chuw" solutions="" manager="RongYu" editor="" />

<tags ms.service="mysql" ms.date="10/10/2016" wacn.date="10/10/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-business-continuity-disaster-recovery/)
- [English](/documentation/articles/mysql-database-enus-business-continuity-disaster-recovery/)

# MySQL on Azure 业务连续性方案

本文将总结典型的几个中断事件的场景，以及MySQL on Azure相应的保障业务连续性的方案。并重点介绍针对区域性灾难，恢复的具体步骤以及指标。

## 1. 概述 ##
业务连续性是指以弹性方式设计、部署和运行应用程序，以应对计划或非计划的中断事件，避免应用程序永久性或暂时性地无法执行其业务功能。非计划事件覆盖了从人为失误、到永久或暂时性的中断、再到区域性的灾难（这可能会导致特定 Azure 区域中出现大规模的机能损失）的整个范围。计划事件包括向不同的区域重新部署应用程序以及应用程序升级。保障业务连续性的目标是减少中断时间对应用业务的影响，尽可能减小影响时间避免数据丢失。

在介绍业务连续性方案前，您需要熟悉以下几个概念：

* 恢复时间目标 (RTO)：在发生中断性事件后，应用程序完全恢复之前的最长可接受时间。RTO 用于度量故障期间的最大可用性损失。
* 恢复点目标 (RPO)：在发生中断性事件后，应用程序在完全恢复时可以丢失的最大最近更新数量（时间间隔）。RPO 用于度量故障期间的最大数据丢失
* 预计恢复时间 (ERT)：在发出还原或故障转移请求后，数据库完全可用之前预计持续的时间。

## 2. 业务连续性方案 ##
下面介绍主要的几个常见的场景和相应的方案。

<table width="100%" border="1" cellspacing="0" cellpadding="0">
	<tr>
		<td>
			<b>场景</b>
		</td>
		<td>
			<b>描述</b>
		</td>
		<td>
			<b>应对方案</b>
		</td>
	</tr>
	<tr>
		<td>
			人为失误导致的业务中断恢复
		</td>
		<td>
			在生产环境下，由于数据库管理员的操作失误，导致某些重要数据丢失，需要快速恢复。
		</td>
		<td>
			备份与恢复- 任意时间点回滚功能
			MySQL 支持七天内的任意时间点回滚，具体步骤请参照：<a href="https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore/" target="_blank">MySQL on Azure备份与恢复- 还原数据库到任意时间点</a>
		</td>
	</tr>
	<tr>
		<td>
			某次升级导致的业务中断恢复
		</td>
		<td>
			在生产环境下，由于某次升级失败，导致发生兼容性问题，业务无法正常进行。
		</td>
		<td>
			在升级前，用户对数据库创建快照备份，升级如遇到问题，可以快速还原完全备份到新实例/原实例，具体步骤请参照：<a href="https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore/" target="_blank">备份与恢复</a>
		</td>
	</tr>
	<tr>
		<td>
			区域性灾难恢复
		</td>
		<td>
			当某些区域性灾难造成大面积服务中断时，需要在异地快速进行恢复。
		</td>
		<td>
			目前提供三种恢复方案，用户可以根据紧急程度进行方案选择；如果用户在生产环境使用，建议采用自助服务方案。<br><br>
			-自助服务方案：当区域性灾难发生，用户无法通过管理门户进行异地还原时，可通过PowerShell命令行，将实例在异地进行还原。<br><br>
			-PaaS方案：在严重区域性灾难发生时，MySQL服务也会针对所有受影响的实例进行异地还原。后文将重点介绍。<br><br>
			-从异地从属实例中恢复：如果用户设置了异地从属实例，则在主实例不可用的时候，可在最短时间内通过手动或自动（在应用程序中设置）方式将应用切换到异地从属实例。
		</td>
	</tr>
</table>

## 3. 区域性灾难恢复 ##

如果发生区域性灾难，如大面积机房掉电、火灾或地震等不可抗力事件，则MySQL Database on Azure的异地还原功能可帮助用户在灾难发生时保持业务连续性。

## 3.1 灾难恢复方案 ##
目前针对灾难恢复的方案有三种：
 
* MySQL服务层灾难恢复：MySQL Database on Azure服务本身会在灾难发生时，紧急响应判断灾难发生原因是否能够快速恢复（在小于RPO的时间范围内），如果不能快速恢复，则MySQL Database on Azure会针对所有受影响的实例在异地进行数据库还原，恢复的时间点将为离故障发生最接近的可恢复时间点。

* 自助服务灾难恢复：如果用户使用的是生产环境，对恢复时间要求较高，则用户可以通过PowerShell命令行手动将受影响的实例在异地进行恢复。

* 异地从属实例灾难恢复：如果用户事先设置了异地从属实例，则当主实例不可用时，用户可通过管理门户（手动）或应用程序（自动）切换到异地从属实例，从而实现快速灾难恢复。


## 3.2 异地还原原理 ##
一旦灾难发生，通过自助服务，用户可以通过指定希望还原的时间点，或是PaaS服务恢复方案将会默认指定中断发生数据完好的时间点，MySQL Database on Azure会尽力而为将实例指定地域进行基于该时间点或是离该时间点最近的可恢复时间点进行还原。MySQL Database on Azure底层将用户数据存储在Azure存储的块blob中，并采用只读访问跨地域冗余级别，保障本地异地各有三份数据拷贝，且异地只读可访问。当用户执行异地还原操作时，MySQL Database on Azure会根据用户指定的时间点，在本地存储可用的情况下将数据从本地拷贝到目的区域后进行数据库还原，若本地存储已不可用，MySQL Database on Azure会在异地使用存储层复制后的离指定时间点最接近的数据对数据库进行还原。

## 3.3 异地还原性能指标 ##
ERT<3小时，RPO< 1小时。<br>
>[AZURE.NOTE]**注意：ERT, RTO和RPO是工程指标，仅提供参考，且该指标仅在区域性灾难中出现，且不属于MySQL数据库服务的SLA。**

## 3.4 用户自助服务步骤 ##
如果发生灾难时，管理门户可用，用户可以通过[备份与恢复](/documentation/articles/mysql-database-point-in-time-restore/)中异地还原的步骤进行操作。但通常发生区域性灾难时，管理门户中无法正确获取实例的信息，建议用户通过PowerShell对实例进行异地还原操作：

```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers" -ResourceName <ResourceName> -ApiVersion 2015-09-01 -ResourceGroupName <ResourceGroupName> -Location <TargetLocation> -SkuObject @{name=<targetSKU>} -Properties @{creationSource=@{server='<SourceServerName>';region='<SourceLocation>';timepoint='<TimeTag>'};version = '<version number>'}
```

>[AZURE.NOTE] **注意:<br>
1. 其中timepoint是Optional的值，如果不填，则默认是当前的时间点，如果用户填写时间需按照Json中date-time格式, 如2016-05-06T08:00:00，我们统一使用的是UTC时间。<br>
2. 通过Azure管理门户创建的实例，按地理位置默认分配在”Default-MySQL-ChinaNorth”和”Default-MySQL-ChinaEast” 资源组中。**

当还原实例创建完毕之后，需要将当前IP加入新实例的防火墙白名单中，并且需要手动将应用程序与数据库的连接字符串更新为新实例的hostname，从而恢复应用层业务。

## 3.5 从异地副本中恢复 ##
如果用户设置了异地副本，发生灾难时，可以将业务切换到异地从属实例上。具体步骤为：

1. 登录管理控制界面，选择异地从属实例；
2. 提升异地从属实例为单个实例（详情请参见《[MySQL主从复制和只读实例](https://www.azure.cn/documentation/articles/mysql-database-read-replica/)》一文中的“提升从属实例”部分）；
3. 修改应用端连接地址，连接到提升后的新实例。

因为提升从属实例只需要花费几秒钟的时间，所以从异地副本恢复时，RPO和ERT是最小的，RPO在几秒，ERT在30秒以内。