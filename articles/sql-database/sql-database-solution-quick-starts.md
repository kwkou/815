<properties
   pageTitle="Azure SQL 数据库解决方案快速入门 | Azure"
   description="了解 Azure SQL 数据库解决方案"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="sqldb-quickstart"
   ms.date="09/06/2016"
   wacn.date="12/26/2016"
   ms.author="carlrab"/>  


# 浏览 Azure SQL 数据库解决方案快速入门

本文包含了对 Azure SQL 数据库解决方案快速入门的概述。这些快速入门在 GitHub SQL Server 示例存储库中提供，根据实际方案演示 SQL 数据库在整个解决方案中的用法。有关演示特定 SQL 数据库功能用法的简单分步教程，请参阅[浏览 Azure SQL 数据库教程](/documentation/articles/sql-database-explore-tutorials/)。

## 尝试 WingTipTickets 演示和动手实验

[Azure SQL 数据库 WingTipTickets](https://github.com/microsoft/wingtiptickets) 演示和动手实验演示基于 Azure SQL 数据库和 Azure 搜索服务的、用于销售音乐会门票的示例应用程序。


## 跨多个池收集和监视资源使用情况数据

[Solution Quick Start: Elastic Pool telemetry using PowerShell](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools)（解决方案快速入门：使用 PowerShell 进行弹性池遥测）针对在一个订阅中跨多个池收集和监视 Azure SQL 数据库资源使用情况提供了解决方案。一个订阅中有大量数据库时，单独监视每个弹性池非常麻烦。

若要解决这个问题，可以将 SQL 数据库 PowerShell cmdlet 和 T-SQL 查询结合使用，从多个池及其数据库收集资源使用情况数据。这有助于更有效地监视和分析资源使用情况。

此快速入门提供一组 PowerShell 脚本和 T-SQL 查询，以及描述解决方案用途和实现方式的文档。

## 在 SaaS 方案中开始使用弹性数据库

 [Solution Quick Start: Elastic Pool custom dashboard for SaaS](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools-custom-dashboard)（解决方案快速入门：适用于 SaaS 的弹性池自定义仪表板）提供了适用于软件即解决方案 (SaaS) 场景的解决方案，可利用 SQL 数据库的弹性数据库功能，为 SaaS 应用程序提供符合成本效益且可缩放的数据库后端。

在此解决方案中，你将逐步完成 Web 应用的实现。使用此 Web 应用，可对由使用用于补充 Azure 门户的自定义仪表板的负载生成器在弹性数据库上创建的负载进行可视化，

此快速入门提供负载生成器和监视 Web 应用，以及描述应用用途和用法的文档。

## 使用 Code First 开发和 Entity Framework 创建 Azure SQL 数据库

[新数据库 Code First](https://msdn.microsoft.com/zh-cn/data/jj193542.aspx) 中的视频和示例介绍以新数据库为目标的 Code First 开发。此方案以尚不存在、但 Code First 将会创建的数据库为目标。或者，该方案创建空数据库，Code First 向其中添加新表。

Code First 允许使用 C# 或 Visual Basic .NET 类定义模型。可以在类和属性中使用特性或使用 Fluent API 执行其他可选配置。

## 将弹性数据库工具集成到 Entity Framework 应用程序

[将弹性数据库客户端库与实体框架配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/)示例演示要将 Entity Framework 应用程序与[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)集成而需要对该应用程序所做的更改。重点是使用 Entity Framework Code First 方法撰写[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)和[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。

[EF 新数据库示例 Code First](http://msdn.microsoft.com/zh-cn/data/jj193542.aspx) 在此示例中充当运行示例。此文档附带的示例代码是 Visual Studio 代码示例中弹性数据库工具示例的一部分。

## 将弹性数据库工具与行级别安全性集成

[使用弹性数据库工具和行级别安全性的多租户应用程序](/documentation/articles/sql-database-elastic-tools-multi-tenant-row-level-security/)演示要将[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)与[行级别安全性](https://msdn.microsoft.com/zh-cn/library/dn765131)集成而需要对 Entity Framework 应用程序做出的更改。此示例将演示如何同时运用这些技术来构建具有高度可伸缩性数据层、支持多租户分片的应用程序。

为此，可以使用 ADO.NET SqlClient 或 Entity Framework。此示例通过添加对多租户分片数据库的支持，扩展[将弹性数据库客户端库与 Entity Framework 配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/)。
它将构建一个用于创建博客和文章的简单控制台应用程序，其中包含四个租户和两个多租户分片数据库。

## 通过 Tailspin Surveys 应用程序创建在线调查表

此 [Tailspin Surveys 示例应用程序](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md)是一个多租户 Web 应用程序（称为 Surveys），可让用户创建在线调查表。该示例演示了在多租户应用程序中管理用户标识（包括注册、身份验证、授权和应用角色）时的一些重要注意事项。

## 通过 Contoso Clinic 演示应用程序了解 SQL 数据库的最新安全功能

这个 [Contoso Clinic 演示应用程序](https://github.com/Microsoft/azure-sql-security-sample)展示了 SQL 数据库最新的安全功能。

## 后续步骤

[浏览 Azure SQL 数据库教程](/documentation/articles/sql-database-explore-tutorials/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->