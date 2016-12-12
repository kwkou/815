<properties
	pageTitle="SQL 数据库教程：创建 SQL 数据库 | Azure"
	description="了解如何设置 SQL 数据库逻辑服务器、服务器防火墙规则、SQL 数据库和示例数据。此外，了解如何使用客户端工具进行连接、配置用户，以及设置数据库防火墙规则。"
	keywords="SQL 数据库教程, 创建 SQL 数据库"
	services="sql-database"
	documentationCenter=""
	authors="CarlRabeler"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/07/2016"
	wacn.date="12/12/2016"
	ms.author="carlrab"/>  



# SQL 数据库教程：使用 Azure 门户在几分钟内创建一个 SQL 数据库

> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/sql-database-get-started/)
- [C#](/documentation/articles/sql-database-get-started-csharp/)
- [PowerShell](/documentation/articles/sql-database-get-started-powershell/)

本教程介绍如何使用 Azure 经典管理门户完成以下任务：

- 使用示例数据创建一个 Azure SQL 数据库。
- 创建单个 IP 地址或一系列 IP 地址的服务器级别防火墙规则。

也可以使用 [C#](/documentation/articles/sql-database-get-started-csharp/) 或 [PowerShell](/documentation/articles/sql-database-get-started-powershell/) 来执行上述相同的任务。

[AZURE.INCLUDE [登录](../../includes/azure-getting-started-portal-login.md)]

[AZURE.INCLUDE [创建 SQL 数据库逻辑服务器](../../includes/sql-database-create-new-server-portal.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-database-portal.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-server-firewall-portal.md)]

## 后续步骤
现已完成本 SQL 数据库教程并创建了包含一些示例数据的数据库，接下来可以使用偏好的工具来探索其他功能。

- 如果熟悉 Transact-SQL 和 SQL Server Management Studio (SSMS)，请学习如何[使用 SSMS 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query-ssms/)。

- 如果熟悉 Excel，请学习如何[使用 Excel 连接到 Azure 中的 SQL 数据库](/documentation/articles/sql-database-connect-excel/)。

- 如果已准备好开始编写代码，请在[用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)中选择所需的编程语言

- 若要将本地 SQL Server 数据库移到 Azure，请参阅[将数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)了解详细信息。

- 若要使用 BCP 命令行工具将 CSV 文件中的某些数据载入新表，请参阅[使用 BCP 将 CSV 文件中的数据载入 SQL 数据库](/documentation/articles/sql-database-load-from-csv-with-bcp/)。

- 若要开始探索 Azure SQL 数据库安全性，请参阅[安全入门](/documentation/articles/sql-database-get-started-security/)


## 其他资源

[什么是 SQL 数据库？](/documentation/articles/sql-database-technical-overview/)

<!---HONumber=Mooncake_Quality_Review_1118_2016-->