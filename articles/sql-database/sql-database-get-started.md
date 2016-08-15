<properties
	pageTitle="SQL 数据库教程：创建 SQL 数据库 | Azure"
	description="了解如何设置 SQL 数据库逻辑服务器、服务器防火墙规则、SQL 数据库、示例性数据，如何使用客户端工具连接以及如何配置用户和数据库防火墙规则。"
	keywords="SQL 数据库教程：创建 SQL 数据库"
	services="sql-database"
	documentationCenter=""
	authors="carlrabeler"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="07/05/2016"
	wacn.date="08/15/2016"/>

# SQL 数据库教程：使用 Azure 经典管理门户在几分钟内创建一个 SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/sql-database-get-started/)
- [C#](/documentation/articles/sql-database-get-started-csharp/)
- [PowerShell](/documentation/articles/sql-database-get-started-powershell/)

在本教程中，你将学习如何使用 Azure 门户来：

- 创建 SQL 数据库逻辑服务器来托管 SQL 数据库
- 创建无数据、包含示例性数据或包含 SQL 数据库备份数据的 SQL 数据库。
- 创建单个 IP 地址或一系列 IP 地址的服务器级别防火墙规则。

通过以下链接来执行使用 [C#](/documentation/articles/sql-database-get-started-csharp/) 或 [PowerShell](/documentation/articles/sql-database-get-started-powershell/) 的这些相同任务。

[AZURE.INCLUDE [登录](../../includes/azure-getting-started-portal-login.md)]

[AZURE.INCLUDE [创建 SQL 数据库逻辑服务器](../../includes/sql-database-create-new-server-portal.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-database-portal.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-server-firewall-portal.md)]

## 后续步骤
现在，你已完成本 SQL 数据库教程并创建了包含一些示例数据的数据库，接下来可以使用偏好的工具来探索其他功能。

- 如果你熟悉 Transact-SQL 和 SQL Server Management Studio，请学习如何[使用 SSMS 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query-ssms/)。

- 如果你了解 Excel，请学习如何[使用 Excel 连接到 Azure SQL 数据库](/documentation/articles/sql-database-connect-excel/)。

- 如果你已准备好开始编码，请在[用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)中选择所需的编程语言

- 如果你想要将本地 SQL Server 数据库移到 Azure，请参阅[将数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)了解详细信息。

- 如果想使用 BCP 将 CSV 文件上的某些数据加载到新表中，请参阅[使用 BCP 将 CSV 文件上的数据加载到 SQL 数据库](/documentation/articles/sql-database-load-from-csv-with-bcp/)。


## 其他资源

[什么是 SQL 数据库？](/documentation/articles/sql-database-technical-overview/)

<!---HONumber=Mooncake_0808_2016-->