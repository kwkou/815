<properties
   pageTitle="Azure SQL 数据库解决方案快速入门 | Azure"
   description="了解 Azure SQL 数据库解决方案"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="06/22/2016"
   wacn.date="08/15/2016"/>

# 浏览 Azure SQL 数据库解决方案快速入门

本文包含了对 Azure SQL 数据库解决方案快速入门的概述。这些快速入门教程演示了 SQL 数据库在基于实际方案的完整解决方案中的使用。有关演示特定 Azure SQL 数据库功能用法的简单分步教程，请参阅[探索 Azure SQL 数据库教程](/documentation/articles/sql-database-explore-tutorials/)。

## WingTipTickets 演示和动手实验

[Azure SQL 数据库 WingTipTickets](https://github.com/microsoft/wingtiptickets) 演示和动手实验 这些文件包括一个动手实验，演示一个基于 Azure SQL 数据库和 Azure 搜素的示例应用程序如何用于销售演唱会门票

## 跨多个池收集和监视资源使用情况数据

针对在一个订阅中跨多个池收集和监视 Azure SQL 数据库资源使用情况，本解决方案快速入门教程提供了解决方案。一个订阅中有大量数据库时，单独监视每个弹性池非常麻烦。要解决这个问题，可以将 SQL 数据库 PowerShell cmdlet 和 T-SQL 查询结合使用，从多个池及其数据库收集资源使用情况数据，以便监视和分析资源使用情况。

GitHub SQL Server 示例存储库中的[使用 PowerShell 和 Power BI 管理 SQL 数据库中的多个弹性池](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools)提供了一组 PowerShell 脚本和 T-SQL 查询，以及有关其作用和使用方法的文档。

## 开始在 SaaS 方案中使用弹性池

针对利用弹性池提供经济实惠的可缩放 SaaS 应用程序数据库后端的软件即解决方案 (SaaS) 方案，本解决方案快速入门教程提供了解决方案。在本解决方案中，你将演练 Web 应用的实现，这种实现让你能够使用补充 Azure 门户的自定义仪表板，对由负载生成器在弹性池上创建的负载进行可视化。

GitHub SQL Server 示例存储库中的 [Saas 的弹性池自定义仪表板](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools-custom-dashboard)提供了一个负载生成器和一个监视 Web 应用，以及有关其作用和使用方法的文档。

## 使用 Entity Framework 和 Code First 开发创建 Azure SQL 数据库

此视频和示例介绍了针对新数据库的 Code First 开发。此方案介绍如何针对 Code First 即将创建的暂不存在的数据库，或者 Code First 将向其添加新表的空数据库。Code First 允许使用 C# 或 VB.Net 类定义模型。也可以选择在类和属性上使用特性或使用 Fluent API 执行其他配置。请参阅[对新数据库使用 Code First](https://msdn.microsoft.com/zh-cn/data/jj193542.aspx)。

## 将弹性数据库工具集成到 Entity Framework 应用程序

此示例介绍与[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)集成所需在 Entity Framework 应用程序中进行的更改。重点是使用 Entity Framework Code First 方法撰写[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)和[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。[Code First - EF 的新数据库示例](http://msdn.microsoft.com/zh-cn/data/jj193542.aspx)在本示例中充当运行示例。本文档附带的示例代码是 Visual Studio 代码示例中弹性数据库工具示例的一部分。请参阅[将弹性数据库客户端库与 Entity Framework 配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/)。

## 具有弹性数据库工具和行级安全性的多租户应用程序

此示例介绍将[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)与[行级安全性](https://msdn.microsoft.com/zh-cn/library/dn765131)相集成所需对实体框架应用程序进行的更改。此示例将演示如何使用 ADO.NET SqlClient 和/或 Entity Framework ，同时运用这些技术来构建具有高度可伸缩性数据层、支持多租户分片的应用程序。此示例通过添加对多租户分片数据库的支持，扩展[将弹性数据库客户端库与 Entity Framework 配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/)。它将构建一个用于创建博客和文章的简单控制台应用程序，其中包含四个租户和两个多租户分片数据库。请参阅[使用弹性数据库工具和行级安全性的多租户应用程序](/documentation/articles/sql-database-elastic-tools-multi-tenant-row-level-security/)。

## Tailspin Surveys 应用程序

此示例是一个多租户 Web 应用程序，称为 Surveys。用户可以用其来创建在线调查。示例演示了在多租户应用程序中管理用户标识（包括注册、身份验证、授权和应用角色）时的一些关键注意事项。若要运行此示例，请参阅[如何运行 Tailspin Surveys 示例应用程序](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md)。

## 后续步骤

[浏览 Azure SQL 数据库教程](/documentation/articles/sql-database-explore-tutorials/)

<!---HONumber=Mooncake_0808_2016-->