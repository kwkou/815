<properties 
	pageTitle="SQL Database 审核入门 - Azure 
	description="SQL Database 审核入门" 
	services="sql-database" 
	documentationCenter="" 
	authors="jeffgoll" 
	manager="jeffreyg" 
	editor=""/>
<tags ms.service="sql-database"
    ms.date="02/23/2015"
    wacn.date="04/15/2015"
    />

 
#SQL Database 审核入门 
Azure SQL Database 审核可以跟踪数据库事件，并将审核的事件写入 Azure 存储帐户中的审核日志。一般而言，可以在基本、标准和高级服务层中使用审核功能。

审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。 

审核工具有助于遵从法规标准，但不能保证遵从法规。有关可帮助你遵从标准的 Azure 计划的详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/support/trust-center/compliance/" target="_blank">Azure 信任中心</a>。

+ [Azure SQL Database 审核基础知识]
+ [为数据库设置审核]
+ [分析审核日志和报告]

##<a id="subheading-1"></a>Azure SQL Database 审核基础知识

可以在 Azure 预览版门户中设置审核，但是，不管数据库是使用 Azure 门户还是 Azure 预览版门户创建的，这种设置都没有差别。SQL Database 审核可让你：

- **保留**选定事件的审核跟踪。定义要记录的数据库操作和事件类别。
- **报告**数据库活动。使用预配置的报告和仪表板快速开始使用活动和事件报告。
- **分析**报告。查找可疑事件、异常活动和趋势。

可以审核以下活动和事件：

- **对数据的访问**
- **架构更改 (DDL)**
- **数据更改 (DML)**
- **帐户、角色和权限 (DCL)**
- **安全异常**

有关记录的活动和事件的更多详细信息，请参阅<a href="http://download.microsoft.com/download/D/8/D/D8D90BA1-977F-466B-A839-7823FF37FD02/03-Azure%20SQL%20DB%20Audit%20Logs%20Format%20Specification.docx" target="_blank">审核日志格式参考（doc 文件下载）</a>。 

你还可以选择要将审核日志保存到的存储帐户。

###已启用安全性的连接字符串
当你设置审核时，Azure 将为你提供已启用安全性的数据库连接字符串。系统只会记录使用此连接字符串的客户端应用程序的活动和事件，因此，你需要更新现有的客户端应用程序才能使用新的字符串格式。

传统的连接字符串格式：<*服务器名称*>.database.chinacloudapi.cn

已启用安全性的连接字符串：<*服务器名称*>.database.**secure**.chinacloudapi.cn


##<a id="subheading-2"></a>为数据库设置审核

1. 启动 Azure 门户</a> ( https://manage.windowsazure.cn/ )。 请参考以下详细信息。
2. 导航到你要审核的数据库配置边栏。向下滚动到"操作"部分，然后单击"审核"以启用审核并启动审核配置边栏。

	![][1]

3. 在审核配置边栏中，选择要将日志保存到的 Azure 存储帐户。**提示：**为所有审核的数据库使用相同的存储帐户，以充分利用预配置的报告模板。

	![][2]

4. 在"审核选项"下，单击"全部"以记录所有事件，或选择单个事件类型。

	![][3]

5. 如果你要将这些设置应用到服务器上的所有未来数据库以及尚未设置审核的数据库，请选中"将这些设置保存为服务器默认值"。以后可以遵循相同的步骤覆盖每个数据库的这些设置。 

6. 单击"显示数据库连接字符串"，然后复制或记下应用程序的已启用安全性的相应连接字符串。在任何你想审核其活动的客户端应用程序中使用此字符串。

	![][5]

7. 单击"确定"。



##<a id="subheading-3">分析审核日志和报告</a>

审核日志将在安装期间选择的 Azure 存储帐户中名为 **AuditLogs** 的单个 Azure 存储表内进行聚合。你可以使用工具（例如 <a href="http://azurestorageexplorer.codeplex.com/" target="_blank">Azure 存储资源管理器</a>）查看日志文件。

预配置的仪表板报告模板作为<a href="http://download.microsoft.com/download/D/8/D/D8D90BA1-977F-466B-A839-7823FF37FD02/01-Azure%20SQL%20DB%20Audit%20Logs%20Report%20Template.xlsx" target="_blank">可下载的 Excel 电子表格</a>提供，可让你快速分析日志数据。若要对审核日志使用模板，需要安装可从<a href="http://www.microsoft.com/zh-CN/download/details.aspx?id=39379">此处</a>下载的 Excel 2013 或更高版本以及 Power Query。 

该模板包含虚构的示例数据，你可以将 Power Query 设置为直接从 Azure 存储帐户导入审核日志。 

有关使用报告模板的详细说明，请阅读<a href="http://download.microsoft.com/download/D/8/D/D8D90BA1-977F-466B-A839-7823FF37FD02/02-Azure%20SQL%20DB%20Audit%20Logs%20Excel%20Report%20How-To.docx">操作指南（文档下载）</a>。

![][6]


##<a id="subheading-4"></a>使用经典 Azure 门户为数据库设置审核

1. 启动<a href= "https://manage.windowsazure.cn/" target="_bank">经典 Azure 门户</a> ( https://manage.windowsazure.cn/ )。 
2. 单击要审核的数据库，然后单击"审核与安全性预览"选项卡。
3. 在审核部分中，单击"启用"。

![][7]

4. 根据需要编辑"事件类型"。

![][8]

5. 选择"存储帐户"。
6. 单击"保存"。
7. 单击连接字符串对应的"显示受保护的连接字符串"。


##<a id="subheading-3">生产环境中的用法实践</a>
本部分中的说明与以上屏幕截图相关。使用<a href= "https://manage.windowsazure.cn/" target="_bank">Azure 门户</a>。
 

##<a id="subheading-4"></a>已启用安全性的访问

在生产环境中，你可能需要审核从所有应用程序和工具传往数据库的所有流量。为此，请将"已启用安全性的访问"从"可选"修改为"必需"，然后保存策略。配置为"必需"后，用户将无法选择通过原始连接字符串访问数据库，而只能通过已启用安全性的连接字符串进行访问。


![][9]


##<a id="subheading-4"></a>重新生成存储密钥

在生产环境中，你可能会定期刷新存储密钥。审核服务不会保留你的存储帐户密钥。保存后，系统会针对审核表生成一个只写共享访问签名 (SAS) 密钥（只有客户能够读取审核日志）。因此，刷新密钥时，你需要重新保存策略。过程如下：


1. 在审核配置边栏中（如以上有关设置审核的部分中所述），将"存储访问密钥"从"主要"切换为"辅助"，然后单击"保存"。
![][10]
2. 转到存储配置边栏，并**重新生成** *主要访问密钥*。

3. 返回审核配置边栏，将"存储访问密钥"从"辅助"切换为"主要"，然后单击"保存"。

4. 返回存储 UI 并**重新生成** *辅助访问密钥*（为下一个密钥刷新周期做好准备）。
  
##<a id="subheading-4"></a>自动化
可以使用多个 PowerShell cmdlet 来配置 Azure SQL Database 中的审核。若要访问审核 cmdlet，你必须以 Azure 资源管理器模式运行 PowerShell。

> [WACOM.NOTE] AzureResourceManager 模块目前以预览版提供。它可能未提供与 Azure 模块相同的管理功能。

可通过运行 Switch-AzureMode cmdlet 来访问  [Azure 资源管理器](https://msdn.microsoft.com/zh-cn/library/dn654592.aspx)模式 (`Switch-AzureMode AzureResourceManager`)。当你处于 Azure 资源管理器模式时，运行 `Get-Command *AzureSql*` 可以列出可用的 cmdlet。








<!---- Anchors ---->
[Azure SQL Database 审核基础知识]: #subheading-1

[为数据库设置审核]: #subheading-2

[分析审核日志和报告]: #subheading-3

[使用经典 Azure 门户为数据库设置审核]: #subheading-4


<!--Image references-->

[1]:  ./media/sql-database-auditing-get-started/sql-database-get-started-auditingpreview.png

[2]: ./media/sql-database-auditing-get-started/sql-database-get-started-storageaccount.png

[3]: ./media/sql-database-auditing-get-started/sql-database-auditing-eventtype.png

[5]: ./media/sql-database-auditing-get-started/sql-database-get-started-connectionstring.png

[6]: ./media/sql-database-auditing-get-started/sql-database-auditing-dashboard.png

[7]: ./media/sql-database-auditing-get-started/sql-database-auditing-classic-portal-enable.png

[8]: ./media/sql-database-auditing-get-started/sql-database-auditing-classic-portal-configure.png

[9]: ./media/sql-database-auditing-get-started/sql-database-auditing-security-enabled-access.png

[10]: ./media/sql-database-auditing-get-started/sql-database-auditing-storage-account.png






<!--Link references-->
[另一个 azure.microsoft.com 文档主题的链接 1]: /documentation/articles/virtual-machines-windows-tutorial/
[另一个 azure.microsoft.com 文档主题的链接 2]: /documentation/articles/web-sites-custom-domain-name/
[另一个 azure.microsoft.com 文档主题的链接 3]: /documentation/articles/storage-whatis-account/


<!--HONumber=50-->