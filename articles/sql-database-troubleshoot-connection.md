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
	ms.date="02/12/2015"
	wacn.date="03/29/2016"/>

# 排查“服务器上的数据库当前不可用。请稍后重试连接”和其他连接错误
“服务器 <servername> 上的数据库 <dbname> 当前不可用...”是 Azure SQL 数据库最常见的暂时性连接错误。暂时性连接错误的发生通常是由于计划内事件（例如软件升级）或计划外事件（例如进程崩溃）。这些错误的发生通常时间很短，从数秒到一分钟（最多）不等。如果你收到了其他错误，请查看[错误消息](/documentation/articles/sql-database-develop-error-messages)以找到原因的线索，确定问题是暂时性的还是永久性的，然后使用本主题中的指导予以解决。

## 解决暂时性连接问题的步骤
1.	请查看 [Azure 服务仪表板](https://azure.microsoft.com/status)以了解已知的中断。
2.	确保你的应用程序使用重试逻辑。有关常规重试策略，请参阅[连接问题](/documentation/articles/sql-database-connectivity-issues)和[最佳实践和设计准则](/documentation/articles/sql-database-connect-central-recommendations)。然后参阅[代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)以了解具体操作。
3.	由于数据库即将达到其资源限制，因此错误看起来像是暂时性连接问题。
4.	如果仍存在连接问题，你可以通过在 [Azure 支持](/support/contact)站点上选择“获取支持”来发出 Azure 支持请求。

## 解决永久性连接问题的步骤
如果应用完全无法连接，原因通常与 IP 和防火墙配置有关。这可能包括客户端上重新配置了网络（例如，使用了新的 IP 地址或代理）。连接参数拼写错误（例如连接字符串）也很常见。

1.	设置[防火墙规则](/documentation/articles/sql-database-configure-firewall-settings)以允许客户端 IP 地址。
2.	在客户端与 Internet 之间的所有防火墙上，确保为出站连接打开端口 1433。
3.	验证连接字符串和其他连接设置。参阅[连接问题主题](/documentation/articles/sql-database-connectivity-issues)中的“连接字符串”部分。
4.	在仪表板中检查服务运行状况。如果你认为没有区域性的中断，请参阅[在中断后恢复](/documentation/articles/sql-database-disaster-recovery)，以了解恢复到新区域的步骤。
5.	如果仍存在连接问题，你可以通过在 [Azure 支持](/support/contact)站点上选择“获取支持”来发出 Azure 支持请求。

<!---HONumber=Mooncake_0321_2016-->
