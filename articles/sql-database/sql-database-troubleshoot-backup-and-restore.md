<properties
	pageTitle="排查 Azure SQL 数据库的备份和还原问题"
	description="了解如何在发生错误和服务中断时，使用 Azure SQL 数据库中的备份和副本恢复云数据库。"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	wacn.date="10/17/2016"
	ms.author="daleche"/>  


# 将数据库还原到以前的时间点、还原已删除的数据库，或者在数据中心服务中断的情况下进行恢复

SQL 数据库保留了你数据库的副本，因此你可以在发生服务中断和用户错误的情况下进行恢复。可用的选项取决于数据库服务层和你选择的选项。有关详细信息和设计注意事项，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

## 将数据库还原到以前的时间点
1.	在 [Azure 门户预览](https://portal.azure.cn)中，单击“SQL 数据库”。
2.	从列表中选择你的数据库，然后单击“还原”。
3.	键入数据库的新名称、选择要从中还原的日期和时间，然后单击“创建”。
4.	根据需要调整应用以引用新的数据库。请参阅[将数据库恢复到某个时间点](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore)。

## 还原意外删除的数据库
1.	在 [Azure 门户预览](https://portal.azure.cn)中，单击“SQL Server”。
2.	从列表中选择托管该数据库的服务器。
3.	在“服务器”边栏选项卡上，向下滚动并单击“已删除的数据库”。
4.	选择要还原的数据库，然后单击“创建”。
5.	根据需要调整应用以引用新的数据库。请参阅[恢复已删除的数据库](/documentation/articles/sql-database-recovery-using-backups/#deleted-database-restore)。

## 发生区域性的数据中心服务中断后进行恢复
使用标准和高级数据库时，如果你设置了异地复制的辅助数据库，则你可以使用这些辅助数据库进行恢复。这样，你就可以还原数据库，并且不太可能会丢失数据。有关详细信息，请参阅[使用自动数据库备份恢复 Azure SQL 数据库](/documentation/articles/sql-database-disaster-recovery/)。

Azure 还在不同的区域提供每个数据库的备份（异地冗余的备份）。你可以从这些备份创建新数据库（称为异地还原），但如果只是依赖于这一种方法，有可能会发生数据丢失。

**使用异地还原恢复数据库：**

- 在 [Azure 门户预览](https://portal.azure.cn)中，依次单击“新建”、“数据和存储”、“SQL 数据库”，然后选择“备份”作为数据库源。有关详细信息，请参阅[在中断后恢复 Azure SQL 数据库](/documentation/articles/sql-database-disaster-recovery/)。

<!---HONumber=Mooncake_1010_2016-->