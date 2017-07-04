<properties
    pageTitle="适用于 SQL 数据库的 Azure PowerShell 示例 | Azure"
    description="Azure CLI 示例 - 创建和管理 Azure SQL 数据库服务器、弹性池、数据库和防火墙。"
    services="sql-database"
    documentationcenter="sql-database"
    author="CarlRabeler"
    manager="jhubbard"
    editor="tysonn"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="sample"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="sql-database"
    ms.workload="database"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="janeng"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="f8e11c348da1d783b691a60c68055f2d394e6a94"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-powershell-samples-for-azure-sql-database"></a>适用于 Azure SQL 数据库的 Azure PowerShell 示例

下表包括了适用于 Azure SQL 数据库的示例 Azure PowerShell 脚本的链接。

| |  |
|---|---|
|**创建单一数据库和弹性池**||
| [创建单一数据库和配置防火墙规则](/documentation/articles/sql-database-create-and-configure-database-powershell/) | 创建单个 Azure SQL 数据库并配置服务器级防火墙规则。 |
| [创建弹性池并移动入池数据库](/documentation/articles/sql-database-move-database-between-pools-powershell/) | 创建弹性池，移动入池数据库，并更改性能级别。|
|**配置异地复制和故障转移**||
| [配置单一数据库并使用活动异地复制对其进行故障转移](/documentation/articles/sql-database-setup-geodr-and-failover-database-powershell/)| 为单个 Azure SQL 数据库配置活动异地复制，并将其故障转移到辅助副本。 |
| [配置入池数据库并使用活动异地复制对其进行故障转移](/documentation/articles/sql-database-setup-geodr-and-failover-pool-powershell/)| 为弹性池中的 Azure SQL 数据库配置活动异地复制，并将其故障转移到辅助副本。 |
|**缩放单一数据库和弹性池**||
| [缩放单一数据库](/documentation/articles/sql-database-monitor-and-scale-database-powershell/) | 监视一个 Azure SQL 数据库的性能指标，将其缩放为更高的性能级别，并基于性能指标之一创建警报规则。 |
| [缩放弹性池](/documentation/articles/sql-database-monitor-and-scale-pool-powershell/) | 监视一个弹性池的性能指标，将其缩放为更高的性能级别，并基于性能指标之一创建警报规则。  |
| **审核和威胁检测** |
| [配置审核和威胁检测](/documentation/articles/sql-database-auditing-and-threat-detection-powershell/)| 为 Azure SQL 数据库配置审核和威胁检测策略。 |
| **还原、复制和导入数据库**||
| [还原数据库](/documentation/articles/sql-database-restore-database-powershell/)| 从异地冗余备份中还原 Azure SQL 数据库以及将已删除的 Azure SQL 数据库还原到最新备份。 |
| [将数据库复制到新服务器](/documentation/articles/sql-database-copy-database-to-new-server-powershell/)| 在新的 Azure SQL 服务器中创建现有 Azure SQL 数据库的副本。 |
| [从 bacpac 文件导入数据库](/documentation/articles/sql-database-import-from-bacpac-powershell/)| 从 bacpac 文件将数据库导入到 Azure SQL 服务器。 |
|||