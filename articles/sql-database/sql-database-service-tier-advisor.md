<properties 
   pageTitle="Azure SQL 数据库的定价层建议" 
   description="在 Azure 门户预览中更改定价层时，提供的定价层建议将会推荐最适合用于运行现有 Azure SQL 数据库负载的层。定价层描述 SQL 数据库的服务层和性能级别。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management" 
   ms.date="08/08/2016" 
   wacn.date="09/26/2016"
   ms.author="sstein"/>

# SQL 数据库定价层建议

 定价层建议将会推荐最适合用于运行现有 Azure SQL 数据库工作负荷的服务层和性能级别。

> [AZURE.NOTE] 定价层建议仅适用于 Web 和企业数据库及弹性数据库池，并且只会在 [Azure 门户预览](https://portal.azure.cn/)中提供。


在执行以下任务期间获取定价层建议：

- [更改 SQL 数据库的服务层和性能级别（定价层）](/documentation/articles/sql-database-scale-up/)
- [将 Azure SQL 服务器升级到 V12](/documentation/articles/sql-database-upgrade-server-portal/)
- 浏览到 V12 服务器。请参阅 [SQL 数据库定价层建议](/documentation/articles/sql-database-service-tier-advisor/)。
- [创建弹性数据库池](/documentation/articles/sql-database-elastic-pool/#elastic-database-pool-pricing-tier-recommendations)





## 概述

SQL 数据库服务会通过评估 SQL 数据库的历史资源使用量，来分析当前的性能和功能要求。此外，会根据数据库大小和启用的[业务连续性](/documentation/articles/sql-database-business-continuity/)功能确定可接受的最低服务层。

服务将分析这些信息，然后推荐最适合用于运行数据库的典型工作负载和保留其当前功能集的服务层与性能级别。

- 服务将会调查过去 15 到 30 天的历史数据（资源使用量、数据库大小和数据库活动），并在消耗资源数量与当前可用服务层和性能级别的实际限制之间执行比较。
- 数据将以 15 秒为间隔进行分析，每个间隔的结果集将分类到最适合用于处理该结果集的工作负载的现有服务层和性能级别中。
- 然后，这些以 15 秒为间隔的样本将聚合成更大的 15-30 天分析样本，服务将会推荐能够以最佳方式处理 95% 的历史工作负载的服务层和性能级别。

### 建议

根据数据库的使用情况，目前可能会提供两个类别的建议：


| 建议 | 说明 |
| :--- | :--- |
| 升级 | 升级到新层。 |
| 不可用 | 数据库需要最小工作负荷或大约 35 天的活动。数据不足，无法提供有效的建议。 |

## 获取定价层建议

可以通过选择现有的 Web 或企业数据库并单击“所有设置”和“定价层(规模 DTU)”来获取定价层建议。（当你[将 Azure SQL 服务器升级到 V12](/documentation/articles/sql-database-upgrade-server-portal/) 时，也可以获取定价层建议。）

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。
2. 单击“浏览”>“SQL 数据库”。
4. 在“SQL 数据库”边栏选项卡中，单击想要查看其建议的数据库：

    ![选择数据库][1]

5. 在数据库边栏选项卡上，选择“所有设置”，然后选择“定价层(规模 DTU)”。


7. 此时会打开“建议的定价层”，你可以在其中单击建议的层，然后单击“选择”按钮切换到该层。

    ![注册预览版][4]

8. （可选）单击“查看使用情况详细信息”打开“定价层建议详细信息”边栏选项卡，你可以在其中查看数据库的建议层、当前层与建议层之间的功能比较，以及历史资源使用量分析图表。

    ![注册预览版][5]



## 摘要

定价层建议提供自动化体验，可以收集每个 SQL 数据库的遥测数据，并根据数据库的实际性能需求和功能要求推荐最佳的服务层/性能级别组合。在“设置”边栏选项卡上，单击“定价层(规模 DTU)”以查看针对 Web 和企业数据库的定价层建议。



## 后续步骤

根据具体数据库的详细信息，执行升级或降级通常不会立即发生。当数据库过渡到新层时，门户会提供通知；你也可以通过在 SQL 数据库服务器的 master 数据库中查询 [sys.dm\_operation\_status (Azure SQL 数据库)](https://msdn.microsoft.com/zh-cn/library/dn270022.aspx) 视图，来监视升级状态。


<!--Image references-->
[1]: ./media/sql-database-service-tier-advisor/select-database.png
[4]: ./media/sql-database-service-tier-advisor/choose-pricing-tier.png
[5]: ./media/sql-database-service-tier-advisor/usage-details.png


 

<!---HONumber=Mooncake_0919_2016-->