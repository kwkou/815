<properties 
   pageTitle="更改数据库服务层和性能级别" 
   description="了解如何使用 Azure SQL 数据库 的服务层向上和向下缩放云数据库。使用服务层中可以根据业务和成本要求上下调节性能，且不会造成应用程序中断服务。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jeffreyg" 
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services" 
   ms.date="04/02/2015"
   wacn.date="05/25/2015"
   ms.author="sstein"/>


# 更改数据库服务层和性能级别 

本主题介绍有关在服务层和性能级别之间移动 Azure SQL 数据库 的步骤。

## 更改服务层 

使用[将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-new-service-tiers)和 [Azure SQL 数据库 服务层和性能级别](https://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)中的信息，确定适合你的 Azure SQL 数据库 的服务层和性能级别。

你可以使用 Azure 管理门户、[PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/dn546726.aspx) 或 [REST API](https://msdn.microsoft.com/zh-cn/library/dn505719.aspx) 在任何服务层之间轻松转移。

在服务层之间转移时，请考虑以下事项：
- 在服务层或性能级别之间升级之前，请确保你在服务器上有可用配额。如果需要更多的配额，请致电客户支持。
- 联合数据库无法升级到基本、标准或高级服务层。

- 若要降级数据库，数据库应小于目标服务层允许的最大大小。有关每个服务层允许的数据库大小的详细信息，请参阅本部分后面的服务层和数据库大小表。

- 从高级服务层降级时，你必须首先终止所有地域复制关系。可以执行[终止连续复制关系](https://msdn.microsoft.com/zh-cn/library/azure/dn741323.aspx)主题中所述的步骤来停止在主数据库与活动辅助数据库之间进行的复制过程。

- 还原服务解决方案根据服务层的不同而异。如果你要降级，则以后可能无法还原到时间点，或者会缩短备份保留期。有关详细信息，请参阅 [Azure SQL 数据库 备份和还原](https://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)。

- 在 24 小时期限内，最多可以单独进行四次数据库更改（服务层或性能级别）。

- 在更改完成之前，数据库的新属性不会应用。

请注意以下事项：
- 企业和 Web 服务层将于 2015 年 9 月停用。有关详细信息，请参阅 [Web 和企业版停用常见问题](https://msdn.microsoft.com/zh-cn/library/azure/dn741330.aspx)。

<注意事项>
- 必须注意，[联合的当前实现将随 Web 和企业服务层一起停用](https://msdn.microsoft.com/zh-cn/library/azure/dn741330.aspx)。建议你使用 [Azure SQL 数据库 的弹性缩放](/documentation/articles/sql-database-elastic-scale-get-started)在 Azure SQL 数据库 上生成分片的向外扩展解决方案。若要试用弹性缩放，请参阅"Azure SQL 数据库 弹性缩放预览版入门"。

## 升级到更高的服务层
使用以下方法之一可以升级数据库。这些步骤专门用于升级到高级服务层，不过也适用于所有升级。

### 使用 Azure 管理门户
1. 使用你的 Microsoft 帐户登录到 Azure 管理门户。
2. 导航到"SQL 数据库"选项卡。
3. 从"数据库"列表中选择数据库。这将在"数据库仪表板"或"快速启动"页上打开该数据库。
4. 为数据库选择"缩放"选项卡。
5. 在"常规"部分下，为服务层选择"高级"。
6. 对于"性能级别"，请选择"P1"、"P2"或"P3"。为每个性能级别提供支持的资源以 DTU 表示。有关性能级别和 DTU 的详细信息，请参阅"Azure SQL 数据库 服务层和性能级别"
8. 在屏幕底部的命令栏中，单击"保存"按钮。
9. 将为你显示"确认"。阅读提供的信息，然后选中该复选框进行确认。


### 使用 Azure PowerShell
1. 使用 Set-AzureSqlDatabase 指定数据库的性能级别、最大数据库大小和服务层。有关不同服务层支持的数据库大小的列表，请参阅"Azure SQL 数据库 服务层（版本）"。
2. 使用 New-AzureSqlDatabaseServerContext cmdlet 设置服务器上下文。"使用 Azure PowerShell 命令"部分中提供了示例语法。
3. 获取数据库和目标性能级别的句柄。使用 Set-AzureSqlDatabase -ServiceObjective 指定性能级别

**用法示例**
在此示例中：
- 本示例演示如何升级到高级服务层。
- 创建了 $db 句柄，它指向数据库名称"somedb"。
- 创建了指向高级性能级别 1 的 $P1 句柄。
- 数据库 $db 的性能级别已设置为 $P1。

		Windows PowerShell:

		$db = Get-AzureSqlDatabase $serverContext -DatabaseName "somedb"

		$P1= Get-AzureSqlDatabaseServiceObjective $serverContext -ServiceObjectiveName "P1"

		Set-AzureSqlDatabase $serverContext -Database $db -ServiceObjective $P1 -Edition Premium



## 降级到更低的服务层
使用以下方法之一可将数据库降级到更低的服务层：

### 使用 Azure 管理门户
1. 使用你的 Microsoft 帐户登录到 Azure 管理门户。
2. 导航到"SQL 数据库"选项卡。
3. 为所需数据库选择"缩放"选项卡。
4. 在"常规"部分下，选择你要降级到的服务层。
5. 在屏幕底部的命令栏中，单击"保存"按钮。
6. 如果适用，请在"确认"页上，阅读提供的信息并选中复选框以确认更改。

### 使用 Azure PowerShell
1. 使用 Set-AzureSqlDatabase 指定数据库的服务层、性能级别和最大大小。
2. 使用 New-AzureSqlDatabaseServerContext 来设置服务器上下文（可以参考"使用 Azure PowerShell 命令"部分中提供的示例语法）。
3. 执行以下操作：
 - 获取数据库的句柄。
 - 获取性能级别的句柄。
 - 使用 Set-AzureSqlDatabase -ServiceObjective 指定数据库的服务层、性能级别和最大大小。

**用法示例**

本示例演示如何将高级服务层数据库降级到标准服务层数据库：

- 创建了 $db 句柄，它指向数据库名称"somedb"。

- 创建了指向标准性能级别 S2 的变量 $S2。

- 数据库 $db 的性能级别已设置为 $S2。

- 使用 -Edition 和 -MaxSizeGB 参数指定数据库服务层以及数据库的最大大小。为 -MaxSizeGB 参数指定的值必须对目标服务层有效。本主题前面部分提供了包含每个服务层的 MaxSize 值的表格。

		Windows PowerShell:

		$db = Get-AzureSqlDatabase $serverContext -DatabaseName "somedb"

		$S2 = Get-AzureSqlDatabaseServiceObjective $serverContext -ServiceObjectiveName "S2"

		Set-AzureSqlDatabase $serverContext -Database $db -ServiceObjective $S2 -Edition Standard -MaxSizeGB 40

## 更改性能级别
可以使用以下方法之一来提高或降低标准数据库或高级数据库的性能级别。更改数据库的性能级别可能需要一些时间。有关详细信息，请参阅后面的[高级数据库更改的影响](https://msdn.microsoft.com/zh-cn/library/azure/dn369872.aspx#Impact)部分。

如果你要更改已配置活动地域复制关系的高级数据库的性能级别，请针对主数据库和活动辅助数据库按以下顺序操作：

这是因为，活动辅助数据库的性能级别必须等于或大于主数据库的性能级别。
- 如果要从更高的性能级别更改为更低的性能级别，请先从主数据库开始更改，然后再更改一个或多个活动辅助数据库的性能级别。

- 如果要从更低的性能级别更改为更高的性能级别，请先从活动辅助数据库开始更改，最后再更改主数据库的性能级别。

### 使用 Azure 管理门户
1. 使用你的 Microsoft 帐户登录到 Azure 管理门户。
2. 导航到"SQL 数据库"选项卡。
3. 从该帐户或特定服务器的"数据库"列表中选择一个数据库。这将在"数据库仪表板"或"快速启动"页上打开该数据库。
5. 为数据库选择"缩放"选项卡。
6. 对于"性能级别"选项，请选择一个性能级别。
7. 在屏幕底部的命令栏中，单击"保存"按钮

### 使用 Azure PowerShell
1. 使用 Set-AzureSqlDatabase 指定数据库的性能级别。
2. 使用 New-AzureSqlDatabaseServerContext cmdlet 设置服务器上下文。"使用 Azure PowerShell 命令"部分中提供了示例语法。
3. 执行以下操作：
- 获取数据库的句柄。
- 获取性能级别的句柄。
- 使用 Set-AzureSqlDatabase -ServiceObjective 指定性能级别。

**用法示例**

在此示例中：

- 创建了 $db 句柄，它指向数据库名称"somedb"。

- 创建了指向高级性能级别 2 的 $P2 句柄。

- 数据库 $db 的性能级别已设置为 $P2。

		Windows PowerShell
		$db = Get-AzureSqlDatabase $serverContext -DatabaseName "somedb"

		$P2 = Get-AzureSqlDatabaseServiceObjective $serverContext -ServiceObjectiveName "P2"

		Set-AzureSqlDatabase $serverContext -Database $db -ServiceObjective $P2

## 数据库更改的影响
本部分提供有关升级到标准或高级服务层，或者对数据库做出性能级别更改所造成的影响的信息。

### 在数据库更改期间处理应用程序连接
在完成性能级别更改或升级/降级时，与数据库的连接可能会暂时断开，并且可能需要几秒钟才能重新建立连接。SQL 数据库 应用程序的代码应当具有从断开的连接恢复的弹性，因为当数据中心内的计算机出现故障，SQL 数据库 服务对数据库执行故障转移时，SQL 数据库 中随时会出现这种情况。应用程序不需要执行实现方面的更改，即可使用高级数据库或更改高级数据库性能级别。

### 了解和估算数据库更改的延迟
数据库的 SLO 更改通常会涉及数据移动，因此可能需要几个小时，更改请求才能完成，关联的计费更改才会生效。在升级或降级数据库时，将会因为更改而发生数据移动；在更改数据库的性能级别时，也有可能会发生数据移动。

**涉及数据移动的 SLO 更改的延迟**

确定数据库的存储大小后，可按以下启发式方法估算 SLO 更改请求的延迟：

3 x (5 分钟 + 数据库大小 / 150 MB/分钟)

例如，如果数据库大小为 50 GB，可按以下启发式方法估算 SLO 更改请求的延迟：

3 x (5 分钟 + 50 GB x 1024 MB/GB / 150 MB/分钟) ≈ 17 小时

使用这种启发式方法得出的值在下限 15 分钟（对于空数据库）和上限 2 天（对于 150 GB 的数据库）之间变化。根据数据中心内的情况，估计值可能会进一步变化。

**从更高性能级别更改为更低性能级别时的延迟**

通常，如果将数据库性能级别从更高的级别更改为更小的大小，则不会发生数据移动。这样，SLO 更改的延迟要短得多，通常在几秒钟之内。

警告：以上说明仅适用于高级和标准服务层之间的降级。降级到 Web、企业或基本服务层需要数据移动。

### 检查数据库更改的状态
可以使用以下方法之一，在升级或降级服务层期间或者在更改性能级别时检查数据库的状态。

#### 使用 Azure 管理门户
1. 使用你的 Microsoft 帐户登录到 Azure 管理门户。
2. 从"数据库"列表中选择数据库。这将在"数据库仪表板"或"快速启动"页上打开该数据库。
3. 在"数据库仪表板"上，查看"版本"部分中的"速览"区域以获取状态信息。
4. 服务级别目标 (SLO) 表示某个服务层内的性能级别。


## 使用 Azure PowerShell 命令
本部分提供使用 Azure PowerShell 命令的先决条件。

**先决条件**

若要使用本主题中描述的 Azure PowerShell cmdlet，你必须在运行 PowerShell 的计算机上安装以下软件。
1. 从 http://www.microsoft.com/zh-cn/download/details.aspx?id=34595 下载最低 3.0 的 Windows PowerShell 版本。

2. 从 [Azure SDK 和工具下载](/downloads)的"命令行工具"部分下载 Azure PowerShell。

执行以下操作：
从"开始"屏幕或"开始"菜单，导航到 Azure PowerShell 并将其启动。

键入服务器的用户名和密码。

使用 **New-AzureSqlDatabaseServerContext** 创建服务器上下文。

**示例**

		Windows PowerShell
		$subId = <Subscription ID>
		$thumbprint = <Certificate Thumbprint>
		$myCert = Get-Item Cert:\CurrentUser\My\$thumbprint
		Set-AzureSubscription -SubscriptionName "mySubscription" -SubscriptionId $subId -Certificate $myCert
		Select-AzureSubscription -SubscriptionName "mySubscription"
		$serverContext = New-AzureSqlDatabaseServerContext -ServerName "myserver" -UseSubscription


**Azure PowerShell 参考**
若要查看有关本主题中使用的 Azure PowerShell cmdlet 的详细信息，请参阅 [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/dn546726.aspx)。

[New-AzureSqlDatabaseServerContext](https://msdn.microsoft.com/zh-cn/library/azure/dn546736.aspx)

[New-AzureSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx)

[Set-AzureSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/dn546732.aspx)

<!--HONumber=55-->