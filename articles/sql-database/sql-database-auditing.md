<properties
    pageTitle="Azure SQL 数据库审核入门 | Azure"
    description="SQL 数据库审核入门"
    services="sql-database"
    documentationcenter=""
    author="giladm"
    manager="jhubbard"
    editor="giladm"
    translationtype="Human Translation" />
<tags
    ms.assetid="89c2a155-c2fb-4b67-bc19-9b4e03c6d3bc"
    ms.service="sql-database"
    ms.custom="secure and protect"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="3/7/2017"
    wacn.date="04/17/2017"
    ms.author="giladm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="05d5b482219f2175a7886b3bd2d662c0b4c719dd"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-with-sql-database-auditing"></a>SQL 数据库审核入门
Azure SQL 数据库审核跟踪数据库事件，并将事件写入 Azure 存储帐户中的审核日志。

审核可帮助你一直保持遵从法规、了解数据库活动，以及深入了解可以指明业务考量因素或疑似安全违规的偏差和异常。

审核有助于遵从法规标准，但不能保证遵从法规。 有关支持标准法规的 Azure 计划的详细信息，请参阅 [Azure 信任中心](https://www.trustcenter.cn/zh-cn/compliance/default.html)。

* [Azure SQL 数据库审核概述]
* [为数据库设置审核]
* [分析审核日志和报告]

## <a id="subheading-1"></a>Azure SQL 数据库审核概述
SQL 数据库审核可让你：

* **保留** 选定事件的审核痕迹。 可以定义要审核的数据库操作的类别。
* **报告** 数据库活动。 可以使用预配置的报告和仪表板快速开始使用活动和事件报告。
* **分析** 报告。 可以查找可疑事件、异常活动和趋势。

有两种**审核方法**：

* **Blob 审核** - 将日志写入 Azure Blob 存储。 这是一种较新的审核方法，可提供**更高的性能**，支持**更高粒度的对象级审核**，且**更具成本效益**。 Blob 审核将取代表审核。
* **表审核（已弃用）**- 将日志写入 Azure 表存储。

> [AZURE.IMPORTANT]
> 引入新的 Blob 审核为数据库继承服务器审核策略的方式带来了重要更改。 有关其他详细信息，请参阅 [Blob/表在服务器审核策略继承中的差异](#subheading-8)部分。

可按[为数据库设置审核](#subheading-2)部分中所述，为不同类型的事件类别配置审核。

<!--For each Event Category, auditing of **Success** and **Failure** operations are configured separately.-->

可以为特定数据库定义审核策略，也可以将审核策略定义为默认服务器策略。 默认的服务器审核策略将应用到服务器上的所有现有数据库和新建的数据库。

## <a id="subheading-2"></a>为数据库设置审核
以下部分介绍如何使用 Azure 门户配置审核。

### <a id="subheading-2-1">Blob 审核</a>
1. 启动 [Azure 门户](https://portal.azure.cn) (https://portal.azure.cn)。
2. 导航到你要审核的 SQL 数据库/SQL Server 的设置边栏选项卡。 在“设置”边栏选项卡中，选择“审核和威胁检测”。

    <a id="auditing-screenshot"></a>
    ![导航窗格][1]

3. 在数据库审核配置边栏选项卡中，可以选中“从服务器继承审核设置”复选框，指定根据服务器的设置对数据库进行审核。 如果选中此选项，将会看到“查看服务器审核设置”  链接，可以使用该链接在此上下文中查看或修改服务器审核设置。

    ![导航窗格][2]

4. 若要在数据库级别启用 Blob 审核（结合服务器级审核启用 Blob 审核，或者取代服务器级审核），请**取消选中**“从服务器继承审核设置”选项，将审核设置为“打开”，然后选择“Blob”审核类型。

    > 如果启用了服务器 Blob 审核，数据库配置的审核将与服务器 Blob 审核并存。  
   
    ![导航窗格][3]

5. 选择“存储详细信息”打开“审核日志存储”边栏选项卡。 选择日志要保存到的 Azure 存储帐户以及保留期（超过此期限的旧日志将被删除），然后单击底部的“确定”。 **提示：**为所有审核的数据库使用相同的存储帐户，以便充分利用审核报告模板。

    <a id="storage-screenshot"></a>
    ![导航窗格][4]

6. 若要自定义审核的事件，可以使用 PowerShell 或 REST API - 有关更多详细信息，请参阅[自动化 (PowerShell/REST API)](#subheading-7) 部分。
7. 配置审核设置后，可以打开新的**威胁检测**（预览版）功能，并配置电子邮件用于接收安全警报。 使用威胁检测可以接收针对异常数据库活动（可能表示潜在的安全威胁）发出的前瞻性警报。 有关更多详细信息，请参阅[威胁检测入门](/documentation/articles/sql-database-threat-detection-get-started/)。
8. 单击“保存” 。

### <a id="subheading-2-2">表审核</a>（已弃用）

> 在设置**表审核**之前，请检查使用的是否为[“下层客户端”](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients/)。 此外，如果有严格的防火墙设置，请注意，在启用表审核时[会更改数据库的 IP 终结点](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients/)。


1. 启动 [Azure 门户](https://portal.azure.cn) (https://portal.azure.cn)。
2. 导航到你要审核的 SQL 数据库/SQL Server 的设置边栏选项卡。 在“设置”边栏选项卡中，选择“审核和威胁检测”（[请参阅“Blob 审核”部分中的屏幕截图](#auditing-screenshot)）。
3. 在数据库审核配置边栏选项卡中，可以选中“从服务器继承审核设置”复选框，指定根据服务器的设置对数据库进行审核。 如果选中此选项，将会看到“查看服务器审核设置”  链接，可以使用该链接在此上下文中查看或修改服务器审核设置。

    ![导航窗格][2]

4. 如果不想要从服务器继承审核设置，请**取消选中**“从服务器继承审核设置”选项，将审核设置为“打开”，然后选择“表”审核类型。

    ![导航窗格][3-tbl]

5. 选择“存储详细信息”打开“审核日志存储”边栏选项卡。 选择日志要保存到的 Azure 存储帐户以及保留期（超过此期限的旧日志将被删除）。 **提示：**为所有审核的数据库使用相同的存储帐户，以便充分利用审核报告模板（[请参阅“Blob 审核”部分中的屏幕截图](#storage-screenshot)）。
6. 单击“审核事件”自定义要审核的事件。 在“按事件记录日志”边栏选项卡中，单击“成功”和“失败”记录所有事件，或者选择单个事件类别。

    > 也可以通过 PowerShell 或 REST API 自定义审核的事件 - 有关更多详细信息，请参阅[自动化 (PowerShell/REST API)](#subheading-7) 部分。

    ![导航窗格][5]

7. 配置审核设置后，可以打开新的**威胁检测**（预览版）功能，并配置电子邮件用于接收安全警报。 使用威胁检测可以接收针对异常数据库活动（可能表示潜在的安全威胁）发出的前瞻性警报。 有关更多详细信息，请参阅[威胁检测入门](/documentation/articles/sql-database-threat-detection-get-started/)。
8. 单击“保存” 。


## <a id="subheading-8"></a>Blob/表在服务器审核策略继承中的差异

###<a name="ablob-auditinga"></a><a>Blob 审核</a>

1. 如果**启用了“服务器 Blob 审核”**，它将**始终应用于数据库**（即将对数据库进行审核），而不考虑：
    - 数据库审核设置。
    - 是否在数据库边栏选项卡中选中了“从服务器继承设置”复选框。

2. 除了在服务器上启用 Blob 审核外，还对数据库启用 Blob 审核**不**会覆盖或更改服务器 Blob 审核的任何设置 - 这两种审核将并存。 换而言之，将并行对数据库审核两次（一次按服务器策略审核，一次按数据库策略审核）。

    > [AZURE.NOTE]
    > <p>应避免同时启用服务器 Blob 审核和数据库 Blob 审核，除非：
    > <p>- 需要对特定数据库使用不同*存储帐户*或*保留期*。
    > <p>- 你希望为特定数据库审核事件类型或类别，而这些事件类型或类别不同于要为此服务器上其余数据库审核的事件类型或类别（例如，如果只有特定数据库需要审核表插入）。
    > <br><br>
    > 否则，**建议仅启用服务器级 Blob 审核**，并将所有数据库的数据库级审核保留为禁用。

###<a name="atable-auditinga-deprecated"></a><a>表审核</a>（已弃用）

如果**启用了服务器级表审核**，则仅当在数据库边栏选项卡中选中了“从服务器继承设置”复选框（默认情况下，将对所有现有数据库和新创建的数据库进行此检查）时，它才应用于数据库。

- 如果*选中了*此复选框，则对数据库中的审核设置所做的任何更改将**覆盖**此数据库的服务器设置。

- 如果*未选中*此复选框且数据库级审核处于禁用状态，则不会审核数据库。

## <a id="subheading-3"></a>分析审核日志和报告
审核日志将在安装期间选择的 Azure 存储帐户中进行聚合。

可以使用 [Azure 存储资源管理器](http://storageexplorer.com/)等工具浏览审核日志。

请参阅下面有关分析 **Blob** 和**表**审核日志的具体信息。

### <a id="subheading-3-1">Blob 审核</a>
Blob 审核日志以 Blob 文件集合的形式保存在名为“**sqldbauditlogs**”的容器中。

有关 Blob 审核日志存储文件夹层次结构、blob 命名约定和日志格式的详细信息，请参阅 [Blob 审核日志格式参考（doc 文件下载）](https://go.microsoft.com/fwlink/?linkid=829599)。

可通过多种方法查看 Blob 审核日志：

1. 通过 [Azure 门户](https://portal.azure.cn) - 打开相关数据库。 在数据库的“审核和威胁检测”边栏选项卡的顶部，单击“查看审核日志”。

    ![导航窗格][10]

    此时将打开“审核记录”边栏选项卡，可在其中查看日志。

    - 可以通过单击“审核记录”边栏选项卡顶部区域中的“筛选”，选择查看特定的日期
    - 可以在服务器策略审核或数据库策略审核创建的审核记录之间切换

    ![导航窗格][11]
2. 通过门户或使用 [Azure 存储资源管理器](http://storageexplorer.com/)等工具从 Azure 存储 Blob 容器下载日志文件。

    在本地下载日志文件后，可以双击该文件将它打开，然后在 SSMS 中查看和分析日志。

    其他方法：

   * 可以通过 Azure 存储资源管理器**同时下载多个文件** - 右键单击特定的子文件夹（例如包含特定日期生成的所有日志文件的子文件夹），然后选择“另存为”，将文件保存到本地文件夹中。

       下载多个文件后（或者一整天的日志文件，如上所述），可按如下所述在本地将它们合并：

       **打开 SSMS，选择“文件”->“打开”->“合并扩展事件”-> 选择要合并的所有文件**
   * 以编程方式：

     * 扩展事件读取器 **C# 库**（[详细信息](https://blogs.msdn.microsoft.com/extended_events/2011/07/20/introducing-the-extended-events-reader/)）
     * 使用 **PowerShell** 查询扩展事件文件（[详细信息](https://sqlscope.wordpress.com/2014/11/15/reading-extended-event-files-using-client-side-tools-only/)）

3. 我们已创建一个**示例应用程序**，它在 Azure 中运行，并利用 OMS 公共 API 将 SQL 审核日志推送到 OMS，以便于通过 OMS 仪表板使用（[此处提供详细信息](https://github.com/Microsoft/Azure-SQL-DB-auditing-OMS-integration)）。

### <a id="subheading-3-2">表审核</a>（已弃用）
表审核日志以 Azure 存储表集合的形式保存，具有 **SQLDBAuditLogs** 前缀。

有关表审核日志格式的更多详细信息，请参阅 [Table Audit Log Format Reference（文档下载）](http://go.microsoft.com/fwlink/?LinkId=506733)（表审核日志格式参考）。

可通过多种方法查看表审核日志：

1. 通过 [Azure 门户](https://portal.azure.cn) - 打开相关数据库。 在数据库的“审核和威胁检测”边栏选项卡的顶部，单击“查看审核日志”。

    ![导航窗格][10]

    此时将打开“审核记录”边栏选项卡，可在其中查看日志。

    * 可以通过单击“审核记录”边栏选项卡顶部区域中的“筛选”  ，选择查看特定的日期
    * 可以通过单击“审核记录”边栏选项卡顶部区域中的“在 Excel 中打开”  ，下载并查看 Excel 格式的审核日志

    ![导航窗格][12]

2. 或者，可以借助以 [可下载 Excel 电子表格](http://go.microsoft.com/fwlink/?LinkId=403540) 形式提供的预配置报告模板来快速分析日志数据。 若要对审核日志使用模板，需要安装可从 [此处](http://www.microsoft.com/download/details.aspx?id=39379)下载的 Excel 2013 或更高版本以及 Power Query。

    还可以使用 Power Query 将审核日志从 Azure 存储帐户直接导入 Excel 模板。 然后可以浏览审核记录，并在日志数据顶部创建仪表板和报表。

    ![导航窗格][9]

## <a id="subheading-5"></a>生产环境中的用法实践
<!--The description in this section refers to screen captures above.-->

### <a id="subheading-6">审核异地复制的数据库</a>
使用异地复制的数据库时，可以根据审核类型，针对主数据库和/或辅助数据库设置审核。

**表审核** - 可以根据[为数据库设置审核](#subheading-2-2)部分中所述，在数据库或服务器级别为两个数据库（主数据库或辅助数据库）中的一个配置不同的策略。

**Blob 审核** - 请遵循以下说明：

1. **主数据库** - 根据[为数据库设置审核](#subheading-2-1)部分中所述，在服务器或数据库本身上打开“Blob 审核”。
2. **辅助数据库** - 仅可以通过主数据库审核设置打开/关闭 Blob 审核。

   * 根据[为数据库设置审核](#subheading-2-1)部分中所述，对“主数据库”打开“Blob 审核”。 必须在*主数据库本身*上启用 Blob 审核，而不要在服务器上启用。
   * 在主数据库上启用 Blob 审核后，也会在辅助数据库上启用 Blob 审核。

    > [AZURE.IMPORTANT]
    > 默认情况下，辅助数据库的**存储设置**与主数据库相同，因而会导致生成**跨区域流量**。 在**辅助服务器**上启用 Blob 审核并在辅助服务器存储设置中配置**本地存储**可以避免此问题（这会覆盖辅助数据库的存储位置，导致每个数据库将审核日志保存到本地存储中）。  

<br>

### <a id="subheading-6"></a>重新生成存储密钥
在生产环境中，可能会定期刷新存储密钥。 刷新密钥时，需要重新保存审核策略。 过程如下：

1. 在存储详细信息边栏选项卡中，将“存储访问密钥”从“主要”切换为“辅助”，然后单击底部的“确定”。 然后，单击审核配置边栏选项卡顶部的“保存”。

    ![导航窗格][6]
2. 转到存储配置边栏选项卡，并 **重新生成***主访问密钥*。

    ![导航窗格][8]
3. 返回到审核配置边栏选项卡，将“存储访问密钥”从“辅助”切换为“主要”，然后单击底部的“确定”。 然后，单击审核配置边栏选项卡顶部的“保存”。
4. 返回到存储配置边栏选项卡并**重新生成***辅助访问密钥*（为下一个密钥刷新周期做好准备）。

## <a id="subheading-7"></a>自动化 (PowerShell/REST API)
也可以使用以下自动化工具在 Azure SQL 数据库中配置审核：

1. **PowerShell cmdlet**

   * [Get-AzureRMSqlDatabaseAuditingPolicy][101]
   * [Get-AzureRMSqlServerAuditingPolicy][102]
   * [Remove-AzureRMSqlDatabaseAuditing][103]
   * [Remove-AzureRMSqlServerAuditing][104]
   * [Set-AzureRMSqlDatabaseAuditingPolicy][105]
   * [Set-AzureRMSqlServerAuditingPolicy][106]
   * [Use-AzureRMSqlServerAuditingPolicy][107]
2. **REST API - Blob 审核**

   * [创建或更新数据库 Blob 审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt695939.aspx)
   * [创建或更新服务器 Blob 审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt771861.aspx)
   * [获取数据库 Blob 审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt695938.aspx)
   * [获取服务器 Blob 审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt771860.aspx)
   * [获取服务器 Blob 审核操作结果](https://msdn.microsoft.com/zh-cn/library/azure/mt771862.aspx)
3. **REST API - 表审核（已弃用）**

   * [创建或更新数据库审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt604471.aspx)
   * [创建或更新服务器审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt604383.aspx)
   * [获取数据库审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt604381.aspx)
   * [获取服务器审核策略](https://msdn.microsoft.com/zh-cn/library/azure/mt604382.aspx)

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

[101]: https://msdn.microsoft.com/zh-cn/library/azure/mt603731(v=azure.200).aspx
[102]: https://msdn.microsoft.com/zh-cn/library/azure/mt619329(v=azure.200).aspx
[103]: https://msdn.microsoft.com/zh-cn/library/azure/mt603796(v=azure.200).aspx
[104]: https://msdn.microsoft.com/zh-cn/library/azure/mt603574(v=azure.200).aspx
[105]: https://msdn.microsoft.com/zh-cn/library/azure/mt603531(v=azure.200).aspx
[106]: https://msdn.microsoft.com/zh-cn/library/azure/mt603794(v=azure.200).aspx
[107]: https://msdn.microsoft.com/zh-cn/library/azure/mt619353(v=azure.200).aspx

<!--Update_Description: deprecated table audit; add production practice introduction; content structure refine-->