<properties
    pageTitle="将数据迁移到 SQL 数据仓库 | Azure"
    description="有关在开发解决方案时将数据迁移到 Azure SQL 数据仓库的技巧。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="jrowlandjones"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="d78f954a-f54c-4aa4-9040-919bc6414887"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="04/24/2017"
    ms.author="jrj;barbkess"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="e0c0e7e7994d1fc8b985dc54b7bdcd9fb17a0136"
    ms.lasthandoff="04/14/2017" />

# <a name="migrate-your-data"></a>迁移数据
数据可以使用各种工具从不同源移动到 SQL 数据仓库中。  ADF 复制、SSIS 和 bcp 都可用来实现此目标。 但是，随着数据量的增加，你应该考虑将数据迁移过程划分成多个步骤。 这样，你便有机会优化每个步骤以提高性能和弹性，确保顺利迁移数据。

本文首先讨论 SSIS 和 bcp 的简单迁移方案。 然后稍微深入讨论如何优化迁移。


## <a name="integration-services"></a>Integration Services
集成服务 (SSIS) 是一个功能强大且灵活的提取、转换和加载 (ETL) 工具，支持复杂的工作流、数据转换，以及多个数据加载选项。 使用 SSIS 可以单纯将数据传输到 Azure，或作为更广泛迁移的一部分。

> [AZURE.NOTE]
> SSIS 可以导出为 UTF-8，文件中不需要有字节顺序标记。 若要这样配置，必须先使用派生的列组件，转换数据流中的字符数据来使用 65001 UTF-8 代码页。 转换列之后，将数据写入平面文件目标适配器，但要确保同时选择了 65001 作为文件的代码页。
> 
> 

SSIS 将连接到 SQL 数据仓库，就像连接到 SQL Server 部署一样。 但是，你的连接需要使用 ADO.NET 连接管理器。 你还应谨慎配置“可用时使用批量插入”设置，以最大化吞吐量。 请参阅 [ADO.NET 目标适配器][ADO.NET destination adapter] 一文以深入了解此属性

> [AZURE.NOTE]
> 不支持使用 OLEDB 连接到 Azure SQL 数据仓库。
> 
> 

此外，程序包经常可能会因为限制或网络问题而失败。 请将程序包设计为可以从失败点恢复，而不必重做失败之前已完成的工作。

有关详细信息，请参阅 [SSIS 文档][SSIS documentation]。

## <a name="bcp"></a>bcp
bcp 是专为平面文件数据导入和导出所设计的命令行实用工具。 数据导出期间可以进行某些转换。 若要执行简单的转换，请使用查询选择和转换数据。 导出之后，可将平面文件直接加载到目标 SQL 数据仓库数据库。

> [AZURE.NOTE]
> 通常建议将数据导出期间使用的转换封装在源系统上的视图中。 这可以确保逻辑得到保留，并且过程可重复。
> 
> 

bcp 的优点包括：

* 简单。 bcp 命令的生成和执行十分简单
* 可重新启动的加载过程。 导出后，即可不限次数执行加载

bcp 的限制包括：

* bcp 只能使用表格型平面文件。 无法使用 xml 或 JSON 之类的文件
* 数据转换功能仅限于导出阶段且性质简单
* 当通过 Internet 加载数据时，bcp 的设计尚不够健全。 任何网络不稳定状况都可能导致加载错误。
* bcp 依赖于加载之前存在于目标数据库中的架构

有关详细信息，请参阅 [使用 bcp 将数据载入 SQL 数据仓库][Use bcp to load data into SQL Data Warehouse]。

## <a name="optimizing-data-migration"></a>优化数据迁移
SQLDW 数据迁移过程可以有效地划分成三个独立的步骤：

1. 导出源数据
2. 将数据传输到 Azure
3. 载入目标 SQLDW 数据库

每个步骤可以单独进行优化，创建稳健、可重新启动且富有弹性的迁移过程，让每个步骤发挥最高的性能。

## <a name="optimizing-data-load"></a>优化数据加载
反过来看，加载数据最快的方式是通过 PolyBase。 优化 PolyBase 加载过程对上述步骤规定了先决条件，最好先了解这一点。 它们具有以下特点：

1. 数据文件的编码
2. 数据文件的格式
3. 数据文件的位置

### <a name="encoding"></a>编码
PolyBase 要求数据文件必须为 UTF-8 或 UTF-16FE。 

### <a name="format-of-data-files"></a>数据文件的格式
PolyBase 规定要有固定的行终止符 \n 或换行符。 数据文件必须符合此标准。 字符串或列终止符没有任何限制。

在 PolyBase 中，必须将文件中的每个列定义为外部表的一部分。 请确保所有导出的列都是必需的，且类型符合所需的标准。

有关支持的数据类型的详细信息，请参阅前面的 [迁移架构] 一文。

### <a name="location-of-data-files"></a>数据文件的位置
SQL 数据仓库只使用 PolyBase 从 Azure Blob 存储加载数据。 因此，数据必须先传输到 Blob 存储。

## <a name="optimizing-data-transfer"></a>优化数据传输
数据迁移过程中，最慢的一部分是将数据传输到 Azure。 不只网络带宽可能是个问题，网络可靠性也可能会严重阻碍进度。 默认情况下，将数据迁移到 Azure 是通过 Internet 完成的，因此很可能会发生传输错误。 但是，这些错误可能需要重新发送整个或部分数据。

幸运的是，有多个选项可以提升此过程的速度和恢复能力：

### <a name="expressrouteexpressroute"></a>[ExpressRoute][ExpressRoute]
你可以考虑使用 [ExpressRoute][ExpressRoute] 加速传输。 [ExpressRoute][ExpressRoute] 为你提供可靠的 Azure 专用连接，这种连接不经由公共 Internet。 这不是一项强制措施。 但是，从本地或归置设备向 Azure 推送数据时，它会改善吞吐量。

使用 [ExpressRoute][ExpressRoute] 的优点包括：

1. 更高的可靠性
2. 更快的网络速度
3. 更低的网络延迟
4. 更高的网络安全性

[ExpressRoute][ExpressRoute] 有利于许多情况，而不只是迁移。

感兴趣吗？ 有关详细信息和定价，请访问 [ExpressRoute 文档][ExpressRoute documentation]。

### <a name="azure-import-and-export-service"></a>Azure 导入和导出服务
Azure 导入和导出服务是一个数据传输进程，用于将大量 (GB++) 和巨量 (TB++) 的数据传输到 Azure。 它涉及到将数据写入磁盘并传送到 Azure 数据中心。 然后代用户将磁盘内容载入 Azure 存储 Blob。

导入导出过程的概略性视图如下所示：

1. 配置要接收数据的 Azure Blob 存储容器
2. 将数据导出到本地存储
3. 使用 [Azure 导入/导出工具] 将数据复制到 3.5 英寸 SATA II/III 硬盘驱动器
4. 使用 Azure 导入和导出服务，并提供 [Azure 导入/导出工具] 生成的日记文件，创建导入作业
5. 将磁盘寄送到指定的 Azure 数据中心
6. 你的数据将传输到 Azure Blob 存储容器
7. 使用 PolyBase 将数据载入 SQLDW

### <a name="azcopyazcopy-utility"></a>[AZCopy][AZCopy] 实用工具
[AZCopy][AZCopy] 实用程序是将数据传输到 Azure 存储 Blob 的一个极佳工具。 它旨在用于少量 (MB++) 到极大量 (GB++) 的数据传输。 将数据传输到 Azure 时，[AZCopy] 还能提供高度灵活的吞吐量，因此是数据传输措施的理想选择。 传输后，你可以使用 PolyBase 将数据载入 SQL 数据仓库。 你还可以使用“执行进程”任务将 AZCopy 合并到 SSIS 包中。

若要使用 AZCopy，必须先下载并安装它。 可用版本包括[生产版][production version]和[预览版][preview version]。

若要从文件系统上传文件，需要执行如下所示的命令：

    AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:abc.txt

高级过程摘要如下：

1. 配置要接收数据的 Azure 存储 Blob 容器
2. 将数据导出到本地存储
3. 使用 AZCopy 复制 Azure Blob 存储容器中的数据
4. 使用 PolyBase 将数据载入 SQL 数据仓库

提供的完整文档： [AZCopy][AZCopy]的一部分。

## <a name="optimizing-data-export"></a>优化数据导出
除了确保导出符合 PolyBase 规定的要求外，你还可以设法优化数据导出，以进一步改善过程。

### <a name="data-compression"></a>数据压缩
PolyBase 可以读取 gzip 压缩数据。 如果你可以将数据压缩成 gzip 文件，就能将网络上推送的数据量减到最少。

### <a name="multiple-files"></a>多个文件
将大型表分割成多个文件不仅有助于改善导出速度，还有助于重新开始传输，以及数据进入 Azure Blob 存储之后的整体易管理性。 PolyBase 的众多优点之一是可以读取文件夹内的所有文件，并将其视为一个表。 因此，最好将每个表的文件隔离到它自身的文件夹中。

PolyBase 还支持名为“递归文件夹遍历”的功能。 你可以使用此功能进一步增强所导出数据的组织方式，以改善数据管理。

若要深入了解如何使用 PolyBase 加载数据，请参阅 [使用 PolyBase 将数据载入 SQL 数据仓库][Use PolyBase to load data into SQL Data Warehouse]。

## <a name="next-steps"></a>后续步骤
有关迁移的详细信息，请参阅 [将解决方案迁移到 SQL 数据仓库][Migrate your solution to SQL Data Warehouse]。
有关更多开发技巧，请参阅 [开发概述][development overview]。

<!--Image references-->


<!--Article references-->
[AZCopy]: /documentation/articles/storage-use-azcopy/
<!--Azure Data Factory (ADF) Not supported in ACN-->
[ADF 复制]: /documentation/articles/data-factory-copy-activity/
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Migrate your solution to SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-overview-migrate/
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Use bcp to load data into SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[Use PolyBase to load data into SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-get-started-load-with-polybase/


<!--MSDN references-->

<!--Other Web references-->
[Azure 数据工厂]: /documentation/services/data-factory/
[ExpressRoute]: /documentation/services/expressroute/
[ExpressRoute documentation]: /documentation/services/expressroute/

[production version]: http://aka.ms/downloadazcopy/
[preview version]: http://aka.ms/downloadazcopypr/
[ADO.NET destination adapter]: https://msdn.microsoft.com/zh-cn/library/bb934041.aspx
[SSIS documentation]: https://msdn.microsoft.com/zh-cn/library/ms141026.aspx

<!--Update_Description:update meta properties;wording update-->