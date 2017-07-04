<properties
    pageTitle="在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库 | Azure"
    description="演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该示例使用 JDBC，并在 Windows 客户端计算机上运行。"
    services="sql-database"
    documentationcenter=""
    author="LuisBosquez"
    manager="jhubbard"
    editor="genemi" />
<tags
    ms.assetid="08fc49b1-cd48-4dcc-a293-ff22a4d2d62c"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="java"
    ms.topic="article"
    ms.date="02/03/2017"
    wacn.date="04/06/2017"
    ms.author="lbosq" />

# 在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题演示可用于连接到 Azure SQL 数据库的 Java 代码示例。该 Java 示例依赖于 Java 开发工具包 (JDK) 版本 1.8。该示例使用 JDBC 驱动程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境
[配置用于 Java 开发的开发环境](https://docs.microsoft.com/sql/connect/jdbc/step-1-configure-development-environment-for-java-development/)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建数据库。

## 步骤 3：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-jdbc-20-portalshots](../../includes/sql-database-include-connection-string-jdbc-20-portalshots.md)]

> [AZURE.NOTE] 如果使用的是 JTDS JDBC 驱动程序，则需要将“ssl=require”添加到连接字符串的 URL，并需要设置以下 JVM 选项：“-Djsse.enableCBCProtection=false”。此 JVM 选项会禁用针对某个安全漏洞的修复程序，因此在设置此选项之前，请确保已了解涉及哪些风险。

## 步骤 4：运行示例代码
* [“使用 Java 连接到 SQL”概念证明](https://docs.microsoft.com/sql/connect/jdbc/step-3-proof-of-concept-connecting-to-sql-using-java/)

## 后续步骤

* 访问 [Java 开发人员中心](/develop/java/)。
* 参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft JDBC Driver for SQL Server](https://docs.microsoft.com/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server/) 的详细信息

## 其他资源 

* [多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库功能](/home/features/sql-database/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->