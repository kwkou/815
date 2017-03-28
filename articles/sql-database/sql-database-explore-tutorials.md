<properties
    pageTitle="浏览 Azure SQL 数据库教程 | Azure"
    description="了解 SQL 数据库的特性和功能"
    keywords=""
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="04c0fd7f-d260-4e43-a4f0-41cdcd5e3786"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-management"
    ms.date="02/08/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab" />  


# 浏览 Azure SQL 数据库教程
单击下表中的链接可以转到所列的每个功能区域的概述，以及每个区域的简单分步入门教程。

## 创建服务器、数据库和服务器级防火墙规则
以下教程介绍如何创建服务器、数据库和服务器级防火墙规则，以及如何连接和查询服务器和数据库。

| 教程 | 说明 |
| --- | --- | 
| [第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/) | 完成此快速入门教程后，将获得一个示例数据库，以及一个在 Azure 资源组中运行并已附加到逻辑服务器的空白数据库。此外，还会获得两个服务器级防火墙规则并将其配置为启用服务器级主体，用于从两个指定的 IP 地址登录到服务器。最后，将会知道如何在 Azure 门户预览中查询数据库，以及如何使用 SQL Server Management Studio 进行连接和查询。 |
| [使用 PowerShell 预配和访问 Azure SQL 数据库](/documentation/articles/sql-database-get-started-powershell/) | 完成此教程后，将会获得一个示例数据库，以及一个在 Azure 资源组中运行并已附加到逻辑服务器的空白数据库。此外，还会配置一项服务器级防火墙规则，目的是让服务器级主体从指定 IP 地址（或 IP 地址范围）登录到服务器。 |
| [使用 C# 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/)| 在本教程中，将使用 C# 创建 SQL 数据库服务器、防火墙规则和 SQL 数据库。还将创建一个 Active Directory (AD) 应用程序和对 C# 应用进行身份验证所需的服务主体。 |
| | |

## 备份和数据库恢复
以下教程介绍如何使用[数据库备份](/documentation/articles/sql-database-automated-backups/)和[使用备份恢复数据库](/documentation/articles/sql-database-recovery-using-backups/)。

| 教程 | 说明 |
| --- | --- | 
| [使用 Azure 门户预览进行备份和还原](/documentation/articles/sql-database-get-started-backup-recovery-portal/) | 此教程介绍如何使用 Azure 门户预览查看备份、还原到某个时间点，以及通过 Azure 恢复服务保管库的备份进行还原
| [使用 PowerShell 进行备份和还原](/documentation/articles/sql-database-get-started-backup-recovery-powershell/) | 此教程介绍如何使用 PowerShell 查看备份、还原到某个时间点，以及通过 Azure 恢复服务保管库的备份进行还原
| | |

## 分片的数据库
以下教程介绍如何[使用分片映射管理器扩展数据库](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

| 教程 | 说明 |
| --- | --- | 
| [创建分片应用程序](/documentation/articles/sql-database-elastic-scale-get-started/) |本教程介绍如何使用简单的分片应用程序来使用弹性数据库工具的功能。 |
| [部署拆分/合并服务](/documentation/articles/sql-database-elastic-scale-configure-deploy-split-and-merge/) |此教程介绍如何在分片数据库之间移动数据。 |
| | |


## 弹性查询

以下教程介绍如何运行[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)。

| 教程 | 说明 |
| --- | --- | 
| 创建[弹性查询](/documentation/articles/sql-database-elastic-query-getting-started-vertical/) | 此教程将介绍如何使用单一连接点运行跨多个数据库的 T-SQL 查询 |
| [跨已扩展的云数据库进行报告](/documentation/articles/sql-database-elastic-query-getting-started/) |此教程介绍如何从位于横向分区（分片）数据库中的所有数据库创建报告 |
| [在具有不同架构的云数据库中查询](/documentation/articles/sql-database-elastic-query-vertical-partitioning/) | 此教程介绍如何针对具有不同架构的多个数据库运行 T-SQL 查询 |
| [跨已扩展的云数据库进行报告](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/) |此教程介绍如何在分片数据库中创建跨所有数据库的报表。 |
| | |

## 数据库身份验证和授权
以下教程介绍如何[创建和管理登录名与用户](/documentation/articles/sql-database-manage-logins/)。

| 教程 | 说明 |
| --- | --- | --- |
| [SQL 身份验证和授权](/documentation/articles/sql-database-control-access-sql-authentication-get-started/) | 此教程介绍如何使用 SQL Server 身份验证创建登录名和用户、将其添加到角色，以及向其授予权限 |
| [Azure AD 身份验证和授权](/documentation/articles/sql-database-control-access-aad-authentication-get-started/) | 此教程介绍如何使用 Azure Active Directory 身份验证创建登录名和用户 |
| | |

## 保护数据
以下教程介绍如何[保护 Azure SQL 数据库数据](/documentation/articles/sql-database-security-overview/)。

| 教程 | 说明 |
| --- | --- | 
| [使用 Always Encrypted 保护敏感数据](/documentation/articles/sql-database-always-encrypted-azure-key-vault/) |此教程介绍如何使用 Always Encrypted 向导保护 Azure SQL 数据库中的敏感数据。 |
| | |

## 开发
以下教程介绍应用程序和数据库开发。

| 教程 | 说明 |
| --- | --- | 
| [使用 Excel 创建报表](/documentation/articles/sql-database-connect-excel/) |此教程介绍如何将 Excel 连接到云中的 SQL 数据库，以便导入数据并根据数据库中的值创建表和图表。 |
| [使用 SQL Server 构建应用](https://www.microsoft.com/sql-server/developer-get-started/) |此教程介绍如何使用 SQL Server 构建应用程序 |
| [临时表](/documentation/articles/sql-database-temporal-tables/) | 此教程介绍临时表。
| [将实体框架与弹性工具配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/) |此教程介绍需要在实体框架应用程序中做哪些更改才能与弹性数据库工具集成。 |


## 数据同步
此教程介绍[数据同步](http://download.microsoft.com/download/4/E/3/4E394315-A4CB-4C59-9696-B25215A19CEF/SQL_Data_Sync_Preview.pdf)。

| 教程 | 说明 |
| --- | --- | 
| [Azure SQL 数据同步入门（预览）](/documentation/articles/sql-database-get-started-sql-data-sync/) | 本教程介绍使用 Azure 经典门户进行 Azure SQL 数据同步的基础知识。 |
| | |

## 监视和优化
以下教程介绍如何进行监视和优化。
| 教程 | 说明 |
| --- | --- | 
| [使用 PowerShell 收集弹性池遥测数据](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools)| 此教程介绍如何使用 PowerShell 收集弹性池遥测数据。 |
| [SaaS 的弹性池自定义仪表板](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools-custom-dashboard) | 此教程介绍如何创建用于监视弹性池的自定义仪表板。 |
| [将扩展事件捕获到事件文件目标](/documentation/articles/sql-database-xevent-code-event-file/)| 此教程介绍如何将扩展事件捕获到事件目标文件。|
| [将扩展事件捕获到环形缓冲区目标](/documentation/articles/sql-database-xevent-code-ring-buffer/)| 此教程介绍如何将扩展事件捕获到代码环形缓冲区。|
| | |

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: content index update-->