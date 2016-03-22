<properties
	pageTitle="创建可缩放的弹性数据库池 | Azure"
	description="如何将可缩放的弹性数据库池添加到 SQL 数据库配置，以简化多个数据库的管理和资源共享。"
	keywords="可缩放的数据库,数据库配置"
	services="sql-database"
	documentationCenter=""
	authors="sidneyh"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="02/12/2016"
	wacn.date="03/21/2016"/>


# 在 Azure 门户中为 SQL 数据库创建可缩放的弹性数据库池

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-portal)
- [C#](/documentation/articles/sql-database-elastic-pool-csharp)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell)

本文介绍如何使用 Azure 门户创建可缩放的[弹性数据库池](/documentation/articles/sql-database-elastic-pool)。包含弹性数据库池的 SQL 数据库配置简化了对多个数据库进行的管理和资源共享。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-powershell)。


## 先决条件

* SQL 数据库 V12 服务器上的数据库。如果你没有这样的数据库，请参阅[创建首个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)，在不到五分钟的时间内创建一个。 
* 或者，如果你已经有 SQL 数据库 V11 服务器，则可以[在门户中升级到 V12](/documentation/articles/sql-database-v12-plan-prepare-upgrade)，然后返回来按照这些说明创建一个池。


## 步骤 1：创建新池

通过向服务器添加新池来创建弹性数据库池。可以将多个池添加到服务器，但不能将数据库从不同的服务器添加到同一个池中。


1. 在 [Azure 门户](http://manage.windowsazure.cn)中，单击“SQL 服务器”。
2. 选择包含你要添加到池的数据库的服务器。
3. 单击“新建池”（或者如果你看到一条消息说已经有建议的池，则单击该消息即可轻松查看和创建针对服务器的数据库优化的池。有关建议的详细信息如下所示。
4. 在“弹性数据库池”边栏选项卡上，可以保留默认名称，也可以键入新池的名称。

### 建议的弹性数据库池

浏览到 SQL 数据库服务器，你可能会看到一条消息说该服务器已有建议的弹性数据库池（仅限 V12）。

### 为什么我会收到建议？

SQL 数据库服务将评估使用情况历史记录，并在比使用单一数据库更符合成本效益时，建议你使用一个或多个弹性数据库池。

每项建议是使用最适合该池的服务器数据库的唯一子集配置的。

每个池建议包含以下元素：

- 池的定价层（基本、标准或高级）。
- 适当的池 eDTU 数量。
- 弹性数据库最小/最大 eDTU 设置。
- 建议的数据库列表。

在建议弹性数据库池时，服务将考虑过去 30 天的遥测数据。被视为弹性数据库池候选项的数据库必须至少存在 7 天。已在弹性数据库池中的数据库不被视为建议的弹性数据库池候选项。

服务会评估将每个服务层中的单一数据库移到同一层的弹性数据库池的资源需求和成本效益。例如，评估服务器上的所有标准数据库是否适合标准弹性池。这意味着，服务不进行跨层建议，例如将标准数据库移到高级池。


### 创建建议的池

1. 单击该消息可以查看建议的池的列表。
1. 单击某个池即可查看详细的建议。
2. 查看建议中的所有设置并按原样接受，或编辑建议以满足你特定的业务需求。


## 步骤 2：选择一个定价层。

该池的定价层决定了池中弹性数据库的可用功能、eDTU 数目上限 (DTU MAX)，以及每个数据库的可用存储 (GB)。有关详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/#Service-tiers-for-elastic-database-pools)。

>[AZURE.NOTE] 目前，在预览版中，弹性数据库池的定价层在创建之后无法更改。若要更改现有弹性池的定价层，请在所需的定价层中创建新的弹性池，然后将弹性数据库移转到这个新池。

   ![选择一个定价层](./media/sql-database-elastic-pool-portal/pricing-tier.png)

### 配置池

设置定价层后，请单击“配置池”，在其中添加数据库、设置池 eDTU 和存储（池 GB），并且在其中为池中的弹性数据库设置最小值和最大值 eDTU。

## 步骤 3：将数据库添加到池

你可以随时对池添加或删除数据库。

1. 在池的创建过程中，单击“配置池”边栏选项卡上的“添加数据库”。
2. 选择你要添加到池的数据库：

    当你选择要添加到池的数据库时，必须满足以下条件：

    - 该池必须为数据库留出足够的空间（不能已经包含最大数目的数据库）。更具体地说，若要满足每个数据库的 eDTU 保障，该池必须具有足够的可用 eDTU。例如，如果一个组的 eDTU 保障为 400，且每个数据库的 eDTU 保障为 10，则池中允许的最大数据库数目为 40（400 eDTU/每个 DB 10 个 eDTU 保障 = 40 最大数据库数目）。
    - 数据库当前所用的功能必须在池中可用。

### 动态建议

将数据库添加到池中后，将基于所选数据库的历史使用情况动态地生成建议。这些建议将显示在 eDTU 和容量 (GB) 使用情况图表中以及在“配置池”边栏选项卡顶部的建议横幅中。这些建议旨在帮助你创建针对特定数据库进行了优化的池。

## 步骤 4：设置池的性能特征

通过设置池以及池中弹性数据库的性能参数，来配置池的性能。请记住，**弹性数据库设置**将应用到池中所有数据库。

 ![配置弹性池](./media/sql-database-elastic-pool-portal/configure-performance.png)

你可以设置三个参数来定义池的性能，即池的 eDTU 保障，以及池中弹性数据库的 eDTU MIN 和 eDTU MAX。下表描述了每个参数，并提供了一些设置方面的指导。有关特定的可用值设置，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。

| 性能参数 | 说明 |
| :--- | :--- |
| **POOL eDTU** - 池的 eDTU 保障 | 池的 eDTU 保障是指可供池中所有数据库使用和共享的 eDTU 的保障数目。<br> 设置某个组的 eDTU 保障的具体大小时，应考虑该组 eDTU 的历史使用率。此外，也可以通过每个数据库所需的 eDTU 保障以及并发活动数据库的使用率来设置此大小。池的 eDTU 保障还与池的可用存储容量相关联，每分配给池一个 eDTU，你都可以获得固定量的数据库存储。<br> **我应该将池的 eDTU 保障设置为何值？** <br>你至少应将池的 eDTU 保障设置为（[数据库数] x [每个数据库的平均 eDTU 使用率]） |
| **eDTU MIN** - 每个数据库的 eDTU 保障 | 每个数据库的 eDTU 保障是指池中的单一数据库可以确保获得的 eDTU 数。<br> **我应该将每个数据库的 eDTU 保障设置为何值？** <br> 通常，每个数据库的最小 eDTU 保障 (eDTU MIN) 应设置为介于 0 和（[每个数据库的平均使用率]）之间的某个数字。每个数据库的 eDTU 保障是一种全局设置，用于设置池中所有数据库的 eDTU 保障。 |
| **eDTU MAX** - 单个数据库的 eDTU 最大值 | 单个数据库的 eDTU MAX 是指池中单一数据库可以使用的 eDTU 的最大数目。必须将单个数据库的 eDTU 最大值设置得足够高，否则无法处理数据库可能会遇到的最大突发或峰值情况。你可以将此最大值设置为系统上限，后者取决于池的定价层（高级层为 1000 eDTU）。此最大值的具体大小取决于组中数据库的峰值使用率模式。组可能会碰到某种程度的超量使用情况，因为池通常会假定数据库存在热使用模式和冷使用模式，即所有数据库不会同时达到峰值。<br> **我应该将单个数据库的 eDTU 最大值设置为何值？** <br> 将单个数据库的 eDTU MAX 或 eDTU 最大值设置为（[数据库高峰使用率]）。例如，假设单个数据库的高峰使用率为 50 DTU，在该组的 100 个数据库中，仅有 20% 同时爆发到峰值。如果将单个数据库的 eDTU 最大值设置为 50 eDTU，则可以认为超量 5 倍使用该池是合理的，因此可以将该组的 eDTU 保障 (POOL eDTU) 设置为 1,000 eDTU。此外还需要提到的是，eDTU 最大值不是数据库的资源保障，而是在可能情况下能够达到的 eDTU 上限。 |


## 对池添加和删除数据库

创建池后，你可以通过在“弹性数据库”页上添加或删除数据库，在池中添加或删除现有数据库（浏览到你的池，然后在“基本组件”中单击“弹性数据库”）。

- 单击“添加数据库”以打开可以添加到池中的数据库列表。

    -或-

- 选择你想要从池中删除的数据库，然后单击“删除数据库”。

## 在池中创建新数据库

通过浏览到所需的池并单击“创建数据库”在池中创建新数据库。

系统已经为正确的服务器和池配置了 SQL 数据库，因此请输入名称并选择数据库选项，然后单击“确定”创建新数据库：

## 监视和管理弹性数据库池

创建弹性数据库池后，你可以在门户中监视和管理该池，只需浏览到现有池的列表，然后选择所需池即可。

创建一个池之后，你可以：

- 选择“配置池”来更改池 eDTU 和单数据库 eDTU 设置。
- 通过创建弹性作业来选择“创建作业”并管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 Transact-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。
- 选择“管理作业”可管理现有弹性作业。
- 单击“创建数据库”以在池中创建新数据库。
- 单击池设置链接，调整每个池的 eDTU 和容量 (GB)
- 单击数据库链接以对池添加或删除数据库。
- 单击数据库设置链接来针对池中的弹性数据库调整最小和最大的 eDTU。 

当你选择现有池时，你可以查看池的资源使用率。单击“资源使用率”图表可以打开“度量值”边栏选项卡，你可以在其中自定义图表和设置警报。

![监视弹性池][4]

单击“编辑图表”可以添加参数，这样你就可以轻松地查看池的遥测数据。

## 后续步骤
创建后弹性数据库池后，可以通过创建弹性作业来管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 Transact-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。



## 其他资源

- [SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool)
- [使用 PowerShell 创建 SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool-powershell)
- [使用 C# 创建和管理 SQL 数据库](/documentation/articles/sql-database-client-library)
- [弹性数据库参考](/documentation/articles/sql-database-elastic-pool-reference)


<!--Image references-->
[4]: ./media/sql-database-elastic-pool-portal/monitor-elastic-pool.png
[6]: ./media/sql-database-elastic-pool-portal/metric.png
[7]: ./media/sql-database-elastic-pool-portal/edit-chart.png
[10]: ./media/sql-database-elastic-pool-portal/star.png

<!---HONumber=Mooncake_0307_2016-->
