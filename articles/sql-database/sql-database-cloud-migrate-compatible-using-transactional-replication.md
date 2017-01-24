<properties
    pageTitle="使用事务复制迁移到 SQL 数据库 | Azure"
    description="Azure SQL 数据库，数据库迁移，导入数据库，事务复制"
    services="sql-database"
    documentationcenter=""
    author="jognanay"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="eebdd725-833d-4151-9b2b-a0303f39e30f"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="sqldb-migrate"
    ms.date="12/09/2016"
    wacn.date="01/20/2017"
    ms.author="carlrab; jognanay;" />

# 使用事务复制将 SQL Server 数据库迁移到 Azure SQL 数据库

> [AZURE.SELECTOR]
- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/)
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/)

在本文中，你将了解如何使用 SQL Server 事务复制以最短的停机时间将兼容的 SQL Server 数据库迁移到 Azure SQL 数据库。

## 了解事务复制的体系结构
如果在发生迁移时你无法承受从生产中删除 SQL Server 数据库的后果，可以使用 SQL Server 事务复制作为你的迁移解决方案。若要使用此解决方案，请将 Azure SQL 数据库配置为你想要迁移的本地 SQL Server 实例的订阅服务器。在新的事务不断发生时，本地事务复制分发器将对要同步的本地数据库（发布服务器）中的数据进行同步。

还可以使用事务复制来迁移本地数据库的子集。复制到 Azure SQL 数据库的发布可以限制为复制的数据库中表的子集。对于所复制的每一个表，可以将数据限制为行的子集和/或列的子集。

使用事务复制时，对数据或架构所做的所有更改都会显示在 Azure SQL 数据库中。同步完成后，如果已准备好进行迁移，则可更改应用程序的连接字符串，使其指向 Azure SQL 数据库。一旦事务复制清空保留在本地数据库中的任何更改，并且所有应用程序都指向 Azure DB，即可卸载事务复制。Azure SQL 数据库现在是用户的生产系统。

 ![SeedCloudTR 示意图](./media/sql-database-cloud-migrate/SeedCloudTR.png)  


## 事务复制的工作原理

事务复制涉及到 3 个主要组件，分别是发布服务器、分发服务器和订阅服务器。复制过程由这些组件共同执行。分发服务器负责控制在服务器之间移动数据的过程。设置分发时，SQL 将创建分发数据库。每个发布服务器需要绑定到分发数据库。分发数据库包含每个关联发布内容的元数据，以及有关每个复制操作的进度的数据。在事务复制中，分发数据库包含所有事务，但要在订阅服务器中执行的事务除外。

发布服务器是所有要迁移的数据的来源数据库。发布服务器中可能包含许多发布内容。这些发布内容包含映射到所有表的文章，以及需要复制的数据。根据发布内容和文章的定义方式，可以复制所有或一部分数据库。

在复制中，订阅服务器是从发布服务器接收所有数据和事务的服务器。每个发布服务器可能包含许多复制内容。

## 事务复制要求
[转到更新的要求列表的链接。](https://msdn.microsoft.com/zh-cn/library/mt589530.aspx)
> [AZURE.IMPORTANT]
使用最新版本的 SQL Server Management Studio 以与 Azure 和 SQL 数据库的更新保持同步。较旧版本的 SQL Server Management Studio 不能将 SQL 数据库设置为订阅服务器。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。
> 

## 使用事务复制工作流迁移到 SQL 数据库

1. 设置分发
   -  [使用 SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/zh-cn/library/ms151192.aspx#Anchor_1)
   -  [使用 Transact-SQL](https://msdn.microsoft.com/zh-cn/library/ms151192.aspx#Anchor_2)
2. 创建发布
   -  [使用 SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/zh-cn/library/ms151160.aspx#Anchor_1)
   -  [使用 Transact-SQL](https://msdn.microsoft.com/zh-cn/library/ms151160.aspx#Anchor_2)
3. 创建订阅
   -  [使用 SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/zh-cn/library/ms152566.aspx#Anchor_0)
   -  [使用 Transact-SQL](https://msdn.microsoft.com/zh-cn/library/ms152566.aspx#Anchor_1)

## 有关迁移到 SQL 数据库的一些提示和差异

1. 使用本地分发服务器
   - 这会对服务器的性能造成影响。
   - 如果对性能的影响不可接受，可以使用另一台服务器，但这又会增大管理的复杂性。
2. 选择快照文件夹时，请确保选择的文件夹足够大，可以保存想要复制的每个表的 BCP。
3. 请注意，快照创建操作在完成之前会锁定关联的表，因此，在计划快照时请记住这一点。
4. Azure SQL 数据库中仅支持推送订阅
   - 只能从本地数据库端添加订阅服务器。

## 后续步骤

- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)
- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [SQL Server 2016](https://www.microsoft.com/en-us/sql-server/sql-server-2016)

## 其他资源
* 若要了解有关事务复制的详细信息，请参阅[事务复制](https://msdn.microsoft.com/zh-cn/library/mt589530.aspx)。
* 若要了解整个迁移过程和选项，请参阅[将 SQL Server 数据库迁移到云中的 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。

<!---HONumber=Mooncake_0116_2017-->
<!--update: add three sections("事务复制工作原理","使用事务复制工作流迁移到 SQL 数据库","有关迁移到 SQL 数据库的一些提示和差异";remove two links) ; -->