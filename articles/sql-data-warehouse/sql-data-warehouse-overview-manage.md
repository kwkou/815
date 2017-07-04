<properties
    pageTitle="在 Azure SQL 数据仓库中管理数据库 | Azure"
    description="管理 SQL 数据仓库数据库的概述。 包括管理工具、DWU 和向外扩展性能，对查询性能进行故障排除，建立良好的安全策略，以及从数据损坏或区域中断还原数据库。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="8576fdb3-71fe-4b3b-a4e0-5e8a7f148acf"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="manage"
    ms.date="10/31/2016"
    wacn.date="05/08/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="b4a9313fffff4c3fcc012dcbb98fefe3529e7820"
    ms.lasthandoff="04/28/2017" />

# <a name="manage-databases-in-azure-sql-data-warehouse"></a>在 Azure SQL 数据仓库中管理数据库
SQL 数据仓库自动执行管理数据库的许多方面的操作。 例如，若要缩放性能，你只需调整相应级别的计算资源并为这些资源付费，然后即可让 SQL 数据仓库执行向外扩展和缩减的所有工作。

你肯定需要监视工作负荷以确定所需性能，并对长时间运行的查询进行故障排除。 你还需要执行几个安全任务来管理用户和登录名的权限。

本概述介绍管理 SQL 数据仓库的这些方面内容。

* 管理工具
* 缩放计算
* 暂停和恢复
* 性能最佳实践
* 监视查询
* “安全”
* 备份和还原

## <a name="management-tools"></a>管理工具
可以使用多种工具来管理 SQL 数据仓库中的数据库。 管理数据库时，你将为需要执行的每种类型的任务制定工具首选项。

### <a name="azure-portal"></a>Azure 门户
[Azure 门户][Azure portal] 是一个基于 Web 的门户，你可以从中创建、更新和删除数据库以及监视数据库资源。 如果你刚开始使用 Azure、管理少量的数据仓库数据库或需要快速执行某些操作，该工具是理想之选。

若要开始使用 Azure 门户，请参阅[创建 SQL 数据仓库（Azure 门户）][Create a SQL Data Warehouse (Azure portal)]。

### <a name="sql-server-data-tools-in-visual-studio"></a>Visual Studio 中的 SQL Server Data Tools
使用 Visual Studio 中的 [SQL Server Data Tools][SQL Server Data Tools] (SSDT)，可连接到你的数据库并对其进行管理和开发。 如果你是熟悉 Visual Studio 或其他集成开发环境 (IDE) 的应用程序开发人员，请尝试使用 Visual Studio 中的 SSDT。

使用 SSDT 包含的 SQL Server 对象资源管理器，可以针对 SQL 数据仓库数据库进行可视化、连接和执行脚本。 若要快速连接到 SQL 数据仓库，只需在 Azure 经典管理门户中查看数据库详细信息时，单击命令栏中的“在 Visual Studio 中打开”按钮。  

若要开始使用 Visual Studio 中的 SSDT，请参阅 [使用 Visual Studio 查询 Azure SQL 数据仓库][Query Azure SQL Data Warehouse with Visual Studio]。

### <a name="command-line-tools"></a>命令行工具
命令行工具最适合用于自动执行工作负荷。PowerShell 和 sqlcmd 是自动执行过程的两个很好方法。由于可为所需的作业编写脚本并自动执行此类作业，因此我们建议使用这些工具来管理大量的逻辑服务器，以及在生产环境中部署资源更改。

### <a name="dynamic-management-views"></a>动态管理视图
DMV 是管理 SQL 数据仓库的必备工具。 在门户中显示的所有信息几乎都依赖于 DMV。 若要查看 SQL 数据仓库 DMV 的列表，请参阅 [SQL 数据仓库系统视图][SQL Data Warehouse system views]。

若要开始，请参阅[使用 sqlcmd 进行连接和查询][Connect and query with sqlcmd]以及[创建数据库 (PowerShell)][Create a database (PowerShell)]。

## <a name="scale-compute"></a>缩放计算
在 SQL 数据仓库中，通过增加或减少 CPU 计算资源、内存和 I/O 带宽，可以快速地进行性能缩放。 进行性能缩放时，只需调整 SQL 数据仓库分配给数据库的数据仓库单位 (DWU) 数即可。 SQL 数据仓库可以快速地进行更改，并处理针对硬件或软件的所有基础更改。

若要了解有关缩放 DWU 的详细信息，请参阅[缩放性能]。

## <a name="pause-and-resume"></a>暂停和恢复
为了节省成本，可以按需暂停和恢复计算资源。 例如，如果你晚上和周末不使用数据库，那么可以在这些时间暂停数据库的使用，然后在白天时恢复使用。 当数据库暂停时不对 DWU 进行收费。

有关详细信息，请参阅[暂停计算][Pause compute]和[恢复计算][Resume compute]。

## <a name="performance-best-practices"></a>性能最佳实践
开始使用一种新技术时，从一开始就发现最适用的提示和技巧可以节省大量时间。用户会发现，最佳实践贯穿了许多主题。

若要查看在开发工作负荷时最重要的注意事项的摘要，请参阅 [SQL 数据仓库最佳实践][SQL Data Warehouse Best Practices]。

## <a name="query-monitoring"></a>监视查询
有时某个查询运行时间太长，但你不能确定哪个查询才是问题所在。 SQL 数据仓库包含动态管理视图 (DMV)，可用于找出哪个查询用时过长。

若要查找长时间运行的查询，请参阅[使用 DMV 监视工作负荷][Monitor your workload using DMVs]。

## <a name="security"></a>“安全”
若要维护一个安全系统，必须警惕和防范任何类型的未经授权的访问。 安全系统需要确保防火墙规则已到位，以便只有经过授权的 IP 地址才能连接。 它需要对用户凭据进行相应身份验证。 用户连接到数据库后，用户只应有权执行最小数量的操作。 若要保护数据，可以使用加密。 具有审核和跟踪功能也很重要，以便在有任何可疑活动时可以追溯事件。

若要了解管理安全性，请直接访问 [安全性概述][Security overview]。

## <a name="backup-and-restore"></a>备份和还原
创建数据的可靠备份是任何生产数据库必不可少的组成部分。 SQL 数据仓库可通过定期自动备份活动数据库来使数据保持安全。 通过这些备份可以从损坏了数据或是意外删除了数据或数据库的情形中恢复。  有关数据备份计划、保留策略以及如何还原数据库，请参阅 [从快照还原][Restore from snapshot]。

## <a name="next-steps"></a>后续步骤
使用好的数据库设计原则可更轻松地在 SQL 数据仓库中管理数据库。 若要了解详细信息，请直接访问[开发概述][Development overview]。

<!--Image references-->

<!--Article references-->
[Create a SQL Data Warehouse (Azure Portal)]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Create a database (PowerShell)]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[connection]: /documentation/articles/sql-data-warehouse-develop-connections/
[Query Azure SQL Data Warehouse with Visual Studio]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[Connect and query with sqlcmd]: /documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/
[Development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Monitor your workload using DMVs]: /documentation/articles/sql-data-warehouse-manage-monitor/
[Pause compute]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#pause-compute-bk
[Restore from snapshot]: /documentation/articles/sql-data-warehouse-restore-database-overview/
[Resume compute]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#resume-compute-bk
[缩放性能]: /documentation/articles/sql-data-warehouse-manage-compute-overview/#scale-compute
[Security overview]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[SQL Data Warehouse Best Practices]: /documentation/articles/sql-data-warehouse-best-practices/
[SQL Data Warehouse system views]: /documentation/articles/sql-data-warehouse-reference-tsql-system-views/

<!--MSDN references-->
[SQL Server Data Tools]: https://msdn.microsoft.com/zh-cn/library/mt204009.aspx

<!--Other web references-->
[Azure portal]: http://portal.azure.cn/

<!--Update_Description:update meta properties;wording update -->