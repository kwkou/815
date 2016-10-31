<properties
   pageTitle="Azure SQL 数据仓库中的审核 | Azure"
   description="Azure SQL 数据仓库中的审核入门"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.workload="data-management"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="09/24/2016" 
   wacn.date="10/31/2016"/>  


# Azure SQL 数据仓库中的审核

> [AZURE.SELECTOR]
- [审核](/documentation/articles/sql-data-warehouse-auditing-overview/)
- [威胁检测](/documentation/articles/sql-data-warehouse-security-threat-detection/)

SQL 数据仓库审核允许用户将数据库中的事件记录到 Azure 存储帐户中的审核日志。审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。SQL 数据仓库审核还与 Microsoft Power BI 集成，适用于向下钻取报告和分析数据。

审核工具有助于遵从法规标准，但不能保证遵从法规。有关可帮助你遵从标准的 Azure 计划的详细信息，请参阅 <a href="/support/trust-center/" target="_blank">Azure 信任中心</a>。

+ [数据库审核基础知识]
+ [为数据库设置审核]
+ [分析审核日志和报告]

##<a id="subheading-1"></a>Azure SQL 数据仓库数据库审核基础知识


SQL 数据仓库数据库审核可让你：

- **保留**选定事件的审核痕迹。可以定义要审核的数据库操作的类别。
- **报告**数据库活动。可以使用预配置的报告和仪表板快速开始使用活动和事件报告。
- **分析**报告。可以查找可疑事件、异常活动和趋势。

你可以为以下事件类别配置审核：

**普通 SQL** 和**参数化 SQL**，且为其收集的审核日志归类为

- **对数据的访问**
- **架构更改 (DDL)**
- **数据更改 (DML)**
- **帐户、角色和权限 (DCL)**
- **存储过程**、**登录**和**事务管理**。

针对每个事件类别，分别配置**成功**和**失败**操作的审核。

有关审核的活动和事件的更多详细信息，请参阅<a href="http://go.microsoft.com/fwlink/?LinkId=506733" target="_blank">审核日志格式参考（doc 文件下载）</a>。

审核日志存储在 Azure 存储帐户中。你可以定义审核日志的保持期。

可以为特定数据库定义审核策略，也可以将审核策略定义为默认服务器策略。默认服务器审核策略将应用于服务器上所有未定义特定重写数据库审核策略的数据库。

在设置审核之前，请检查是否正在使用[“下层客户端”](/documentation/articles/sql-data-warehouse-auditing-downlevel-clients/)。


##<a id="subheading-2"></a>为数据库设置审核

1. 启动 <a href="https://portal.azure.cn" target="_blank">Azure 门户预览</a>。

2. 导航到你要审核的 SQL 数据仓库数据库/SQL Server 的配置边栏选项卡。单击顶部的“设置”按钮，然后在“设置”边栏选项卡中选择“审核”。

	![][1]  


3. 在审核配置边栏选项卡中，首先取消选中“从服务器继承审核设置”复选框。这样，你便可以指定特定数据库的设置。

	![][2]  


4. 接下来，通过单击“打开”按钮启用审核。

	![][3]  


5. 在审核配置边栏选项卡中，选择“存储详细信息”以打开“审核日志存储”边栏选项卡。选择要用于保存日志的 Azure 存储帐户以及保持期。**提示：**为所有审核的数据库使用相同的存储帐户，以便充分利用预配置的报告模板。

	![][4]  


6. 单击“确定”按钮保存存储详细信息配置。


7. 在“按事件记录日志”下，单击“成功”和“失败”以记录所有事件，或者选择单个事件类别。


8. 如果你要为某个数据库配置审核，可能需要更改客户端的连接字符串，以确保正确捕获数据审核。对于下层客户端连接，请查看[修改连接字符串中的服务器 FDQN](/documentation/articles/sql-data-warehouse-auditing-downlevel-clients/) 主题。

9. 单击**“确定”**。


##<a id="subheading-3"></a>分析审核日志和报告

审核日志将在设置期间选择的 Azure 存储帐户中前缀为 **SQLDBAuditLogs** 的一系列存储表内进行聚合。你可以使用工具（比如 <a href="http://azurestorageexplorer.codeplex.com/" target="_blank">Azure 存储资源管理器</a>）查看日志文件。

预配置的仪表板报告模板作为<a href="http://go.microsoft.com/fwlink/?LinkId=403540" target="_blank">可下载的 Excel 电子表格</a>提供，可帮助你快速分析日志数据。若要对审核日志使用模板，需要安装可从<a href="http://www.microsoft.com/download/details.aspx?id=39379">此处</a>下载的 Excel 2013 或更高版本以及 Power Query。

该模板包含虚构的示例数据，你可以将 Power Query 设置为直接从 Azure 存储帐户导入审核日志。

有关使用报告模板的更多详细说明，请阅读<a href="http://go.microsoft.com/fwlink/?LinkId=506731">操作说明（doc 下载）</a>。

![][5]  



##<a id="subheading-4"></a>生产环境中的用法实践
本部分中的说明与以上屏幕截图相关。可以使用 <a href="https://portal.azure.cn" target="_blank">Azure 门户预览</a>或<a href= "https://manage.windowsazure.cn/" target="_bank"> Azure 经典门户</a>。


##<a id="subheading-5"></a>重新生成存储密钥

在生产环境中，你可能会定期刷新存储密钥。刷新密钥时，你需要重新保存策略。过程如下：


1. 在审核配置边栏选项卡中（如以上有关设置审核的部分中所述），将“存储访问密钥”从“主”切换为“辅助”，然后单击“保存”。![][4]
2. 转到存储配置边栏选项卡，并**重新生成***主访问密钥*。

3. 返回审核配置边栏选项卡，将“存储访问密钥”从“辅助”切换为“主”，然后按“保存”。

4. 返回存储 UI 并**重新生成***辅助访问密钥*（为下一个密钥刷新周期做好准备）。

##<a id="subheading-6"></a>自动化
可以使用多个 PowerShell cmdlet 来配置 Azure SQL 数据库中的审核。若要访问审核 cmdlet，你必须以 Azure 资源管理器模式运行 PowerShell。

> [AZURE.NOTE] [Azure 资源管理器](https://msdn.microsoft.com/zh-cn/library/dn654592.aspx)模块目前以预览版提供。它可能未提供与 Azure 模块相同的管理功能。

当你处于 Azure 资源管理器模式时，运行 `Get-Command *AzureSql*` 可以列出可用的 cmdlet。


<!--Anchors-->

[数据库审核基础知识]: #subheading-1
[为数据库设置审核]: #subheading-2
[分析审核日志和报告]: #subheading-3


<!--Image references-->
[1]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing.png
[2]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-inherit.png
[3]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-enable.png
[4]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-storage-account.png
[5]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-dashboard.png


<!--Link references-->

<!---HONumber=Mooncake_1024_2016-->