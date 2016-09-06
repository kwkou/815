<properties
   pageTitle="公共预览版 Azure 备份常见问题 | Microsoft Azure"
   description="此版常见问题支持 Azure 备份服务公共预览版。针对备份代理、备份和保留、恢复、安全性的常见问题以及针对 Azure 备份解决方案的其他常见问题的答案。"
   services="backup"
   documentationCenter=""
   authors="markgalioto"
   manager="jwhit"
   editor=""
   keywords="备份解决方案; 备份服务"/>

<tags
    ms.service="backup"
    ms.date="07/01/2016"
    wacn.date="09/06/2016"/>

# 公共预览版 Azure 备份服务 - 常见问题

> [AZURE.SELECTOR]
- [经典模式备份常见问题](/documentation/articles/backup-azure-backup-faq/)
- [ARM 模式备份常见问题](/documentation/articles/backup-azure-backup-ibiza-faq/)

本文提供特定于 Azure 备份服务公共预览版的信息。本文在有新的常见问题时会更新，并补充 [Azure 备份常见问题](backup-azure-backup-faq)。Azure 备份常见问题提供整套有关 Azure 备份服务的问答。

你可以在本文或相关章的 Disqus 部分中提出有关 Azure 备份的问题。你也可以在[论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazureonlinebackup)中发布有关 Azure 备份服务的问题。

## 公共预览版有哪些功能？
公共预览版引入了在保护 Azure VM 时的恢复服务保管库和 ARM 支持。恢复服务保管库是下一代保管库。它是 Azure 备份和 Azure Site Recovery (ASR) 服务使用的保管库。可以将它视为 v.2 保管库。

## 恢复服务和备份保管库

**问 1.恢复服务保管库版本为 v.2，那么是否仍支持备份保管库 (v.1)？** <br/>
答 1.是，仍然支持备份保管库。可在经典门户中创建备份保管库，使用 Azure PowerShell 创建恢复服务保管库。

**问 2.是否可以将备份保管库迁移到恢复服务保管库？** <br/>
答 2.很遗憾不可以，目前无法将备份保管库的内容迁移到恢复服务保管库。我们正着手添加此功能，但公共预览中并未提供。

**问 3.恢复服务保管库支持 v.1 还是 v.2 VM？** <br/>
答 3.恢复服务保管库支持 v.1 和 v.2 VM。可以将经典门户中创建的 VM（即 V.1）或 Azure PowerShell 创建的 VM（即 V.2）备份到恢复服务保管库。


<!---HONumber=Mooncake_0801_2016-->