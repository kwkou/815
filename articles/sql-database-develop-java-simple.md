<properties
	pageTitle="在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库 | Azure"
	description="演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该示例使用 JDBC，并在 Windows 客户端计算机上运行。"
	services="sql-database"
	documentationCenter=""
	authors="LuisBosquez"
	manager="jeffreyg"
	editor="genemi"/>


<tags
	ms.service="sql-database"
	ms.date="04/27/2016"
	wacn.date="06/14/2016"/>


# 在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该 Java 示例依赖于 Java 开发工具包 (JDK) 版本 1.8。该示例将使用 JDBC 驱动程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[配置用于 Java 开发的开发环境](https://msdn.microsoft.com/zh-cn/library/mt720658.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建数据库。

## 步骤 3：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-jdbc-20-portalshots](../includes/sql-database-include-connection-string-jdbc-20-portalshots.md)]

> [AZURE.NOTE] 如果你使用的是 JTDS JDBC 驱动程序，则需要将“ssl=require”添加到连接字符串的 URL，并需要设置以下 JVM 选项：“-Djsse.enableCBCProtection=false”。此 JVM 选项会禁用针对某个安全漏洞的修复程序，因此在设置此选项之前，请确保你了解涉及哪些风险。

## 步骤 4：运行示例代码

* [使用 Java 连接到 SQL 以进行概念认证](https://msdn.microsoft.com/zh-cn/library/mt720656.aspx)

## 后续步骤

有关详细信息，请参阅 [Java 开发人员中心](/develop/java)。

<!---HONumber=Mooncake_0530_2016-->
