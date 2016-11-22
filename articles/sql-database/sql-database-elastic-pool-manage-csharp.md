<properties
    pageTitle="使用 C# 监视和管理弹性数据库池 | Azure"
    description="使用 C# 数据库开发技术来管理 Azure SQL 数据库弹性数据库池。"
    services="sql-database"
    documentationCenter=""
    authors="stevestein"
    manager="jhubbard"
    editor=""/>  


<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="csharp"
    ms.workload="data-management"
    ms.date="10/04/2016"
    wacn.date="10/31/2016"
    ms.author="sstein"/>  


# 使用 C&#x23; 监视和管理弹性数据库池 

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-elastic-pool-manage-powershell/)
- [C#](/documentation/articles/sql-database-elastic-pool-manage-csharp/)
- [T-SQL](/documentation/articles/sql-database-elastic-pool-manage-tsql/)


了解如何使用 C#; 管理[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

>[AZURE.NOTE] SQL 数据库的许多新功能仅在使用 [Azure Resource Manager 部署模型](/documentation/articles/resource-group-overview/)时才可用，因此，始终应该使用最新版本的**用于 .NET 的 Azure SQL 数据库管理库（[文档](https://msdn.microsoft.com/zh-cn/library/azure/mt349017.aspx) | [NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)）**。以前的[基于经典部署模型的库](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Sql)只是为了向后兼容而受到支持，因此，建议使用较新的基于 Resource Manager 的库。

若要完成本文中的步骤，需要准备好以下项：

- 弹性池（想要管理的池）。若要创建池，请参阅 [Create an elastic database pool with C#](/documentation/articles/sql-database-elastic-pool-create-csharp/)（使用 C# 创建弹性数据库池）。
- Visual Studio。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)页。


## 将数据库移入弹性池

你可将独立的数据库移入或移出池。

    // Retrieve current database properties.

    currentDatabase = sqlClient.Databases.Get("resourcegroup-name", "server-name", "Database1").Database;

    // Configure create or update parameters with existing property values, override those to be changed.
    DatabaseCreateOrUpdateParameters updatePooledDbParameters = new DatabaseCreateOrUpdateParameters()
    {
        Location = currentDatabase.Location,
        Properties = new DatabaseCreateOrUpdateProperties()
        {
            Edition = "Standard",
            RequestedServiceObjectiveName = "ElasticPool",
            ElasticPoolName = "ElasticPool1",
            MaxSizeBytes = currentDatabase.Properties.MaxSizeBytes,
            Collation = currentDatabase.Properties.Collation,
        }
    };

    // Update the database.
    var dbUpdateResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database1", updatePooledDbParameters);

## 列出弹性池中的数据库

若要在池中检索所有数据库，可调用 [ListDatabases](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.management.sql.elasticpooloperationsextensions.listdatabases) 方法。

    //List databases in the elastic pool
    DatabaseListResponse dbListInPool = sqlClient.ElasticPools.ListDatabases("resourcegroup-name", "server-name", "ElasticPool1");
    Console.WriteLine("Databases in Elastic Pool {0}", "server-name.ElasticPool1");
    foreach (Database db in dbListInPool)
    {
        Console.WriteLine("  Database {0}", db.Name);
    }

## 更改池的性能设置

检索现有池属性。修改值并执行 CreateOrUpdate 方法。

    var currentPool = sqlClient.ElasticPools.Get("resourcegroup-name", "server-name", "ElasticPool1").ElasticPool;

    // Configure create or update parameters with existing property values, override those to be changed.
    ElasticPoolCreateOrUpdateParameters updatePoolParameters = new ElasticPoolCreateOrUpdateParameters()
    {
        Location = currentPool.Location,
        Properties = new ElasticPoolCreateOrUpdateProperties()
        {
            Edition = currentPool.Properties.Edition,
            DatabaseDtuMax = 50, /* Setting DatabaseDtuMax to 50 limits the eDTUs that any one database can consume */
            DatabaseDtuMin = 10, /* Setting DatabaseDtuMin above 0 limits the number of databases that can be stored in the pool */
            Dtu = (int)currentPool.Properties.Dtu,
            StorageMB = currentPool.Properties.StorageMB,  /* For a Standard pool there is 1 GB of storage per eDTU. */
        }
    };

    newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);


## 弹性池操作延迟

- 更改每个数据库的最小 eDTU 数或每个数据库的最大 eDTU 数通常可在五分钟或更少的时间内完成。
- 更改池大小 (eDTU) 所需的时间取决于池中所有数据库的总大小。更改平均起来每 100 GB 需要 90 分钟或更短的时间。例如，如果池中所有数据库的总空间为 200 GB，则更改每个池的池 eDTU 时，预计延迟为 3 小时或更短的时间。




## 其他资源

- [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题](/documentation/articles/sql-database-develop-error-messages/)。
- [SQL 数据库](/documentation/services/sql-databases/)
- [Azure 资源管理 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx)
- [使用 C# 创建新的弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-csharp/)
- [何时使用弹性数据库池？](/documentation/articles/sql-database-elastic-pool-guidance/)
- 请参阅[使用 Azure SQL 数据库进行扩展](/documentation/articles/sql-database-elastic-scale-introduction/)：使用弹性数据库工具扩展、移动数据、查询或创建事务。

<!---HONumber=Mooncake_1024_2016-->