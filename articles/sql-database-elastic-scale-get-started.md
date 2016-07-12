<properties 
	pageTitle="弹性数据库工具入门" 
	description="大致介绍 Azure SQL 数据库的弹性数据库工具功能，包括易于使用的示例应用。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jhubbard" 
	authors="ddove" 
	editor="sidneyh"/>

<tags 
	ms.service="sql-database" 
	ms.date="03/22/2016" 
	wacn.date="05/16/2016"/>

# 弹性数据库工具入门

本文介绍开发人员运行示例应用时的体验。此示例将创建一个简单的分片应用程序，并探讨弹性数据库工具的主要功能。此示例演示[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)的功能

## 先决条件

1. 必须安装 C# 版 Visual Studio 2012 或更高版本。可以从 [Visual Studio 下载](http://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)页面下载免费版本。
2. Nuget 2.7 或更高版本。若要获取最新版本，请参阅[安装 NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)

## 下载并运行示例应用

**支持 Azure SQL 的弹性数据库 - 入门**示例应用程序演示了使用 Azure SQL 弹性数据库工具进行分片应用程序开发的最重要体验方面。它注重于[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)、[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)和[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)的主要用例。若要下载并运行该示例，请遵循以下步骤：

1. 打开 Visual Studio 并选择“文件”->“新建”->“项目”。
2. 在对话框中，单击“联机”。

    ![“新建项目”>“联机”][2]
3. 然后单击“示例”下面的“Visual C#”。

    ![单击“Visual C#”][3]
4. 在搜索框中，键入 **elastic db** 以搜索示例。随后将出现标题 **Azure SQL 弹性数据库工具 - 入门**。

    ![搜索框][1]
 
5. 选择该示例，选择新项目的名称和位置，然后按“确定”创建该项目。
6. 在示例项目的解决方案中打开 **app.config** 文件，然后遵循该文件中的说明添加 Azure SQL 数据库服务器名称和你的登录信息（用户名和密码）。
7. 构建并运行应用程序。当出现提示时，请允许 Visual Studio 还原该解决方案的 NuGet 包。这将会从 NuGet 下载最新版本的弹性数据库客户端库。
8. 尝试使用不同的选项，以了解有关弹性缩放功能的详细信息。请注意应用程序在控制台输出中执行的步骤，并随意浏览后台代码。

    ![进度][4]

祝贺你 - 你已成功地使用弹性数据库池在 Azure SQL 数据库上生成并运行了第一个分片应用程序。通过将 Visual Studio 或 SQL Server Management Studio 连接到 Azure DB 服务器，快速查看一下该示例创建的分片。你将会看到该示例创建的新示例分片数据库和分片映射管理器数据库。

**注意：**如果你没有 SQL Server Management Studio，请参阅[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms/)，其中提供了有关获取该工具的说明。

### 重要的代码示例片段

1. **管理分片和分片映射：**该代码演示如何在文件 **ShardMapManagerSample.cs** 中处理分片、范围和映射。你可以在以下位置找到有关此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。  
2. **数据相关的路由**：**DataDependentRoutingSample.cs** 中演示了如何向正确的分片路由事务。有关详细信息，请参阅[数据相关的路由](http://go.microsoft.com/?linkid=9862596)。 
3. **查询多个分片**：文件 **MultiShardQuerySample.cs** 中演示了如何查询多个分片。有关详细信息，请参阅[多分片查询](http://go.microsoft.com/?linkid=9862597)。
4. **添加空分片**：文件 **AddNewShardsSample.cs** 中的代码以迭代方式添加新的空分片。以下位置提供了此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

### 其他弹性缩放操作

1. **拆分现有分片**：拆分分片的功能是通过**拆分/合并工具**提供的。可在以下位置找到有关此服务的详细信息：[拆分/合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。
2. **合并现有分片**：分片合并也是使用**拆分/合并工具**执行的。有关详细信息，请参阅：[拆分/合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。   


## 成本

弹性数据库工具是免费的。弹性数据库工具不会在 Azure 使用费的基础上施加额外的费用。

例如，示例应用程序会创建新的数据库。这种费用取决于你选择的 Azure SQL DB 数据库版本，以及你的应用程序的 Azure 使用情况。

有关价格信息，请参阅 [SQL 数据库定价详细信息](/home/features/sql-database/pricing/)。

## 后续步骤
有关弹性数据库工具的详细信息，请参阅：

* [弹性数据库工具文档结构图](https://azure.microsoft.com/documentation/learning-paths/sql-database-elastic-scale) 
-    代码示例： 
    -    [Azure SQL 弹性数据库 - 入门](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-a80d8dc6?SRC=VSIDE)
    -    [Azure SQL 弹性数据库 - 与实体框架集成](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE)
    -    [脚本中心上的分片弹性](https://gallery.technet.microsoft.com/scriptcenter/Elastic-Scale-Shard-c9530cbe)
-    博客：[弹性缩放产品通告](https://azure.microsoft.com/blog/2014/10/02/introducing-elastic-scale-preview-for-azure-sql-database)
-    论坛：[Azure SQL 数据库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=ssdsgetstarted)


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
 

<!---HONumber=Mooncake_0509_2016-->
