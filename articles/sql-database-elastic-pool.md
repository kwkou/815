<properties
	pageTitle="SQL 数据库的弹性数据库池 | Azure"
	description="了解如何使用弹性数据库池（在多个数据库之间共享可用资源的一种方式）来顺应 SQL 数据库中的爆炸式数据增长。"
	keywords="弹性数据库,sql 数据库"	
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor="cgronlun"/>

<tags
	ms.service="sql-database"
	ms.date="12/01/2015"
	wacn.date="01/29/2016"/>


# 使用弹性数据库池共享资源以顺应 SQL 数据库中的爆炸式数据增长

如果你是 SaaS 开发人员，使用数十、数百甚至数千个 SQL 数据库，则可以通过弹性数据库池来简化这些数据库的创建、维护以及性能管理流程，将预算置于自己的控制之下。使用 Azure 门户、PowerShell 或 C#，在几分钟内即可为 SQL 数据库[创建一个弹性数据库池](/documentation/articles/sql-database-elastic-pool-portal)。

常见的 SaaS 应用模式是，每一个数据库的客户各不相同，每一个的资源消耗（以 eDTU 汇总的 CPU/IO/内存）也各不相同且不可预知。对每个数据库的需求都存在这种高峰和低谷，因此很难预测和设置资源。你将面临两种选择，一种是基于高峰使用情况过度设置数据库资源，因此需要支付额外的费用；另一种是为了节省成本而采用低配，但在高峰期间会出现性能下降而导致客户满意度降低。

Microsoft 创建弹性数据库池就是专门帮你解决这个问题。


弹性数据库池提供解决方案，让客户确保他们的数据库在必要时获取所需的效能资源，同时提供简单的资源分配机制和可预测的预算。弹性数据库池中的每个数据库可按需调整性能，因为池中的每个数据库都使用该池关联的共享集的 eDTU。这样，负载较重的数据库便会消耗较多的 eDTU 来满足需求，负载较轻的数据库消耗较少的 eDTU，而完全无负载的数据库则不使用任何 eDTU。为池预配资源而不是为单一数据库设置资源不仅可以简化多个数据库的管理，而且对于无法预测的工作负荷也能有可预测的预算。

如果需要更多的 eDTU 来满足池的需求（将更多的数据库添加到池中，或现有数据库开始使用更多的 eDTU），可将其他 eDTU 加入现有池，而不会使数据库停机或对数据库造成负面影响。同样，你随时可以从现有池中删除不再需要的额外 eDTU。

![弹性数据库池中共享 eDTU 的 SQL 数据库。][1]

最适合添加到弹性数据库池的数据库通常是有时活动，有时不活动。在上述示例中，你可以看到单一数据库的活动、4 个数据库的活动，最后是包含 20 个数据库的弹性数据库池的活动。活动随时间而不同的这些数据库很适合添加到弹性数据库，因为它们不是永远都在使用中，而且可以共享 eDTU。并非所有数据库都符合此模式。有的数据库对资源的需求较为稳定，这些数据库更适合“基本”、“标准”和“高级”服务层级，这些层级的资源是单独分配的。有关确定数据库是否因弹性数据库池而受益的帮助，请参阅[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance)。

有关弹性数据库池的详细信息，包括 API 和错误详细信息，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。


> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。

## 使用弹性数据库工具轻松管理大量 SQL 数据库

除了提供更高效的资源利用率和可预测的性能，弹性数据库池还可以使 SaaS 应用程序开发更容易，其提供的工具可以简化数据层的构建和管理。跨大型数据库集执行维护任务和实施更改在以往都是既耗时又复杂的流程，但现在只需在弹性作业中运行脚本就可以了。创建和运行弹性数据库作业功能消除了几乎所有在管理数百甚至数千数据库时需要完成的繁重任务。有关用于跨池中所有弹性数据库运行 Transact-SQL 脚本的弹性数据库作业服务的信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。

此外还提供一组功能丰富且强大的开发人员工具，用于实施弹性数据库应用模式。对于分片数据库，可以使用其他工具（例如拆分-合并工具）来拆分某个分片的数据，然后将其与其他分片合并到一起。这极大地减少了管理大规模分片数据库的工作。有关详细信息，请参阅[弹性数据库工具主题映射](/documentation/articles/sql-database-elastic-scale-documentation-map)。

## 池中数据库的业务连续性功能

弹性数据库支持 V12 服务器上单一数据库可用的大部分[业务连续性功能](/documentation/articles/sql-database-business-continuity)（目前以预览版提供）。

### 备份和还原数据库（时间点还原）

系统会自动备份弹性数据库池中的数据库，备份保留策略与单一数据库对应的服务层相同。更具体地说，基本池中的弹性数据库可以还原到过去 7 天内的任何还原点，标准池中的弹性数据库可以还原到过去 14 天内的任何还原点，高级池中的弹性数据库可以还原到过去 35 天内的任何还原点。在预览期，池中的数据库将还原到相同池中的新数据库。删除的数据库始终还原为池外的独立数据库，因而将使用该服务层的最低性能层级。例如，标准池中删除的弹性数据库将还原为 S0 数据库。你可以通过 Azure 门户或以编程方式使用 REST API 执行数据库还原操作。PowerShell cmdlet 支持即将推出。

### 异地还原

异地还原功能允许你将池中的数据库恢复到其他区域中的服务器。在使用预览版期间，若要将池中的数据库还原到其他服务器，目标服务器需要有一个名称与源池相同的池。如果需要，请在目标服务器上创建新的池，并在还原该数据库之前为其提供一个相同的名称。如果目标服务器上不存在相同名称的池，则异地还原操作将失败。有关详细信息，请参阅[使用异地还原进行恢复](/documentation/articles/sql-database-disaster-recovery/#recover-using-geo-restore)。


### 异地复制

标准异地复制适用于标准或高级弹性数据库池中的任何数据库。只要服务层相同，异地复制合作关系中的一个或所有数据库就可以处于一个弹性数据库池中。可以使用 [Azure 门户](/documentation/articles/sql-database-geo-replication-portal)、[PowerShell](/documentation/articles/sql-database-geo-replication-powershell) 或 [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql) 为弹性数据库池配置异地复制。

### 导入和导出

支持从池中导出数据库。目前不支持直接将数据库导入池，但可以先导入至单一数据库，然后将数据库移入池中。


<!--Image references-->
[1]: ./media/sql-database-elastic-pool/databases.png

<!---HONumber=Mooncake_0118_2016-->
