<properties
	pageTitle="SQL 数据库审核入门 | Azure"
	description="SQL 数据库审核入门"
	services="sql-database"
	documentationCenter=""
	authors="ronitr"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="04/11/2016"
	wacn.date="05/16/2016"/>
 
# SQL 数据库审核入门
Azure SQL 数据库审核可以跟踪数据库事件，并将审核的事件写入 Azure 存储帐户中的审核日志。一般而言，可以在基本、标准和高级服务层中使用审核功能。

审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。

审核工具有助于遵从法规标准，但不能保证遵从法规。有关可帮助你遵从标准的 Azure 计划的详细信息，请参阅 [Azure 信任中心](/support/trust-center/compliance)。

+ [Azure SQL 数据库审核基础知识]
+ [为数据库设置审核]
+ [分析审核日志和报告]

##<a id="subheading-1"></a>Azure SQL 数据库审核基本信息

以下部分介绍如何使用 Azure 经典管理门户配置审核。

SQL 数据库审核可让你：

- **保留**选定事件的审核痕迹。可以定义要审核的数据库操作的类别。
- **报告**数据库活动。可以使用预配置的报告和仪表板快速开始使用活动和事件报告。
- **分析**报告。可以查找可疑事件、异常活动和趋势。

> [AZURE.NOTE] 现在，你可以使用新的**威胁检测**功能（目前以预览版提供），针对可能表示出现安全威胁的异常数据库活动接收前瞻性的警报。可以在审核配置边栏选项卡中启用和配置威胁检测。

你可以为以下事件类别配置审核：

**普通 SQL** 和**参数化 SQL**，且为其收集的审核日志归类为

- **对数据的访问**
- **架构更改 (DDL)**
- **数据更改 (DML)**
- **帐户、角色和权限 (DCL)**
- **存储过程**、**登录**和**事务管理**。

针对每个事件类别，分别配置**成功**和**失败**操作的审核。

有关审核的活动和事件的更多详细信息，请参阅[审核日志格式参考（doc 文件下载）](http://go.microsoft.com/fwlink/?LinkId=506733)。

审核日志存储在 Azure 存储帐户中。你可以定义审核日志的保留期，保留期过后，将删除旧日志。

可以为特定数据库定义审核策略，也可以将审核策略定义为默认服务器策略。默认服务器审核策略将应用于服务器上所有未定义特定重写数据库审核策略的数据库。

在设置审核之前，请检查是否正在使用[“下层客户端”](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients/)。

##<a id="subheading-3"></a>分析审核日志和报告

审核日志将在设置期间选择的 Azure 存储帐户中前缀为 **SQLDBAuditLogs** 的一系列存储表内进行聚合。你可以使用工具（比如 [Azure 存储资源管理器](http://azurestorageexplorer.codeplex.com)）查看日志文件。

预配置的报告模板作为[可下载的 Excel 电子表格](http://go.microsoft.com/fwlink/?LinkId=403540)提供，可帮助你快速分析日志数据。若要对审核日志使用模板，需要安装可从[此处](http://www.microsoft.com/zh-cn/download/details.aspx?id=39379)下载的 Excel 2013 或更高版本以及 Power Query。

可以使用 Power Query 将审核日志从 Azure 存储帐户直接导入 Excel 模板。然后可以浏览审核记录，并在日志数据顶部创建仪表板和报表。


![导航窗格][4]


##<a id="subheading-4"></a>使用 Azure 经典管理门户为数据库设置审核

1. 通过 https://manage.windowsazure.cn/ 启动 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 单击要审核的 SQL 数据库/SQL Server，然后单击“审核和安全性”选项卡。

3. 将审核设置为“已启用”。

	![导航窗格][5]

4. 根据需要编辑“按事件记录日志”，以自定义要审核的事件。

	![导航窗格][6]

5. 选择“存储帐户”并配置“保持期”。

	![导航窗格][7]

6. 单击“保存”。


##<a id="subheading-6"></a>重新生成存储密钥

在生产环境中，你可能会定期刷新存储密钥。刷新密钥时，需要重新保存审核策略。过程如下：


1. 在审核配置边栏选项卡中，将“存储访问密钥”从“主要”切换为“辅助”，然后单击“保存”。

	![][8]

2. 转到存储配置边栏选项卡，并**重新生成**主访问密钥。

3. 返回审核配置边栏选项卡，将“存储访问密钥”从“辅助”切换为“主要”，然后单击“保存”。

4. 返回存储 UI 并**重新生成**辅助访问密钥（为下一个密钥刷新周期做好准备）。
  
##<a id="subheading-7"></a>自动化
可以使用多个 PowerShell cmdlet 来配置 Azure SQL 数据库中的审核：

- [Get-AzureRMSqlDatabaseAuditingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt603731.aspx)
- [Get-AzureRMSqlServerAuditingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt619329.aspx)
- [Remove-AzureRMSqlDatabaseAuditing](https://msdn.microsoft.com/zh-cn/library/azure/mt603796.aspx)
- [Remove-AzureRMSqlServerAuditing](https://msdn.microsoft.com/zh-cn/library/azure/mt603574.aspx)
- [Set-AzureRMSqlDatabaseAuditingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt603531.aspx)
- [Set-AzureRMSqlServerAuditingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt603794.aspx)
- [Use-AzureRMSqlServerAuditingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt619353.aspx)




<!--Anchors-->
[Azure SQL 数据库审核基础知识]: #subheading-1
[为数据库设置审核]: #subheading-2
[分析审核日志和报告]: #subheading-3
[使用 Azure 经典管理门户为数据库设置审核]: #subheading-4
[Practices for usage in production]: #subheading-5
[Storage Key Regeneration]: #subheading-6
[Automation]: #subheading-7


<!--Image references-->
[1]: ./media/sql-database-auditing-get-started/1_auditing_get_started_settings.png
[2]: ./media/sql-database-auditing-get-started/2_auditing_get_started_storage_account.png
[3]: ./media/sql-database-auditing-get-started/3_auditing_get_started_inherit_from_server.png
[4]: ./media/sql-database-auditing-get-started/4_auditing_get_started_report_template.png
[5]: ./media/sql-database-auditing-get-started/5_auditing_get_started_classic_portal_enable.png
[6]: ./media/sql-database-auditing-get-started/6_auditing_get_started_classic_portal_events.png
[7]: ./media/sql-database-auditing-get-started/7_auditing_get_started_classic_portal_storage.png
[8]: ./media/sql-database-auditing-get-started/8_auditing_get_started_storage_key_rotation.png


 

<!---HONumber=Mooncake_0503_2016-->