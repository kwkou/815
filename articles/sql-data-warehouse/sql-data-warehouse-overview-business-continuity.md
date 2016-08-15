<properties
   pageTitle="在 SQL 数据仓库中规划业务连续性 | Azure"
   description="SQL 数据仓库中的业务连续性概述"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/07/2016"
   wacn.date="05/23/2016"/>


# 在 SQL 数据仓库中规划业务连续性

本文逐步说明有关在 SQL 数据仓库中规划业务连续性和灾难恢复的基本知识。数据仓库是企业信息的中央存储库，在组织的各个层面，它都在涉及分析和业务智能的日常操作中扮演着重要角色。因此，数据仓库必须很可靠，且支持恢复和连续操作。具体而言，本文将说明 SQL 数据仓库如何让你使用“数据库还原”和“异地还原”从用户错误和灾难中恢复。

## 概念

业务连续性和灾难恢复计划需要优化以下指标：

**恢复时间目标 (RTO)：**在发生中断性事件，数据库完全恢复之前的最长可接受时间，即失败期间可用性的最大损失。

**恢复点目标 (RTO)：**发生中断性事件时，数据库在恢复之前可接受更新丢失的最长时间范围，即失败期间最大数据丢失量。

**预计恢复时间 (ERT)：**在发出还原请求后，数据库完全正常运行之前预计持续的时间。

## 业务连续性方案

**发生基础结构故障后恢复：**此方案是指在发生基础结构问题（例如磁盘故障等）后恢复。客户想要使用容错和高可用性基础结构来确保业务连续性。

**发生用户错误后恢复：**此方案是指发生意外或偶发的数据损坏或删除后进行恢复。如果用户无意中或不小心修改或删除了数据，客户希望能够将数据库还原到较早的时间点，以确保业务连续性。

**发生灾难后恢复 (DR)：**此方案是指发生重大灾难后进行恢复。在自然灾害或区域性停电等中断性事件造成数据库变得不可用的情况下，客户想要恢复不同区域的数据库，使业务得以持续运营。


## 业务连续性功能

让我们看看 SQL 数据仓库如何增强数据库的可靠性，并在发生上述情况时实现恢复和持续运营。


### 数据冗余

由于 SQL 数据仓库会隔离计算和存储，因此所有数据将直接写入异地冗余的 Azure 存储空间 (RA-GRS)。地域冗余存储将数据复制到距主区域数百英里以外的辅助区域。在主要区域和次要区域，你的数据将分别复制到次要区域三次，并且是在不同的容错域和升级域之间复制。即使其中一个区域发生全区域性的停电或灾难，造成服务中断，也仍可确保数据的持久性。若要了解有关读取访问权限异地冗余存储的详细信息，请参阅 [Azure 存储冗余选项][]。

### 数据库还原

数据库点还原旨在将数据库还原到以前的某个时间点。Azure SQL 数据仓库服务至少每隔 8 小时使用自动存储快照来保护所有数据库，并将快照保留 7 天，为你提供一系列不同的还原点。这些备份存储在 RA-GRS Azure 存储空间中，因此，默认情况下是异地冗余的。自动备份和还原功能不收取额外的费用，提供零成本和免管理的方式来避免数据库遭到意外损坏或删除。若要了解有关数据库还原的详细信息，请参阅[在发生用户错误后恢复][]。

### 异地还原

在由于发生中断性事件而使数据库不可用时，异地还原可以恢复数据库。你可以联系支持人员，以便从异地冗余的备份还原数据库，并在任一 Azure 区域中创建新数据库。由于备份是异地冗余的，即使由于停电而无法访问数据库，也能够使用备份来恢复数据库。异地还原功能不收取额外的费用。


## 后续步骤
若要详细了解其他 SQL 数据库版本的业务连续性功能，请阅读 [SQL 数据库业务连续性概述][]。

<!--Image references-->

<!--Article references-->
[business continuity overview]: /documentation/articles/sql-database-business-continuity/
[Finalize a recovered database]: /documentation/articles/sql-database-recovered-finalize/
[Azure 存储冗余选项]: /documentation/articles/storage-redundancy/#read-access-geo-redundant-storage-ra-grs
[SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[在发生用户错误后恢复]: /documentation/articles/sql-data-warehouse-business-continuity-recover-from-user-error/

<!--MSDN references-->
[Create database restore request]: http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx
[Database operation status]: http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx
[Get restorable dropped database]: http://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx
[List restorable dropped databases]: http://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0307_2016-->