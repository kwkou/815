<properties 
    pageTitle="使用 Azure 门户预览为 Azure SQL 数据库配置异地复制 | Azure" 
    description="使用 Azure 门户预览为 Azure SQL 数据库配置异地复制" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
   ms.workload="NA"
    ms.date="07/14/2016"
    wacn.date="09/19/2016"
    ms.author="sstein"/>

# 使用 Azure 门户预览为 Azure SQL 数据库配置异地复制


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-replication-overview/)
- [Azure 门户预览](/documentation/articles/sql-database-geo-replication-portal/)
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)
- [T-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/)

本文说明如何使用 [Azure 门户预览](http://portal.azure.cn)为 SQL 数据库配置活动异地复制。

若要使用 Azure 门户预览启动故障转移，请参阅[使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-portal/)。

>[AZURE.NOTE] 活动异地复制（可读辅助数据库）现在可供所有服务层中的所有数据库使用。非可读辅助类型将在 2017 年 4 月停用，现有的非可读数据库将自动升级到可读辅助数据库。

若要使用 Azure 门户预览配置异地复制，需提供：

- Azure 订阅。
- 一个 Azure SQL 数据库数据库 - 需要复制到其他地理区域的主数据库。

## 添加辅助数据库

以下步骤在异地复制合作关系中创建新的辅助数据库。

只有订阅所有者或共同所有者才可添加辅助数据库。

辅助数据库将使用与主数据库相同的名称，并将默认使用相同的服务级别。辅助数据库可以是可读或不可读，并且可以是单一数据库或弹性数据库。有关详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/)。
创建辅助数据库并设定种子之后，开始将数据从主数据库复制到新的辅助数据库。

> [AZURE.NOTE] 如果伙伴数据库已存在（例如，在终止旧的异地复制关系的情况下），命令将会失败。

### 添加辅助数据库

1. 在 [Azure 门户预览](http://portal.azure.cn)中，浏览到需要设置以便进行异地复制的数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 选择要创建辅助数据库的区域。


    ![添加辅助数据库][1]


4. 配置“辅助数据库类型”（“可读”或“不可读”）。
5. 选择或配置辅助数据库的服务器。

    ![创建辅助数据库][3]

5. 你可以选择性地将辅助数据库添加到弹性数据库池：

       - 单击“弹性数据库池”，然后选择要在其中创建辅助数据库的目标服务器上的池。目标服务器上必须已存在池，因为此工作流不创建新池。

6. 单击“创建”添加辅助数据库。
 
6. 此时会创建辅助数据库，种子设定过程开始。
 
    ![种子设定][6]

7. 完成种子设定过程时，辅助数据库会显示其状态（“不可读”）。

    ![辅助数据库准备就绪][9]



## 删除辅助数据库

此操作会永久终止到辅助数据库的复制，并会将辅助数据库的角色更改为常规的读-写数据库。如果与辅助数据库的连接断开，命令将会成功，但辅助数据库必须等到连接恢复后才会变为可读写。

1. 在 [Azure 门户预览](http://portal.azure.cn)中浏览到异地复制合作关系中的主数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 在“辅助数据库”列表中选择需要从异地复制合作关系中删除的数据库。
4. 单击“停止复制”。

    ![删除辅助数据库][7]


5. 单击“停止复制”将打开一个确认窗口，此时单击“是”即可从异地复制合作关系中删除该数据库（将其设置为不属于任何复制的读写数据库）。


    ![确认删除][8]


## 后续步骤

- 若要深入了解活动异地复制，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)


<!--Image references-->
[1]: ./media/sql-database-geo-replication-portal/configure-geo-replication.png
[2]: ./media/sql-database-geo-replication-portal/add-secondary.png
[3]: ./media/sql-database-geo-replication-portal/create-secondary.png
[4]: ./media/sql-database-geo-replication-portal/secondary-type.png
[5]: ./media/sql-database-geo-replication-portal/create.png
[6]: ./media/sql-database-geo-replication-portal/seeding0.png
[7]: ./media/sql-database-geo-replication-portal/remove-secondary.png
[8]: ./media/sql-database-geo-replication-portal/stop-confirm.png
[9]: ./media/sql-database-geo-replication-portal/seeding-complete.png
[10]: ./media/sql-database-geo-replication-portal/failover.png

<!---HONumber=Mooncake_0912_2016-->