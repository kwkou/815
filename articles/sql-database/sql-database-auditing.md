<properties
    pageTitle="Azure SQL 数据库 - 审核 | Azure"
    description="Azure SQL 数据库审核跟踪数据库事件，并将事件写入 Azure 存储帐户中的审核日志。"
    services="sql-database"
    documentationcenter=""
    author="ronitr"
    manager="jhubbard"
    editor="giladm" />
<tags
    ms.assetid="89c2a155-c2fb-4b67-bc19-9b4e03c6d3bc"
    ms.service="sql-database"
    ms.custom="secure and protect"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/05/2016"
    wacn.date="03/24/2017"
    ms.author="ronitr; giladm" />  


# SQL 数据库审核的概念
Azure SQL 数据库审核跟踪数据库事件，并将事件写入 Azure 存储帐户中的审核日志。

审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。

审核有助于遵从法规标准，但不能保证遵从法规。有关支持标准法规的 Azure 计划的详细信息，请参阅 [Azure 信任中心](https://www.trustcenter.cn/zh-cn/compliance/default.html)。

* [Azure SQL 数据库审核概述]
* [为数据库设置审核]
* [分析审核日志和报告]

## <a id="subheading-1"></a>Azure SQL 数据库审核概述
SQL 数据库审核可让你：

* **保留**选定事件的审核痕迹。可以定义要审核的数据库操作的类别。
* **报告**数据库活动。可以使用预配置的报告和仪表板快速开始使用活动和事件报告。
* **分析**报告。可以查找可疑事件、异常活动和趋势。

有两种**审核方法**：

* **Blob 审核** - 将日志写入 Azure Blob 存储。这是一种较新的审核方法，可提供更高的性能、支持更高粒度的对象级审核且更具成本效益。
* **表审核** - 将日志写入 Azure 表存储。

> [AZURE.IMPORTANT]
>引入新的 Blob 审核为数据库继承服务器审核策略的方式带来了重大变革。

可为不同类型的事件类别配置审核。

* 若要使用 Azure 门户预览配置和管理审核，请参阅[使用 Azure 门户预览进行审核](/documentation/articles/sql-database-auditing-portal/)。
* 若要使用 PowerShell 配置和管理审核，请参阅[使用 PowerShell 进行审核](/documentation/articles/sql-database-auditing-powershell/)。
* 若要使用 REST API 门户配置和管理审核，请参阅[使用 REST API 进行审核](/documentation/articles/sql-database-auditing-rest/)。

<!--For each Event Category, auditing of **Success** and **Failure** operations are configured separately.-->


可以为特定数据库定义审核策略，也可以将审核策略定义为默认服务器策略。默认的服务器审核策略将应用到服务器上的所有现有数据库和新建的数据库。

## Blob 审核

如果启用了服务器 Blob 审核，该项审核始终会应用到数据库（将审核服务器上的所有数据库），不管数据库审核设置如何，也不管是否在数据库边栏选项卡中选中了“从服务器继承设置”复选框。

> [AZURE.IMPORTANT]
>在数据库和服务器中启用 Blob 审核**不会**覆盖或更改服务器 Blob 审核的任何设置 - 这两项审核将同时存在。换而言之，数据库将同时审核两次（由服务器策略审核一次，由数据库策略审核一次）。

除非有以下需要，否则应该避免同时启用服务器 Blob 审核和数据库 Blob 审核：
 * 需要对特定的数据库使用不同的*存储帐户*或*保留期*。
 * 想要针对特定的数据库审核与此服务器上其他数据库不同的事件类型或类别（例如，只需要审核特定数据库的表插入事件）。

否则，我们建议仅启用服务器级 Blob 审核，并为所有数据库禁用数据库级审核。

## 表审核

如果启用了服务器级表审核，则仅当已在数据库边栏选项卡中选中“从服务器继承设置”复选框时（对于所有现有和新建的数据库，默认已选中此复选框），才会将此项审核应用到数据库。

- 如果此复选框处于*选中状态*，对数据库中的审核设置进行任何更改将**覆盖**此数据库的服务器设置。

- 如果此复选框处于*未选中状态*并且已禁用数据库级审核，将不会审核数据库。

## 分析审核日志和报告
审核日志将在安装期间选择的 Azure 存储帐户中进行聚合。

可以使用 [Azure 存储资源管理器](http://storageexplorer.com/)等工具浏览审核日志。

## 后续步骤

* 若要使用 Azure 门户预览配置和管理审核，请参阅[在 Azure 门户预览中配置审核](/documentation/articles/sql-database-auditing-portal/)。
* 若要使用 PowerShell 配置和管理审核，请参阅[使用 PowerShell 配置审核](/documentation/articles/sql-database-auditing-powershell/)。
* 若要使用 REST API 配置和管理审核，请参阅[使用 REST API 配置审核](/documentation/articles/sql-database-auditing-rest/)。


<!--Anchors-->

[Azure SQL 数据库审核概述]: #subheading-1
[为数据库设置审核]: #subheading-2
[分析审核日志和报告]: #subheading-3
[Practices for usage in production]: #subheading-5
[Storage Key Regeneration]: #subheading-6
[Automation (PowerShell / REST API)]: #subheading-7
[Blob/Table differences in Server auditing policy inheritance]: (#subheading-8)

<!--Image references-->

[1]: ./media/sql-database-auditing-get-started/1_auditing_get_started_settings.png
[2]: ./media/sql-database-auditing-get-started/2_auditing_get_started_server_inherit.png
[3]: ./media/sql-database-auditing-get-started/3_auditing_get_started_turn_on.png
[3-tbl]: ./media/sql-database-auditing-get-started/3_auditing_get_started_turn_on_table.png
[4]: ./media/sql-database-auditing-get-started/4_auditing_get_started_storage_details.png
[5]: ./media/sql-database-auditing-get-started/5_auditing_get_started_audited_events.png
[6]: ./media/sql-database-auditing-get-started/6_auditing_get_started_storage_key_regeneration.png
[7]: ./media/sql-database-auditing-get-started/7_auditing_get_started_activity_log.png
[8]: ./media/sql-database-auditing-get-started/8_auditing_get_started_regenerate_key.png
[9]: ./media/sql-database-auditing-get-started/9_auditing_get_started_report_template.png
[10]: ./media/sql-database-auditing-get-started/10_auditing_get_started_blob_view_audit_logs.png
[11]: ./media/sql-database-auditing-get-started/11_auditing_get_started_blob_audit_records.png
[12]: ./media/sql-database-auditing-get-started/12_auditing_get_started_table_audit_records.png

<!---HONumber=Mooncake_0320_2017-->