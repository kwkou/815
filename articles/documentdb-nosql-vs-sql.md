<properties
	pageTitle="何时使用 NoSQL 与 SQL | Azure"
	description="比较使用非关系 NoSQL 解决方案与使用 SQL 解决方案的好处。了解 Microsoft Azure NoSQL 服务还是 SQL Server 解决方案最适合你的方案。"
	keywords="nosql 与 sql, 何时使用 NoSQL, sql 与 nosql"
	services="documentdb"
	documentationCenter=""
	authors="mimig1"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="documentdb"
	ms.date="03/28/2016"
	wacn.date="06/29/2016"/>

# NoSQL 与 SQL

SQL Server 和关系数据库 (RDBMS) 成为重要专业数据库已经 20 多年了。但是，处理更高容量和各种数据的快速增长需求改变了应用程序开发人员对数据存储存储需求的性质。为了实现这种需求，能够大规模存储非结构化和异类数据的 NoSQL 数据库得到了普及。

NoSQL 是一种与 SQL 数据库截然不同的数据库。NoSQL 常用来指代“非 SQL”的数据管理系统，或者指代“不仅限于 SQL”的数据管理方法。NoSQL 类别中有多种技术，包括文档数据库、键值存储、列系列存储和图形数据库，这些技术在游戏、社交和 IoT 应用中非常流行。

![显示常用方案和数据模型的 NoSQL 与 SQL 概述关系图](./media/documentdb-nosql-vs-sql/nosql-vs-sql-overview.png)

本文旨在帮助你了解 SQL、NoSQL 之间的区别，并为你提供来自 Microsoft 的 NoSQL 和 SQL 产品简介。

## 何时使用 NoSQL？

假设你正在构建新的社交网站。用户可以创建帖子，并向其中添加图片、视频和音乐。其他用户可以对帖子发表评论，并对帖子打分（或类似）进行评级。登录页将提供一些帖子，用户可以进行共享和互动。

那么，如何存储此数据？ 如果你熟悉 SQL，你可能会开始绘制类似如下的内容：

![显示社交网站关系数据模型的 NoSQL 与 SQL 关系图](./media/documentdb-nosql-vs-sql/nosql-vs-sql-social.png)

目前为止，一切都好，但现在想想单个帖子的结构以及如何将其显示。如果你想要在网站或应用程序上显示帖子和关联的图像、音频、视频、评论、分数以及用户信息，你将必须使用八表联接执行查询，只是为了检索内容。现在，假设帖子流动态加载并出现在屏幕上，并且你可轻松地预测它将需要数千个查询和多个联接来完成任务。

现可使用类似于 SQL Server 的关系解决方案来存储数据 - 但还有其他选项，一种将此方法简化了的 NoSQL 选项。通过将帖子转换为与以下内容类似的 JSON 文档并将其存储在 DocumentDB（一个 Azure NoSQL 文档数据库服务）中，可以提高性能并使用一个查询，而不需要连接就可检索整个帖子。它更简单、更直观，并且性能更高。

    {
        "id":"ew12-res2-234e-544f",
        "title":"post title",
        "date":"2016-01-01",
        "body":"this is an awesome post stored on NoSQL",
        "createdBy":User,
        "images":["http://myfirstimage.png","http://mysecondimage.png"],
        "videos":[
            {"url":"http://myfirstvideo.mp4", "title":"The first video"},
            {"url":"http://mysecondvideo.mp4", "title":"The second video"}
        ],
        "audios":[
            {"url":"http://myfirstaudio.mp3", "title":"The first audio"},
            {"url":"http://mysecondaudio.mp3", "title":"The second audio"}
        ]
    }

此外，此数据可以按帖子 ID 进行分区，从而允许数据自然扩展并利用 NoSQL 缩放特征。此外，NoSQL 系统使开发人员能够放宽一致性，并提供高度可用的应用。最后，此解决方案不需要开发人员在数据层定义、管理和维护架构，实现快速迭代。

然后你可以使用其他 Azure 服务生成此解决方案：

- [Azure App Services](/documentation/services/web-sites/) 可用来托管应用程序和后台进程。
- [Azure Blob 存储](/documentation/services/storage/)可用来存储包括图像的完整的用户配置文件。
- [Azure SQL 数据库](/documentation/services/sql-databases/)可用来存储大量数据，例如登录信息和使用情况分析数据。


此社交网站只是 NoSQL 数据库是针对作业的适当数据模型的其中一种方案。如果你对阅读有关此方案以及如何在社交媒体应用程序中对 DocumentDB 的数据建模的详细信息感兴趣，请参阅 [Going social with DocumentDB（使用 DocumentDB 进行社交）](/documentation/articles/documentdb-social-media-apps/)。

## NoSQL 与 SQL 比较

下表比较了 NoSQL 和 SQL 的主要区别。

![显示何时使用 NoSQL 以及何时使用 SQL 的 NoSQL 与 SQL 关系图。SQL 与 NoSQL 比较](./media/documentdb-nosql-vs-sql/nosql-vs-sql-comparison.png)

如果 NoSQL 数据库最适合你的要求，继续进行下一部分，了解有关 Azure 中可用 NoSQL 服务的详细信息。相反，如果 SQL 数据库最适合你的要求，请跳到 [What are the Microsoft SQL offerings?（Microsoft SQL 产品/服务有哪些？）](#what-are-the-microsoft-sql-offerings)

## Microsoft Azure NoSQL 产品/服务有哪些？

Azure 具有以下四种完全托管的 NoSQL 服务：

- [Azure DocumentDB](/services/documentdb/)
- [Azure 表存储](/services/storage/)
- [作为 HDInsight 一部分的 Azure HBase](/services/hdinsight/)
- [Azure Redis Cache](/documentation/services/redis-cache/)

下面的比较图表详细说明了每种服务的关键差异。哪个最能准确描述你应用程序的需求？

![显示何时使用 Microsoft Azure 中 NoSQL 产品/服务的 NoSQL 与 SQL 关系图，包括 DocumentDB、表存储、作为 HDInsight 一部分的 HBase 和 Redis 缓存](./media/documentdb-nosql-vs-sql/nosql-vs-sql-documentdb-storage-hbase-hdinsight-redis-cache.png)

如果这些服务中的一个或多个可能满足你应用程序的需要，可使用以下资源了解详细信息：

- [Azure 表存储入门](/documentation/articles/storage-dotnet-how-to-use-tables/)
- [HDInsight 中的 HBase 是什么](/documentation/articles/hdinsight-hbase-overview/)


然后转到[后续步骤](#next-steps)，获取免费试用版信息。

## <a name="what-are-the-microsoft-sql-offerings"></a> Microsoft SQL 产品/服务有哪些？

Microsoft 提供了五种 SQL 产品/服务：

- [Azure SQL 数据库](/services/sql-databases/)
- [SQL Server](https://www.microsoft.com/server-cloud/products/sql-server-2016/)
- [Azure SQL 数据仓库（预览版）](/documentation/services/sql-data-warehouse)
- [分析平台系统（本地设备）](https://www.microsoft.com/zh-cn/server-cloud/products/analytics-platform-system/)

如果你对虚拟机上的 SQL Server 或 SQL 数据库感兴趣，则阅读 [选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)](/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas/)，了解两者区别的详细信息。

如果 SQL 听起来像是最佳选择，然后转到 [SQL Server](https://www.microsoft.com/server-cloud/products/) 了解有关 Microsoft SQL 产品和服务所提供内容的详细信息。

然后转到[后续步骤](#next-steps)，获取免费试用版和评估版的链接。

## <a name="next-steps"></a>后续步骤

我们邀请你通过免费试用 SQL 和 NoSQL，了解有关二者的详细信息。

- 对于所有 Azure 服务，你可以注册 [free one-month trial（一个月试用版）](/pricing/free-trial/)，并收到 200 美元专用在任何 Azure 服务上。
    - [Azure DocumentDB](/documentation/services/documentdb/)
    - [作为 HDInsight 一部分的 Azure HBase](/documentation/services/hdinsight/)
    - [Azure Redis Cache](/documentation/services/cache/)
    - [Azure SQL 数据仓库（预览版）](/documentation/services/sql-data-warehouse/)
    - [Azure SQL 数据库](/documentation/services/sql-database/)
    - [Azure 表存储](/documentation/services/storage/)

- 你可以注册[虚拟机上 SQL Server 2016 的评估版](/marketplace/partners/microsoft/sqlserver2016ctp33evaluationwindowsserver2012r2/)或下载 [SQL Server 评估版](/evalcenter/evaluate-sql-server-2016)。
    - [SQL Server](https://www.microsoft.com/server-cloud/products/sql-server-2016/)
    - [Azure 虚拟机中的 SQL Server](/services/virtual-machines/sql-server/)


<!---HONumber=Mooncake_0425_2016-->