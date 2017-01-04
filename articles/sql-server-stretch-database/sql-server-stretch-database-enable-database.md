<properties
	pageTitle="为数据库启用 Stretch Database | Azure"
	description="了解如何为延伸数据库配置数据库。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/05/2016"
	wacn.date="01/03/2017"
	ms.author="douglasl"/>

# 为数据库启用延伸数据库

若要为 Stretch Database 配置现有数据库，请在 SQL Server Management Studio 中选择该数据库对应的“任务 | 延伸 | 启用”，以打开“启用数据库延伸”向导。也可以使用 Transact-SQL 为数据库启用延伸数据库。

如果你针对单个表选择了“任务 | 延伸 | 启用”但尚未为数据库启用 Stretch Database，该向导将为 Stretch Database配置数据库，并让你在此过程中选择表。请遵循本主题中的步骤，而不要遵循[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)中的步骤。

对数据库或表启用延伸数据库需要有 db\_owner 权限。对数据库启用延伸数据库还需要有 CONTROL DATABASE 权限。

 >   [AZURE.NOTE] 若要在以后禁用 Stretch Database，请记住，针对表或数据库禁用 Stretch Database 不会删除远程对象。若要删除远程表或远程数据库，必须使用 Azure 管理门户。远程对象在手动删除之前，会持续产生 Azure 费用。

## 准备工作

-   在为延伸配置数据库之前，我们建议你运行延伸数据库顾问来识别符合延伸条件的数据库和表。延伸数据库顾问还可识别阻碍性问题。有关详细信息，请参阅[识别符合 Stretch Database 条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

-   请参阅 [Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)。

-   延伸数据库会将数据迁移到 Azure。因此，必须拥有一个 Azure 帐户和订阅以供计费。若要获取 Azure 帐户，请[单击此处](/pricing/1rmb-trial/)。

-   使用所需的连接和登录信息创建新的 Azure 服务器，或者选择现有的 Azure 服务器。

## <a name="EnableTSQLServer"></a>先决条件：在服务器上启用 Stretch Database
在对数据库或表启用延伸数据库之前，必须先在本地服务器上启用延伸数据库。此操作需要 sysadmin 或 serveradmin 权限。

-   如果你拥有所需的管理权限，“为数据库启用 Stretch”向导将为 Stretch 配置服务器。

-   如果你没有所需的权限，则只有在某位管理员通过运行 **sp\_configure** 手动启用该选项之后才可以运行该向导，或者必须由管理员运行该向导。

若要在服务器上手动启用 Stretch Database，请运行 **sp\_configure** 并打开 **remote data archive** 选项。以下示例通过将 **remote data archive** 选项的值设置为 1 来启用该选项。


	EXEC sp_configure 'remote data archive' , '1';
	GO

	RECONFIGURE;
	GO

有关详细信息，请参阅[配置 remote data archive 服务器配置选项](https://msdn.microsoft.com/zh-cn/library/mt143175.aspx)和 [sp\_configure (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms188787.aspx)。

## <a name="Wizard"></a>使用向导来对数据库启用 Stretch Database
有关“为数据库启用 Stretch 向导”的信息，包括必须输入的信息以及必须选择的选项，请参阅[通过运行“为数据库启用 Stretch 向导”开始操作](/documentation/articles/sql-server-stretch-database-wizard/)。

## <a name="EnableTSQLDatabase"></a>使用 Transact-SQL 对数据库启用 Stretch Database
在对各个表启用延伸数据库之前，必须先对数据库启用延伸数据库。

对数据库或表启用延伸数据库需要有 db\_owner 权限。对数据库启用延伸数据库还需要有 CONTROL DATABASE 权限。

1.  在开始之前，请选择 Stretch Database 要将数据迁移到的现有 Azure 服务器，或创建一个新的 Azure 服务器。

2.  在该 Azure 服务器上，创建一个允许 SQL Server 与远程服务器通信的防火墙规则，该规则使用 SQL Server 的 IP 地址范围。

    你可以通过尝试从 SQL Server Management Studio (SSMS) 的对象资源管理器连接到 Azure 服务器，轻松找到所需的值并创建防火墙规则。SSMS 通过打开以下对话框（其中已包括所需的 IP 地址值）帮助你创建规则。

	![在 SSMS 中创建防火墙规则][FirewallRule]  


3.  若要为延伸数据库配置某个 SQL Server 数据库，该数据库必须具有数据库主密钥。数据库主密钥用于保护延伸数据库在连接到远程数据库时所用的凭据。下面是一个示例，用于创建新的数据库主密钥。

        USE <database>
        GO
    
        CREATE MASTER KEY ENCRYPTION BY PASSWORD ='<password>'

    有关数据库主密钥的详细信息，请参阅 [CREATE MASTER KEY (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) 和[创建数据库主密钥](https://msdn.microsoft.com/zh-cn/library/aa337551.aspx)。

4.  在为延伸数据库配置数据库时，必须提供用于在本地 SQL Server 与远程 Azure 服务器之间进行通信的延伸数据库凭据。可以使用两个选项。
    -   你可以提供管理员凭据。
    -   如果通过运行向导来启用延伸数据库，则可以在运行向导时创建凭据。
    -   如果你打算通过运行 **ALTER DATABASE** 启用 Stretch Database，则必须在运行 **ALTER DATABASE** 启用 Stretch Database 之前手动创建凭据。

		下面是一个示例，用于创建新的凭据。

        ```tsql
        CREATE DATABASE SCOPED CREDENTIAL <db_scoped_credential_name>
            WITH IDENTITY = '<identity>' , SECRET = '<secret>';
        GO
        ```

		有关凭据的详细信息，请参阅 [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。创建凭据需要 ALTER ANY CREDENTIAL 权限。
    -   如果符合以下所有条件，则可以使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。
    -   运行 SQL Server 实例的服务帐户是一个域帐户。
    -   域帐户属于某个域，该域的 Active Directory 与 Azure Active Directory 相联合。
    -   远程 Azure 服务器配置为支持 Azure Active Directory 身份验证。
    -   必须将运行 SQL Server 实例的服务帐户配置为远程 Azure 服务器上的 dbmanager 或 sysadmin 帐户。

5.  若要为延伸数据库配置数据库，请运行 ALTER DATABASE 命令。

    1.  对于 SERVER 参数，请提供现有 Azure 服务器的名称，包括该名称的 `.database.chinacloudapi.cn` 部分 - 例如 `MyStretchDatabaseServer.database.chinacloudapi.cn`。

    2.  请提供具有 CREDENTIAL 参数的现有管理员凭据，或指定 FEDERATED\_SERVICE\_ACCOUNT = ON。以下示例提供了一个现有的凭据。

    ```tsql
    ALTER DATABASE <database name>
        SET REMOTE_DATA_ARCHIVE = ON
            (
                SERVER = '<server_name>',
                CREDENTIAL = <db_scoped_credential_name>
            ) ;
    GO
    ```

## 后续步骤
-   参阅[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)来启用其他表。

-   参阅[监视 Stretch Database](/documentation/articles/sql-server-stretch-database-monitor/) 来查看数据迁移状态。

-   [暂停和恢复延伸数据库](/documentation/articles/sql-server-stretch-database-pause/)

-   [延伸数据库的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

-   [备份启用了延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)

## 另请参阅

[标识 Stretch Database 的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)

[ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)

[FirewallRule]: ./media/sql-server-stretch-database-enable-database/firewall.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->