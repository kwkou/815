<properties
    pageTitle="T-SQL：创建和管理单一 Azure SQL 数据库 | Azure"
    description="快速参考：如何使用 Azure 门户预览创建和管理单一 Azure SQL 数据库"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="single databases"
    ms.devlang="NA"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.date="02/06/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab" />  


# 使用 Transact-SQL 创建和管理单一 Azure SQL 数据库

可以使用 [Azure 门户预览](https://portal.azure.cn/)、PowerShell、Transact-SQL、REST API 或 C# 创建和管理单一 Azure SQL 数据库。本主题说明如何使用 Azure 门户预览。有关 PowerShell，请参阅[使用 Powershell 创建和管理单一数据库](/documentation/articles/sql-database-manage-single-databases-powershell/)。有关 Transact-SQL，请参阅[使用 Transact-SQL 创建和管理单一数据库](/documentation/articles/sql-database-manage-single-databases-tsql/)。

## 在 SQL Server Management Studio 中使用 Transact-SQL 创建 Azure SQL 数据库

若要在 SQL Server Management Studio 中使用 Transact-SQL 创建 SQL 数据库，请执行以下操作：

1. 使用服务器级主体登录名或属于 **dbmanager** 角色的登录名通过 SQL Server Management Studio 连接到 Azure 数据库服务器。有关登录名的详细信息，请参阅[管理登录名](/documentation/articles/sql-database-manage-logins/)。
2. 在对象资源管理器中，打开“数据库”节点，展开“系统数据库”文件夹，右键单击“master”，然后单击“新建查询”。
3. 使用 **CREATE DATABASE** 语句可创建数据库。有关详细信息，请参阅 [CREATE DATABASE（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)。以下语句将创建名为 **myTestDB** 的数据库，并指定它是默认大小上限为 250 GB 的“标准 S0 版本”数据库。
  
        CREATE DATABASE myTestDB
        (EDITION='Standard',
        SERVICE_OBJECTIVE='S0');

4. 单击“执行”运行查询。
5. 在对象资源管理器中，右键单击“数据库”节点，然后单击“刷新”在对象资源管理器中查看新的数据库。


## 更改单一数据库的服务层和性能级别
使用 [Transact-SQL (T-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 更改数据库最大大小

> [AZURE.TIP]
>有关使用 Transact-SQL 创建数据库的教程，请参阅[创建数据库 - Azure 门户预览](/documentation/articles/sql-database-get-started/)。
>

## 后续步骤
* 有关管理工具的概述，请参阅[管理工具概述](/documentation/articles/sql-database-manage-overview/)。
* 若要了解如何使用 Azure 门户预览执行管理任务，请参阅[使用 Azure 门户预览管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-portal/)。
* 若要了解如何使用 PowerShell 执行管理任务，请参阅[使用 PowerShell 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-powershell/)。
* 若要了解如何使用 SQL Server Management Studio 执行管理任务，请参阅 [SQL Server Management Studio](/documentation/articles/sql-database-manage-azure-ssms/)。
* 有关 SQL 数据库服务的信息，请参阅[什么是 SQL 数据库](/documentation/articles/sql-database-technical-overview/)。
* 有关 Azure 数据库服务器和数据库功能的信息，请参阅[功能](/documentation/articles/sql-database-features/)。

<!---HONumber=Mooncake_0320_2017-->