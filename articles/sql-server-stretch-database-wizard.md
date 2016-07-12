<properties
	pageTitle="通过运行“为数据库启用延伸向导”开始操作 | Azure"
	description="了解如何通过运行“为数据库启用延伸向导”，来为延伸数据库配置数据库。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="04/25/2016"/>

# 通过运行“为数据库启用延伸向导”开始操作

若要为SQL Server Stretch Database配置数据库，请运行“为数据库启用延伸向导”。本主题介绍你要在该向导中输入的信息以及必须选择的选项。

若要了解有关SQL Server Stretch Database的详细信息，请参阅[SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-overview/)。

## 启动向导

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要对其启用延伸的数据库。

2.  单击右键，然后依次选择“任务”、“延伸”、“启用”以启动向导。

## <a name="Intro"></a>介绍
查看向导的用途和先决条件。

![Stretch Database 向导的简介页][StretchWizardImage1]

## <a name="Tables"></a>选择表
选择你要为其启用延伸的表。

![Stretch Database 向导的选择表页][StretchWizardImage2]

|列|说明|
|----------|---------------|
|(无标题)|选中此列中的复选框可为选定的表启用延伸。|
|**Name**|指定表中的列名。|
|(无标题)|此列中的符号通常指出由于阻碍性问题，你不能为选定的表启用延伸。这可能是因为该表使用了不受支持的数据类型。将鼠标悬停于符号上可在工具提示中显示更多信息。有关详细信息，请参阅 [Surface area limitations and blocking issues for Stretch Database（延伸数据库的外围应用限制与阻碍性问题）](/documentation/articles/sql-server-stretch-database-limitations/)。|
|**已延伸**|指示该表是否已启用。|
|**行**|指定表中的行数。|
|**大小(KB)**|指定表的大小，以 KB 为单位。|
|**迁移**|在 CTP 3.1 到 RC1 版本中，你只能使用向导迁移整个表。如果你要指定一个谓词以便在包含历史数据和当前数据的表中选择要迁移的行，请在退出向导之后运行 ALTER TABLE 语句指定谓词。有关详细信息，请参阅 [Enable Stretch Database for a table（为表启用 Stretch Database）](/documentation/articles/sql-server-stretch-database-enable-table/)或 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。|

## <a name="Configure"></a>配置 Azure 部署

1.  使用 Microsoft 帐户登录到 Azure。

    ![登录到 Azure - Stretch Database 向导][StretchWizardImage3]

2.  选择用于延伸数据库的 Azure 订阅。

3.  选择一个 Azure 区域。如果你要创建新服务器，该服务器将在此区域中创建。

4.  指定是要使用现有的服务器还是创建新的 Azure 服务器。

    如果 SQL Server 上的 Active Directory 已与 Azure Active Directory 联合，则你可以选择使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。有关此选项的要求详细信息，请参阅 [ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)。

	-   **创建新服务器**

        1.  为服务器管理员创建登录名和密码。

        2.  （可选）使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。

		![创建新的 Azure 服务器 - Stretch Database 向导][StretchWizardImage4]

    -   **现有服务器**

        1.  选择或输入现有 Azure 服务器的名称。

        2.  选择身份验证方法。

            -   如果选择“SQL Server 身份验证”，请创建新的登录名和密码。

            -   选择“Active Directory 集成身份验证”可以使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。

		![选择现有的 Azure 服务器 - Stretch Database 向导][StretchWizardImage5]

## <a name="Credentials"></a>安全凭据
输入一个强密码用于创建数据库主密钥，或者，如果数据库主密钥已存在，则输入该密钥的密码。

必须使用数据库主密钥来保护SQL Server Stretch Database在连接到远程数据库时所用的凭据。

![Stretch Database 向导的安全凭据页][StretchWizardImage6]

有关数据库主密钥的详细信息，请参阅 [CREATE MASTER KEY (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) 和[创建数据库主密钥](https://msdn.microsoft.com/zh-cn/library/aa337551.aspx)。有关向导创建的凭据的详细信息，请参阅 [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。

## <a name="Network"></a>选择 IP 地址
使用 SQL Server 的公共 IP 地址或输入 IP 地址范围，以便在 Azure 上创建一个可使 SQL Server 与远程 Azure 服务器通信的防火墙规则。

在此页上提供的一个或多个 IP 地址告知 Azure 服务器允许 SQL Server 发起的传入数据、查询和管理操作通过 Azure 防火墙。向导不会更改 SQL Server 上的任何防火墙设置。

![Stretch Database 向导的选择 IP 地址页][StretchWizardImage7]

## <a name="Summary"></a>摘要
查看在向导中输入的值和选择的选项。然后选择“完成”以启用延伸。

![Stretch Database 向导的摘要页][StretchWizardImage8]

## <a name="Results"></a>结果
查看结果。

（可选）选择“监视”启动监视器，以便在SQL Server Stretch Database监视器中查看数据迁移状态。有关详细信息，请参阅[数据迁移的监视和故障排除 (SQL Server Stretch Database)](/documentation/articles/sql-server-stretch-database-monitor/)。

## <a name="KnownIssues"></a>排查向导问题
**延伸数据库向导失败。** 如果SQL Server Stretch Database尚未在服务器级别启用，而你在不使用系统管理员权限的情况下运行向导来启用SQL Server Stretch Database，则向导将会失败。请求系统管理员在本地服务器实例上启用SQL Server Stretch Database，然后再次运行向导。有关详细信息，请参阅[先决条件：在服务器上启用SQL Server Stretch Database所需的权限](/documentation/articles/sql-server-stretch-database-enable-database/#EnableTSQLServer)。

## 后续步骤
为SQL Server Stretch Database启用其他表。监视数据迁移与管理已启用延伸的数据库和表。

-   参阅[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)来启用其他表。

-   参阅[监视SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-monitor/)来查看数据迁移状态。

-   [暂停和恢复SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-pause/)

-   [SQL Server Stretch Database的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

-   [备份和还原已启用延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)

## 另请参阅

[为数据库启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)

[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)

[StretchWizardImage1]: ./media/sql-server-stretch-database-wizard/stretchwiz1.png
[StretchWizardImage2]: ./media/sql-server-stretch-database-wizard/stretchwiz2.png
[StretchWizardImage3]: ./media/sql-server-stretch-database-wizard/stretchwiz3.png
[StretchWizardImage4]: ./media/sql-server-stretch-database-wizard/stretchwiz4.png
[StretchWizardImage5]: ./media/sql-server-stretch-database-wizard/stretchwiz5.png
[StretchWizardImage6]: ./media/sql-server-stretch-database-wizard/stretchwiz6.png
[StretchWizardImage7]: ./media/sql-server-stretch-database-wizard/stretchwiz7.png
[StretchWizardImage8]: ./media/sql-server-stretch-database-wizard/stretchwiz8.png

<!---HONumber=Mooncake_0418_2016-->