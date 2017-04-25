<!-- Remove solution-partners -->
<properties
    pageTitle="将数据加载到 Azure SQL 数据仓库 | Azure"
    description="了解将数据加载到 SQL 数据仓库的常见方案。 这些常见方案包括使用 PolyBase、Azure Blob 存储、平面文件以及磁盘寄送。 也可使用第三方工具。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="2253bf46-cf72-4de7-85ce-f267494d55fa"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    ms.author="barbkess"
    wacn.date="04/24/2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="02d57a17601f5cce34dbf10ab06aa42f22b4df7c"
    ms.lasthandoff="04/14/2017" />

# <a name="load-data-into-azure-sql-data-warehouse"></a>将数据载入 Azure SQL 数据仓库
本文汇总了将数据载入 SQL 数据仓库时的方案选项和建议。

加载数据时，最难的部分通常是准备要加载的数据。 Azure 使用 Azure Blob 存储作为许多此类服务的常用数据存储，并使用 Azure 数据工厂来协调 Azure 服务之间的通信和数据移动，使得加载过程得到简化。 这些过程集成了 PolyBase 技术，因此可以通过大规模并行处理 (MPP) 将数据从 Azure Blob 存储并行加载到 SQL 数据仓库。 

有关加载示例数据库的教程，请参阅[加载示例数据库][Load sample databases]。

## <a name="load-from-azure-blob-storage"></a>从 Azure Blob 存储加载
将数据导入 SQL 数据仓库时，最快的方式是使用 PolyBase 从 Azure Blob 存储加载数据。 PolyBase 使用 SQL 数据仓库的大规模并行处理 (MPP) 设计以并行方式从 Azure Blob 存储加载数据。 若要使用 PolyBase，可以使用 T-SQL 命令或 Azure 数据工厂管道。

### <a name="use-polybase-and-t-sql"></a>使用 PolyBase 和 T-SQL
加载过程摘要：

1. 将数据移到 Azure Blob 存储并存储在文本文件中。    <!--Data Lake Store Not supported in ACN-->
2. 在 SQL 数据仓库中配置外部对象，以便定义数据的位置和格式
3. 运行 T-SQL 命令，将数据以并行方式加载到新的数据库表。

有关教程，请参阅[将数据从 Azure Blob 存储加载到 SQL 数据仓库 (PolyBase)][Load data from Azure blob storage to SQL Data Warehouse (PolyBase)]。
<!-- ADF Not supported in ACN-->

## <a name="load-from-sql-server"></a>从 SQL Server 加载
若要将数据从 SQL Server 加载到 SQL 数据仓库，可以使用 Integration Services (SSIS)，可以传输平面文件，还可以将磁盘寄送到 Microsoft。 继续阅读下去，你会看到各种不同加载过程的摘要，以及教程链接。

若打算将数据从 SQL Server 完整迁移到 SQL 数据仓库，请参阅[迁移概述][Migration overview]。 

### <a name="use-integration-services-ssis"></a>使用 Integration Services (SSIS)
如果你已经在使用 Integration Services (SSIS) 包载入 SQL Server，则可更新这些包，将 SQL Server 用作源并且将 SQL 数据仓库用作目标。 如果不打算迁移加载过程以使用云中的已有数据，那么采用这种方式既快速又方便。 不利的一面是，加载速度会比使用 PolyBase 慢，因为此 SSIS 不会以并行方式进行加载。

加载过程摘要：

1. 修改 Integration Services 包，使之指定 SQL Server 实例作为源，指定 SQL 数据仓库数据库作为目标。
2. 将架构迁移到 SQL 数据仓库（如果尚未迁移到该处）。
3. 更改包中的映射方式，仅使用 SQL 数据仓库支持的数据类型。
4. 计划和运行包。

有关教程，请参阅[将数据从 SQL Server 加载到 Azure SQL 数据仓库 (SSIS)][Load data from SQL Server to Azure SQL Data Warehouse (SSIS)]。

### <a name="use-azcopy-recommended-for--10-tb-data"></a>使用 AZCopy（建议用于数据量 < 10 TB 的情况）
如果数据大小 < 10 TB，则可先将数据从 SQL Server 导出到平面文件，然后将这些文件复制到 Azure Blob 存储，最后再使用 PolyBase 将数据载入 SQL 数据仓库

加载过程摘要：

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到平面文件。
2. 使用 AZCopy 命令行实用程序将数据从平面文件复制到 Azure Blob 存储。
3. 通过 PolyBase 加载到 SQL 数据仓库中。

有关教程，请参阅[将数据从 Azure Blob 存储加载到 SQL 数据仓库 (PolyBase)][Load data from Azure blob storage to SQL Data Warehouse (PolyBase)]。

### <a name="use-bcp"></a>使用 bcp
如果你的数据量小，则可使用 bcp 将数据直接载入 Azure SQL 数据仓库。

加载过程摘要：

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到平面文件。
2. 使用 bcp 将数据从平面文件直接加载到 SQL 数据仓库。

有关教程，请参阅[将数据从 SQL Server 加载到 Azure SQL 数据仓库 (bcp)][Load data from SQL Server to Azure SQL Data Warehouse (bcp)]。

### <a name="use-importexport-recommended-for--10-tb-data"></a>使用导入/导出（建议用于数据量 > 10 TB 的情况）
如果数据大小 > 10 TB 并且需要将数据移至 Azure，则建议使用磁盘寄送服务：[导入/导出][Import/Export]。 

加载过程摘要

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到可转移磁盘上的平面文件。
2. 将磁盘寄送到 Microsoft。
3. Microsoft 将数据载入 SQL 数据仓库

## <a name="load-from-hdinsight"></a>从 HDInsight 加载
SQL 数据仓库支持通过 PolyBase 从 HDInsight 加载数据。 该过程和从 Azure Blob 存储加载数据一样 - 使用 PolyBase 连接到 HDInsight 以加载数据。 

### <a name="use-polybase-and-t-sql"></a>使用 PolyBase 和 T-SQL
加载过程摘要：

1. 将数据移到 HDInsight，并将其存储为文本文件、ORC 或 Parquet 格式。
2. 在 SQL 数据仓库中配置外部对象，以便定义数据的位置和格式。
3. 运行 T-SQL 命令，将数据以并行方式加载到新的数据库表。

有关教程，请参阅[将数据从 Azure Blob 存储加载到 SQL 数据仓库 (PolyBase)][Load data from Azure blob storage to SQL Data Warehouse (PolyBase)]。

## <a name="recommendations"></a>建议
<!--solution parterners not available in Azure.cn-->
<!--我们的很多合作伙伴都提供加载解决方案。 若要获取更多解决方案，请参阅我们的 [解决方案合作伙伴][solution partners]的列表。 -->

如果你的数据来自非关系源，而你想要将其载入 SQL 数据仓库，则需在加载前先将其转换成行和列。已转换的数据不需存储在数据库中，可以存储在文本文件中。

基于新加载的数据创建统计信息。 Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。  为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。  有关详细信息，请参阅 [统计信息][Statistics]。

## <a name="next-steps"></a>后续步骤
有关更多开发技巧，请参阅 [开发概述][development overview]。

<!--Image references-->

<!--Article references-->
[Load data from Azure blob storage to SQL Data Warehouse (PolyBase)]: /documentation/articles/sql-data-warehouse-load-from-azure-blob-storage-with-polybase/
[Load data from Azure blob storage to SQL Data Warehouse (Azure Data Factory)]: /documentation/articles/sql-data-warehouse-load-from-azure-blob-storage-with-data-factory/
[Load data from SQL Server to Azure SQL Data Warehouse (SSIS)]: /documentation/articles/sql-data-warehouse-load-from-sql-server-with-integration-services/
[Load data from SQL Server to Azure SQL Data Warehouse (bcp)]: /documentation/articles/sql-data-warehouse-load-from-sql-server-with-bcp/
[Load data from SQL Server to Azure SQL Data Warehouse (AZCopy)]: /documentation/articles/sql-data-warehouse-load-from-sql-server-with-azcopy/

[Load sample databases]: /documentation/articles/sql-data-warehouse-load-sample-databases/
[Migration overview]: /documentation/articles/sql-data-warehouse-overview-migrate/
[solution partners]: /documentation/articles/sql-data-warehouse-partner-business-intelligence/
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/

<!--MSDN references-->

<!--Other Web references-->
[Import/Export]: https://azure.microsoft.com/documentation/articles/storage-import-export-service/

<!--Update_Description:update meta properties;wording update-->