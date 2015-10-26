<properties 
	pageTitle="创建和管理 SQL 数据库弹性数据库池 | Windows Azure"
	description="创建一个可在一组 Azure SQL 数据库之间共享的资源池。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="08/25/2015"
	wacn.date="10/17/2015"/>


# 创建和管理 SQL 数据库弹性数据库池

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-portal)
- [C#](/documentation/articles/sql-database-client-library)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell)

本文介绍如何通过 [Azure 预览门户](https://manage.windowsazure.cn)创建弹性数据库池。

弹性数据库池可以简化大量数据库的创建、维护以及性能和成本的管理流程。


> [AZURE.NOTE]弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。




## 先决条件

若要创建弹性数据库池，你需要满足以下条件：

- Azure 订阅！ 如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库 V12 服务器。如果你没有 V12 服务器，可以遵循本文中的以下步骤创建一个：[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)。



## 创建弹性数据库池

通过向服务器添加新池来创建弹性数据库池。可以将多个池添加到一个服务器，但每个池只能有一 (1) 个关联的服务器。此外，你还可以将将服务器上的所有或部分数据库添加到一个池中。


1.	选择包含你要添加到池的数据库的 SQL 数据库 V12 服务器。
2.	通过选择 SQL Server 边栏选项卡顶部的“添加池”来创建池。

   ![创建弹性池][1]

## 配置弹性数据库池

通过设置定价层、添加数据库和配置池的性能特征来配置池。

> [AZURE.NOTE]当你选择“添加池”命令时，你必须通过选择“预览版条款”并完成“预览版条款”边栏选项卡来接受预览版条款。每次订阅只需接受条款一次。

   ![配置弹性池][2]

## 配置定价层

弹性数据库池的定价层决定了池中弹性数据库的可用功能、eDTU 数目上限 (DTU MAX)，以及每个数据库的可用存储 (GB)。有关详细信息，请参阅[弹性池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool-reference#edtu-and-storage-limits-for-elastic-pools-and-elastic-databases)。

>[AZURE.NOTE]目前，在预览版中，弹性数据库池的定价层在创建之后无法更改。若要更改现有弹性池的定价层，请在所需的定价层中创建新的弹性池，然后将弹性数据库移转到这个新池。

   ![定价层][9]


## 弹性数据库池定价层建议

SQL 数据库服务将评估使用历史记录，并在比使用单一数据库更符合成本效益时，建议你使用一个或多个弹性数据库池。星号 (![*][10]) 代表根据你的数据库工作负荷建议的定价层。

如果建议了多个定价层，则表示应该创建多个弹性数据库池。每项建议是使用最适合该池的服务器数据库的唯一子集配置的。

除了只建议弹性数据库池定价层之外，每个池建议还包含以下项：

- 池的定价层（基本、标准或高级）。
- 适当的池 eDTU 数量。
- 数据库最小/最大 eDTU 设置。  
- 建议的数据库列表。

在建议弹性数据库池时，服务将考虑过去 30 天的遥测数据。被视为弹性数据库池候选项的数据库必须至少存在 7 天。已在弹性数据库池中的数据库不被视为建议的弹性数据库池候选项。

服务会评估将每个服务层中的单一数据库移到同一层的弹性数据库池的资源需求和成本效益。例如，评估服务器上的所有标准数据库是否适合标准弹性池。这意味着，服务不进行跨层建议，例如将标准数据库移到高级池。

>[AZURE.NOTE]Web 和企业数据库根据其使用历史记录和数据库大小，映射到一个新的基本、标准或高级层。通过映射到新层，为适当的池建议 Web 和企业数据库。


### 添加数据库

在任何时候，你都可以选择想要其包含在池中的特定数据库。创建新池时，Azure 会为你建议可以添加到池中的数据库，并将其标记为包含。你可以添加服务器上提供的所有数据库，也可以根据需要选择或清除初始列表中的数据库。

   ![添加数据库][5]

当你选择要添加到池的数据库时，必须满足以下条件：

- 该池必须为数据库留出足够的空间（不能已经包含最大数目的数据库）。更具体地说，若要满足每个数据库的 eDTU 保障，该池必须具有足够的可用 eDTU。例如，如果一个组的 eDTU 保障为 400，且每个数据库的 eDTU 保障为 10，则池中允许的最大数据库数目为 40（400 eDTU/每个 DB 10 个 eDTU 保障 = 40 最大数据库数目）。
- 数据库当前所用的功能必须在池中可用。


### 配置性能

通过设置池以及池中弹性数据库的性能参数，来配置池的性能。请记住，**弹性数据库设置**将应用到池中所有数据库。

   ![配置弹性池][3]

你可以设置三个参数来定义池的性能，即池的 eDTU 保障，以及池中弹性数据库的 eDTU MIN 和 eDTU MAX。下表描述了每个参数，并提供了一些设置方面的指导。有关特定的可用值设置，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。

| 性能参数 | 说明 |
| :--- | :--- |
| **POOL eDTU** - 池的 eDTU 保障 | 池的 eDTU 保障是指可供池中所有数据库使用和共享的 eDTU 的保障数目。<br> 设置某个组的 eDTU 保障的具体大小时，应考虑该组 eDTU 的历史使用率。此外，也可以通过每个数据库所需的 eDTU 保障以及并发活动数据库的使用率来设置此大小。池的 eDTU 保障还与池的可用存储容量相关联，每分配给池一个 eDTU，你都可以获得固定量的数据库存储。<br> **我应该将池的 eDTU 保障设置为何值？** <br>你至少应将池的 eDTU 保障设置为（[数据库数] x [每个数据库的平均 eDTU 使用率]） |
| **eDTU MIN** - 每个数据库的 eDTU 保障 | 每个数据库的 eDTU 保障是指池中的单一数据库可以确保获得的 eDTU 数。例如，在标准池中，你可以将此保障设置为 0、10、20、50 或 100 个 eDTU，也可以选择不为组中的数据库提供保障 (eDTU MIN=0)。<br> **我应该将每个数据库的 eDTU 保障设置为何值？** <br> 通常，每个数据库的最小 eDTU 保障 (eDTU MIN) 应设置为介于 0 和（[每个数据库的平均使用率]）之间的某个数字。每个数据库的 eDTU 保障是一种全局设置，用于设置池中所有数据库的 eDTU 保障。 |
| **eDTU MAX** - 单个数据库的 eDTU 最大值 | 单个数据库的 eDTU MAX 是指池中单一数据库可以使用的 eDTU 的最大数目。必须将单个数据库的 eDTU 最大值设置得足够高，否则无法处理数据库可能会遇到的最大突发或峰值情况。你可以将此最大值设置为系统上限，后者取决于池的定价层（高级层为 1000 eDTU）。此最大值的具体大小取决于组中数据库的峰值使用率模式。组可能会碰到某种程度的超量使用情况，因为池通常会假定数据库存在热使用模式和冷使用模式，即所有数据库不会同时达到峰值。<br> **我应该将单个数据库的 eDTU 最大值设置为何值？** <br> 将单个数据库的 eDTU MAX 或 eDTU 最大值设置为（[数据库高峰使用率]）。例如，假设单个数据库的高峰使用率为 50 DTU，在该组的 100 个数据库中，仅有 20% 同时爆发到峰值。如果将单个数据库的 eDTU 最大值设置为 50 eDTU，则可以认为超量 5 倍使用该池是合理的，因此可以将该组的 eDTU 保障 (POOL eDTU) 设置为 1,000 eDTU。此外还需要提到的是，eDTU 最大值不是数据库的资源保障，而是在可能情况下能够达到的 eDTU 上限。 |


## 将数据库添加到池，以及从池中删除弹性数据库

创建池后，你可以通过在“添加数据库”页上选择或清除数据库，在池中添加或删除数据库。

*创建池后，你还可以使用 Transact-SQL 在该池中创建新的弹性数据库，以及将现有数据库移入和移出池。有关详细信息，请参阅[弹性数据库池参考 - Transact-SQL](/documentation/articles/sql-database-elastic-pool-reference/#Transact-SQL)。*


## 监视和管理弹性数据库池

创建弹性数据库池后，你可以在门户中监视和管理该池，只需浏览到现有池的列表，然后选择所需池即可。

创建一个池之后，你可以：

- 选择“配置池”来更改池 eDTU 和单数据库 eDTU 设置。
- 通过创建弹性作业来选择“创建作业”并管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 Transact-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。
- 选择“管理作业”可管理现有弹性作业。



![监视弹性池][8]




![监视弹性池][4]

当你选择现有池时，你可以查看池的资源使用率。单击“资源使用率”图表可以打开“度量值”边栏选项卡，你可以在其中自定义图表和设置警报。


![资源利用率][6]

单击“编辑图表”可以添加参数，这样你就可以轻松地查看池的遥测数据。


![编辑图表][7]




## 后续步骤
创建后弹性数据库池后，可以通过创建弹性作业来管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 Transact-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。



## 其他资源

- [SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool)
- [使用 PowerShell 创建 SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool-powershell)
- [使用 C# 创建和管理 SQL 数据库](/documentation/articles/sql-database-client-library)
- [弹性数据库参考](/documentation/articles/sql-database-elastic-pool-reference)


<!--Image references-->
[1]: ./media/sql-database-elastic-pool-portal/new-elastic-pool.png
[2]: ./media/sql-database-elastic-pool-portal/configure-elastic-pool.png
[3]: ./media/sql-database-elastic-pool-portal/configure-performance.png
[4]: ./media/sql-database-elastic-pool-portal/monitor-elastic-pool.png
[5]: ./media/sql-database-elastic-pool-portal/add-databases.png
[6]: ./media/sql-database-elastic-pool-portal/metric.png
[7]: ./media/sql-database-elastic-pool-portal/edit-chart.png
[8]: ./media/sql-database-elastic-pool-portal/configure-pool.png
[9]: ./media/sql-database-elastic-pool-portal/pricing-tier.png
[10]: ./media/sql-database-elastic-pool-portal/star.png

<!---HONumber=74-->