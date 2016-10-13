<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,区域性灾难,业务连续性方案,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="本文提供三种灾难恢复方案，保障用户的业务连续性。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="v-chenyh" solutions="" manager="RongYu" editor="" />

<tags ms.service="mysql" ms.date="10/13/2016" wacn.date="10/13/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-business-continuity-disaster-recovery/)
- [English](/documentation/articles/mysql-database-enus-business-continuity-disaster-recovery/)

# MySQL Database on Azure业务连续性方案

本文将总结几个典型的中断事件场景以及MySQL Database on Azure保障业务连续性的方案，并重点介绍针对区域性灾难恢复的具体步骤和指标。

## 概述 ##

**业务连续性**指以弹性方式设计、部署和运行应用程序，以应对计划或非计划的中断事件，并避免应用程序永久性或暂时性无法执行业务功能。

**保障业务连续性的目标**是减少中断时间对应用业务的影响、尽可能缩短影响时间，以及避免数据丢失。

**非计划事件**涵盖从人为失误到永久或暂时性中断，再到区域性灾难（这可能会导致特定 Azure 区域中出现大规模的机能损失）的整个范围。

**计划事件**包括向不同的区域重新部署应用程序以及应用程序升级。
在介绍业务连续性方案前，您需要熟悉以下几个概念：

* **恢复时间目标（RTO）**：在发生中断性事件后，应用程序完全恢复之前的最长可接受时间。RTO用来度量故障期间的最大可用性损失。
* **恢复点目标（RPO）**：在发生中断性事件后，应用程序在完全恢复时可以丢失的最大最近更新数量（时间间隔）。RPO用于度量故障期间的最大数据丢失。
* **预计恢复时间（ERT）**：在发出还原或故障转移请求后，数据库完全可用之前预计持续的时间。


## 业务连续性方案 ##

下文介绍几个主要的场景和相应方案。

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
			备份与恢复——任意时间点回滚功能<br><br>
			MySQL Database on Azure支持七天内的任意时间点回滚，具体步骤请参照：<a href="https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore/" target="_blank">MySQL Database on Azure备份和恢复——还原数据库到任意时间点</a>。

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
			在升级前，用户对数据库创建快照备份；如升级遇到问题，可以快速还原完全备份到新实例/原实例。具体步骤请参照：<a href="https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore/" target="_blank">MySQL Database on Azure备份和恢复</a>
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
			目前提供三种恢复方案，用户可以根据紧急程度选择不同方案。后文将重点介绍。<br><br>
			- MySQL PaaS方案<br>
			- 自助服务方案<br>
			- 异地从属实例恢复方案
		</td>
	</tr>
</table>

## 区域性灾难恢复 ##

MySQL Database on Azure具有Geo Restore（异地还原）和Geo Replication（异地复制）技术，可帮助您在发生区域性灾难（比如大面积机房掉电、火灾或地震等不可抗力事件）时保持业务连续性。

**Geo Restore异地还原**

MySQL Database on Azure底层将用户数据存储在Azure存储的块blob中，并采用只读访问跨地域冗余级别，保障本地异地各有三份数据拷贝，且异地只读可访问。当用户执行异地还原操作时，MySQL Database on Azure会根据用户指定的时间点，在本地存储可用的情况下将数据从本地拷贝到目的区域后进行数据库还原，若本地存储已不可用，MySQL Database on Azure会在异地使用存储层复制后的离指定时间点最接近的数据对数据库进行还原。

利用异地还原技术，用户可以选择PaaS方案或自助服务方案实现灾难恢复。异地还原技术的性能指标为：**ERT < 3小时，RPO < 1小时**。

**Geo Replication异地复制**

如果用户事先通过管理门户创建了异地从属实例，则区域性灾难发生时可以提升该从属实例用于业务切换，保障业务的连续性。异地复制技术的性能指标为：**ERT < 30秒，RPO < 10秒**。

>[AZURE.NOTE]ERT、RTO和RPO是工程指标，仅供参考，且仅适用于区域性灾难，不属于MySQL数据库服务的SLA。


## 灾难恢复方案 ##

目前针对灾难恢复的方案有三种，分别是：MySQL PaaS方案、自助服务方案、异地从属实例恢复方案。

* **MySQL PaaS方案（利用异地还原技术）**	MySQL Database on Azure服务本身会在灾难发生时，紧急响应判断灾难发生原因是否能够快速恢复（在小于RPO的时间范围内），如果不能快速恢复，则MySQL Database on Azure会针对所有受影响的实例在异地进行数据库还原，恢复的时间点将为离故障发生最接近的可恢复时间点。

>[AZURE.NOTE]PaaS方案将会默认指定中断发生时数据完好的时间点，而MySQL Database on Azure会尽力将实例指定地域进行基于该时间点或是离该时间点最近的可恢复时间点进行还原。

* **自助服务方案（利用异地还原技术）**	如果用户使用的是生产环境，对恢复时间要求较高，则用户可以通过PowerShell命令行手动将受影响的实例在异地进行恢复。

*用户自助服务说明*

如果发生灾难时管理门户可用，则用户可以通过《[MySQL Database on Azure备份和恢复](http://wacn-ppe.chinacloudsites.cn/documentation/articles/mysql-database-point-in-time-restore/)》一文中提供的异地还原步骤进行操作。但通常发生区域性灾难时，管理门户无法正确获取实例信息，因此建议用户通过PowerShell对实例进行异地还原操作：

	New-AzureRmResource -ResourceType "Microsoft.MySql/servers" -ResourceName <ResourceName> -ApiVersion 2015-09-01 -ResourceGroupName <ResourceGroupName> -Location <TargetLocation> -SkuObject @{name=<targetSKU>} -Properties @{creationSource=@{server='<SourceServerName>';region='<SourceLocation>';timepoint='<TimeTag>'};version = '<version number>'}

>[AZURE.NOTE]
>1. timepoint是Optional的值，如果不填，则默认是当前的时间点；如果用户填写时间，则需按照Json中date-time格式填写（比如2016-05-06T08:00:00），我们统一使用的是UTC时间。
>2. 通过Azure管理门户创建的实例，按地理位置默认分配在“Default-MySQL-ChinaNorth”和“Default-MySQL-ChinaEast”资源组中。

当还原实例创建完毕之后，需要将当前IP加入新实例的防火墙白名单中，并且需要手动将应用程序与数据库的连接字符串更新为新实例的hostname，从而恢复应用层业务。

* **异地从属实例恢复方案 （利用异地复制技术）**	如果用户事先设置了异地从属实例，则当主实例不可用时，用户可通过管理门户（手动）或应用程序（自动）切换到异地从属实例，从而实现快速灾难恢复。

*异地从属实例恢复步骤*

1. 登录管理控制界面，选择异地从属实例。
2. 提升异地从属实例为单个实例（详情请参见《[MySQL主从复制和只读实例](https://www.azure.cn/documentation/articles/mysql-database-read-replica/)》一文中的“提升从属实例”部分）。
3. 修改应用端连接地址，连接到提升后的新实例。