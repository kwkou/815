<properties
   pageTitle="恢复服务保管库常见问题 | Azure"
   description="此版常见问题支持 Azure 备份服务公共预览版。针对备份代理、备份和保留、恢复、安全性的常见问题以及针对 Azure 备份解决方案的其他常见问题的答案。"
   services="backup"
   documentationCenter=""
   authors="markgalioto"
   manager="jwhit"
   editor=""
   keywords="备份解决方案; 备份服务"/>  


<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/21/2016"
	ms.author="trinadhk; markgal; jimpark;"
   	wacn.date="10/26/2016"/>  


# 恢复服务保管库 - 常见问题

> [AZURE.SELECTOR]
- [经典模式备份常见问题](/documentation/articles/backup-azure-backup-faq/)
- [资源管理器模式备份常见问题](/documentation/articles/backup-azure-backup-ibiza-faq/)

本文提供特定于恢复服务保管库的信息，是 [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq/)的补充。Azure 备份常见问题提供整套有关 Azure 备份服务的问答。

你可以在本文或相关章的 Disqus 部分中提出有关 Azure 备份的问题。还可以在[论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazureonlinebackup)中发布有关 Azure 备份服务的问题。

## 既然恢复服务保管库基于资源管理器，备份保管库（经典模式）是否仍受支持？<br/>
是，仍然支持备份保管库。可以在[经典管理门户](https://manage.windowsazure.cn)中创建备份保管库。

## 是否可以将备份保管库迁移到恢复服务保管库？<br/>
很遗憾不可以，目前无法将备份保管库的内容迁移到恢复服务保管库。我们正着手添加此功能，但公共预览中并未提供。

## 恢复服务保管库是否支持经典 VM 或基于资源管理器的 VM？<br/>
恢复服务保管库支持这两种模型。可以将经典管理门户中创建的 VM（经典模式 VM）或 Azure 门户预览中创建的 VM（基于资源管理器）备份到恢复服务保管库。


<!---HONumber=Mooncake_0801_2016-->