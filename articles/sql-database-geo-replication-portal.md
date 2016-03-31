<properties 
    pageTitle="使用 Azure 门户为 Azure SQL 数据库配置异地复制 | Azure" 
    description="使用 Azure 门户为 Azure SQL 数据库配置异地复制" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jeffreyg" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="12/01/2015"
    wacn.date="01/29/2016"/>

# 使用 Azure 门户为 Azure SQL 数据库配置异地复制


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-geo-replication-portal)
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell)
- [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql)


本文说明如何使用 [Azure 门户](https://manage.windowsazure.cn)为 Azure SQL 数据库配置异地复制。

异地复制可让你在不同的数据中心位置（区域）最多创建 4 个副本（辅助）数据库。在数据中心发生服务中断或无法连接到主数据库时，可以使用辅助数据库。

异地复制仅适用于标准和高级数据库。

标准数据库可以有一个不可读取的辅助副本，并且必须使用建议的区域。高级数据库最多可以有四个位于任何可用区域并且可读取的辅助副本。


若要配置异地复制，需要提供以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 一个 Azure SQL 数据库数据库 - 需要复制到其他地理区域的主数据库。



## 添加辅助数据库

以下步骤在异地复制合作关系中创建新的辅助数据库。

若要添加辅助数据库，你必须是订阅所有者或共同所有者。

辅助数据库将使用与主数据库相同的名称，并将默认使用相同的服务级别。辅助数据库可以是可读数据库（仅限高级版），也可以是不可读数据库；可以是单一数据库，也可以是弹性数据库。有关详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers)。
创建辅助数据库并设定种子之后，开始将数据从主数据库复制到新的辅助数据库。

> [AZURE.NOTE] 如果伙伴数据库已存在（例如，由于终止前面的异地复制关系），命令将会失败。




### 添加辅助数据库

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，浏览到需要设置以便进行异地复制的数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 选择要创建辅助数据库的区域。高级数据库可以将任何区域用于辅助数据库，标准数据库必须使用建议的区域：





4. 配置“辅助数据库类型”（“可读”或“不可读”）。仅高级数据库可以有可读辅助数据库，标准数据库的辅助数据库只能设置为“不可读”。
5. 选择或配置辅助数据库的服务器。



5. 你可以选择性地将辅助数据库添加到弹性数据库池：

       - 单击“弹性数据库池”，然后选择要在其中创建辅助数据库的目标服务器上的池。目标服务器上必须已存在池，因为此工作流不创建新池。

6. 单击“创建”添加辅助数据库。
 
6. 此时会创建辅助数据库，种子设定过程开始。
 


7. 完成种子设定过程时，辅助数据库会显示其状态（“不可读”）。





## 删除辅助数据库

此操作会永久终止到辅助数据库的复制，并会将辅助数据库的角色更改为常规的读-写数据库。如果与辅助数据库的连接断开，命令将会成功，但辅助数据库必须等到连接恢复后才会变为可读写。

1. 在 [Azure 门户](https://manage.windowsazure.cn)中浏览到异地复制合作关系中的主数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 在“辅助数据库”列表中选择需要从异地复制合作关系中删除的数据库。
4. 单击“停止复制”。




5. 单击“停止复制”将会打开一个确认窗口，因此请单击“是”从异地复制合作关系中删除该数据库（将其设置为不参与任何复制过程的读-写数据库）。






## 启动故障转移

辅助数据库可以通过切换变为主数据库。

1. 在 [Azure 门户](https://manage.windowsazure.cn)中浏览到异地复制合作关系中的主数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 在“辅助数据库”列表中，选择需要让其成为新的主数据库的数据库。
4. 单击“故障转移”。



该命令将执行以下工作流：

1. 暂时将复制切换到同步模式。这会导致将所有未处理的事务排送到辅助数据库。 

2. 切换异地复制合作关系中两个数据库的主辅角色。

对于计划的故障转移，此顺序可确保不发生任何数据丢失。切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。在正常情况下，完成整个操作所需的时间应该少于一分钟。

   

## 后续步骤

- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)


## 其他资源

- [新异地复制功能的亮点](http://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication)
- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)


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

<!---HONumber=Mooncake_0118_2016-->
