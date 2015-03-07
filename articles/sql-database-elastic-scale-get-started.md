<properties title="Get Started with  Azure SQL Database Elastic Scale" pageTitle="Azure SQL Database 灵活扩展入门" description="Azure SQL Database 的灵活扩展功能的基本介绍，其中包括轻松运行简单的应用。" metaKeywords="sharding scaling, Azure SQL DB sharding, elastic scale" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

#Azure SQL Database 灵活扩展预览版入门

按需增加和缩小容量是主要的云计算约定之一。在过去，针对云应用程序的数据库层交付此约定一直非常枯燥且繁琐。在过去几年中，整个行业都趋向于使用完善的设计模式，通常称为分片。尽管常规分片模式解决了这一挑战，但构建和管理使用分片的应用程序时通常在应用程序的业务逻辑之外，还需要大量的基础结构投资。 

Azure SQL Database 灵活扩展（在预览版中）支持应用程序的数据层通过行业标准分片实践进行向外扩展和向内扩展，同时极大地简化已分片的云应用程序的开发和管理。灵活扩展提供了开发人员功能和管理功能，这些功能是通过一组 .Net 库和 Azure 服务模板提供的，您可以在自己的 Azure 订阅中对这些模板进行托管，以管理高度可扩展的应用程序。Azure DB 灵活扩展实现了分片的基础结构方面，使您能够专注于应用程序的业务逻辑。 

本文档向您介绍了 Azure SQL DB 灵活扩展的开发人员体验。 

有关灵活扩展的工作方式的详细信息，请参阅[灵活扩展概述](/zh-cn/documentation/articles/sql-database-elastic-scale-introduction/)。

有关灵活扩展的所有主题列表，请参阅[灵活扩展文档结构图](/zh-cn/documentation/articles/sql-database-elastic-scale-documentation-map/)

## 灵活扩展示例应用程序

该示例创建了一个简单的分片应用程序，并探讨了灵活扩展的主要功能。若要下载并运行该应用程序，请按照下面所示的步骤操作。 

###先决条件
若要运行示例应用程序，您必须使用 Visual Studio 并且必须有权访问在 Azure 上运行的 Azure SQL Database。如果您还没有 Azure 订阅，请注册一个[试用订阅](/pricing/1rmb-trial/)。
####Visual Studio 和 Nuget

1. 需要采用 C# 的 Visual Studio 2012 或更高版本。通过访问 [Visual Studio 下载](http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx)，下载免费版本。
2. Nuget 2.7 或更高版本。若要获取最新版本，请参阅[安装 NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)
####创建 Azure SQL 数据库

* 请参阅 [Microsoft Azure SQL Database 入门](/zh-cn/documentation/articles/sql-database-get-started/)。

##下载并运行示例应用程序
**Azure SQL Database 灵活扩展 - 入门**示例应用程序通过 Azure SQL DB 灵活扩展演示了分片应用程序的开发体验中最重要的方面。它重点介绍了[分片映射管理](/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/)、[数据依赖路由](/zh-cn/documentation/articles/sql-database-elastic-scale-data-dependent-routing/) 以及[多分片查询]的主要用例(/zh-cn/documentation/articles/sql-database-elastic-scale-multishard-querying/)。若要下载并运行示例，请按照以下步骤操作： 

1. 打开 Visual Studio，然后选择"文件"->"新建"->"项目"****。
2. 在对话框中，单击"联机"****。

    ![New Project>Online][2]
3. 然后，单击"示例"****下的"Visual C#"****。

    ![Click Visual C#][3]
4. 在搜索框中，键入**灵活扩展**以搜索示例。此时将显示标题 **Azure SQL Database 灵活扩展 - 入门**。

    ![Search Box][1]
 
5. 选择示例，为新项目选择名称和位置，然后按"确定"****以创建该项目。
6. 打开示例项目解决方案中的 **app.config** 文件，并按照该文件中的说明添加 Azure SQL Database 服务器名称以及您的登录信息（用户名和密码）。
7. 构建并运行应用程序。当出现系统提示时，请允许 Visual Studio 还原该解决方案的 NuGet 程序包。这将从 NuGet 下载最新版的灵活扩展客户端库。
8. 使用其他选项进行操作，以了解有关灵活扩展功能的详细信息。请注意应用程序在控制台输出中所采取的步骤，并随时浏览后台中的代码。

    ![progress][4]

恭喜 - 您已成功在 Azure SQL DB 上构建并运行第一个灵活扩展应用程序。快速查看通过将 SQL Server Management Studio 连接到您的 Azure DB 服务器所创建的示例的分片。您将注意到该示例创建的新示例分片数据库和分片映射管理器数据库。

**请注意**，如果您没有 SQL Server Management Studio，请参阅[使用 SQL Server Management Studio 管理 Azure SQL Database](/zh-cn/documentation/articles/documentation/articles/sql-database-manage-azure-ssms/)，其中包括有关获取工具的说明。  

### 代码示例的关键部分

1. **管理分片和分片映射**：该代码演示了如何在文件 **ShardMapManagerSample.cs** 中使用分片、范围和映射。您可以在以下位置找到有关该主题的详细信息：[分片映射管理](/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/).  
2. **数据依赖路由**：**DataDependentRoutingSample.cs** 中显示了如何将事务路由到正确的分片。有关详细信息，请参阅[数据依赖路由](/zh-cn/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。 
3. **对多个分片进行查询**：在文件 **MultiShardQuerySample.cs** 中介绍了如何跨分片进行查询。有关详细信息，请参阅[多分片查询](/zh-cn/documentation/articles/sql-database-elastic-scale-multishard-querying/)。
4. **添加空分片**：新的空分片的迭代添加由
文件 **AddNewShardsSample.cs** 中的代码执行。此处介绍了该主题的详细信息：[分片映射管理](/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

### 其他灵活扩展操作

1. **拆分现有分片**：使用**拆分/合并服务**可拆分分片。您可以在以下位置找到有关该服务的详细信息：[拆分/合并服务](/zh-cn/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。
2. **合并现有分片**：也可以使用**拆分/合并服务**执行分片合并。有关详细信息，请参考：[拆分/合并服务](/zh-cn/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。   


## 成本

灵活扩展库和服务模板是免费的。除您的 Azure 使用成本外，灵活扩展不会产生额外费用。 

例如，示例应用程序创建了新的数据库。成本取决于您选择的 Azure SQL DB 数据库版本以及您的应用程序的 Azure 使用情况。

有关定价信息，请参阅 [SQL Database 定价详细信息](/pricing/details/sql-database/)。

## 后续步骤
有关灵活扩展功能的详细信息，请参阅：

* [灵活扩展学习页面](/zh-cn/documentation/articles/sql-database-elastic-scale-documentation-map/)
-    代码示例： 
    -    [Azure SQL Database 灵活扩展 - 入门](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-a80d8dc6?SRC=VSIDE)
    -    [Azure SQL Database 灵活扩展 - 与实体框架集成](http://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE)

-    博客：[灵活扩展公告](http://go.microsoft.com/?linkid=9862608)



<!--Anchors-->
[灵活扩展示例应用程序]: #The-Elastic-Scale-Sample-Application
[下载并运行示例应用]: #Download-and-Run-the-Sample-App
[成本]: #Cost
[后续步骤]: #next-steps

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-get-started/newProject.png
[2]: ./media/sql-database-elastic-scale-get-started/click-online.png
[3]: ./media/sql-database-elastic-scale-get-started/click-CSharp.png
[4]: ./media/sql-database-elastic-scale-get-started/output2.png
