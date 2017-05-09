<properties
    pageTitle="Azure HDInsight 上的 Hadoop 组件发行说明 | Azure"
    description="Azure HDInsight 的 Hadoop 组件的最新发行说明和版本。 获取有关 Hadoop、Apache Storm 和 HBase 的开发技巧和详细信息。"
    services="hdinsight"
    documentationcenter=""
    editor="cgronlun"
    manager="jhubbard"
    author="nitinme"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="a363e5f6-dd75-476a-87fa-46beb480c1fe"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="4/06/2017"
    wacn.date="05/08/2017"
    ms.author="nitinme"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="9e640c7547562a5e2018393aa084556d0d1fccc2"
    ms.lasthandoff="04/28/2017" />

# <a name="release-notes-for-hadoop-components-on-azure-hdinsight"></a>Azure HDInsight 上的 Hadoop 组件发行说明

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

本文提供有关**最新** Azure HDInsight 版本更新的信息。 有关之前版本的信息，请参阅 [HDInsight 发行说明存档](/documentation/articles/hdinsight-release-notes-archive/)。

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 版本控制文章](/documentation/articles/hdinsight-component-versioning/)。

## <a name="04062017---general-availability-of-hdinsight-36"></a>2017 年 4 月 6 日：HDInsight 3.6 公开上市

* 在此次发布中，Azure HDInsight 添加了基于 HDP 2.6 的 3.6 版。 [此处](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_release-notes/content/ch_relnotes.html)提供 HDP 2.6 发行说明；[此处](/documentation/articles/hdinsight-component-versioning/)提供有关 HDInsight 版本的详细信息。 HDInsight 3.6 仅针对以下工作负荷提供。

    * Hadoop v2.7.3
    * HBase v1.1.2
    * Storm v1.1.0
    * Spark v2.1.0
    * Interactive Hive v2.1.0

* **Hive View 2.0 支持**。 这应该能够改善 Interactive Hive 的用户体验。 有关详细信息，请参阅 [Hortonworks 文档](http://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-views/content/ch_using_hive_view.html)。

* **Hive LLAP 性能增强**。 有关详细信息，请参阅 [Hortonworks 文档](https://hortonworks.com/blog/top-5-performance-boosters-with-apache-hive-llap/)。

* **Hive 中的新增功能**。 有关详细信息，请参阅 [Hortonworks 文档](https://hortonworks.com/apache/hive/#section_4)。

* **Hive CLI 弃用**：已弃用 Hive CLI，建议客户改为使用 Beeline。 有关详细信息，请参阅 [Apache 文档](https://cwiki.apache.org/confluence/display/Hive/Replacing+the+Implementation+of+Hive+CLI+Using+Beeline)。 有关如何将 Beeline 与 HDInsight 配合使用的说明，请参阅[将 Beeline 与 HDInsight Hadoop 群集配合使用](/documentation/articles/hdinsight-hadoop-use-hive-beeline/)。

* **Apache Phoenix 和 HBase 中的新增功能**。
    * 存储配额支持：常用于多租户环境，可按表和按命名空间提供有限的存储空间。
    * Phoenix 索引改进：增量创建索引以及从之前失败的场景中重新生成/恢复索引。
    * Phoenix 数据完整性工具：支持验证架构、索引和其他元数据。

* **HBase 问题**：运行 CSV 批量上传 MapReduce 作业时，以下语法可能会导致错误。

        HADOOP_CLASSPATH=$(hbase mapredcp):/path/to/hbase/conf hadoop jar phoenix-<version>-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table EXAMPLE --input /data/example.csv

    应改为使用以下语法。

        HADOOP_CLASSPATH=/path/to/hbase-protocol.jar:/path/to/hbase/conf hadoop jar phoenix-<version>-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table EXAMPLE --input /data/example.csv

## <a name="02282017---release-of-spark-21-on-hdinsight-36-preview"></a>2017 年 2 月 28 日：发布 Spark 2.1 on HDInsight 3.6（预览版）
* [Spark 2.1](http://spark.apache.org/releases/spark-release-2-1-0.html) 纠正了旧版中的许多稳定性和可用性问题。 它还带来了针对所有 Spark 工作负荷（如 Spark Core、SQL、ML 和流式处理）的新功能。
* 结构化流式处理通过支持事件时间水印和 Kafka 0.10 连接器提高了可伸缩性。
* 现在使用新的可缩放分区处理机制处理 Spark SQL 分区。 有关如何升级，请参阅 [此处](http://spark.apache.org/releases/spark-release-2-1-0.html) 的更多详细信息。
* Azure HDInsight 3.6 预览版中的 Spark 2.1 目前不支持使用 ODBC 驱动程序的 BI 工具连接。

## <a name="11182016---release-of-spark-201-on-hdinsight-35"></a>2016 年 11 月 18 日：发布 Spark 2.0.1 on HDInsight 3.5
Spark 2.0.1 现已在 Spark 群集（HDInsight 版本 3.5）上发行。

## <a name="11162016---release-of-r-server-90-on-hdinsight-35-spark-20"></a>2016 年 11 月 16 日：发布 R Server 9.0 on HDInsight 3.5 (Spark 2.0)
*    R Server 群集现在包括适用于两个版本的选项：R Server 9.0 on HDI 3.5 (Spark 2.0) 和 R Server 8.0 on HDI 3.4 (Spark 1.6)。
*    R Server 9.0 on HDI 3.5 (Spark 2.0) 以 R 3.3.2 为基础构建，包括名为 RxHiveData 和 RxParquetData 的全新 ScaleR 数据源功能，可以将数据从 Hive 和 Parquet 直接加载到 Spark DataFrames 供 ScaleR 分析。 有关详细信息，可以使用 ?RxHiveData 和 ?RxParquetData 命令查看 R 中这些函数的内联帮助。
*    现在，作为预配流的一部分，RStudio Server 社区版（通过选择退出选项）默认安装在“群集预配”边栏选项卡上。

## <a name="11092016---release-of-spark-20-on-hdinsight"></a>2016 年 11 月 9 日：发布 Spark 2.0 on HDInsight
* HDInsight 3.5 上的 Spark 2.0 群集现支持 Livy 和 Jupyter 服务。

## <a name="10262016---release-of-r-server-on-hdinsight"></a>2016 年 10 月 26 日：发布 R Server on HDInsight
* 进行边缘节点访问的 URI 已更改为 **clustername**-ed-ssh.azurehdinsight.cn
* R Server on HDInsight 群集预配已简化。
* R Server on HDInsight 现已作为常用的 HDInsight“R Server”群集类型推出，不再作为单独的 HDInsight 应用程序安装。 边缘节点和 R Server 二进制文件现在会在 R Server 群集部署过程中进行预配。 这将提高预配的速度和可靠性。 R Server 的定价模型会相应地进行更新。

## <a name="08302016---release-of-r-server-on-hdinsight"></a>2016 年 8 月 30 日：发布 R Server on HDInsight
随此版本一起部署的基于 Linux 的 HDInsight 群集的所有版本号包括：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 | Ambari 内部版本 |
| --- | --- | --- | --- | --- |
| 3.2 |3.2.1000.0.8268980 |2.2 |2.2.9.1-19 |2.2.1.12-4 |
| 3.3 |3.3.1000.0.8268980 |2.3 |2.3.3.1-25 |2.2.1.12-4 |
| 3.4 |3.4.1000.0.8269383 |2.4 |2.4.2.4-5 |2.2.1.12-4 |

随此版本一起部署的基于 Windows 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 2.1 |2.1.10.1033.2559206 |1.3 |1.3.12.0-01795 |
| 3.0 |3.0.6.1033.2559206 |2.0 |2.0.13.0-2117 |
| 3.1 |3.1.4.1033.2559206 |2.1 |2.1.16.0-2374 |
| 3.2 |3.2.7.1033.2559206 |2.2 |2.2.9.1-11 |
| 3.3 |3.3.0.1033.2559206 |2.3 |2.3.3.1-25 |

<!--Update_Description: add version 3.6-->