<properties
	pageTitle="计划升级到 SQL 数据库 V12 | Windows Azure"
	description="介绍升级到 Azure SQL 数据库 V12 版本所涉及的准备工作和限制。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>


# 规划和准备升级到 SQL 数据库 V12


本主题介绍将 Azure SQL 数据库从 V11 版本升级到 V12 时必须执行的规划和准备工作。


我们提供了新的 [Microsoft Azure 门户](https://manage.windowsazure.cn)来支持你升级到 V12。


下表列出了有关 V12 的其他帮助主题。


| 标题和链接 | 内容说明 |
| :--- | :--- |
| [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new) | 介绍 V12 如何使 Azure SQL 数据库几乎能够完全与 Microsoft SQL Server 兼容。 |
| [演练：注册最新的 SQL 数据库更新版 V12](/documentation/articles/sql-database-v12-sign-up) | 介绍将 Azure SQL 数据库 s 升级到 V12 时必须执行的步骤。 |
| [在 SQL 数据库更新版 V12 中创建数据库](/documentation/articles/sql-database-create) | 介绍如何在版本 V12 中创建新的 Azure SQL 数据库。其中介绍了各个选项，而不仅仅是如何创建空数据库。 |


## 提前规划


以下小节介绍了在采取行动将 Azure SQL 数据库升级到 V12 之前，必须了解的知识和做出的决策。

### 版本说明


本文档涉及从 Windows Azure SQL 数据库 版本 V11 到 V12 的升级。更准确地讲，版本号非常类似于 Transact-SQL 语句 **SELECT @@version;** 报告的以下两个值：


- 12\.0.2000.8 *(or a bit higher, V12)*
- 11\.0.9228.18 *(V11)*
 - V11 有时也称为 SAWA v2。

### 服务层规划


从 V12 开始，Azure SQL 数据库仅支持名为“基本”、“标准”和“高级”的服务层。V12 不支持 Web 和企业服务层。因此，如果你打算升级当前位于 Web 或企业服务层的 Azure SQL 数据库，必须决定该数据库要位于哪个新的服务层。


有关基本、标准和高级服务层的详细信息，请参阅：


- [将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-new-service-tiers)
- [Azure SQL 数据库定价](/home/features/sql-database/#price)


### 查看地域复制配置


如果你的 Azure SQL 数据库已针对地域复制进行配置，则你应记录其当前配置并停止地域复制，然后才能启动升级准备操作。升级完成后，你必须重新为地域复制配置数据库。


策略是将源保持不变，并在数据库的副本上进行测试。


## 准备操作


完成规划后，你可以执行以下小节中描述的操作步骤，以准备最后的升级阶段。


链接到本帮助主题顶部的帮助主题中提供了最终升级阶段的详细说明。


### 服务层操作


V12 不支持 Web 和企业服务定价层。


如果你的 V11 Azure SQL 数据库是 Web 或企业数据库，则升级过程允许你将数据库切换到受支持的层。升级过程会根据你的数据库工作负载历史记录推荐一个层。但是，你可以根据需要选择任何受支持的层。


在开始升级之前，通过将 V11 数据库从 Web 和企业层切换到其他层，可以减少升级过程中需要执行的步骤数。可以使用新的 [Windows Azure 门户](https://manage.windowsazure.cn)来实现此目的。


如果不确定要切换到哪个服务层，标准层的 S2 级别可能是合理的初始选择。更低的任何层具有的资源比 Web 和企业层要少。


### 升级过程中暂停地域复制


如果地域复制在数据库上处于活动状态，将无法运行升级到 V12 的步骤。首先必须重新配置你的数据库，以停止使用地域复制。


在升级完成后，你可以将数据库配置为再次使用地域复制。


### Azure VM 上的客户端


如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开 VM 上的以下端口范围：

- 11000-11999
- 14000-14999


单击[此处](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)可了解有关 SQL 数据库 V12 的端口的详细信息。SQL 数据库 V12 中的性能增强功能需要这些端口。


##<a id="limitations"></a>升级到 V12 期间和之后的限制


### V12 的门户


Azure 有三个门户，每个门户针对 SQL 数据库 V12 提供不同的功能。


- [http://manage.windowsazure.cn](http://manage.windowsazure.cn/)<br/>此 Azure 预览门户是新门户，仍处于预览状态。此门户尚未完全正式发布 (GA)。此门户：
 - 可以管理 V12 服务器和数据库。
 - 可以将 V11 数据库升级到 V12。


- [http://manage.windowsazure.cn/](http://manage.windowsazure.cn/)<br/>此 Azure 门户最终可能会淘汰。此门户：
 - 可以管理 V12 服务器和数据库。
 - *无法*将 V11 数据库升级到 V12。


- （http://*yourservername*.database.chinacloudapi.cn)<br/>
Azure SQL 数据库管理门户：
 - 无法管理 V12 服务器。


我们建议你使用 Visual Studio 2013 (VS2013) 连接到 Azure SQL 数据库。VS2013 可用于如下所述的任务：


- 运行 Transact-SQL 语句。
- 设计架构。
- 联机或脱机开发数据库。


你可以改为使用 [Visual Studio Community 2013](https://www.visualstudio.com/zh-cn/news/vs2013-community-vs.aspx)（免费但具有完整功能的 VS2013 版本）进行连接。


在旧版 Azure 门户上的数据库页中，可以单击“在 Visual Studio 中打开”，以便在计算机上启动 VS2013，从而与 Azure SQL 数据库 建立连接。


另一种方法是使用装有 [CU6](https://support.microsoft.com/zh-CN/kb/3031047) 的 SQL Server Management Studio (SSMS) 2014 来连接到 Azure SQL 数据库。以下博客文章提供了更多详细信息：<br/>[Azure SQL 数据库 的客户端工具更新](http://azure.microsoft.com/blog/2014/12/22/client-tooling-updates-for-azure-sql-database)。


### 升级到 V12 *期间*的限制


在升级到 V12 期间，V11 数据库仍然支持数据访问。但你要考虑到几个限制。


| 限制 | 说明 |
| :--- | :--- |
| 升级持续时间 | 升级持续时间取决于服务器中数据库的大小、版本和数量。如果服务器中的数据库存在以下情况，升级过程可能会持续几小时甚至几天：<br/><br/>* 大于 50 GB<br/>* 位于非高级服务层上<br/><br/>升级期间在服务器上创建新数据库也可能会延长升级持续时间。 |
| 没有地域复制 | 目前，从 V11 升级的 V12 服务器不支持地域复制。 |


### 升级到 V12 *之后*的限制


| 限制 | 说明 |
| :--- | :--- |
| 无法还原到 V11 | 执行就地升级之后，无法还原或撤消升级结果。 |
| Web 或企业层 | 在升级开始后，新 V12 数据库的服务器不再能够识别或接受 Web 或企业服务层。 |



### 升级到 V12 *之后*的导出和导入操作


你可以使用 [Windows Azure 门户](https://manage.windowsazure.cn)导出或导入 V12 数据库。或者使用下列任一工具进行导出或导入：


- SQL Server Management Studio (SSMS)
- Visual Studio 2013
- 数据层应用程序框架 (DacFx)


但是，若要使用这些工具，必须先安装其最新更新，以确保它们支持新的 V12 功能：


- [SQL Server Management Studio 2014 累积更新 6](http://support2.microsoft.com/kb/3031047)
- [Visual Studio 2013 中的 SQL Server 数据库工具 2015 年 2 月更新版](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [Azure SQL 数据库 V12 数据层应用程序框架 (DacFx) 2015 年 2 月版](https://www.microsoft.com/zh-CN/download/details.aspx?id=45886)


> [AZURE.NOTE] 前面的工具链接已在 2015 年 3 月 2 日或之后更新。我们建议你使用这些工具的较新更新版。


#### 自动导出


像在预览版中一样，自动导出继续可用。使用自动导出时，不需要进行客户端工具更新。如果你选择采用生成的文件并导入到本地服务器，则需要使用上面列出的工具更新，这样才能成功导入。


### 将已删除的 V11 数据库还原到 V12

以下方案说明了可以将已删除的 V11 Azure SQL 数据库还原到 V12 Azure SQL 数据库服务器。

1. 假设你有一个 V11 Azure SQL 数据库服务器。<br/> 在该服务器中，有一个数据库位于已弃用的 Web 和企业服务层上。
2. 删除该数据库。
3. 将服务器升级到 V12。
4. 接下来，将该数据库还原到服务器。<br/> 这样，该数据库便会升级到 V12，并位于标准服务层的 S0 级别上。
5. 如果 S0 不是你的首选级别，你可以将数据库切换到任何受支持的服务层。


### PowerShell cmdlet


可以使用 Powershell cmdlet 来启动、停止或监视从 Azure SQL 数据库 V11 或其他任何低于 V12 的版本到 V12 的升级。


有关这些 Powershell cmdlet 的参考文档，请参阅：


- [Get-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143621.aspx)
- [Start-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143623.aspx)
- [Stop-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143622.aspx)


Stop- cmdlet 表示取消，而不是暂停。你无法在中途恢复升级，而只能从头开始重新升级。Stop- cmdlet 将清理并释放所有相应的资源。


## 失败解决方法


如果任何奇怪的原因导致升级失败，V11 数据库将保持活动状态并像平常一样工作。


> [AZURE.NOTE]
> 在升级到 V12 期间，版本低于 V12 的数据库*仍然支持*数据访问。

<!--
## 相关链接


- Microsoft Azure [预览版功能](/home/features/preview/)
-->

<!--Anchors-->
[Subheading 1]: #subheading-1

<!---HONumber=76-->