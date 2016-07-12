<properties
	pageTitle="排查 Azure SQL 数据库的“服务器上的数据库当前不可用”错误"
	description="识别和解决 Azure SQL 数据库连接错误的步骤。"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/29/2016"
	wacn.date="05/16/2016"/>

# 排查“&lt;x&gt;服务器上的&lt;y&gt;数据库当前不可用。请稍后重试连接”错误
[AZURE.INCLUDE [support-disclaimer](../includes/support-disclaimer.md)]

当应用程序连接到 Azure SQL 数据库时，你将收到以下错误消息：

	
	Error code 40613: "Database <x> on server <y> is not currently available. Please retry the connection later. If the problem persists, contact customer support, and provide them the session tracing ID of <z>"


> [AZURE.NOTE] 此错误消息通常是暂时的（生存期较短）。

正在移动（或重新配置）Azure 数据库时发生该错误，并且你的应用程序失去与 SQL 数据库的连接。SQL 数据库重新配置事件发生是由于计划内事件（例如软件升级）或计划外事件（例如进程崩溃或负载平衡）。大多数重新配置事件的生存期通常较短，并且应在最多 60 秒内完成。但是，偶尔这些事件可能需要更长时间才能完成，例如当一个大型事务导致长时间运行的恢复时。

## 解决暂时性连接问题的步骤
1.	检查 [Azure 服务仪表板](https://azure.microsoft.com/status)在由应用程序报告错误期间是否发生任何已知的服务中断。
2. 连接到云服务的应用程序（如 Azure SQL 数据库）应期望定期重新配置事件并实施重试逻辑来处理这些错误，而不是将它们作为应用程序错误展现给用户。请查看[暂时性错误](/documentation/articles/sql-database-connectivity-issues/)部分和[最佳实践和设计指南](/documentation/articles/sql-database-connect-central-recommendations/)，了解详细信息和常规重试策略。然后参阅[代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples/)以了解具体操作。
3.	由于数据库即将达到其资源限制，因此错误看起来像是暂时性连接问题。
4.	如果连接问题继续，或者应用程序遇到错误的持续时间超过 60 秒，或者在给定日期内看到错误多次发生，请通过选择[Azure 支持](/support/contact)站点上的“获取支持”提出 Azure 支持请求。

## 后续步骤
- 如果你收到其他错误，请评估[错误信息](/documentation/articles/sql-database-develop-error-messages/)，以便获取有关原因的线索。
- 如果问题持续，请访问[排查 SQL Azure 数据库的常见连接问题](/documentation/articles/sql-database-troubleshoot-common-connection-issues/)中的指南。

<!---HONumber=Mooncake_0509_2016-->
