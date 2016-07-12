<properties 
	pageTitle="将弹性数据库客户端库与 Dapper 配合使用 | Azure" 
	description="将弹性数据库客户端库与 Dapper 配合使用。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="torsteng"/>

<tags 
	ms.service="sql-database"
	ms.date="04/26/2016" 
	wacn.date="05/23/2016"/>

# 将弹性数据库客户端库与 Dapper 配合使用 

本文档面向依赖于使用 Dapper 生成应用程序，但同时想要运用[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-introduction/)创建应用程序来实现分片，以向外缩放其应用程序的开发人员。本文档演示了与弹性数据库工具集成所需的基于 Dapper 的应用程序发生的更改。我们将重点介绍如何使用 Dapper 构建弹性数据库分片管理和数据相关的路由。

**示例代码**：[Azure SQL 数据库的弹性数据库工具 - Dapper 集成](https://code.msdn.microsoft.com/Elastic-Scale-with-Azure-e19fc77f)。
 
将**Dapper** 和 **DapperExtensions** 与 Azure SQL 数据库的弹性数据库客户端库的过程很简单。应用程序可以通过将新 [SqlConnection](http://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.aspx) 对象的创建和打开方式更改为使用来自[客户端库](http://msdn.microsoft.com/zh-cn/library/azure/dn765902.aspx)的 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) 调用，来使用数据相关的路由。这会将应用程序中的更改限制为已创建和打开新连接的位置。

## Dapper 概述
**Dapper** 是对象关系映射器。它将应用程序中的 .NET 对象映射到关系数据库（或者执行相反的映射）。示例代码的第一个部分演示了如何将弹性数据库客户端库与基于 Dapper 的应用程序相集成。示例代码的第二个部分演示了同时使用 Dapper 和 DapperExtensions 时如何集成。

Dapper 中的映射器功能对数据库连接提供扩展方法，可以简化用于执行或查询数据库的 T-SQL 语句的提交。例如，使用 Dapper 可以轻松地在 .NET 对象与用于 **Execute** 调用的 SQL 语句参数之间进行映射，或者在 Dapper 中通过 **Query** 调用来使用对 .NET 对象执行 SQL 查询后返回的结果。

使用 DapperExtensions 时，你不再需要提供 SQL 语句。对数据库连接执行 **GetList** 或 **Insert** 等扩展方法会在幕后创建 SQL 语句。
 
Dapper 和 DapperExtensions 的另一个优点在于，应用程序可以控制数据库连接的创建。这有助于与弹性数据库客户端库交互，从而可以通过将 shardlet 映射到数据库来中转数据库连接。

若要获取 Dapper 程序集，请参阅 [Dapper .NET](http://www.nuget.org/packages/Dapper)。有关 Dapper 扩展，请参阅 [DapperExtensions](http://www.nuget.org/packages/DapperExtensions)。

## 弹性数据库客户端库速览

使用弹性数据库客户端库，你可以定义应用程序数据的分区（称为 *shardlet*），将它们映射到数据库，并根据*分片键*来识别这些分区。你可以根据需要创建任意数目的数据库，并在这些数据库之间分布 shardlet。分片键值到数据库的映射由库的 API 提供的分片映射存储。此功能称为**分片映射管理**。分片映射还为带有分片键的请求充当数据库连接的代理。此功能称为**数据相关的路由**。

![分片映射和数据相关的路由][1]

分片映射管理器可防止用户不一致的视图到 shardlet 数据时并发 shardlet 管理操作发生在数据库上可能会出现。为此，分片映射将会代理使用库生成的应用程序的数据库连接。当分片管理操作可能会影响 shardlet 时，这可以允许分片映射功能自动终止数据库连接。

我们需要使用 [OpenConnectionForKey 方法](http://msdn.microsoft.com/zh-cn/library/azure/dn824099.aspx)，而不是使用传统方法来创建 Dapper 的连接。这可确保所有验证都会发生，并在分片之间移动任何数据时正确管理连接。

### Dapper 集成的要求

在使用弹性数据库客户端库和 Dapper API 时，我们希望保留以下属性：

* **向外缩放**：我们需要根据应用程序的容量需求，在分片应用程序的数据层中添加或删除数据库。 

-    **一致性**：由于应用程序是使用分片向外缩放的，因此我们需要执行数据相关的路由。我们需要使用库的数据相关路由功能来实现此目的。具体而言，我们需要保留验证，以避免损坏或错误的查询结果情况下通过分片映射管理器中转连接提供的一致性保证。这可确保（举例而言）当前已使用 Split/Merge API 将 shardlet 移至其他分片时，拒绝或停止与给定 shardlet 的连接。

-    **对象映射**：我们需要保留 Dapper 为了在应用程序中的类和基础数据库结构之间进行转换而提供的映射方便性。

以下部分基于 **Dapper** 和 **DapperExtensions** 的应用程序的这些要求的指南。

## 技术指南
### 数据相关的路由与 Dapper 

使用 Dapper 时，应用程序通常负责创建和打开与基础数据库的连接。如果应用程序指定了类型 T，则 Dapper 将查询结果返回为 T 类型的 .NET 集合。Dapper 执行从 T-SQL 结果行到 T 类型对象的映射。同样，Dapper 将 .NET 对象映射到数据操作语言 (DML) 语句的 SQL 值或参数。Dapper 通过扩展方法的常规 [SqlConnection](http://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.aspx) ADO.NET SQL 客户端库中的对象提供此功能。DDR 弹性缩放 API 返回的 SQL 连接也是常规 [SqlConnection](http://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.aspx) 对象。这样，我们便可以针对客户端库的 DDR API 返回的类型直接使用 Dapper 扩展，因为它也是一个简单的 SQL 客户端连接。

根据这些规则，可以方便地使用 Dapper 的弹性数据库客户端库中转的连接。

此代码示例（摘自随附的示例）演示了应用程序在库中提供分片键，以将连接中转到正确分片的方案。

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                     key: tenantId1, 
                     connectionString: connStrBldr.ConnectionString, 
                     options: ConnectionOptions.Validate))
    {
        var blog = new Blog { Name = name };
        sqlconn.Execute(@"
                      INSERT INTO
                            Blog (Name)
                            VALUES (@name)", new { name = blog.Name }
                        );
    }

调用 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) API 会替换 SQL 客户端连接的默认创建和打开方法。[OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) 调用采用了进行数据相关路由所需的参数：

-    用于访问数据相关路由接口的分片映射
-    用于标识 shardlet 的分片键
-    用于连接分片的凭据（用户名和密码）

分片映射对象将与保存给定分片键 shardlet 的分片建立连接。弹性数据库客户端 API 还会标记连接以实现一致性保证。由于调用 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) 会返回一个常规 SQL 客户端连接对象，因此从 Dapper 后续调用 **Execute** 扩展方法遵循标准的 Dapper 做法。

查询的工作方式非常类似 - 首先从客户端 API 使用 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) 打开连接。然后，可以使用常规 Dapper 扩展方法将 SQL 查询的结果映射到 .NET 对象：

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId1, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate ))
    {    
           // Display all Blogs for tenant 1
           IEnumerable<Blog> result = sqlconn.Query<Blog>(@"
                                SELECT * 
                                FROM Blog
                                ORDER BY Name");

           Console.WriteLine("All blogs for tenant id {0}:", tenantId1);
           foreach (var item in result)
           {
                Console.WriteLine(item.Name);
            }
    }

请注意，将块与 DDR 连接一起**使用**会将块中的所有数据库操作划归到保存 tenantId1 的一个分片。该查询仅返回当前分片中存储的博客，而不是任何其他分片中存储的博客。

## 数据相关的路由与 Dapper 和 DapperExtensions

Dapper 随附了可以在开发数据库应用程序时提供更大方便性和从数据库抽象其他扩展的生态系统。DapperExtensions 就是一个示例。

在应用程序中使用 DapperExtensions 不会更改创建和管理数据库连接的方式。应用程序仍要负责打开连接，并且扩展方法要求使用常规 SQL 客户端连接对象。我们可以依赖于上述 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx)。如以下代码示例所示，唯一的变化是我们不再需要编写 T-SQL 语句：

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId2, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate))
    {
           var blog = new Blog { Name = name2 };
           sqlconn.Insert(blog);
    }

下面是查询的代码示例：

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId2, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate))
    {
           // Display all Blogs for tenant 2
           IEnumerable<Blog> result = sqlconn.GetList<Blog>();
           Console.WriteLine("All blogs for tenant id {0}:", tenantId2);
           foreach (var item in result)
           {
               Console.WriteLine(item.Name);
           }
    }

### 处理暂时性故障

Microsoft 模式和实践团队发布了[暂时性故障处理应用程序块](http://msdn.microsoft.com/zh-cn/library/hh680934.aspx)，以帮助应用程序开发人员消除在云中运行应用程序时遇到的常见暂时性故障状态。有关详细信息，请参阅[锲而不舍是一切成功的秘密：使用暂时性故障处理应用程序块](http://msdn.microsoft.com/zh-cn/library/dn440719.aspx)。

该代码示例依赖于暂时性故障库来防止暂时性故障。

    SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
    {
       using (SqlConnection sqlconn = 
          shardingLayer.ShardMap.OpenConnectionForKey(tenantId2, connStrBldr.ConnectionString, ConnectionOptions.Validate))
          {
              var blog = new Blog { Name = name2 };
              sqlconn.Insert(blog);
          }
    });

上述代码中的 **SqlDatabaseUtils.SqlRetryPolicy** 定义为 **SqlDatabaseTransientErrorDetectionStrategy**，重试计数为 10，每两次重试的等待时间为 5 秒。如果你正在使用事务，请确保在出现暂时性故障的情况下重试范围可以恢复为事务开始时间。

## 限制

本文档中概述的方法存在一些限制：

* 本文档示例代码未演示如何管理不同分片的架构。
* 对于给定的请求，我们假设它的所有数据库处理都包含在该请求提供的分片键标识的单个分片内。但是，这种假设并不总是合理，例如，在无法使用某个分片键的情况下。为了解决此问题，弹性数据库客户端库包含了 [MultiShardQuery 类](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardexception.aspx)。该类实现了一个连接抽象用于查询多个分片。MultiShardQuery 与 Dapper 的结合使用超出了本文档的讨论范围。

## 结束语

使用 Dapper 和 DapperExtensions 的应用程序很容易从 Azure SQL 数据库的弹性数据库工具受益。通过本文档中所述的步骤，这些应用程序可以使用该工具的功能，通过将新 [SqlConnection](http://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.aspx) 对象的创建和打开方式更改为使用弹性数据库客户端库的 [OpenConnectionForKey](http://msdn.microsoft.com/zh-cn/library/azure/dn807226.aspx) 调用，来实现数据相关的路由。这会将应用程序更改限制为已创建和打开新连接的位置。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-working-with-dapper/dapperimage1.png
 

<!---HONumber=Mooncake_1221_2015-->
