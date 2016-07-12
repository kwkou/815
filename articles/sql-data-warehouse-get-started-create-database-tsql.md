<properties
   pageTitle="使用 TSQL 创建 SQL 数据仓库 | Azure"
   description="了解如何使用 TSQL 创建 Azure SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""
   tags="azure-sql-data-warehouse"/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/20/2016"
   wacn.date="06/23/2016"/>

# 使用 Transact-SQL (TSQL) 创建 SQL 数据仓库数据库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文说明如何使用 Transact-SQL (TSQL) 创建 SQL 数据仓库数据库。

## 开始之前

若要完成本文中的步骤，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 一个 V12 逻辑服务器。你将需要使用 V12 SQL 服务器来创建 SQL 数据仓库。如果没有 V12 逻辑 SQL 服务器，请参阅[如何从 Azure 门户创建 SQL 数据仓库][]一文中的**配置和创建服务器**。
- Visual Studio。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载][]页。


> [AZURE.NOTE] 创建新的 SQL 数据仓库可能会导致新的计费服务。有关定价的更多详细信息，请参阅 [SQL 数据仓库定价][]。

## 使用 Visual Studio 创建数据库

如果不熟悉 Visual Studio，请参阅[使用 Visual Studio 连接到 SQL 数据仓库][]一文。若要开始操作，请在 Visual Studio 中打开 SQL Server 对象资源管理器，并连接到要托管 SQL 数据仓库数据库的服务器。连接后，可以针对 **master** 数据库运行以下 SQL 命令来创建 SQL 数据仓库。此命令会创建服务目标为 DW400 的数据库 MySqlDwDb，并允许此数据库增长到大小上限 10 TB。

```
CREATE DATABASE MySqlDwDb (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB);
```

## 使用 sqlcmd 创建数据库

或者，你可以在命令提示符处运行以下命令，以使用 sqlcmd 运行相同的命令。

```
sqlcmd -S <Server Name>.database.chinacloudapi.cn -I -U <User> -P <Password> -Q "CREATE DATABASE MySqlDwDb (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB)"
```

**MAXSIZE** 和 **SERVICE\_OBJECTIVE** 参数指定数据库可使用的最大磁盘空间，以及分配给数据仓库实例的计算资源。服务目标实质上是以线性方式随着 DWU 大小缩放的 CPU 和内存分配。

MAXSIZE 可以介于 250 GB 与 60 TB 之间。服务目标可以介于 DW100 与 DW2000 之间。有关所有 MAXSIZE 和 SERVICE\_OBJECTIVE 有效值的完整列表，请参阅 MSDN 文档中的 [CREATE DATABASE][]。使用 [ALTER DATABASE][] T-SQL 命令也可以更改 MAXSIZE 和 SERVICE\_OBJECTIVE。更改 SERVICE\_OBJECTIVE 时应谨慎，因为这会导致服务重新启动而取消所有正在进行的查询。更改 MAXSIZE 时则不必如此，因为这只是简单的元数据操作。

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][]或了解如何[开发][]、[加载][]，或[迁移][]数据。

<!--Article references-->
[如何从 Azure 门户创建 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[使用 Visual Studio 连接到 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-get-started-connect/
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-overview-load/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/

<!--MSDN references--> 
[CREATE DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx
[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx

<!--Other Web references-->
[SQL 数据仓库定价]: /home/features/sql-data-warehouse/pricing/
[Visual Studio 下载]: https://www.visualstudio.com/downloads/download-visual-studio-vs

<!---HONumber=Mooncake_0530_2016-->