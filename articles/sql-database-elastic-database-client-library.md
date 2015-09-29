<properties
    pageTitle="Azure SQL 数据库 - 客户端库"
    description="生成可缩放的 .NET 数据库应用"
    services="sql-database"
    documentationCenter=""
    manager="jeffreyg"
    authors="sidneyh"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="07/29/2015"
    wacn.date="09/15/2015"/>

# 弹性数据库客户端库概述

**弹性数据库客户端库**可帮助你使用 Windows Azure 上托管的数百甚至数千个 Azure SQL 数据库轻松开发分片应用程序。此类设计通常用于软件即服务 (SaaS) 应用程序，这种应用程序往往采用单租户体系结构 -- 其中的每个租户是使用数据库设置的。生成和管理此类应用程序就是该库的目标。客户端库是一种 .NET Framework 库，可以使用 [Visual Studio](/documentation/articles/sql-database-elastic-scale-add-references-visual-studio) 和 [Nuget](http://go.microsoft.com/?linkid=9862605) 将它安装到任何应用程序项目。客户端库属于弹性数据库工具，具体而言，是一项[弹性数据库功能](/documentation/articles/sql-database-elastic-scale-introduction)。

## 客户端功能

对于开发人员和管理员来说，使用*分片*（下面将会介绍）在开发、扩展和管理向外扩展的应用程序时都存在挑战。客户端库减轻松这两个角色的负担。下图描绘了弹性数据库客户端库提供的主要功能。图中演示了一个包含许多数据库的环境，而每个数据库对应于一个分片。尽管每个客户（租户）有一个数据库的情况与此相同，但是在此示例中，许多客户使用范围映射共置于同一数据库中。这些工具通过以下特定功能，使开发分片 Azure SQL 数据库应用程序变得更轻松：

关于本文使用的术语定义，请参阅[弹性数据库工具词汇表](/documentation/articles/sql-database-elastic-scale-glossary)。

![弹性缩放功能][1]

1.  **分片映射管理**：为管理分片的集合创建称为“分片映射管理器”的特殊数据库。分片映射管理是使应用程序能够管理有关其分片的各种元数据的功能。开发人员可使用此功能将数据库注册为分片（描述各个分片键或键范围到这些数据库的映射），并随着数据库的数量和组成发展来维护此元数据，以反映容量更改。如果不使用弹性数据库客户端库，实现分片时你必须花费大量时间来编写管理代码。有关详细信息，请参阅[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management)。

* **数据相关的路由**：假设将一个请求传入应用程序。基于该请求的分片键值，应用程序需要确定为此分片键值保存数据的正确分片，然后打开到此分片的连接以处理该请求。借助数据依赖路由，您能够通过对应用程序的分片映射的单个简单调用打开连接。数据相关的路由是基础结构代码的另一个区域，现在它由弹性数据库客户端库中的功能所取代。有关详细信息，请参阅[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)。

* **多分片查询 (MSQ)**：当一个请求涉及多个（或所有）分片时，多分片查询将生效。多分片查询在所有分片或一组分片上执行相同的 T-SQL 代码。使用 UNION ALL 语义，将参与分片中的结果合并到一个总结果集中。该功能是通过该客户端库处理多个任务公开的，其中包括连接管理、线程管理、故障处理和中间结果处理。MSQ 最多可以查询数百个分片。有关详细信息，请参阅[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying)。

一般而言，当使用弹性数据库的客户提交具有其自己的语义的局部分片操作（与跨分片操作相对）时，预期可获取完整的 T-SQL 功能。

## 后续步骤

尝试运行用于演示客户端功能的[示例应用](/documentation/articles/sql-database-elastic-scale-get-started)。

若要安装该库，请转到[弹性数据库客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。

有关使用拆分/合并工具的说明，请参阅[拆分/合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge)。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]: ./media/sql-database-elastic-database-client-library/glossary.png

<!---HONumber=69-->