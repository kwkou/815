<properties
	pageTitle="什么是 SQL 数据库？ SQL 数据库简介 | Azure"
	description="获取 SQL 数据库简介：Microsoft 在云中的关系数据库管理系统 (RDBMS) 的技术详细信息与功能。"
	keywords="sql 简介,什么是 sql 数据库,DTU"
	services="sql-database"
	documentationCenter=""
	authors="shontnew"
	manager="jeffreyg"
	editor="cgronlun"/>

<tags
   ms.service="sql-database"
   ms.date="12/22/2015"
   wacn.date="02/26/2016"/>

# 什么是 SQL 数据库？ SQL 数据库简介、技术细节以及 DTU 说明

SQL 数据库是云中的关系数据库服务，它基于行业领先的 Microsoft SQL Server 引擎并提供任务关键型功能。SQL 数据库提供可预测的性能、在不停机的情况下进行缩放、业务连续性和数据保护以及近乎实时的管理功能。你可以将注意力集中在如何快速进行应用开发、加快推向市场，而不是管理虚拟机和基础结构上。由于 SQL 数据库基于 [SQL Server](https://msdn.microsoft.com/zh-cn/library/bb545450.aspx) 引擎，因此可以支持现有的 SQL Server 工具、库和 API，方便你迁移和扩展到云中。

本文将介绍 SQL 数据库在性能、缩放性和易管理性方面的核心概念与功能，并提供链接让你进一步了解详细信息。如果你已准备就绪，可以立即[创建第一个 SQL 数据库](/documentation/articles/sql-database-get-started)，或者[创建弹性数据库池](/documentation/articles/sql-database-elastic-pool-portal)。

## 无需停机即可调整性能和规模
SQL 数据库采用基本、标准和高级*服务层*。每个服务层提供[不同级别的性能和功能](/documentation/articles/sql-database-service-tiers)，以支持从轻型到重型的数据库工作负荷。你可以在小型数据库中构建第一个应用，每个月只需花费少量的资金。然后在你的应用受到广泛欢迎之后，随时手动或以编程方式[更改服务层](/documentation/articles/sql-database-scale-up)，这不会给你的应用或客户造成停机。

许多业务和应用只要能够创建数据库并按需调高或调低单一数据库的性能即可，尤其是当使用模式相对容易预测时。但如果有无法预测的使用模式，则管理成本和业务模式就会变得相当困难。

SQL 数据库中的[弹性数据库池](/documentation/articles/sql-database-elastic-pool)可解决此问题。概念很简单。你可以向池分配性能，并且仅需为池的总体性能付费，而无需为单一数据库的性能付费。无需调整数据库的性能。池中的数据库称为*弹性数据库*，它们能够自动增减以满足需求。弹性数据库会使用该池，但不会超出其限制，因此即使数据库的使用情况仍不可预测，你的成本也仍是可预测的。此外，你可以[在池中添加和删除数据库](/documentation/articles/sql-database-elastic-pool-portal)，将应用从少量数据库扩展到数千个，而一切费用不会超出由你控制的预算范围。

无论使用单一数据库还是弹性数据库，你都可以随时更改选择。可将单一数据库与弹性数据库池混合使用，并更改单一数据库和池的服务级别，以便进行创新设计。再者，凭借 Azure 的功能和作用范围，你可以将 Azure 服务与 SQL 数据库搭配使用以满足独特的现代应用程序设计需求，提高成本和资源效益，发掘新的商机。

但是，要如何比较数据库和数据库池的相对性能？ 当调高和调低性能时，如何知道该在何处停止？ 答案就是使用单一数据库的数据库事务单位 (DTU)，以及弹性数据库和数据库池的弹性 DTU (eDTU)。

## 了解 DTU

[AZURE.INCLUDE [SQL 数据库 DTU 说明](../includes/sql-database-understanding-dtus.md)]

## 使应用和业务持续运转

Azure 行业领先的 99.99% 可用性服务级别协议 [(SLA)](/support/legal/sla)（由 Microsoft 管理的数据中心的全球网络提供支持），可帮助你保持应用全天候运行。使用每个 SQL 数据库时，你可以使用内置的数据保护、容错功能，以及可能需要另外设计、购买、构建和管理的数据保护功能。即便如此，根据业务要求，你还可以请求额外级别的保护，以确保在发生灾难、错误或其他事件时，应用和业务可以快速恢复。使用 SQL 数据库时，每个服务层提供不同的功能菜单，使你能够立即启动并运行。你可以使用时间点还原将数据库还原到以前的状态，最长可还原到 35 天前。此外，如果托管数据库的数据中心发生服务中断，你可以故障转移到其他区域中的数据库副本。或者，你可以使用副本进行升级，或将其重新放置在不同的区域。

![SQL 数据库异地复制](./media/sql-database-technical-overview/azure_sqldb_map.png)


有关可用于不同服务层的其他业务连续性功能的详细信息，请参阅[业务连续性](/documentation/articles/sql-database-business-continuity)。

## 保护数据
SQL Server 的数据安全性一贯可靠，SQL 数据库也包含类似的功能，可以限制访问、保护数据，并帮助你监视活动。有关 SQL 数据库提供的安全选项的快速概览，请参阅[保护你的 SQL 数据库](/documentation/articles/sql-database-security)。有关更全面的安全功能视图，请参阅 [SQL Server 数据库引擎和 SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)。有关 Azure 平台安全性的信息，请访问 [Azure 信任中心](/support/trust-center/security)。

## 后续步骤
现在，你已阅读 SQL 数据库简介并回答了“什么是 SQL 数据库？”这个问题，因此可以继续完成以下内容：

- 有关单一数据库和弹性数据库的定价与计算器，请参阅[定价页](/home/features/sql-database/#price)。

- 从[创建第一个数据库](/documentation/articles/sql-database-get-started)开始。然后使用 [C#](/documentation/articles/sql-database-connect-query)、[Java](/documentation/articles/sql-database-develop-java-simple-windows)、[Node.js](/documentation/articles/sql-database-develop-nodejs-simple-windows)、[PHP](/documentation/articles/sql-database-develop-php-retry-windows)、[Python](/documentation/articles/sql-database-develop-python-simple-windows) 或 [Ruby](/documentation/articles/sql-database-develop-ruby-simple-linux) 构建你的第一个应用。

<!---HONumber=Mooncake_0215_2016-->