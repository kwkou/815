<properties
   pageTitle="将数据迁移到 SQL 数据仓库 | Azure"
   description="有关在开发解决方案时将数据迁移到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/03/2016"
   wacn.date="04/11/2016"/>

# 迁移数据
迁移数据时的主要目标是填充 SQLDW 数据库。此过程可以通过多种方式来完成。SSIS 和 bcp 都可用来实现此目标。但是，随着数据量的增加，你应该考虑将数据迁移过程划分成多个步骤。这样，你便有机会优化每个步骤以提高性能和弹性，确保顺利迁移数据。

本文首先讨论 SSIS 和 bcp 的简单迁移方案。然后稍微深入讨论如何优化迁移。

## Integration Services ##
集成服务 (SSIS) 是一个功能强大且灵活的提取、转换和加载 (ETL) 工具，支持复杂的工作流、数据转换，以及多个数据加载选项。使用 SSIS 可以单纯将数据传输到 Azure，或作为更广泛迁移的一部分。

> [AZURE.NOTE] SSIS 可以导出为 UTF-8，文件中不需要有字节顺序标记。若要这样配置，必须先使用派生的列组件，转换数据流中的字符数据来使用 65001 UTF-8 代码页。转换列之后，将数据写入平面文件目标适配器，但要确保同时选择了 65001 作为文件的代码页。

SSIS 将连接到 SQL 数据仓库，就像连接到 SQL Server 部署一样。但是，你的连接需要使用 ADO.NET 连接管理器。你还应谨慎配置“可用时使用批量插入”设置，以最大化吞吐量。请参阅 [ADO.NET 目标适配器][]一文以深入了解此属性

> [AZURE.NOTE] 不支持使用 OLEDB 连接到 Azure SQL 数据仓库。

此外，包经常可能会因为限制或网络问题而失败。请将包设计为可以从失败点恢复，而不必重做失败之前已完成的工作。

有关详细信息，请参阅 [SSIS 文档][]。

## bcp
bcp 是专为平面文件数据导入和导出所设计的命令行实用程序。数据导出期间可以进行某些转换。若要执行简单的转换，请使用查询来选择和转换数据。导出之后，可将平面文件直接加载到目标 SQL 数据仓库数据库。

> [AZURE.NOTE] 通常建议将数据导出时使用的转换封装在源系统上的视图中。这可以确保逻辑得到保留，并且过程可重复。

bcp 的优点包括：

- 简单。bcp 命令的构建和执行十分简单
- 可重新启动的加载过程。导出后，即可不限次数执行加载

bcp 的限制包括：

- bcp 只能使用表格型平面文件。无法使用 xml 或 JSON 之类的文件
- bcp 不支持导出为 UTF-8。这可能导致无法对 bcp 导出的数据使用 PolyBase
- 数据转换功能仅限于导出阶段且性质简单
- 当通过 Internet 加载数据时，bcp 的设计尚不够健全。任何网络不稳定状况都可能造成加载错误。
- bcp 依赖于加载之前存在于目标数据库中的架构

有关详细信息，请参阅[使用 bcp 将数据载入 SQL 数据仓库][]。

## 优化数据迁移
SQLDW 数据迁移过程可以有效地划分成三个独立的步骤：

1. 导出源数据
2. 将数据传输到 Azure
3. 载入目标 SQLDW 数据库

每个步骤可以单独进行优化，以创建稳健、可重新启动且富有弹性的迁移过程，让每个步骤发挥最高的性能。

## 优化数据加载
反过来看，加载数据最快的方式是通过 PolyBase。优化 PolyBase 加载过程对上述步骤规定了先决条件，最好先了解这一点。它们具有以下特点：

1. 数据文件的编码
2. 数据文件的格式
3. 数据文件的位置

### 编码
PolyBase 规定数据文件必须采用 UTF-8 编码。这意味着在导出数据时，数据必须符合这项要求。如果数据只包含基本 ASCII 字符（不是扩展的 ASCII），则这些直接映射到 UTF-8 标准，不需要太担心编码。但是，如果数据包含任何特殊字符，例如变音符、腔调符或符号，或数据支持非拉丁语言，则必须确保导出文件经过适当的 UTF-8 编码。

> [AZURE.NOTE] bcp 不支持将数据导出为 UTF-8。

数据传输***之前***，任何使用 UFT-16 编码的文件都需要经过重新编写。

### 数据文件的格式
PolyBase 规定要有固定的行终止符 \\n 或换行符。数据文件必须符合此标准。字符串或列终止符没有任何限制。

在 PolyBase 中，必须将文件中的每个列定义为外部表的一部分。请确保所有导出的列都是必需的，且类型符合所需的标准。

有关支持的数据类型的详细信息，请参阅前面的 [迁移架构] 一文。

### 数据文件的位置
SQL 数据仓库只使用 PolyBase 从 Azure Blob 存储加载数据。因此，数据必须先传输到 Blob 存储。

## 优化数据传输
数据迁移过程中，最慢的一部分是将数据传输到 Azure。不只网络带宽可能是个问题，网络可靠性也可能会严重阻碍进度。默认情况下，将数据迁移到 Azure 是通过 Internet 完成的，因此很可能会发生传输错误。但是，这些错误可能需要重新发送整个或部分数据。

幸运的是，有多个选项可以提升此过程的速度和恢复能力：

### [ExpressRoute][]
你可以考虑使用 [ExpressRoute][] 加速传输。[ExpressRoute][] 为你提供可靠的 Azure 专用连接，这种连接不经由公共 Internet。这不是一项强制措施。但是，从本地或共置设备向 Azure 推送数据时，它会改善吞吐量。

使用 [ExpressRoute][] 的优点包括：

1. 更高的可靠性
2. 更快的网络速度
3. 更低的网络延迟
4. 更高的网络安全性

[ExpressRoute][] 有利于许多情况，而不只是迁移。

感兴趣吗？ 有关详细信息和价格，请访问 [ExpressRoute 文档][]。

### Azure 导入和导出服务
Azure 导入和导出服务是一个数据传输进程，用于将大量 (GB++) 和巨量 (TB++) 的数据传输到 Azure。它涉及到将数据写入磁盘并传送到 Azure 数据中心。然后代你将磁盘内容载入 Azure 存储 Blob。

导入导出过程的高级视图如下：

1. 配置要接收数据的 Azure Blob 存储容器
2. 将数据导出到本地存储
2. 使用 [Azure 导入/导出工具] 将数据复制到 3.5 英吋 SATA II/III 硬盘
3. 使用 Azure 导入和导出服务，并提供 [Azure 导入/导出工具] 生成的日记文件，来创建导入作业
4. 将磁盘寄送到指定的 Azure 数据中心
5. 你的数据将传输到 Azure Blob 存储容器
6. 使用 PolyBase 将数据载入 SQLDW

### [AZCopy][] 实用程序
[AZCopy][] 实用程序是将数据传输到 Azure 存储 Blob 的一个极佳工具。它旨在用于少量 (MB++) 到极大量 (GB++) 的数据传输。将数据传输到 Azure 时，[AZCopy] 还能提供高度灵活的吞吐量，因此是数据传输措施的理想选择。传输后，你可以使用 PolyBase 将数据载入 SQL 数据仓库。你还可以使用“执行进程”任务将 AZCopy 合并到 SSIS 包中。

若要使用 AZCopy，必须先下载并安装它。可用版本包括[生产版][]和[预览版][]。

若要从文件系统上载文件，需要执行如下所示的命令：

```
AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:abc.txt
```

高级过程摘要如下：

1. 配置要接收数据的 Azure 存储 Blob 容器
2. 将数据导出到本地存储
3. 使用 AZCopy 复制 Azure Blob 存储容器中的数据
4. 使用 PolyBase 将数据载入 SQL 数据仓库

完整文档：[AZCopy][]。

## 优化数据导出
除了确保导出符合 PolyBase 规定的要求外，你还可以设法优化数据导出，以进一步改善过程。

> [AZURE.NOTE] 由于 PolyBase 要求数据必须采用 UTF-8 格式，因此你不太可能会使用 bcp 来执行数据导出。bcp 不支持将数据文件输出为 UTF-8。SSIS 比较适合执行此类数据导出。

### 数据压缩
PolyBase 可以读取 gzip 压缩数据。如果你可以将数据压缩成 gzip 文件，就能将网络上推送的数据量减到最少。

### 多个文件
将大型表分割成多个文件不仅有助于改善导出速度，还有助于重新开始传输，以及数据进入 Azure Blob 存储之后的整体易管理性。PolyBase 的众多优点之一是可以读取文件夹内的所有文件，并将其视为一个表。因此，最好将每个表的文件隔离到它自身的文件夹中。

PolyBase 还支持名为“递归文件夹遍历”的功能。你可以使用此功能来进一步增强所导出数据的组织方式，以改善数据管理。

若要深入了解如何使用 PolyBase 加载数据，请参阅[使用 PolyBase 将数据载入 SQL 数据仓库][]。


## 后续步骤
有关迁移的详细信息，请参阅[将解决方案迁移到 SQL 数据仓库][]。

<!--Image references-->

<!--Article references-->
[AZCopy]: /documentation/articles/storage-use-azcopy/
[ADF 复制]: /documentation/articles/data-factory-copy-activity/
[ADF 复制示例]: /documentation/articles/data-factory-copy-activity-examples/
[开发概述]: /documentation/articles/sql-data-warehouse-develop-overview/
[将解决方案迁移到 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-overview-migrate/
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[使用 bcp 将数据载入 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[使用 PolyBase 将数据载入 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-load-with-polybase/


<!--MSDN references-->

<!--Other Web references-->
[Azure 数据工厂]: /services/data-factory/
[ExpressRoute]: /services/expressroute/
[ExpressRoute 文档]: /documentation/services/expressroute/
[生产版]: http://aka.ms/downloadazcopy/
[预览版]: http://aka.ms/downloadazcopypr/
[ADO.NET 目标适配器]: https://msdn.microsoft.com/zh-cn/library/bb934041.aspx
[SSIS 文档]: https://msdn.microsoft.com/zh-cn/library/ms141026.aspx

<!---HONumber=Mooncake_0307_2016-->