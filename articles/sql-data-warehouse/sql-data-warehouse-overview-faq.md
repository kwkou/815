<properties
    pageTitle="Azure SQL 数据仓库常见问题解答 | Azure"
    description="本文列出客户和开发人员提出的 Azure SQL 数据仓库常见问题"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="hirokib"
    manager="johnmac"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="812CA525-3BF3-49DF-8DF3-FB4342464F4F"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="overview"
    ms.date="3/1/2017"
    wacn.date="05/08/2017"
    ms.author="elbutter"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="5b8246bd732b99e7fe24acb97dea95125fa66f8b"
    ms.lasthandoff="04/28/2017" />

# <a name="sql-data-warehouse-frequently-asked-questions"></a>SQL 数据仓库常见问题解答

## <a name="general"></a>常规

问： SQL DW 为数据安全提供哪些功能？

A. SQL DW 提供若干解决方案，用于保护数据（如 TDE）和审核。 有关详细信息，请参阅[安全性]。

问： 从何处可以查明 SQL DW 符合哪些法律或业务标准？

A. 请访问 [Microsoft 符合性]页面，查明产品（如 SOC 和 ISO）的各种符合性规定。 首先选择“符合性”标题，然后在页面右侧的“Microsoft 范围内云服务”部分展开 Azure，查看哪些服务是 Azure 符合性服务。

问： 是否可以连接 PowerBI？

A. 能！ 尽管 PowerBI 支持使用 SQL DW 进行直接查询，但不适合大量用户或实时数据。 若要将 PowerBI 用于生产用途，建议基于 Azure Analysis Services 或 Analysis Service IaaS 使用 PowerBI。 

问： SQL 数据仓库容量限制有哪些？

A. 请参阅当前[容量限制]页。 

问： 为什么缩放/暂停/恢复会花费很长时间？

A. 多种因素可能会影响计算管理操作的时间。 一个常见的长时运行操作的例子是事务回退。 缩放或暂停操作启动时，会阻止所有传入会话和查询。 为了使系统处于稳定状态，必须在操作开始前回退事务。 事务数量越多，其日志大小越大，使系统恢复到稳定状态的操作耗时越久。

## <a name="user-support"></a>用户支持

问： 该如何操作？

A. 有关使用 SQL 数据仓库进行开发的帮助，可在 [Stack Overflow] 页提出问题。 

## <a name="sql-languagefeature-support"></a>SQL 语言/功能支持 

问： SQL 数据仓库支持哪些数据类型？

A. 请参阅 SQL 数据仓库[数据类型]。

问： 支持哪些表功能？

A. 虽然 SQL 数据仓库支持许多功能，但不支持某些功能，这些功能记录在[不支持的表功能]中。

## <a name="tooling-and-administration"></a>工具和管理

问： Visual Studio 中是否支持数据库项目？

A. 目前 Visual Studio 对于 SQL 数据仓库，不支持数据库项目。 如果想通过投票获取此功能，请访问 User Voice [数据库项目功能请求]。

问： SQL 数据仓库是否支持 REST API？

A. 是的。 SQL 数据仓库还提供可与 SQL 数据库搭配使用的大多数 REST 功能。 可以在 REST 文档页或 [MSDN] 中找到 API 信息。


## <a name="loading"></a>加载

问： 支持哪些客户端驱动程序？

A. 可在[连接字符串]页找到 DW 驱动程序支持

问：使用 SQL 数据仓库时，PolyBase 支持哪些文件格式？

答：Orc、RC、Parquet 和平面分隔文本

问：连接 Azure 存储 Blob 或 ADLS 时能否进行计算下推？ 

答：不能，SQL DW PolyBase 仅与存储组件交互。 

问：能否连接到 HDI？

答：HDI 可使用 ADLS 或 WASB 作为 HDFS 层。 如果将两者中任意一种作为 HDFS 层，可以将该数据加载到 SQL DW。 但是，无法生成 HDI 实例的下推计算。 

## <a name="next-steps"></a>后续步骤
若要深入了解 SQL 数据仓库的概述信息，请参阅[概述]页。

<!-- Article references -->
[UserVoice]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[连接字符串]: /documentation/articles/sql-data-warehouse-connection-strings/
[Stack Overflow]: http://stackoverflow.com/questions/tagged/azure-sqldw
[支持票证]: /documentation/articles/sql-data-warehouse-get-started-create-support-ticket/
[安全性]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[Microsoft 符合性]: https://www.microsoft.com/en-us/trustcenter/compliance/complianceofferings
[容量限制]: /documentation/articles/sql-data-warehouse-service-capacity-limits/
[数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[不支持的表功能]: /documentation/articles/sql-data-warehouse-tables-overview/#unsupported-table-features
[Azure Data Lake Store]: /documentation/articles/sql-data-warehouse-load-from-azure-data-lake-store/
[Azure 存储 Blob]: /documentation/articles/sql-data-warehouse-load-from-azure-blob-storage-with-polybase/
[数据库项目功能请求]: https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/13313247-database-project-from-visual-studio-to-support-azu
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx
[概述]: /documentation/articles/sql-data-warehouse-overview-faq/

<!--Update_Description:new content about the SQL Data warehouse FAQ-->