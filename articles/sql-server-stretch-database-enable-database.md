<properties
	pageTitle="为数据库启用SQL Server Stretch Database | Azure"
	description="了解如何为SQL Server Stretch Database配置数据库。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="03/09/2016"
	wacn.date="03/10/2016"/>

# 为数据库启用SQL Server Stretch Database

若要为SQL Server Stretch Database配置某个数据库，请在 SQL Server Management Studio 中选择该数据库对应的“任务 | 延伸 | 启用”，打开“为数据库启用延伸”向导。也可以使用 Transact-SQL 为数据库启用SQL Server Stretch Database。

如果你针对某个表选择了“任务 | 延伸 | 启用”但尚未为数据库启用SQL Server Stretch Database，该向导将为SQL Server Stretch Database配置数据库，并让你在此配置过程中配置表。请遵循本主题中的步骤，而不要遵循[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)中的步骤。

对数据库或表启用SQL Server Stretch Database需要有 db\_owner 权限。对数据库启用SQL Server Stretch Database还需要有 CONTROL DATABASE 权限。

## 准备工作

-   在为延伸配置数据库之前，我们建议你运行SQL Server Stretch Database顾问来识别符合延伸条件的数据库和表。SQL Server Stretch Database顾问还可识别阻碍性问题。有关详细信息，请参阅[识别符合SQL Server Stretch Database条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

-   请查看[SQL Server Stretch Database的外围应用限制与阻碍性问题](/documentation/articles/sql-server-stretch-database-limitations/)。

-   SQL Server Stretch Database会将数据迁移到 Azure。因此，你必须拥有一个 Azure 帐户和订阅以供计费。若要获取 Azure 帐户，请[单击此处](/pricing/1rmb-trial/)。

-   准备好所需的信息，以便能够创建新的远程数据库或选择现有的远程数据库，以及创建可使本地服务器能够与远程服务器通信的防火墙规则。

## <a name="EnableTSQLServer"></a>先决条件：在服务器上启用SQL Server Stretch Database所需的权限
在对数据库或表启用SQL Server Stretch Database之前，必须先在本地服务器上启用SQL Server Stretch Database。此操作需要 sysadmin 或 serveradmin 权限。

-   如果你拥有所需的管理权限，“为数据库启用延伸”向导将为延伸配置服务器。

-   如果你没有所需的权限，则只有在某位管理员通过运行 **sp\_configure** 手动启用该选项之后你才可以运行该向导，或者必须由管理员运行该向导。

若要在服务器上手动启用SQL Server Stretch Database，请运行 **sp\_configure** 并打开 **remote data archive** 选项。以下示例通过将 **remote data archive** 选项的值设置为 1 来启用该选项。

```
EXEC sp_configure 'remote data archive' , '1';
GO
RECONFIGURE;
GO
```
有关详细信息，请参阅[配置 remote data archive 服务器配置选项](https://msdn.microsoft.com/zh-cn/library/mt143175.aspx) 和 [sp\_configure (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms188787.aspx)。

## <a name="Wizard"></a>使用向导来对数据库启用SQL Server Stretch Database
有关“为数据库启用延伸向导”的信息，包括必须输入的信息以及必须选择的选项，请参阅[为数据库启用延伸向导](/documentation/articles/sql-server-stretch-database-wizard/)。

## <a name="EnableTSQLDatabase"></a>使用 Transact-SQL 对数据库启用SQL Server Stretch Database
在对各个表启用SQL Server Stretch Database之前，必须先对数据库启用SQL Server Stretch Database。

对数据库或表启用SQL Server Stretch Database需要有 db\_owner 权限。对数据库启用SQL Server Stretch Database还需要有 CONTROL DATABASE 权限。

1.  在开始之前，请选择SQL Server Stretch Database要将数据迁移到的现有 Azure 服务器，或创建一个新的服务器。

2.  在该 Azure 服务器上，创建一个可使 SQL Server 能够与远程服务器通信的防火墙规则，该规则使用 SQL Server 的 IP 地址（或 IP 地址范围）。

3.  若要为SQL Server Stretch Database配置某个 SQL Server 数据库，该数据库必须具有数据库主密钥。数据库主密钥用于保护SQL Server Stretch Database在连接到远程数据库时所用的凭据。若要手动创建数据库主密钥，请参阅 [CREATE MASTER KEY (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) 和[创建数据库主密钥](https://msdn.microsoft.com/zh-cn/library/aa337551.aspx)。

    ```tsql
    USE <database>
    GO

    CREATE MASTER KEY ENCRYPTION BY PASSWORD='<password>'
    ```

4.  在为SQL Server Stretch Database配置数据库时，必须提供用于在本地 SQL Server 与远程 Azure 服务器之间进行通信的SQL Server Stretch Database凭据。可以使用两个选项。

    -   你可以提供管理员凭据。

        -   如果通过运行向导来启用SQL Server Stretch Database，则可以在运行向导时创建凭据。

        -   如果通过运行 **ALTER DATABASE** 启用SQL Server Stretch Database，则必须在启用SQL Server Stretch Database之前手动创建凭据。

        若要手动创建凭据，请参阅 [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。创建凭据需要 ALTER ANY CREDENTIAL 权限。

        ```tsql
        CREATE DATABASE SCOPED CREDENTIAL <db_scoped_credential_name>
            WITH IDENTITY = '<identity>' , SECRET = '<secret>'
        GO
        ```

    -   如果符合以下所有条件，则可以使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。

        -   运行 SQL Server 实例的服务帐户是一个域帐户。

        -   域帐户属于某个域，该域的 Active Directory 与 Azure Active Directory 相联合。

        -   远程 Azure 服务器配置为支持 Azure Active Directory 身份验证。

        -   必须将运行 SQL Server 实例的服务帐户配置为远程 Azure 服务器上的 dbmanager 或 sysadmin 帐户。

5.  若要为SQL Server Stretch Database配置数据库，请运行 ALTER DATABASE 命令。

    1.  对于 SERVER 参数，请提供现有 Azure 服务器的名称，包括该名称的 `.database.windows.net` 部分 - 例如 `MyStretchDatabaseServer.database.windows.net`。

    2.  提供包含 CREDENTIAL 参数的现有管理员凭据，或指定 FEDERATED\_SERVICE\_ACCOUNT \\= ON。以下示例提供了一个现有的凭据。

    ```tsql
    ALTER DATABASE <database name>
        SET REMOTE_DATA_ARCHIVE = ON
            (
                SERVER = '<server_name>' ,
                CREDENTIAL = <db_scoped_credential_name>
            ) ;
    GO;
    ```

## 后续步骤
为SQL Server Stretch Database启用其他表。监视数据迁移与管理已启用延伸的数据库和表。

-   参阅[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)来启用其他表。

-   参阅[监视SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-monitor/)来查看数据迁移状态。

-   [暂停和恢复SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-pause/)

-   [SQL Server Stretch Database的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

-   [备份和还原已启用延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)

## 另请参阅
[识别符合SQL Server Stretch Database条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)
[ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)

<!---HONumber=Mooncake_0307_2016-->