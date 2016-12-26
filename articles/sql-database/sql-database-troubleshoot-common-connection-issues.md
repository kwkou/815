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
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	wacn.date="12/26/2016"
	ms.author="daleche"/>  


# 排查 Azure SQL 数据库的连接问题

与 Azure SQL 数据库的连接失败时，收到[错误信息](/documentation/articles/sql-database-develop-error-messages/)。本文是一个集中化主题，可帮助用户对 Azure SQL 数据库连接问题进行故障排除。本文介绍连接问题的[常见原因](#cause)，推荐可帮助识别问题的[故障排除工具](#try-the-troubleshooter-for-azure-sql-database-connectivity-issues)，还提供解决[暂时性错误](#troubleshoot-transient-errors)和[永久或非暂时性错误](#troubleshoot-the-persistent-errors)的故障排除步骤。最后，本文列出了 [Azure SQL 数据库连接问题的所有相关文章](#all-topics-for-azure-sql-database-connection-problems)。

如果遇到连接问题，请尝试本文中介绍的故障排除步骤。

##<a name="cause"></a> 原因

连接问题可能由以下任何原因引起：

- 在应用程序设计过程中无法应用最佳实践和设计准则。参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)了解入门信息。
- Azure SQL 数据库重新配置
- 防火墙设置
- 连接超时
- 不正确的登录信息
- 一些 Azure SQL 数据库资源达到最大限制

通常，Azure SQL 数据库的连接问题可分类如下：

- [暂时性错误（短期或间歇性）](#troubleshoot-transient-errors)
- [持续性或非暂时性错误（定期重复发生的错误）](#troubleshoot-the-persistent-errors)

##<a id="try-the-troubleshooter-for-azure-sql-database-connectivity-issues"></a> 尝试使用 Azure SQL 数据库连接问题的故障排除工具

如果遇到特定的连接错误，请尝试使用[此工具](https://support.microsoft.com/help/10085/troubleshooting-connectivity-issues-with-microsoft-azure-sql-database)，以帮助快速识别并解决问题。

##<a id="troubleshoot-transient-errors"></a> 对暂时性错误进行故障排除
如果应用程序发生暂时性错误，请查看以下主题以获取有关如何排查这些错误并减少其发生频率的提示：

- [对“服务器 &lt;y&gt; 上的数据库 &lt;x&gt; 不可用”（错误：40613）进行故障排除](/documentation/articles/sql-database-troubleshoot-connection/)
- [排查、诊断和防止 SQL 数据库中的 SQL 连接错误和暂时性错误](/documentation/articles/sql-database-connectivity-issues/)

<a id="troubleshoot-the-persistent-errors" name="troubleshoot-the-persistent-errors"></a>

##<a id="troubleshoot-the-persistent-errors"></a> 对永久性错误（非暂时性错误）进行故障排除

如果应用程序一直无法连接到 Azure SQL 数据库，通常表示下列其中一项出现了问题：

- 防火墙配置。Azure SQL 数据库或客户端防火墙阻止了与 Azure SQL 数据库的连接。
- 客户端上重新配置了网络：例如，使用了新的 IP 地址或代理服务器。
- 用户失误：例如，连接参数（例如连接字符串中的服务器名称）键入错误。

### 解决永久性连接问题的步骤

1.	设置[防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/)以允许客户端 IP 地址。
2.	在客户端与 Internet 之间的所有防火墙上，确保为出站连接打开端口 1433。请参阅[配置 Windows 防火墙以允许 SQL Server 访问](https://msdn.microsoft.com/zh-cn/library/cc646023.aspx)，以获取更多指导。
3.	验证连接字符串和其他连接设置。请参阅[连接问题主题](/documentation/articles/sql-database-connectivity-issues/#connections-to-azure-sql-database)中的“连接字符串”部分。
4.	在仪表板中检查服务运行状况。如果认为存在区域性的中断，请参阅[从中断恢复](/documentation/articles/sql-database-disaster-recovery/)，以了解恢复到新区域的步骤。


##<a id="all-topics-for-azure-sql-database-connection-problems"></a> 有关 Azure SQL 数据库连接问题的所有主题

下表列出了直接适用于 Azure SQL 数据库服务的所有连接问题主题。


| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 1 | [排查 Azure SQL 数据库的连接问题](/documentation/articles/sql-database-troubleshoot-common-connection-issues/) | 这是对 Azure SQL 数据库中的连接问题进行故障排除的登陆页面。该页面介绍如何识别并解决 Azure SQL 数据库中的暂时性错误和永久性或非暂时性错误。 |
| 2 | [排查、诊断和防止 SQL 数据库中的 SQL 连接错误和暂时性错误](/documentation/articles/sql-database-connectivity-issues/) | 了解如何排查、诊断和防止 Azure SQL 数据库中的 SQL 连接错误或暂时性错误。 |
| 3 | 暂时性错误处理的一般指南 | 介绍连接到 Azure SQL 数据库时的暂时性错误处理的一般指南。 |
| 4 | [对 Azure SQL 数据库的连接问题进行故障排除](https://support.microsoft.com/help/10085/troubleshooting-connectivity-issues-with-microsoft-azure-sql-database) | 该工具可帮助识别并解决连接错误。 |
| 5 | [对错误“&lt;y&gt; 服务器上的 &lt;x&gt; 数据库当前不可用。请稍后重试连接”进行故障排除](/documentation/articles/sql-database-troubleshoot-connection/) | 介绍如何识别并解决 40613 错误：“&lt;y&gt; 服务器上的 &lt;x&gt; 数据库当前不可用。请稍后重试连接。” |
| 6 | [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题](/documentation/articles/sql-database-develop-error-messages/) | 介绍有关 SQL 数据库客户端应用程序的 SQL 错误代码的信息，例如常见的数据库连接错误、数据库复制问题和常规错误。 |
| 7 | [Azure SQL 数据库的单一数据库性能指导](/documentation/articles/sql-database-performance-guidance/) | 提供了帮助确定适用于用户应用程序的服务层的指导。还提供了调整应用程序以充分利用 Azure SQL 数据库的建议。 |
| 8 | [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/) | 提供了各项技术的代码示例的链接，可使用这些技术连接到 Azure SQL 数据库并与之进行交互。 |
| 9 | “升级到 Azure SQL 数据库 v12”页面（[Azure 门户预览](/documentation/articles/sql-database-upgrade-server-portal/)、[PowerShell](/documentation/articles/sql-database-upgrade-server-powershell/)） | 提供有关使用 Azure 门户预览或 PowerShell 将现有 Azure SQL 数据库 V11 服务器和数据库升级到 Azure SQL 数据库 V12 的指导。 |


## 后续步骤

- [对 Azure SQL 数据库性能问题进行故障排除](/documentation/articles/sql-database-troubleshoot-performance/)
- [对 Azure SQL 数据库权限问题进行故障排除](/documentation/articles/sql-database-troubleshoot-permissions/)
- [参阅有关 Azure SQL 数据库服务的所有主题](/documentation/articles/sql-database-index-all-articles/)



## 其他资源

- [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
- [SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->