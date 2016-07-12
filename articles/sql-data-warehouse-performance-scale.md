<properties
   pageTitle="SQL 数据仓库的弹性性能与缩放性 | Azure"
   description="了解 SQL 数据仓库的弹性，介绍如何使用数据仓库单位来向上或向下缩放计算资源。提供了代码示例。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/29/2016"
   wacn.date="05/05/2016"/>

# SQL 数据仓库的弹性性能与缩放性
若要弹性增加或减少计算能力，只需调整分配给 SQL 数据仓库的数据仓库单位 (DWU) 数量。数据仓库单位是 SQL 数据仓库引入的新概念，可让你轻松有效地进行管理。本主题是数据仓库单位的简介，说明如何使用数据仓库单位弹性调整计算能力。本文还提供有关针对环境设置合理 DWU 值的一些初步指导。

## 什么是数据仓库单位？
Microsoft 在幕后执行许多性能基准测试，以判断需要多少硬件和哪些配置才能为客户提供具有竞争力的产品。调高和调低计算能力可通过 100 个 DWU 的块来实现，但不支持 100 个 DWU 的所有倍数。

## 应该使用多少 DWU？
我们不会针对某个客户类提供可能合适的规范性 DWU 起点，而是通过实用的方法着手解决此问题。SQL 数据仓库的性能以线性方式缩放，在几秒内就能从某个计算比例更改为另一个（例如从 100 个 DWU 到 2000 个 DWU）。这样，你就可以灵活地尝试和判断你的方案的最适合数目。

1. 对于正在开发的数据仓库，可以从少量的 DWU 开始。
2. 监视应用程序性能，将所选 DWU 数目与观测到的性能变化进行比较。
3. 通过假设线性缩放，判断需要将性能加快或减慢多少才能达到要求的最佳性能级别。
4. 增加或减少选择的 DWU 数目。服务将快速响应并调整计算资源，以符合 DWU 要求。
5. 继续进行调整，直到达到业务要求的最佳性能级别。

如果应用程序的工作负荷持续变动，请将性能级别上调或下调，以配合高峰和离峰点。比方说，如果工作负荷高峰通常出现在月底，则可以规划在高峰期添加更多 DWU，然后在高峰期结束后减少。

## 向上和向下缩放计算资源
不管使用哪种云存储，借助 SQL 数据仓库的弹性，你可以使用数据仓库单位 (DWU) 的可调缩放性扩大、收缩或暂停计算容量。这样你就可以将计算能力弹性调整为最适合业务的计算能力。

若要提升计算能力，可以使用 Azure 管理门户中的缩放滑块将更多 DWU 添加到服务。你还可以通过 T-SQL、REST API 或 Azure Powershell cmdlet 来添加 DWU。尽管向上和向下缩放计算能力会取消所有运行中或已排队的活动，但此操作可在几秒内完成，使你能够以更大或更小的计算能力继续操作。

在 [Azure 管理门户][]中，可以单击 SQL 数据仓库页面顶部的“缩放”图标，然后使用滑块增加或减少应用到数据仓库的 DWU 数量，然后单击“保存”。如果你想要以编程方式更改缩放级别，以下 T-SQL 代码演示了如何针对 SQL 数据仓库调整 DWU 分配：


    ALTER DATABASE MySQLDW
    MODIFY (SERVICE_OBJECTIVE = 'DW1000')
    ;

请注意，应该针对逻辑服务器而不是 SQL 数据仓库实例本身运行此 T-SQL。

也可以通过导入 AzureRM.Sql 模块并使用以下代码，在 Azure Powershell 中实现相同的结果：


    Set-AzureRmSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer.database.chinacloudapi.cn" -RequestedServiceObjectiveName "DW1000"


## 暂停计算资源
SQL 数据仓库的独到之处就是能够根据需要暂停和恢复计算。如果团队将有一段时间不使用数据仓库实例（例如在晚上、周末、特定节假日，或出于任何其他原因），可以在该时段暂停数据仓库实例，并在恢复使用时从以前的中断处继续。

暂停操作将计算资源返回到数据中心的可用资源池；而恢复操作针对设置的 DWU 获取所需的计算资源，并将其分配到数据仓库实例。

> [AZURE.NOTE] 由于存储和计算能力相互独立，因此你的存储不受暂停的影响。

可以通过 [Azure 管理门户][]的 REST API 或 Powershell 来暂停和恢复计算。暂停取消所有运行中或已排队的活动；要继续使用时，可在几秒内恢复计算资源。

若要使用 Azure Powershell 暂停和继续服务，首先需要按如下所示导入 AzureRM.Sql 模块：


    Import-Module AzureRM.Sql


以下代码演示如何使用 Azure PowerShell 执行暂停：


    Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"


使用 Azure PowerShell 还可轻松恢复服务：


    Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"


有关如何使用 Azure PowerShell 的详细信息，请参阅 [在 SQL 数据仓库中使用 PowerShell cmdlet 和 REST API][]。

## 后续步骤
有关性能概述，请参阅[性能概述][]。

<!--Image references-->

<!--Article references-->
[性能概述]: /documentation/articles/sql-data-warehouse-overview-performance/
[在 SQL 数据仓库中使用 PowerShell cmdlet 和 REST API]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/

<!--MSDN references-->


<!--Other Web references-->

[Azure 管理门户]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0425_2016-->