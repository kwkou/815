<properties 
	pageTitle="SQL 数据库审核入门 | Azure" 
	description="SQL Database 审核入门" 
	services="sql-database" 
	documentationCenter="" 
	authors="jeffgoll" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" ms.date="02/23/2015" wacn.date="08/29/2015"/>
 
# SQL Database 审核入门 
Azure SQL Database 审核可以跟踪数据库事件，并将审核的事件写入 Azure 存储帐户中的审核日志。一般而言，可以在基本、标准和高级服务层中使用审核功能。

审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。

审核工具有助于遵从法规标准，但不能保证遵从法规。有关可帮助你遵从标准的 Azure 计划的详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/support/trust-center/compliance/" target="_blank">Azure 信任中心</a>。

+ [Azure SQL Database 审核基础知识] 
+ [为数据库设置审核]
+ [分析审核日志和报告]

##<a id="subheading-1"></a>Azure SQL 数据库审核基本信息

以下各节介绍如何使用 Azure 预览门户配置审核。你也可以[使用经典 Azure 门户为数据库设置审核]。

SQL 数据库审核可让你：

- **保留**选定事件的审核痕迹。可以定义要审核的数据库操作的类别。
- **报告**数据库活动。可以使用预配置的报告和仪表板快速开始使用活动和事件报告。
- **分析**报告。可以查找可疑事件、异常活动和趋势。

你可以为以下事件类别配置审核：

**普通 SQL** 和**参数化 SQL**，且为其收集的审核日志归类为

- **对数据的访问**
- **架构更改 (DDL)**
- **数据更改 (DML)**
- **帐户、角色和权限 (DCL)**

**存储过程**、**登录**和**事务管理**。

针对每个事件类别，分别配置**成功**和**失败**操作的审核。

有关审核的活动和事件的更多详细信息，请参阅<a href="http://go.microsoft.com/fwlink/?LinkId=506733" target="_blank">审核日志格式参考（doc 文件下载）</a>。

审核日志存储在 Azure 存储帐户中。你可以定义审核日志的保持期。

可以为特定数据库定义审核策略，也可以将审核策略定义为默认服务器策略。默认服务器审核策略将应用于服务器上所有未定义特定重写数据库审核策略的数据库。

在设置审核之前，请检查是否正在使用[“下层客户端”](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients)。


##<a id="subheading-2"></a>为数据库设置审核

1. 启动 <a href="https://manage.windowsazure.cn" target="_blank">Azure 预览门户</a> (https://manage.windowsazure.cn)。或者，你也可以启动<a href= "https://manage.windowsazure.cn/" target="_bank">经典 Azure 门户</a> (https://manage.windowsazure.cn/)。请参考以下详细信息。
2. 导航到你要审核的 SQL 数据库/SQL Server 的配置边栏选项卡。单击顶部的“设置”按钮，然后在“设置”边栏选项卡中选择“审核”。

	![][1]

3. 在审核配置边栏选项卡中，选择“存储详细信息”以打开“审核日志存储”边栏选项卡。选择要用于保存日志的 Azure 存储帐户以及保持期。**提示：**为所有审核的数据库使用相同的存储空间帐户，以便充分利用预配置的报告模板。

	![][2]

4. 在“按事件记录日志”下，单击“成功”和“失败”以记录所有事件，或者选择单个事件类别。


5. 如果要为 SQL 数据库配置审核，请单击“若要实施审核，请单击此处...”，并在“已启用安全的访问”中选择“必需”。如果要为 SQL Server 配置审核，有两个选项可供选择：(a) 在步骤 6 之后，导航到服务器上的每个 SQL 数据库，并应用此步骤，或者 (2) [修改连接字符串中的服务器 FDQN](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients)。


	![][5]

6. 单击**“确定”**。



##<a id="subheading-3">分析审核日志和报告</a>

审核日志将在设置期间选择的 Azure 存储帐户中前缀为 **SQLDBAuditLogs** 的一系列存储表内进行聚合。你可以使用工具（比如 <a href="http://azurestorageexplorer.codeplex.com/" target="_blank">Azure 存储资源管理器</a>）查看日志文件。

预配置的仪表板报告模板作为<a href="http://go.microsoft.com/fwlink/?LinkId=403540" target="_blank">可下载的 Excel 电子表格</a>提供，可帮助你快速分析日志数据。若要对审核日志使用模板，需要安装可从<a href="http://www.microsoft.com/zh-CN/download/details.aspx?id=39379">此处</a>下载的 Excel 2013 或更高版本以及 Power Query。

该模板包含虚构的示例数据，你可以将 Power Query 设置为直接从 Azure 存储帐户导入审核日志。

有关使用报告模板的更多详细说明，请阅读<a href="http://go.microsoft.com/fwlink/?LinkId=506731">操作说明（doc 下载）</a>。

![][6]


##<a id="subheading-4"></a>使用经典 Azure 门户为数据库设置审核

1. 启动<a href= "https://manage.windowsazure.cn/" target="_bank">经典 Azure 门户</a> (https://manage.windowsazure.cn/)。 
2.   单击要审核的 SQL 数据库/SQL Server，然后单击“审核和安全”选项卡。 
3.   如果要为 SQL 数据库配置审核，请在“已启用安全的访问”中选择“必需”。如果要为 SQL Server 配置审核，有两个选项可供选择：(a) 在步骤 7 之后，导航到服务器上的每个 SQL 数据库，并应用此步骤，或者 (2) [修改连接字符串中的服务器 FDQN](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients)。
4. 在审核部分中，单击“已启用”。

![][7]

5. 根据需要编辑“按事件记录日志”。

![][8]

6. 选择“存储帐户”并配置“保持期”。![][11]
7. 单击“保存”。



##<a id="subheading-3">生产环境中的用法实践</a>
本部分中的说明与以上屏幕截图相关。可以使用 <a href="https://manage.windowsazure.cn" target="_blank">Azure 预览门户</a>或<a href= "https://manage.windowsazure.cn/" target="_bank">经典 Azure 门户</a>。
 






##<a id="subheading-4"></a>重新生成存储密钥

在生产环境中，你可能会定期刷新存储密钥。刷新密钥时，你需要重新保存策略。过程如下：


1. 在审核配置边栏选项卡中（如以上有关设置审核的部分中所述），将“存储访问密钥”从“主”切换为“辅助”，然后单击“保存”。![][10]
2. 转到存储配置边栏选项卡，并**重新生成***主访问密钥*。

3. 返回审核配置边栏选项卡，将“存储访问密钥”从“辅助”切换为“主”，然后按“保存”。

4. 返回存储 UI 并**重新生成***辅助访问密钥*（为下一个密钥刷新周期做好准备）。
  
##<a id="subheading-4"></a>自动化
可以使用多个 PowerShell cmdlet 来配置 Azure SQL Database 中的审核。若要访问审核 cmdlet，你必须以 Azure 资源管理器模式运行 PowerShell。

> [AZURE.NOTE][Azure 资源管理器](https://msdn.microsoft.com/zh-cn/library/dn654592.aspx)模块目前以预览版提供。它可能未提供与 Azure 模块相同的管理功能。

当你处于 Azure 资源管理器模式时，运行 `Get-Command *AzureSql*` 可以列出可用的 cmdlet。







<!--Anchors-->
[Azure SQL Database 审核基础知识]: #subheading-1
[为数据库设置审核]: #subheading-2
[分析审核日志和报告]: #subheading-3
[使用经典 Azure 门户为数据库设置审核]: #subheading-4


<!--Image references-->
[1]: ./media/sql-database-auditing-get-started/sql-database-get-started-auditingpreview.png
[2]: ./media/sql-database-auditing-get-started/sql-database-get-started-storageaccount.png
[3]: ./media/sql-database-auditing-get-started/sql-database-auditing-eventtype.png
[5]: ./media/sql-database-auditing-get-started/sql-database-get-started-connectionstring.png
[6]: ./media/sql-database-auditing-get-started/sql-database-auditing-dashboard.png
[7]: ./media/sql-database-auditing-get-started/sql-database-auditing-classic-portal-enable.png
[8]: ./media/sql-database-auditing-get-started/sql-database-auditing-classic-portal-configure.png
[9]: ./media/sql-database-auditing-get-started/sql-database-auditing-security-enabled-access.png
[10]: ./media/sql-database-auditing-get-started/sql-database-auditing-storage-account.png
[11]: ./media/sql-database-auditing-get-started/sql-database-auditing-classic-portal-configure-storage.png






<!--Link references-->
[Link 1 to another azure.microsoft.com documentation topic]: /documentation/articles/virtual-machines-windows-tutorial/
[Link 2 to another azure.microsoft.com documentation topic]: /documentation/articles/web-sites-custom-domain-name/
[Link 3 to another azure.microsoft.com documentation topic]: /documentation/articles/storage-whatis-account/

<!---HONumber=67-->