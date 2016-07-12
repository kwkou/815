<properties
   pageTitle="使用 Azure 备份来取代磁带基础结构 | Azure"
   description="了解如何 Azure 备份如何提供类似于磁带的语义，让你在 Azure 中备份和还原数据"
   services="backup"
   documentationCenter=""
   authors="Jim-Parker"
   manager="jwhit"
   editor=""/>
<tags
   ms.service="backup"
   ms.date="06/08/2016"
   wacn.date="07/04/2016"/>

# 使用 Azure 备份来取代磁带基础结构

Azure 备份和 System Center Data Protection Manager 客户可以：
- 根据最适合组织需求的计划备份数据
- 长期保留备份数据
- 使用 Azure（而不是磁带）解决其长期数据保留需求。

本文将介绍客户如何启用备份和保留策略。使用磁带解决长期数据保留需求的客户现在可以使用一种强大而可行并且同样提供此功能的替代解决方案。最新版本的 Azure 备份（可从[此处](http://aka.ms/azurebackup_agent)获取）中已启用该功能。SCDPM 客户在使用此功能之前需要迁移到 UR5。

## 什么是备份计划？
备份计划指示备份操作的频率。例如，下面屏幕中的设置表示要在每天下午 6 点和午夜执行备份。

![每日计划](./media/backup-azure-backup-cloud-as-tape/dailybackupschedule.png)

客户还可以安排每周备份。例如，以下屏幕中的设置表示要在每个星期日和星期三的上午 9:30 和 1:00 创建备份。

![每周计划](./media/backup-azure-backup-cloud-as-tape/weeklybackupschedule.png)

## 什么是保留策略？
保留策略指定备份必须存储的时间长短。客户可以根据备份的创建时间指定不同的保留策略，而不只是为所有备份点指定一个“通用的策略”。例如，在每个季末创建的备份点可能需要保留更长的时间以用于审核目的，而每天创建的备份点（用作操作恢复点）需要保留 90 天。

![保留策略](./media/backup-azure-backup-cloud-as-tape/retentionpolicy.png)

此策略中指定的“保留点”总数为 90（每日点数） + 40（10 年内的每个季度）= 130。

## 示例 - 组合策略

![示例屏幕](./media/backup-azure-backup-cloud-as-tape/samplescreen.png)

1. **每天保留策略**：每天创建的备份存储 7 天。
2. **每周保留策略**：每个星期六午夜和下午 6 点创建的备份保留 4 周
3. “每月保留策略”：每个月最后一个星期六午夜和下午 6 点创建的备份保留 12 个月
4. “每年保持期策略”：每年三月最后一个星期六午夜执行的备份保留 10 年

上图中“保留点”总数（客户可从中还原数据的恢复点）计算方式如下：

- 每天 2 个点 x 7 天 = 14 个恢复点
- 每周 2 个点 x 4 周 = 8 个恢复点
- 每月 2 个点 x 12 个月 = 24 个恢复点
- 每年 1 个点 x 10 年 = 10 个恢复点

恢复点总数为 56。

> [AZURE.NOTE] Azure 备份对恢复点的数量没有限制。

## 高级配置
通过单击上述屏幕中的“修改”，客户可以进一步灵活指定保留计划。

![修改](./media/backup-azure-backup-cloud-as-tape/modify.png)

## 后续步骤
关于 Azure 备份的详细信息，请参阅

- [Azure 备份简介](/documentation/articles/backup-introduction-to-azure-backup/)
- [尝试 Azure 备份](/documentation/articles/backup-try-azure-backup-in-10-mins/)

<!---HONumber=Mooncake_0627_2016-->