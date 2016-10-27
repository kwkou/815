<properties
	pageTitle="在服务器之间或订阅之间移动数据库，以及将数据库移入和移出 Azure。"
	description="在 Azure SQL 数据库中复制、移动和迁移数据与数据库的快速步骤。"
	services="sql-database"
	documentationCenter=""
	authors="v-shysun"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/13/2016"
	wacn.date="10/17/2016"
	ms.author="v-shysun"/>  


# 在服务器之间或订阅之间移动数据库，以及将数据库移入和移出 Azure

##将数据库移到同一订阅中的不同服务器
- 在 [Azure 门户预览](https://portal.azure.cn)中，单击“SQL 数据库”，从列表中选择数据库，然后单击“复制”。有关详细信息，请参阅[复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/)。

## 在订阅之间移动数据库
- 在 [Azure 门户预览](https://portal.azure.cn)中，单击“SQL Server”，然后从列表中选择托管数据库的服务器。单击“移动”，然后选择要移动的资源以及要移动到其中的订阅。

## 将 SQL 数据库迁移到 Azure
- 确定数据库兼容性，然后根据需要选择正确的迁移方法。根据[迁移 SQL Server 数据库](/documentation/articles/sql-database-cloud-migrate/)中的指导和选项进行操作。

## 创建数据库副本以便在 Azure 外部使用
- [导出 BACPAC 文件。](/documentation/articles/sql-database-export/)

<!---HONumber=Mooncake_1010_2016-->