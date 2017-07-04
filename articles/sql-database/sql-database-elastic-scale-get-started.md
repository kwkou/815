<properties
    pageTitle="弹性数据库工具入门"
    description="大致介绍 Azure SQL 数据库的弹性数据库工具功能，包括易于使用的示例应用。"
    services="sql-database"
    documentationcenter=""
    manager="jhubbard"
    author="ddove"
    editor="CarlRabeler"
    translationtype="Human Translation" />
<tags
    ms.assetid="b6911f8d-2bae-4d04-9fa8-f79a3db7129d"
    ms.service="sql-database"
    ms.custom="multiple databases"
    ms.workload="sql-database"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="ddove"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="bb83af56b692771d3154d8e868780fc65ae5f03a"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-with-elastic-database-tools"></a>弹性数据库工具入门
本文介绍开发人员运行示例应用时的体验。 此示例将创建一个简单的分片应用程序，并探讨弹性数据库工具的主要功能。 此示例演示[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)的功能

若要安装该库，请转到 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。 该库和下述示例应用一起安装。

## <a name="prerequisites"></a>先决条件
1. 必须安装 C# 版 Visual Studio 2012 或更高版本。 可以从 [Visual Studio 下载页面](https://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx)下载免费版本。
2. NuGet 2.7 或更高版本。 若要获取最新版本，请参阅 [安装 NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)

## <a name="download-and-run-the-sample-app"></a>下载并运行示例应用
**支持 Azure SQL 的弹性数据库 - 入门**示例应用程序演示了使用 Azure SQL 弹性数据库工具进行分片应用程序开发的最重要体验方面。 它注重于[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)、[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)和[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)的主要用例。 若要下载并运行该示例，请遵循以下步骤： 

1. 从 MSDN 下载 [Elastic DB Tools for Azure SQL - Getting Started sample](https://code.msdn.microsoft.com/windowsapps/Elastic-Scale-with-Azure-a80d8dc6)（Azure SQL 弹性数据库工具 - 入门示例）。 将示例解压缩到所选位置。
2. 从 **C#** 目录打开 **ElasticScaleStarterKit.sln** 解决方案，以便创建项目。
3. 在示例项目的解决方案中打开 **app.config** 文件，然后遵循该文件中的说明添加 Azure SQL 数据库服务器名称和你的登录信息（用户名和密码）。
4. 构建并运行应用程序。 出现提示时，请允许 Visual Studio 还原该解决方案的 NuGet 包。 这会从 NuGet 下载最新版本的弹性数据库客户端库。 注意：请确保[配置数据库防火墙](/documentation/articles/sql-database-configure-firewall-settings/)以允许客户端连接，否则在运行应用时可能会遇到连接错误。
5. 尝试使用不同的选项，以了解有关客户端库功能的详细信息。 请注意应用程序在控制台输出中执行的步骤，并随意浏览后台代码。
   
    ![进度][4]

恭喜 - 你已成功使用弹性数据库池在 Azure SQL 数据库上生成并运行了第一个分片应用程序。 通过将 Visual Studio 或 SQL Server Management Studio 连接到 Azure DB 服务器，快速查看一下该示例创建的分片。 你会看到该示例创建的新示例分片数据库和分片映射管理器数据库。

> [AZURE.IMPORTANT]
>建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。
> 
> 

### <a name="key-pieces-of-the-code-sample"></a>重要的代码示例片段
1. **管理分片和分片映射**：该代码演示如何在文件 **ShardManagementUtils.cs** 中处理分片、范围和映射。 你可以在以下位置找到有关此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。  
2. **数据依赖型路由**：**DataDependentRoutingSample.cs** 中演示了如何向正确的分片路由事务。 有关详细信息，请参阅[数据依赖型路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。 
3. **查询多个分片**：文件 **MultiShardQuerySample.cs** 中演示了如何查询多个分片。 有关详细信息，请参阅[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)。
4. **添加空分片**：文件 **CreateShardSample.cs** 中的代码以迭代方式添加新的空分片。 以下位置提供了此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

### <a name="other-elastic-scale-operations"></a>其他弹性缩放操作
1. **拆分现有分片**：拆分分片的功能是通过**拆分/工具**提供的。 可在以下位置找到有关此服务的详细信息：[拆分 / 合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。
2. **合并现有分片**：分片合并也是使用**拆分 / 合并工具**执行的。 有关详细信息，请参阅：[拆分 / 合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。   

## <a name="cost"></a>成本
弹性数据库工具是免费的。 弹性数据库工具不会在 Azure 使用费的基础上增加额外的费用。 

例如，示例应用程序会创建新的数据库。 这种费用取决于所选择的 Azure SQL DB 数据库版本，以及应用程序的 Azure 使用情况。

有关价格信息，请参阅 [SQL 数据库定价详细信息](/pricing/details/sql-database/)。

## <a name="next-steps"></a>后续步骤
有关弹性数据库工具的详细信息，请参阅：


* 代码示例： 
  * [Azure SQL 弹性数据库 - 入门](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-a80d8dc6?SRC=VSIDE)
  * [Azure SQL 弹性数据库 - 与 Entity Framework 集成](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE)
  * [脚本中心上的分片弹性](https://gallery.technet.microsoft.com/scriptcenter/Elastic-Scale-Shard-c9530cbe)
* 博客：[弹性缩放产品通告](https://azure.microsoft.com/zh-cn/blog/2014/10/02/introducing-elastic-scale-preview-for-azure-sql-database/)
* Microsoft 虚拟大学：[使用分片和弹性数据库客户端库实现向外扩展视频](https://mva.microsoft.com/en-US/training-courses/elastic-database-capabilities-with-azure-sql-db-16554?l=lWyQhF1fC_6306218965) 
* 第 9 频道： [弹性缩放概述视频](http://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Elastic-Scale)
* 要测量性能：[分片映射管理器的性能计数器](/documentation/articles/sql-database-elastic-database-client-library/)

<!--Anchors-->
[The Elastic Scale Sample Application]: #The-Elastic-Scale-Sample-Application
[Download and Run the Sample App]: #Download-and-Run-the-Sample-App
[Cost]: #Cost
[Next steps]: #next-steps

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-get-started/newProject.png
[2]: ./media/sql-database-elastic-scale-get-started/click-online.png
[3]: ./media/sql-database-elastic-scale-get-started/click-CSharp.png
[4]: ./media/sql-database-elastic-scale-get-started/output2.png
<!--Update_Description: update latest sample information-->
