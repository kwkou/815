<properties
    title="Azure SQL Database elastic database query overview"
    pageTitle="Azure SQL 数据库弹性数据库查询概述"
    description="弹性查询功能概述"
	metaKeywords="azure sql 数据库 elastic database queries"
    services="sql-database"
    documentationCenter=""  
    manager="jeffreyg"
    authors="sidneyh"/>

<tags
    ms.service="sql-database"
    ms.date="07/09/2015"
    wacn.date="09/15/2015" />

# 弹性数据库查询 – 2015 年 5 月预览版

弹性数据库查询是 Azure SQL 数据库 在 2015 年 5 月后期推出的一项新预览版功能。随着此功能的引入，用户可以使用与 Azure SQL 数据库 建立的单个连接，跨多个数据库运行 T-SQL 查询。这样，使用弹性数据库分片映射定义的分片数据集便会显示为单个集成式数据存储。用于报告和数据集成目的的所有分布式查询处理将在幕后完成。可以使用能够与 SQL 数据库 通信的任何驱动程序（包括 ADO.Net、ODBC、JDBC、Node.js、PHP 等）来支持连接。

## 弹性数据库查询方案

弹性数据库查询的目标是简化整个分片数据层的报告方案，其中，多个数据库或分片的行参与 T-SQL 查询的单个整体结果。T-SQL 查询可以由用户或应用程序直接撰写，也可以通过与全局查询弹性查询数据库连接的工具间接撰写。对于商业 BI、报告或数据集成工具，以及无法直接使用弹性缩放库扩展的打包软件而言，此查询特别有用。它还可让你通过 SQL Server Management Studio 或 Visual Studio 发出的查询轻松访问整个分片数据集合，并支持从实体框架或其他 ORM 环境访问透明的多分片数据。图 1 演示了此方案：全局查询弹性 DB 查询提供了另一个不同的途径用于将查询提交到分片数据层以及现有的云应用程序（可能使用了弹性缩放库）。

![弹性数据库查询][1]

可以使用以下拓扑来特征化方案：

-	分片数据层（拓扑 1）：数据层用于分片。使用 (1) 弹性数据库客户端库或 (2) 自我分片来执行和管理分片。报告用例用于对许多分片计算报告，弹性数据库客户端库中的多分片查询无法满足这些分片的功能要求和连接要求。

-	多数据库设计（拓扑 2）：在这里，数据层已垂直分区，以便不同类型的数据驻留在不同的数据库上。报告用例用于对分布在多个数据库中，但必须集成到报告的一个总体结果中的数据计算报告。

弹性数据库查询预览版将涵盖这两种用例。使用拓扑 1 的客户如果正在使用弹性数据库客户端库管理分片，则可以依赖于现有的分片映射。自我分片用户若要使用此预览版功能，需要使用弹性数据库工具为其数据层创建分片映射。使用拓扑 2 的客户需要为每个数据库创建不同的分片映射。这些分片映射各自指向单个分片，这个分片随后可以为跨分片查询公开其表。

## 后续步骤
弹性数据库查询计划在 2015 年 5 月底推出预览版。到时请回来查看此页，以了解有关如何使用该功能的详细信息和分步指导。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-overview/overview.png
<!--anchors-->

<!---HONumber=69-->