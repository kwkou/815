<properties
	pageTitle="排查 Azure SQL 数据库的常见连接问题"
	description="识别和解决 Azure SQL 数据库常见连接错误的步骤。"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/29/2016"
	wacn.date="06/14/2016"/>

# 排查 Azure SQL 数据库的常见连接问题
[AZURE.INCLUDE [support-disclaimer](../includes/support-disclaimer.md)]

Azure SQL 数据库的连接问题可以统括分类为以下类型之一：

- 暂时性错误（短期或间歇性）
- 持续性或非暂时性错误（定期重复发生的错误）

如果你的应用程序发生暂时性错误，请查看以下主题以获取有关如何排查这些错误并减少其发生频率的提示：

- [针对“服务器 &lt;y&gt; 上的数据库 &lt;x&gt; 不可用（错误：40613）”进行故障排除](/documentation/articles/sql-database-troubleshoot-connection/)
- [排查、诊断和防止 SQL 数据库中的 SQL 连接错误和暂时性错误](/documentation/articles/sql-database-connectivity-issues/)

如果应用程序一直无法连接到 SQL Azure 数据库，通常表示下列其中一项出现了问题：

- 防火墙配置。Azure SQL 数据库或客户端防火墙阻止了与 Azure SQL 数据库的连接。
- 客户端上重新配置了网络：例如，使用了新的 IP 地址或代理服务器。
- 用户失误：例如，连接参数（例如连接字符串中的服务器名称）键入错误。

## 解决永久性连接问题的步骤
1.	设置防火墙规则以允许客户端 IP 地址。
2.	在客户端与 Internet 之间的所有防火墙上，确保为出站连接打开端口 1433。查看 [Configure the Windows Firewall to Allow SQL Server Access（配置 Windows 防火墙以允许 SQL Server 访问）](https://msdn.microsoft.com/zh-cn/library/cc646023.aspx)，以获取更多指导。
3.	验证连接字符串和其他连接设置。参阅[连接问题主题](/documentation/articles/sql-database-connectivity-issues/#connections-to-azure-sql-database)中的“连接字符串”部分。
4.	在仪表板中检查服务运行状况。如果你认为没有区域性的中断，请参阅[在中断后恢复](/documentation/articles/sql-database-disaster-recovery/)，以了解恢复到新区域的步骤。
5.	如果仍存在连接问题，你可以通过在 [Azure 支持](/support/contact)站点上选择“获取支持”来发出 Azure 支持请求。

<!---HONumber=Mooncake_0606_2016-->
