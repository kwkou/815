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