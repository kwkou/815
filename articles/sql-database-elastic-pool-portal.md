<properties 
	pageTitle="创建和管理 SQL Database 弹性数据库池（预览版）" 
	description="创建一个可在一组 Azure SQL Database 之间共享的资源池。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags ms.service="sql-database" ms.date="04/29/2015" wacn.date="05/29/2015"/>


# 创建和管理 SQL Database 弹性池（预览版）

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-portal)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell)

本文介绍如何通过 [Azure 门户](https://manage.windowsazure.cn)创建弹性池。

弹性池可以简化大量数据库的创建、维护以及性能和成本的管理流程。
 

> [AZURE.NOTE] 弹性池目前为预览版，仅适用于 SQL Database V12 Servers。

 


## 先决条件

若要创建弹性池，你需要满足以下条件：

- Azure 订阅！如果你需要 Azure 订阅，只需单击本页顶部的"试用版"，然后再回来完成本文的相关操作即可。
- Azure SQL Database V12 服务器。如果你没有 V12 服务器，可按以下步骤创建一个：[创建你的第一个 Azure SQL Database](sql-database-get-started)。



## 创建弹性池

通过向服务器添加新池来创建弹性池。可以将多个池添加到一个服务器，但每个池只能有 1 个关联的服务器。此外，你还可以将将服务器上的所有或部分数据库添加到一个池中。


1.	选择包含你要添加到池的数据库的 SQL Database V12 服务器。
2.	通过选择 SQL Server 边栏选项卡顶部的"添加池"来创建池。

   ![Create Elastic Pool][1]

## 配置弹性池

通过设置定价层、添加数据库和配置池的性能特征来配置池。

*当你选择"添加池"命令时，你必须通过选择"预览版条款"并完成"预览版条款"边栏选项卡来接受预览版条款。每次订阅只需接受条款一次。*

   ![Configure elastic pool][2]


### 定价层

弹性池的定价层某种程度上类似于 SQL 数据库的服务层。定价层决定了池中弹性数据库的可用功能、最大数目 DTU (DTU MAX)，以及每个数据库的可用存储 (GB)。 

> [AZURE.NOTE] 预览版目前仅限于**标准**定价层。 

| 定价层 | 每个数据库的 DTU MAX |
| :--- | :--- |
| 标准 | 每个数据库 100 MAX DTU |

### 添加数据库

在任何时候，你都可以选择想要其包含在池中的特定数据库。创建新池时，Azure 会为你建议可以添加到池中的数据库，并将其标记为包含。你可以添加服务器上提供的所有数据库，也可以根据需要选择或清除初始列表中的数据库。

   ![Add databases][5]

当你选择要添加到池的数据库时，必须满足以下条件：

- 该池必须为数据库留出足够的空间（不能已经包含最大数目的数据库）。更具体地说，若要满足每个数据库的 DTU 保障，该池必须具有足够的可用 DTU。例如，如果一个组的 DTU 保障为 400，且每个数据库的 DTU 保障为 10，则池中允许的最大数据库数目为 40（400 DTU/每个 DB 10 个 DTU 保障 = 40 最大数据库数目）。
- 数据库当前所用的功能必须在池中可用。 


### 配置性能

通过设置池以及池中弹性数据库的性能参数，来配置池的性能。请记住，**弹性数据库设置**适用于池中所有数据库。

   ![Configure Elastic Pool][3]

你可以设置三个参数来定义池的性能，即池的 DTU 保障，以及池中弹性数据库的 DTU MIN 和 DTU MAX。下表描述了每个参数，并提供了一些设置方面的指导。

| 性能参数 | 说明 |
| :--- | :--- |
| **POOL DTU/GB** - 池的 DTU 保障 | 池的 DTU 保障是指可供池中所有数据库使用和共享的 DTU 的保障数目。 <br> 目前，你可以将此参数设置为 200、400、800 或 1200。 <br> 设置某个组的 DTU 保障的具体大小时，应考虑该组 DTU 的历史使用率。此外，也可以通过每个数据库所需的 DTU 保障以及并发活动数据库的使用率来设置此大小。池的 DTU 保障还与池的可用存储容量相关联，每分配给池一个 DTU，你都可以获得 1 GB 的数据库存储（1 DTU = 1 GB 存储）。 <br> **我应该将池的 DTU 保障设置为何值？** <br>你至少应将池的 DTU 保障设置为（[数据库数] x [单个数据库的平均 DTU 使用率]）|
| **DTU MIN** - 每个数据库的 DTU 保障 | 每个数据库的 DTU 保障是指池中的单个数据库可以确保获得的 DTU 数。目前，你可以将此保障设置为 0、10、20 或 50 DTU，也可以选择不为组中的数据库提供保障 (DTU MIN=0)。 <br> **我应该将每个数据库的 DTU 保障设置为何值？** <br> 你应该将每个数据库的最小 DTU 保障 (DTU MIN) 设置为（[单个数据库的平均使用率]）。每个数据库的 DTU 保障是一种全局设置，用于设置池中所有数据库的 DTU 保障。 |
| **DTU MAX** - 单个数据库的 DTU 最大值 | 单个数据库的 DTU MAX 是指池中单个数据库可以使用的 DTU 的最大数目。必须将单个数据库的 DTU 最大值设置得足够高，否则无法处理数据库可能会遇到的最大爆发或峰值现象。你可以将此最大值设置为系统上限，后者取决于池的定价层（标准层为 100 DTU）。此最大值的具体大小取决于组中数据库的峰值使用率模式。组可能会碰到某种程度的超量使用情况，因为池通常会假定数据库存在热使用模式和冷使用模式，即所有数据库不会同时达到峰值。<br> **我应该将单个数据库的 DTU 最大值设置为何值？** <br> 将单个数据库的 DTU MAX 或 DTU 最大值设置为（[数据库高峰使用率]）。例如，假设单个数据库的高峰使用率为 50 DTU，在该组的 100 个数据库中，仅有 20% 同时爆发到峰值。如果将单个数据库的 DTU 最大值设置为 50 DTU，则可以认为该组超量 5 倍是合理的，因此可以将该组的 DTU 保障设置为 1,000 DTU。此外还需要提到的是，DTU 最大值不是数据库的资源保障，而是在可能情况下能够达到的 DTU 上限。 |


## 将数据库添加到弹性池中

创建池后，你可以在池中添加或删除数据库


## 监视和管理弹性池

创建弹性池后，你可以在门户中监视和管理该池，只需浏览到现有池的列表，然后选择所需池即可。

创建一个池之后，你可以：

- 选择"配置池"来更改池 DTU 和单数据库 DTU 设置。
- 通过创建弹性作业来选择"创建作业"并管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。<!--有关详细信息，请参阅[弹性数据库作业概述](sql-database-elastic-jobs-overview)。-->
- 选择"管理作业"可管理现有弹性作业。



![Monitor elastic pool][8]




![Monitor elastic pool][4]

当你选择现有池时，你可以查看池的资源使用率。
单击"资源使用率"图表可以打开"指标"边栏选项卡，你可以在其中自定义图表和设置警报。


![resource utilization][6]

单击"编辑图表"可以添加参数，这样你就可以轻松地查看池的遥测数据。


![edit chart][7]







## 后续步骤
创建后弹性池后，可以通过创建弹性作业来管理池中的数据库。弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。

<!--有关详细信息，请参阅[弹性数据库作业概述](sql-database-elastic-jobs-overview)。-->



## 相关内容

- [SQL Database 弹性池](sql-database-elastic-pool)
- [使用 PowerShell 创建 SQL Database 弹性池](sql-database-elastic-pool-powershell)
- [弹性数据库参考](sql-database-elastic-pool-reference)


<!--Image references-->

[1]: ./media/sql-database-elastic-pool-portal/new-elastic-pool.png
[2]: ./media/sql-database-elastic-pool-portal/configure-elastic-pool.png
[3]: ./media/sql-database-elastic-pool-portal/configure-performance.png
[4]: ./media/sql-database-elastic-pool-portal/monitor-elastic-pool.png
[5]: ./media/sql-database-elastic-pool-portal/add-databases.png
[6]: ./media/sql-database-elastic-pool-portal/metric.png
[7]: ./media/sql-database-elastic-pool-portal/edit-chart.png
[8]: ./media/sql-database-elastic-pool-portal/configure-pool.png

<!---HONumber=56-->