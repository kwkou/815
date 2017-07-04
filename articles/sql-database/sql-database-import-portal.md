<properties
    pageTitle="导入 BACPAC 文件以创建 Azure SQL 数据库 | Azure"
    description="通过导入现有 BACPAC 文件创建 Azure SQL 数据库。"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="cf9a9631-56aa-4985-a565-1cacc297871d"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />  


# 使用 Azure 门户导入创建 Azure SQL 数据库所需的 BACPAC 文件

本文说明如何使用 [Azure 门户](https://portal.azure.cn)通过 BACPAC 文件创建 Azure SQL 数据库。

## 先决条件

若要从 PACPAC 导入 SQL 数据库，需要以下内容：

* Azure 订阅。
* Azure SQL 数据库 V12 服务器。如果没有 V12 服务器，可以按照本文中的以下步骤创建一个：[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)。
* 要导入 [Azure 存储帐户（标准）](/documentation/articles/storage-create-storage-account/)blob 容器中的数据库的 .bacpac 文件。

> [AZURE.IMPORTANT]
>从 Azure Blob 存储导入 BACPAC 时，请使用标准存储。不支持从高级存储导入 BACPAC。
> 

## 导入数据库
打开 SQL Server 边栏选项卡：

1. 转到 [Azure 门户](https://portal.azure.cn)。
2. 单击“SQL Server”。
3. 单击要将数据库还原到的服务器。
4. 在 SQL Server 边栏选项卡中，单击“导入数据库”以打开“导入数据库”边栏选项卡：
   
   ![导入数据库][1]
5. 单击“存储”并依次选择存储帐户、blob 容器和 .bacpac 文件，然后单击“确定”。
   
   ![配置存储选项][2]
6. 为新数据库选择定价层，然后单击“选择”。不支持直接将数据库导入弹性池，但可先将其作为独立数据库导入，然后将数据库移入池中。
   
   ![选择定价层][3]  
7. 针对要从 BACPAC 文件创建的数据库输入**数据库名称**。
8. 选择身份验证类型，然后向服务器提供身份验证信息。
9. 单击“创建”以从 BACPAC 创建数据库。
   
   ![创建数据库][4]

单击“创建”会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

## 监视导入操作的进度
1. 单击“SQL Server”。
2. 单击要还原到的服务器。
3. 在 SQL Server 边栏选项卡的操作区域中，单击“导入/导出历史记录”：
   
   ![导入导出历史记录][5] 
   ![导入导出历史记录][6]
4. 若要验证数据库是否位于服务器上，请单击“SQL 数据库”并验证新数据库是否处于“联机”状态。

## 后续步骤
* 若要了解如何连接到导入的 SQL 数据库并对其进行查询，请参阅[使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)。
* 如需 SQL Server 客户顾问团队编写的有关使用 BACPAC 文件进行迁移的博客，请参阅 [Migrating from SQL Server to Azure SQL Database using BACPAC Files](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)（使用 BACPAC 文件从 SQL Server 迁移到 Azure SQL 数据库）。
* 如需 SQL Server 数据库完整迁移过程的介绍（包括性能建议），请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。


<!--Image references-->

[1]: ./media/sql-database-import/import-database.png
[2]: ./media/sql-database-import/storage-options.png
[3]: ./media/sql-database-import/pricing-tier.png
[4]: ./media/sql-database-import/create.png
[5]: ./media/sql-database-import/import-history.png
[6]: ./media/sql-database-import/import-status.png

<!---HONumber=Mooncake_0320_2017-->