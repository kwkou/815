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
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="08/05/2016"
	wacn.date="01/04/2017"
	ms.author="douglasl"/>

# 通过运行“为数据库启用延伸向导”开始操作

若要为延伸数据库配置数据库，请运行“为数据库启用延伸向导”。本主题介绍必须在该向导中输入的信息以及必须选择的选项。

若要了解有关 Stretch Database 的详细信息，请参阅 [Stretch Database](/documentation/articles/sql-server-stretch-database-overview/)。

 >   [AZURE.NOTE] 若要在以后禁用 Stretch Database，请记住，针对表或数据库禁用 Stretch Database 不会删除远程对象。若要删除远程表或远程数据库，必须使用 Azure 管理门户。远程对象在手动删除之前，会持续产生 Azure 费用。

## 启动向导

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要对其启用延伸的数据库。

2.  单击右键，然后依次选择“任务”、“延伸”、“启用”以启动向导。

## <a name="Intro"></a>介绍
查看向导的用途和先决条件。

重要的先决条件包括：

-   必须是管理员才能更改数据库设置。
-   必须有一个 Azure 订阅。
-   SQL Server 必须能与远程 Azure 服务器通信。

![Stretch Database 向导的简介页][StretchWizardImage1]  


## <a name="Tables"></a>选择表
选择你要为其启用延伸的表。

包含大量行的表会显示在已排序列表的顶端。在显示表列表之前，向导会先分析表中是否有 Stretch Database 当前不支持的数据类型。

![Stretch Database 向导的选择表页][StretchWizardImage2]  


|列|说明|
|----------|---------------|
|(无标题)|选中此列中的复选框可为选定的表启用延伸。|
|**名称**|指定表中的列名。|
|(无标题)|此列中的符号可能表示不会阻止你为所选表启用延伸的警告。它也可能表示会阻止你为所选表启用延伸的阻止问题，例如，因为该表使用不支持的数据类型。将鼠标悬停于符号上可在工具提示中显示更多信息。有关详细信息，请参阅 [Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)。|
|**已延伸**|指示该表是否已启用延伸。|
|**迁移**|你可以迁移整个表（**整个表**），也可以对表中的现有列指定一个筛选器。如果想要使用不同的筛选器函数来选择要迁移的行，请运行 ALTER TABLE 语句以在退出向导后指定筛选器函数。有关筛选器函数的详细信息，请参阅[使用筛选器函数选择要迁移的行](/documentation/articles/sql-server-stretch-database-predicate-function/)。有关如何应用函数的详细信息，请参阅[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/) 或 [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)。|
|**行**|指定表中的行数。|
|**大小(KB)**|指定表的大小，以 KB 为单位。|

## <a name="Filter"></a>有选择性地提供行筛选器

如果想提供筛选器函数来选择要迁移的行，请在“选择表”页上执行以下操作。

1.  在“选择要延伸的表”列表中，单击表的行中的“整个表”。此时将打开“选择要延伸的行”对话框。

    ![定义筛选器函数][StretchWizardImage2a]

2.  在“选择要延伸的行”对话框中，选择“选择行”。

3.  在“名称”字段中，提供筛选器函数的名称。

4.  针对 **Where** 子句，从表中选择某列，选择一个运算符，并提供一个值。

5. 单击“检查”以测试该函数。如果该函数返回表中的结果（也就是说，如果有要迁移的行满足条件），则测试会报告“成功”。

    >   [AZURE.NOTE] 显示筛选器查询的文本框是只读的。你无法在文本框中编辑查询。

6.  单击“完成”返回到“选择表”页面。

只有在完成向导后，才会在 SQL Server 中创建筛选器函数。在此之前，你可以返回到“选择表”页面，以更改或重命名筛选器函数。

![定义筛选器函数之后的“选择表”页][StretchWizardImage2b]  


如果想使用不同类型的筛选器函数来选择要迁移的行，请执行下列操作之一。

-   退出向导并运行 ALTER TABLE 语句，以便为表启用延伸，并指定筛选器函数。有关详细信息，请参阅[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)。

-   退出向导后，运行 ALTER TABLE 语句，以便指定筛选器函数。有关所需步骤，请参阅[运行向导后添加筛选器函数](/documentation/articles/sql-server-stretch-database-predicate-function/#addafterwiz)。

## <a name="Configure"></a>配置 Azure 部署

1.  使用 Microsoft 帐户登录到 Azure。

    ![登录到 Azure - Stretch Database 向导][StretchWizardImage3]

2.  选择用于 Stretch Database 的现有 Azure 订阅。

3.  选择一个 Azure 区域。
    -   如果你要创建新服务器，该服务器将在此区域中创建。
    -   如果所选区域中已有服务器，向导会在你选择“现有服务器”时列出这些服务器。

4.  指定是要使用现有的服务器还是创建新的 Azure 服务器。

    如果 SQL Server 上的 Active Directory 已与 Azure Active Directory 联合，则你可以选择使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。如需详细了解此选项的要求，请参阅 [ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)。

	-   **创建新服务器**

        1.  为服务器管理员创建登录名和密码。

        2.  （可选）使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。

		![创建新的 Azure 服务器 - Stretch Database 向导][StretchWizardImage4]  


    -   **现有服务器**

        1.  选择现有的 Azure 服务器。

        2.  选择身份验证方法。

            -   如果选择“SQL Server 身份验证”，请提供管理员登录名和密码。

            -   选择“Active Directory 集成身份验证”可以使用 SQL Server 的联合服务帐户来与远程 Azure 服务器通信。如果所选服务器未与 Azure Active Directory 集成，此选项就不会出现。

		![选择现有的 Azure 服务器 - Stretch Database 向导][StretchWizardImage5]  


## <a name="Credentials"></a>安全凭据
必须使用数据库主密钥来保护延伸数据库在连接到远程数据库时所用的凭据。

如果数据库主密钥已存在，请输入它的密码。

![Stretch Database 向导的安全凭据页][StretchWizardImage6b]

如果数据库没有现有的主密钥，请输入强密码以创建数据库主密钥。

![Stretch Database 向导的安全凭据页][StretchWizardImage6]

有关数据库主密钥的详细信息，请参阅 [CREATE MASTER KEY (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) 和[创建数据库主密钥](https://msdn.microsoft.com/zh-cn/library/aa337551.aspx)。有关向导创建的凭据的详细信息，请参阅 [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。

## <a name="Network"></a>选择 IP 地址
使用子网 IP 地址范围（建议）或 SQL Server 的公共 IP 地址，在 Azure 上创建一个可使 SQL Server 与远程 Azure 服务器通信的防火墙规则。

在此页上提供的一个或多个 IP 地址告知 Azure 服务器允许 SQL Server 发起的传入数据、查询和管理操作通过 Azure 防火墙。向导不会更改 SQL Server 上的任何防火墙设置。

![Stretch Database 向导的选择 IP 地址页][StretchWizardImage7]  


## <a name="Summary"></a>摘要
在 Azure 上查看在向导中输入的值和选择的选项以及预估成本。然后选择“完成”以启用延伸。

![Stretch Database 向导的摘要页][StretchWizardImage8]

## <a name="Results"></a>结果
查看结果。

若要监视数据迁移的状态，请参阅[数据迁移的监视和故障排除 (Stretch Database)](/documentation/articles/sql-server-stretch-database-monitor/)。

![Stretch Database 向导的“结果”页][StretchWizardImage9]

## <a name="KnownIssues"></a>排查向导问题
**延伸数据库向导失败。** 如果延伸数据库尚未在服务器级别启用，而你在不使用系统管理员权限的情况下运行向导来启用延伸数据库，则向导将会失败。请求系统管理员在本地服务器实例上启用延伸数据库，然后再次运行向导。有关详细信息，请参阅[先决条件：在服务器上启用 Stretch Database 所需的权限](/documentation/articles/sql-server-stretch-database-enable-database/#EnableTSQLServer)。

## 后续步骤
为延伸数据库启用其他表。监视数据迁移与管理已启用延伸的数据库和表。

-   参阅[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/) 来启用其他表。

-   参阅[数据迁移的监视和故障排除](/documentation/articles/sql-server-stretch-database-monitor/)来查看数据迁移的状态。

-   [暂停和恢复延伸数据库](/documentation/articles/sql-server-stretch-database-pause/)

-   [延伸数据库的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

-   [备份启用了延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)

## 另请参阅

[为数据库启用延伸数据库](/documentation/articles/sql-server-stretch-database-enable-database/)

[为表启用延伸数据库](/documentation/articles/sql-server-stretch-database-enable-table/)

[StretchWizardImage1]: ./media/sql-server-stretch-database-wizard/stretchwiz1.png
[StretchWizardImage2]: ./media/sql-server-stretch-database-wizard/stretchwiz2.png
[StretchWizardImage2a]: ./media/sql-server-stretch-database-wizard/stretchwiz2a.png
[StretchWizardImage2b]: ./media/sql-server-stretch-database-wizard/stretchwiz2b.png
[StretchWizardImage3]: ./media/sql-server-stretch-database-wizard/stretchwiz3.png
[StretchWizardImage4]: ./media/sql-server-stretch-database-wizard/stretchwiz4.png
[StretchWizardImage5]: ./media/sql-server-stretch-database-wizard/stretchwiz5.png
[StretchWizardImage6]: ./media/sql-server-stretch-database-wizard/stretchwiz6.png
[StretchWizardImage6b]: ./media/sql-server-stretch-database-wizard/stretchwiz6b.png
[StretchWizardImage7]: ./media/sql-server-stretch-database-wizard/stretchwiz7.png
[StretchWizardImage8]: ./media/sql-server-stretch-database-wizard/stretchwiz8.png
[StretchWizardImage9]: ./media/sql-server-stretch-database-wizard/stretchwiz9.png

<!---HONumber=Mooncake_Quality_Review_0104_2017-->