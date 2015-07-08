<properties title="Get Started with  Azure SQL 数据库 Elastic Scale" pageTitle="Azure SQL 数据库 弹性缩放入门" description="大致介绍 Azure SQL 数据库 的弹性缩放功能，包括易于使用的示例应用程序。" metaKeywords="sharding scaling, Azure SQL DB sharding, elastic scale" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

<tags
   ms.service="sql-database"
   ms.date="02/03/2015"
   wacn.date="05/25/2015"/>

# Azure SQL 数据库 弹性缩放预览版入门

按需扩大和缩小容量是云计算应承的重要功能之一。从云应用程序的数据库层角度来讲，要兑现这个承诺在过去一直都很繁琐而复杂。近年来，本行业融合了一些称为分片的成熟设计模式。尽管一般的分片模式能够解决难题，但是，无论应用程序采用哪种业务逻辑，使用分片生成和管理应用程序都需要进行大量的基础结构投资。 

Azure SQL 数据库 弹性缩放（预览版）能使应用程序的数据层通过行业标准的分片实务进行扩展和收缩，同时还能大大简化分片云应用程序的开发和管理。弹性缩放通过一组 .Net 库和可以在你自己的 Azure 订阅中托管的 Azure 服务模板提供开发和管理功能，方便你管理高缩放性应用程序。Azure DB 弹性缩放实施的基础结构以分片技术为基础，因此你可以将重心转移到应用程序的业务逻辑上。 

本文档将介绍 Azure SQL DB 弹性缩放的开发人员体验。 

有关弹性缩放工作原理的详细信息，请参阅[弹性缩放概述](/documentation/articles/sql-database-elastic-scale-introduction)。

有关弹性缩放的所有主题列表，请参阅[弹性缩放文档结构图](/documentation/articles/sql-database-elastic-scale-documentation-map)

## 弹性缩放示例应用程序

此示例将创建一个简单的分片应用程序，并探讨弹性缩放的主要功能。若要下载并运行该应用程序，请遵循下面所示的步骤。 

### 先决条件
若要运行该示例应用程序，你必须使用 Visual Studio，并且必须有权访问 Azure 上运行的 Azure SQL 数据库。如果你没有 Azure 订阅，请注册[试用订阅](/pricing/1rmb-trial)。
#### Visual Studio 和 Nuget

1. 必须安装 C# 版 Visual Studio 2012 或更高版本。可以从 [Visual Studio 下载](http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx)下载免费版。
2. Nuget 2.7 或更高版本。若要获取最新版本，请参阅[安装 NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)
#### 创建 Azure SQL 数据库

* 参阅 [Windows Azure SQL 数据库 入门](/documentation/articles/sql-database-get-started)。

## 下载并运行示例应用程序
**Azure SQL 数据库 弹性缩放 - 入门**示例应用程序演示了使用 Azure SQL DB 弹性缩放进行分片应用程序开发的最重要体验方面。它注重于[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management)、[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)和[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying)的主要用例。若要下载并运行该示例，请遵循以下步骤： 

1. 打开 Visual Studio 并选择"文件"->"新建"->"项目"。
2. 在对话框中，单击"联机"。

    ![New Project>Online][2]
3. 然后单击"示例"下面的"Visual C#"。

    ![Click Visual C#][3]
4. 在搜索框中，键入**弹性缩放**以搜索示例。随后将出现标题 **Azure SQL 数据库 弹性缩放 - 入门**。

    ![Search Box][1]
 
5. 选择该示例，选择新项目的名称和位置，然后按"确定"创建该项目。
6. 在示例项目的解决方案中打开 **app.config** 文件，然后遵循该文件中的说明添加 Azure SQL 数据库 服务器名称和你的登录信息（用户名和密码）。
7. 生成并运行该应用程序。当出现提示时，请允许 Visual Studio 还原该解决方案的 NuGet 程序包。这将会从 NuGet 下载最新版本的弹性缩放客户端库。
8. 尝试使用不同的选项，以了解有关弹性缩放功能的详细信息。请注意应用程序在控制台输出中执行的步骤，并随意浏览后台代码。

    ![progress][4]

祝贺你 - 你已成功地在 Azure SQL DB 上生成并运行了第一个弹性缩放应用程序。通过将 SQL Server Management Studio 连接到 Azure DB 服务器，快速查看一下该示例创建的分片。你将会看到该示例创建的新示例分片数据库和分片映射管理器数据库。

**注意**   如果你没有 SQL Server Management Studio，请参阅[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/documentation/articles/sql-database-manage-azure-ssms)，其中提供了有关获取该工具的说明。

### 重要的代码示例片段

1. **管理分片和分片映射**：该代码演示如何使用文件 **ShardMapManagerSample.cs** 中的分片、范围和映射。你可以在以下位置找到有关此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management)。  
2. **数据相关的路由**：**DataDependentRoutingSample.cs** 中演示了如何向正确的分片路由事务。有关详细信息，请参阅[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)。 
3. **查询多个分片**：文件 **MultiShardQuerySample.cs** 中演示了如何查询多个分片。有关详细信息，请参阅[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying)。
4. **添加空分片**：文件 **AddNewShardsSample.cs** 中的代码以迭代方式添加新的空分片。以下位置提供了此主题的详细信息：[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management)。

### 其他弹性缩放操作

1. **拆分现有分片**：拆分分片的功能是通过**拆分/合并服务**提供的。可在以下位置找到有关此服务的详细信息：[拆分/合并服务](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge)。
2. **合并现有分片**：分片合并也是使用**拆分/合并服务**执行的。有关详细信息，请参阅：[拆分/合并服务](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge)。   


## 成本

弹性缩放库和服务模板是免费的。弹性缩放不会在 Azure 使用费的基础上施加额外的费用。 

例如，示例应用程序会创建新的数据库。这种成本取决于你选择的 Azure SQL DB 数据库版本，以及你的应用程序的 Azure 使用情况。

有关定价信息，请参阅 [SQL 数据库 定价详细信息](/home/features/sql-database/#price)。

## 后续步骤
有关弹性缩放功能的详细信息，请参阅：

* [弹性缩放学习页](/documentation/articles/sql-database-elastic-scale-documentation-map)
-    代码示例： 
    -    [Azure SQL 数据库 弹性缩放 - 入门](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-a80d8dc6?SRC=VSIDE)
    -    [Azure SQL 数据库 弹性缩放 - 与实体框架集成](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE)

-    博客：[弹性缩放产品宣告](http://go.microsoft.com/?linkid=9862608)



<!--Anchors-->
[弹性缩放示例应用程序]: #The-Elastic-Scale-Sample-Application
[下载并运行示例应用程序]: #Download-and-Run-the-Sample-App
[成本]: #Cost
[后续步骤]: #next-steps

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-get-started/newProject.png
[2]: ./media/sql-database-elastic-scale-get-started/click-online.png
[3]: ./media/sql-database-elastic-scale-get-started/click-CSharp.png
[4]: ./media/sql-database-elastic-scale-get-started/output2.png

<!--HONumber=55-->