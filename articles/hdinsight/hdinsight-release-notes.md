<properties
    pageTitle="Azure HDInsight 上的 Hadoop 组件发行说明 | Azure"
    description="Azure HDInsight 的 Hadoop 组件的最新发行说明和版本。获取有关 Hadoop、Apache Storm 和 HBase 的开发技巧和详细信息。"
    services="hdinsight"
    documentationcenter=""
    editor="cgronlun"
    manager="jhubbard"
    author="nitinme"
    tags="azure-portal" />
<tags
    ms.assetid="a363e5f6-dd75-476a-87fa-46beb480c1fe"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="2/28/2017"
    wacn.date="03/31/2017"
    ms.author="nitinme" />

# Azure HDInsight 上的 Hadoop 组件发行说明

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 版本控制文章](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

##HDInsight 3.6 预览版中的 Spark 2.1 的 2017 年 2 月 28 日发行说明
* [Spark 2.1](http://spark.apache.org/releases/spark-release-2-1-0.html) 改进了以前版本中存在的许多稳定性和可用性问题。它还带来了针对所有 Spark 工作负荷（如 Spark Core、SQL、ML 和流式处理）的新功能。
* 结构化流式处理通过支持事件时间水印和 Kafka 0.10 连接器提高了可伸缩性。
* 现在使用新的可缩放分区处理机制处理 Spark SQL 分区。有关如何升级，请参阅[此处](http://spark.apache.org/releases/spark-release-2-1-0.html)的更多详细信息。
* Azure HDInsight 3.6 预览版中的 Spark 2.1 目前不支持使用 ODBC 驱动程序的 BI 工具连接。

##Spark 2.0.1 on HDInsight 3.5 2016 年 11 月 18 日发行说明
Spark 2.0.1 现可在 Spark 群集（HDInsight 版本 3.5）上使用。

## R Server 9.0 on HDInsight 3.5 (Spark 2.0) 2016 年 11 月 16 日发行说明
*	R Server 群集现在包括以下两个版本的选项：R Server 9.0 on HDI 3.5 (Spark 2.0) 和 R Server 8.0 on HDI 3.4 (Spark 1.6)。
*	R Server 9.0 on HDI 3.5 (Spark 2.0) 以 R 3.3.2 为基础构建，包括名为 RxHiveData 和 RxParquetData 的全新 ScaleR 数据源功能，可以将数据从 Hive 和 Parquet 直接加载到 Spark DataFrames 供 ScaleR 分析。有关详细信息，可以使用 ?RxHiveData 和 ?RxParquetData 命令查看 R 中这些函数的内联帮助。
*	RStudio Server 社区版现在可作为预配流的一部分在“群集配置”边栏选项卡上默认安装（附带退出选项）。

## Spark 2.0 on HDInsight 2016 年 11 月 9 日发行说明
* HDInsight 3.5 上的 Spark 2.0 群集现在支持 Livy 和 Jupyter 服务。

## R Server on HDInsight 2016 年 10 月 26 日发行说明
* 进行边缘节点访问的 URI 已更改为 **clustername**-ed-ssh.azurehdinsight.cn
* R Server on HDInsight 群集预配已简化。
* R Server on HDInsight 现已作为常用的 HDInsight“R Server”群集类型推出，不再作为单独的 HDInsight 应用程序安装。边缘节点和 R Server 二进制文件现在会在 R Server 群集部署过程中进行预配。这将提高预配的速度和可靠性。R Server 的定价模型会相应地进行更新。

## R Server on HDInsight 2016 年 8 月 30 日发行说明
随此版本一起部署的基于 Linux 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 | Ambari 内部版本 |
| --- | --- | --- | --- | --- |
| 3\.2 |3\.2.1000.0.8268980 |2\.2 |2\.2.9.1-19 |2\.2.1.12-4 |
| 3\.3 |3\.3.1000.0.8268980 |2\.3 |2\.3.3.1-25 |2\.2.1.12-4 |
| 3\.4 |3\.4.1000.0.8269383 |2\.4 |2\.4.2.4-5 |2\.2.1.12-4 |

随此版本一起部署的基于 Windows 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 2\.1 |2\.1.10.1033.2559206 |1\.3 |1\.3.12.0-01795 |
| 3\.0 |3\.0.6.1033.2559206 |2\.0 |2\.0.13.0-2117 |
| 3\.1 |3\.1.4.1033.2559206 |2\.1 |2\.1.16.0-2374 |
| 3\.2 |3\.2.7.1033.2559206 |2\.2 |2\.2.9.1-11 |
| 3\.3 |3\.3.0.1033.2559206 |2\.3 |2\.3.3.1-25 |

## R Server on HDInsight 2016 年 8 月 17 日发行说明
* R Server 8.0.5 - 此版本主要包含 Bug 修复。有关详细信息，请参阅 [R Server Release Notes](https://msdn.microsoft.com/microsoft-r/notes/r-server-notes)（R Server 发行说明）。
* 边缘节点上的 AzureML 包 - 可以通过[此 R 包](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html)以 Azure ML Web 服务形式发布和使用 R 模型。
* [最常用的 100 个 R 包](https://github.com/metacran/cranlogs)的 Linux 依赖项 - 现已预装这些 Linux 包依赖项。
* 将 R 包添加到数据节点时的 CRAN 存储库使用选项。
* 改进在创建群集时进行 R Server 预配的可靠性。

## HDInsight 2016 年 8 月 1 日发行说明
随此版本一起部署的基于 Linux 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 | Ambari 内部版本 |
| --- | --- | --- | --- | --- |
| 3\.2 |3\.2.1000.0.8028416 |2\.2 |2\.2.9.1-19 |2\.2.1.12-4 |
| 3\.3 |3\.3.1000.0.8028416 |2\.3 |2\.3.3.1-25 |2\.2.1.12-4 |
| 3\.4 |3\.4.1000.0.8053402 |2\.4 |2\.4.2.4-5 |2\.2.1.12-4 |

随此版本一起部署的基于 Windows 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 2\.1 |2\.1.10.1005.2488842 |1\.3 |1\.3.12.0-01795 |
| 3\.0 |3\.0.6.1005.2488842 |2\.0 |2\.0.13.0-2117 |
| 3\.1 |3\.1.4.1005.2488842 |2\.1 |2\.1.16.0-2374 |
| 3\.2 |3\.2.7.1005.2488842 |2\.2 |2\.2.9.1-11 |
| 3\.3 |3\.3.0.1005.2488842 |2\.3 |2\.3.3.1-25 |

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Spark、Hadoop、HBase 或 Storm） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 对 HDInsight 3.4 群集的更改 |更改了以下 hive 配置的默认值以获得更佳性能：<ul><li>`hive.vectorized.execution.reduce.enabled=true`</li><li>`hive.tez.min.partition.factor=1f`</li><li>`hive.tez.max.partition.factor=3f`</li><li>`tez.shuffle-vertex-manager.min-src-fraction=0.9`</li><li>`tez.shuffle-vertex-manager.max-src-fraction=0.95`</li><li>`tez.runtime.shuffle.connect.timeout= 30000`</li></ul> |服务 |全部 |不适用 |
| 此版本中包括下列修补程序 |HIVE-13632、HIVE-12897、HIVE-12907、HIVE-12908、HIVE-12988、HIVE-13510、HIVE-13572、HIVE-13716、HIVE-13726、HIVE-12505、HIVE-13632、HIVE-13661、HIVE-13705、HIVE-13743、HIVE-13810、HIVE-13857、HIVE-13902、HIVE-13911、HIVE-13933 |服务 |全部 |不适用 |

## HDInsight 2016 年 7 月 14 日发行说明
随此版本一起部署的基于 Linux 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 | Ambari 内部版本 |
| --- | --- | --- | --- | --- |
| 3\.2 |3\.2.1000.0.7932505 |2\.2 |2\.2.9.1-11 |2\.2.1.12-2 |
| 3\.3 |3\.3.1000.0.7932505 |2\.3 |2\.3.3.1-18 |2\.2.1.12-2 |
| 3\.4 |3\.4.1000.0.7933003 |2\.4 |2\.4.2.0 |2\.2.1.12-2 |

随此版本一起部署的基于 Windows 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 2\.1 |2\.1.10.989.2441725 |1\.3 |1\.3.12.0-01795 |
| 3\.0 |3\.0.6.989.2441725 |2\.0 |2\.0.13.0-2117 |
| 3\.1 |3\.1.4.989.2441725 |2\.1 |2\.1.16.0-2374 |
| 3\.2 |3\.2.7.989.2441725 |2\.2 |2\.2.9.1-11 |
| 3\.3 |3\.3.0.989.2441725 |2\.3 |2\.3.3.1-21 |

## HDInsight 2016 年 7 月 7 日发行说明
随此版本一起部署的基于 Linux 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 3\.2 |3\.2.1000.0.7864996 |2\.2 |2\.2.9.1-11 |
| 3\.3 |3\.3.1000.0.7864996 |2\.3 |2\.3.3.1-18 |
| 3\.4 |3\.4.1000.0.7861906 |2\.4 |2\.4.2.0 |

随此版本一起部署的基于 Windows 的 HDInsight 群集的所有版本号：

| HDI | HDI 群集版本 | HDP | HDP 内部版本 |
| --- | --- | --- | --- |
| 2\.1 |2\.1.10.977.2413853 |1\.3 |1\.3.12.0-01795 |
| 3\.0 |3\.0.6.977.2413853 |2\.0 |2\.0.13.0-2117 |
| 3\.1 |3\.1.4.977.2413853 |2\.1 |2\.1.16.0-2374 |
| 3\.2 |3\.2.7.977.2413853 |2\.2 |2\.2.9.1-11 |
| 3\.3 |3\.3.0.977.2413853 |2\.3 |2\.3.3.1-21 |

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Spark、Hadoop、HBase 或 Storm） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 用于 IntelliJ 的 HDInsight 工具 |用于 HDInsight Spark 群集的 IntelliJ IDEA 插件现在集成了用于 IntelliJ 的 Azure 工具包。它支持 Azure SDK v2.9.1（最新 Java SDK），包括用于 IntelliJ 的独立 HDInsight 插件提供的所有功能。 |工具 |Spark |不适用 |
| 用于 Eclipse 的 HDInsight 工具 |用于 Eclipse 的 Azure 工具包现在支持 HDInsight Spark 群集。它启用以下功能。<ul><li>在 Scala 和 Java 中轻松创建和编写 Spark 应用程序，提供对 IntelliSense、自动格式、错误检查等功能的一流创作支持。</li><li>在本地测试 Spark 应用程序。</li><li>将作业提交到 HDInsight Spark 群集并检索结果。</li><li>登录 Azure 并访问与 Azure 订阅关联的所有 Spark 群集。</li><li>导航 HDInsight Spark 群集的所有关联的存储资源。</li></ul> |工具 |Spark |不适用 |

从此版本开始，我们更改了基于 Linux 的 HDInsight 群集的来宾 OS 修补策略。新策略的目标是显著减少因修补而导致的重启次数。新策略将继续修补 Linux 群集上的虚拟机 (VM)，开始时间为每个星期一或星期四的凌晨 0 点 (UTC)，采用在任何给定群集中跨节点进行交错修补的方式。但是，任何给定 VM 最多只会因来宾 OS 修补而 30 天重启一次。此外，对于新创建的群集，第一次重启的时间不会早于群集创建日期之后的 30 天。

> [AZURE.NOTE]
这些更改只适用于其版本号不低于此发行版本的新建群集。
>
>

## HDInsight 2016 年 6 月 6 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

| HDP | HDI 版本 | Spark 版本 | Ambari 内部版本号 | HDP 内部版本号 |
| --- | --- | --- | --- | --- |
| 2\.3 |3\.3.1000.0.7702215 |1\.5.2 |2\.2.1.8-2 |2\.3.3.1-18 |
| 2\.4 |3\.4.1000.0.7702224 |1\.6.1 |2\.2.1.8-2 |2\.4.2.0 |

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Spark、Hadoop、HBase 或 Storm） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| Spark on HDInsight 已正式发布 |此版本改进了开源 Apache Spark on HDInsight 的可用性、可伸缩性和工作效率。<ul><li>行业领先的可用性 SLA (99.9%)，使之适用于要求严苛的企业工作负荷。</li><li>生产力工具，适用于每个阶段的数据浏览和开发。使用自定义 Spark 内核的 Jupyter 笔记本允许进行交互式数据浏览；与 BI 仪表板（例如 Power BI、Tableau、Qlik）集成后，即可快速进行数据共享和持续进行报告；IntelliJ 插件是进行长期代码项目开发和调试的可靠选项。</li></ul> |服务 |Spark |不适用 |
| 用于 IntelliJ 的 HDInsight 工具 |这是用于 HDInsight Spark 群集的 IntelliJ IDEA 插件。它启用以下功能。<ul><li>在 Scala 和 Java 中轻松创建和编写 Spark 应用程序，提供对 IntelliSense、自动格式、错误检查等功能的一流创作支持。</li><li>在本地测试 Spark 应用程序。</li><li>将作业提交到 HDInsight Spark 群集并检索结果。</li><li>登录 Azure 并访问与 Azure 订阅关联的所有 Spark 群集。</li><li>导航 HDInsight Spark 群集的所有关联的存储资源。</li><li>导航 HDInsight Spark 群集的所有作业历史记录和作业信息。</li><li>通过台式计算机远程调试 Spark 作业。</li></ul> |工具 |Spark |不适用 |

## HDInsight 2016 年 5 月 13 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.875.2159884（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.875.2159884（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.922.2266903（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.922.2266903 (HDP 2.2.9.1-11)
* HDInsight (Windows) 3.3.0.922.2266903 (HDP 2.3.3.1-18)
* HDInsight (Linux) 3.2.1000.0.7565644 (HDP 2.2.9.1-11)
* HDInsight (Linux) 3.3.1000.0.7565644 (HDP 2.3.3.1-18)
* HDInsight (Linux) 3.4.1000.0.7548380 (HDP 2.4.2.0)

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Spark、Hadoop、HBase 或 Storm） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| Spark 版本更新和其他 Bug 修复 |此版本将 HDInsight 群集中的 Spark 版本更新到 1.6.1，并修复其他 Bug |服务 |Spark |不适用 |

## HDInsight 2016 年 4 月 11 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.889.2191206（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.889.2191206（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.889.2191206（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.889.2191206 (HDP 2.2.9.1-10)
* HDInsight (Windows) 3.3.0.889.2191206（HDP 2.3.3.1-16 -保持不变）
* HDInsight (Linux) 3.2.1000.0.7339916 (HDP 2.2.9.1-10)
* HDInsight (Linux) 3.3.1000.0.7339916 (HDP 2.3.3.1-16)
* HDInsight (Linux) 3.4.1000.0.7338911 (HDP 2.4.1.1-3)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 自定义 HDI 3.4 的元存储升级问题 |如果使用的是此前已在另一低版 HDInsight 群集上使用过的自定义元存储，则无法创建群集。这是由于升级脚本错误（现已修复） |群集创建 |全部 |不适用 |
| Livy 崩溃恢复 |为通过 Livy 提交的任何作业提供作业状态复原 |可靠性 |Linux 上的 Spark |不适用 |
| Jupyter 内容 HA |提供通过与群集关联的存储帐户来回保存和加载 Jupyter 笔记本内容的功能。有关详细信息，请参阅[可供 Jupyter 笔记本使用的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)。 |笔记本 |Linux 上的 Spark |不适用 |
| 删除 Jupter 笔记本中的 hiveContext |使用 `%%sql` 幻数而非 `%%hive` 幻数。SqlContext 相当于 hiveContext。有关详细信息，请参阅[可供 Jupyter 笔记本使用的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/) |笔记本 |Linux 上的 Spark 群集 |不适用 |
| 不推荐使用较早的 Spark 版本 |较早的 Spark 版本 1.3.1 将于 5 月 31 日从服务中删除 |服务 |Windows 上的 Spark 群集 |不适用 |

## HDInsight 2016 年 3 月 29 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.875.2159884（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.875.2159884（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.875.2159884（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.875.2159884（HDP 2.2.9.1-7 - 保持不变）
* HDInsight (Windows) 3.3.0.875.2159884 (HDP 2.3.3.1-16)
* HDInsight (Linux) 3.2.1000.0.7193255（HDP 2.2.9.1-8 - 保持不变）
* HDInsight (Linux) 3.3.1000.0.7193255（HDP 2.3.3.1-7 - 保持不变）
* HDInsight (Linux) 3.4.1000.0.7195842 (HDP 2.4.1.0-327)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 添加了 HDInsight 3.4 版本并更新了所有 HDInsight 群集的 HDP 版本 |在此版本中，我们添加了 HDInsight v3.4（基于 HDP 2.4）并更新了其他 HDP 版本。[此处](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_HDP_RelNotes/content/ch_relnotes_v240.html)提供了 HDP 2.4 发行说明；[此处](/documentation/articles/hdinsight-component-versioning/)提供了有关 HDInsight 版本的详细信息。 |服务 |所有 Linux 群集 |不适用 |
| Spark 1.6.0 |HDInsight 3.4 群集现在包括 Spark 1.6.0 |服务 |Linux 上的 Spark 群集 |不适用 |
| Jupyter 笔记本增强功能 |适用于 Spark 群集的 Jupyter 笔记本现在提供更多的 Spark 内核。该笔记本还包括多种增强功能，例如使用 %%幻数、自动可视化，以及集成 Python 可视化库（例如 matplotlib）。有关详细信息，请参阅[可供 Jupyter 笔记本使用的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)。 |服务 |Linux 上的 Spark 群集 |不适用 |

## HDInsight 2016 年 3 月 22 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.875.2159884（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.875.2159884（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.875.2159884（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.875.2159884（HDP 2.2.9.1-7 - 保持不变）
* HDInsight (Windows) 3.3.0.875.2159884 (HDP 2.3.3.1-16)
* HDInsight (Linux) 3.2.1000.0.7193255（HDP 2.2.9.1-8 - 保持不变）
* HDInsight (Linux) 3.3.1000.0.7193255（HDP 2.3.3.1-7 - 保持不变）
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新所有 HDInsight 群集的 HDInsight 版本 |此发行版已更新所有 HDInsight 群集的 HDInsight 版本 |服务 |全部 |不适用 |

## HDInsight 2016 年3 月 10 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.859.2123216（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.859.2123216（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.859.2123216（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.859.2123216 (HDP 2.2.9.1-7)
* HDInsight (Windows) 3.3.0.859.2123216（HDP 2.3.3.1-5 - 保持不变）
* HDInsight (Linux) 3.2.1000.7076817 (HDP 2.2.9.1-8)
* HDInsight (Linux) 3.3.1000.7076817 (HDP 2.3.3.1-7)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新所有 HDInsight 群集的 HDInsight 版本 |此发行版已更新所有 HDInsight 群集的 HDInsight 版本 |服务 |全部 |不适用 |

## HDInsight 2016 年 1 月 27 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.817.2028315（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.817.2028315（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.817.2028315（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.817.2028315 (HDP 2.2.9.1-1)
* HDInsight (Windows) 3.3.0.817.2028315（HDP 2.3.3.1-5 - 保持不变）
* HDInsight (Linux) 3.2.1000.4072335 (HDP 2.2.9.1-1)
* HDInsight (Linux) 3.3.1000.4072335 (HDP 2.3.3.1-1)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新所有 HDInsight 群集的 HDInsight 版本 |此发行版已更新所有 HDInsight 群集的 HDInsight 版本 |服务 |全部 |不适用 |

## HDInsight 2015 年 12 月 2 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.763.1931434（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.763.1931434（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.763.1931434（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.763.1931434（HDP 2.2.7.1-34 - 保持不变）
* HDInsight (Windows) 3.3.1000.0 (HDP 2.3.3.1-5)
* HDInsight (Linux) 3.2.1000.0.6392801（HDP 2.2.7.1-34 - 保持不变）
* HDInsight (Linux) 3.3.1000.0 (HDP 2.3.3.0-3039)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 添加了 HDInsight 3.3 版本并更新了所有 HDInsight 群集的 HDP 版本 |在此版本中，我们添加了 HDInsight v3.3（基于 HDP 2.3）并更新了其他 HDP 版本。[此处](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_HDP_RelNotes/content/ch_relnotes_v230.html)提供了 HDP 2.3 发行说明；[此处](/documentation/articles/hdinsight-component-versioning/)提供了有关 HDInsight 版本的详细信息。 |服务 |全部 |不适用 |

## HDInsight 2015 年 11 月 30 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.757.1923908（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.757.1923908（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.757.1923908（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.757.1923908 (HDP 2.2.7.1-34)
* HDInsight (Linux) 3.2.1000.0.6392801 (HDP 2.2.7.1-34)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新了所有 HDInsight 群集的 HDInsight 版本，以及 HDInsight 3.2 群集的 HDP 版本（Windows 和 Linux） |在此版本中，HDInsight 和 HDP 版本已更新 |服务 |全部 |不适用 |

## HDInsight 2015 年 10 月 27 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight (Windows) 2.1.10.726.1866228（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight (Windows) 3.0.6.726.1866228（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight (Windows) 3.1.4.726.1866228（HDP 2.1.15.0-2374 - 保持不变）
* HDInsight (Windows) 3.2.7.726.1866228 (HDP 2.2.7.1-33)
* HDInsight (Linux) 3.2.1000.0.6035701 (HDP 2.2.7.1-33)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新了所有 HDInsight 群集的 HDInsight 版本（Windows 和 Linux） |在此版本中，HDInsight 和 HDP 版本已更新 |服务 |全部 |不适用 |
| 修复了包含大写字母群集的 Jupyter for Windows Spark 群集的问题 |以大写字母指定 DNS 名称的群集在使用 Jupyter 笔记本时会因为源请求检查而出现问题。解决方法是将 Jupyter 的配置的 DNS 名称更改为小写。 |服务 |HDInsight Spark (Windows) |不适用 |

## HDInsight 2015 年 10 月 20 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.716.1846990 (Windows)（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.716.1846990 (Windows)（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.716.1846990 (Windows) (HDP 2.1.16.0-2374)
* HDInsight 3.2.7.716.1846990 (Windows) (HDP 2.2.7.1-0004)
* HDInsight 3.2.1000.0.5930166 (Linux) (HDP 2.2.7.1-0004)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 默认 HDP 版本已更改为 HDP 2.2 |HDInsight Windows 群集的默认版本已更改为 HDP 2.2。2015 年 2 月已正式推出 HDInsight 版本 3.2 (HDP 2.2)。此项更改仅在使用 Azure 门户预览、PowerShell cmdlet 或 SDK 预配群集时未进行明确选择的情况下，才切换默认群集版本。 |服务 |全部 |不适用 |
| 在将多个 HDInsight on Linux 群集部署到单一虚拟网络时更改 VM 名称格式 |此版本增加了支持将多个 HDInsight Linux 群集部署在单一虚拟网络中的功能。其中，群集虚拟机名称的格式已分别从 headnode*、workernode* 和 zookeepernode* 更改为 hn*、wn* 和 zk*。不建议使用虚拟机名称格式的直接依赖项，因为该格式会变化。请使用“hostname -f”（在本地计算机上）或 Ambari API 来确定主机的列表，以及从组件到主机的映射。有关详细信息，请访问 [https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/hosts.md](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/hosts.md) 和 [https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/host-components.md](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/host-components.md)。 |服务 |Linux 上的 HDInsight 群集 |不适用 |
| 配置更改 |对于 HDInsight 3.1 群集，现在会启用以下配置：<ul><li>tez.yarn.ats.enabled 和 yarn.log.server.url。这使得 Application Timeline Server 和日志服务器能够提供日志。</li></ul>对于 HDInsight 3.2 群集，已修改以下配置：<ul><li>mapreduce.fileoutputcommitter.algorithm.version 已设置为 2。这样就可以使用 FileOutputCommitter 的 V2 版本。</li></ul> |服务 |全部 |不适用 |

## HDInsight 2015 年 9 月 9 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.675.1768697（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.675.1768697（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.675.1768697（HDP 2.1.15.0-2334 - 保持不变）
* HDInsight 3.2.6.675.1768697（HDP 2.2.6.1-0012 - 保持不变）
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新所有 HDInsight 群集的 HDInsight 版本 |在此版本中，HDInsight 版本已更新 |服务 |全部 |不适用 |

## HDInsight 2015 年 7 月 31 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.640.1695824（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.640.1695824（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.640.1695824（HDP 2.1.15.0-2334 - 保持不变）
* HDInsight 3.2.6.640.1695824（HDP 2.2.6.1-0012 - 保持不变）
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 修复 Spark 群集节点的重新制作映像工作流 |修复造成 Spark 群集节点重新制作映像后不恢复的错误 |服务 |Spark |不适用 |

## HDInsight 2015 年 7 月 31 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.635.1684502（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.635.1684502（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.635.1684502（HDP 2.1.15.0-2334 - 保持不变）
* HDInsight 3.2.6.635.1684502（HDP 2.2.6.1-0012 - 保持不变）
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| 更新所有 HDInsight 群集的 HDInsight 版本 |在此版本中，HDInsight 版本已更新 |服务 |全部 |不适用 |

## HDInsight 2015 年 7 月 7 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.610.1630216（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.610.1630216（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.610.1630216（HDP 2.1.15.0-2334 - 保持不变）
* HDInsight 3.2.4.610.1630216 (HDP 2.2.6.1-0012)
* SDK 1.5.8

此版本包含以下更新。

| 标题 | 说明 | 受影响区域（例如服务、组件或 SDK） | 群集类型（例如 Hadoop、HBase 或 STORM） | JIRA（如果适用） |
| --- | --- | --- | --- | --- |
| HDInsight 3.2 群集的更新后 HDP 版本 |在此版本中，HDInsight 3.2 部署 HDP 2.2.6.1-0012 |服务 |全部 |不适用 |

## HDInsight 2015 年 6 月 26 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.601.1610731（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.601.1610731（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.601.1610731（HDP 2.1.15.0-2334 - 保持不变）
* HDInsight 3.2.4.601.1610731 (HDP 2.2.6.1-0011)
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>HDInsight 3.2 群集的更新后 HDP 版本</td>
<td>在此版本中，HDInsight 3.2 部署 HDP 2.2.6.1</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 6 月 18 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.596.1601657（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.596.1601657（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.4.596.1601657 (HDP 2.1.15.0-2334)
* HDInsight 3.2.4.596.1601657 (HDP 2.2.6.1-0002)
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>其他打开的 HTTPS 端口</td>
<td>云服务现在会打开群集上的 5 个端口（8001 到 8005），例如 https://\&lt;clustername>.azurehdinsight.cn:8001/。对这些 URL 的请求使用和端口 443 相同的基本身份验证密码机制进行验证。这些端口绑定到活动头节点上的相同端口。使用脚本操作可让客户服务在头节点的这些端口上侦听并路由到群集外部。</td>
<td>云服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>HDInsight 3.2 的间歇性 MapReduce 随机排布问题</td>
<td>修复大型群集上间歇性的 MapReduce 随机排布，导致非经常性任务失败的罕见情况。有关详细信息，请参阅 <a href="https://issues.apache.org/jira/browse/MAPREDUCE-6361" target="_blank">MAPREDUCE-6361</a>。</td>
<td>Hadoop 核心</td>
<td>全部</td>
<td><a href="https://issues.apache.org/jira/browse/MAPREDUCE-6361" target="_blank">MAPREDUCE-6361</a></td>
</tr>
<tr>
<td>移到最新的 Azure Java SDK 2.2 for HDInsight 3.2</td>
<td>已移到 WASB 驱动程序所用的最新版 Azure SDK for Java。最新的 SDK 有一些修复程序，https://github.com/Azure/azure-storage-java/blob/master/ChangeLog.txt 上提供了相同的发行说明。</td>
<td>Hadoop 核心</td>
<td>全部</td>
<td><a href="https://issues.apache.org/jira/browse/HADOOP-11959" target="_blank">HADOOP-11959</a></td>
</tr>
<tr>
<td>移到 HDP 2.1.15 for HDInsight 3.1 群集</td>
<td>这些版本的 Hortonworks 版本信息位于<a href="http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.15-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.15.html" target="_blank">此处</a>。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 6 月 4 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.583.1575584（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.583.1575584（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.583.1575584（HDP 2.1.12.1-0003 - 保持不变）
* HDInsight 3.2.4.583.1575584 (HDP 2.2.6.1-1)
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>修复 Storm 群集的 502 网关不正确错误</td>
<td>此版本修复了影响作业提交 API 并导致网站在重新启动后关闭的 Bug。</td>
<td>服务</td>
<td>Storm</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 6 月 1 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.577.1563827（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.577.1563827（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.577.1563827（HDP 2.1.12.1-0003 - 保持不变）
* HDInsight 3.2.4.577.1563827（HDP 2.2.6.0-2800 - 保持不变）
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>各种 Bug 修复</td>
<td>此版本修复了与群集预配相关的 Bug。</td>
<td>服务</td>
<td>所有群集类型</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 5 月 27 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 3.2.4.570.1554102 (HDP 2.2.6.0-2800)
* 其他群集版本和 SDK 不部署为此版本的一部分。

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>HDP 2.2 更新</td>
<td>此版本的 HDInsight 3.2 包含 HDP 2.2.6，并在 HDInsight 中添加了多个重要的 Bug 修复程序。在 <a href="http://dev.hortonworks.com.s3.amazonaws.com/HDPDocuments/HDP2/HDP-2.2.6/HDP_RelNotes_v226/index.html">HDP 2.2.6 发行说明</a>中可获取完整的发行说明。</td>
<td>HDP</td>
<td>所有群集类型</td>
<td>不适用</td>
</tr>
<tr>
<td>更改为默认 Yarn 容器内存配置</td>
<td>在此更新中，由节点管理器启动的 YARN 容器 (yarn.nodemanager.resource.memory-mb and yarn.scheduler.maximum-allocation-mb) 的默认可用内存已增加到 5632MB。以前此内存量缩减为 4608MB，但根据不同的作业运行，新值必须为大多数作业提供更高的可靠性与性能，因此此默认值更佳。照例，如果严重依赖于此内存配置，请在创建群集时显式设置内存。</td>
<td>HDP</td>
<td>所有群集类型</td>
<td>不适用</td>
</tr>
<tr>
<td>HBase 和 Storm 群集的默认配置奇偶校验</td>
<td>此更新将 Hbase 和 Storm 群集还原为使用与 Hadoop 群集相同的 YARN 配置值。这是为了对所有群集类型进行奇偶校验。</td>
<td>HDP</td>
<td>Hbase、Storm</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 5 月 20 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.564.1542093（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.564.1542093（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.564.1542093 (HDP 2.1.12.1-0003)
* HDInsight 3.2.4.564.1542093 (HDP 2.2.4.6-2)
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>SCP.NET EventHub 支持</td>
<td>HDInsight Storm 的更新群集包添加了 SCP.NET 的新功能。现在可以访问拓扑生成器中的新 API，并更轻松地使用 EventHubSpout 或 Java Spouts。必须更新 SCP.NET 客户端 SDK 才能在合约更新后使用新群集。有关新 API、用法和发行说明（包括 Bug 修复程序）的详细信息，请参阅 SCP.NET Nuget 包中的 Readme 文件。</td>
<td>VS 工具</td>
<td>Storm HDInsight 3.2 群集</td>
<td>不适用</td>
</tr>
<tr>
<td>JDBC 驱动程序更新</td>
<td>将驱动程序更新为 sqljdbc_4.1.5605.100 中支持的 SQL Server 版本。</td>
<td>元存储</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 4 月 27 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.537.1486660（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.537.1486660（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.537.1486660（HDP 2.1.12.0-2329 - 保持不变）
* HDInsight 3.2.3.537.1486660 (HDP 2.2.2.2-4)
* SDK 1.5.8

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>修复了 DLL 依赖性</td>
<td>删除了对单元测试框架的 HDInsight 依赖性。	</td>
<td>SDK</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>争用状态 Bug 修复</td>
<td>群集创建请求现在会等待 PUT 请求获得接受，然后才进行状态轮询</td>
<td>SDK</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 4 月 14 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.521.1453250（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.521.1453250（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.521.1453250（HDP 2.1.12.0-2329 - 保持不变）
* HDInsight 3.2.3.525.1459730 (HDP 2.2.2.2-2)
* SDK 1.5.6

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>Tez Bug 修复</td>
<td>此版本的 HDI 3.2 包含 Apache TEZ 2214 和 TEZ 1923 的修复程序。对于需要随机排列大量数据、对 Tez 执行的某些 Hive 查询而言，特别需要这些修复程序。
</td>
<td>HDP</td>
<td>Hadoop</td>
<td><a href="https://issues.apache.org/jira/browse/TEZ-2214">TEZ 2214</a></br><a href="https://issues.apache.org/jira/browse/TEZ-1923">TEZ 1923</a></td>
</tr>
</table>

## HDInsight 2015 年 4 月 6 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.521.1453250（HDP 1.3.12.0-01795 - 保持不变）
* HDInsight 3.0.6.521.1453250（HDP 2.0.13.0-2117 - 保持不变）
* HDInsight 3.1.3.521.1453250（HDP 2.1.12.0-2329 - 保持不变）
* HDInsight 3.2.3.521.1453250 (HDP 2.2.2.2-1)
* SDK 1.5.6

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>HDInsight .NET SDK 1.5.6</td>
<td>已做更新，删除了 HDInsight on Linux 的某些内部类。</td>
<td>SDK</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>Avro Library 1.5.6</td>
<td>添加了方法 <b>GetAllKnownTypes</b> 的 <b>KnownTypeAttribute</b>。修复了 GetAllKnownTypes 方法的类型为 null 时引发的 NullReferenceException</td>
<td>SDK</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>Bug 修复</td>
<td>修复了服务的多个 bug</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 4 月 1 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.513.1431705 (HDP 1.3.12.0-01795)
* HDInsight 3.0.6.513.1431705 (HDP 2.0.13.0-2117)
* HDInsight 3.1.3.513.1431705 (HDP 2.1.12.0-2329)
* HDInsight 3.2.3.513.1431705 (HDP 2.2.2.1-2600)
* SDK 1.5.5

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>可以通过 .NET SDK 在 Windows 群集上启用/禁用远程桌面凭据</td>
<td>支持以编程方式在 Windows 群集上启用或禁用 RDP 凭据。</td>
<td>SDK</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>可以在设置群集时在群集上启用远程桌面凭据</td>
<td>支持在创建群集时以编程方式启用远程桌面凭据。这样就消除了先预配群集，然后启用远程桌面的双步过程。</td>
<td>SDK</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>已将 Python 升级到 2.7.8</td>
<td>已将 HDInsight 群集上的 Python 升级到 Python 2.7.8，其中包含 HDInsight 版本 2.1、3.0、3.1 和 3.2 的一些重要安全修复程序</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>YARN 配置更改</td>
<td>已经为 HDInsight 版本 3.1 和 3.2 的所有群集类型将 YARN 配置 yarn.resourcemanager.max-completed-applications 更改为 1000。此值仅控制 YARN UI 中已完成应用程序的列表。要获取有关在应用程序用户界面上显示的列表之前已提交的应用程序的信息，可以直接转到历史记录服务器。</td>
<td>YARN</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>调整 HBase 群集中节点的大小</td>
<td>现在，HBase 群集允许调整 HDInsight 版本 3.1 和 3.2 节点的大小（向上和向下）</td>
<td>服务</td>
<td>HBase</td>
<td>不适用</td>
</tr>
<tr>
<td>JDBC 升级</td>
<td>对于 HDInsight 版本 3.2，SQL JDBC 驱动程序升级到版本 sqljdbc_4.0.2206.100。此版本包含重要的安全增强功能。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>JVM 配置更新</td>
<td>对于 HDInsight 版本 3.1 和 3.2，已将 JVM 配置 networkaddress.cache.ttl 更新为 300 秒，以前的默认值为 -1。此配置值控制通过名称服务的成功名称查找的缓存策略。这修复了增长和收缩 HBase 群集相关的 Bug。</td>
<td>服务</td>
<td>HBase</td>
<td>不适用</td>
</tr>
<tr>
<td>升级到 Microsoft Azure Storage SDK for Java 2.0</td>
<td>HDInsight 版本 3.2 已升级为使用最新版本的 Microsoft Azure Storage SDK for Java。与当前 0.6.0 版本相比，该版本包含几个重要的 bug 修复程序。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>已升级到 Apache WASB 源代码</td>
<td>HDInsight 版本 3.2 现在使用最新的代码从 Apache Hadoop WASB 文件系统驱动程序。做出此更改后，WASB 驱动程序现在打包为一个单独的 jar。这是纯粹的打包更改，不包含对 WASB 驱动程序行为的任何更改。此 JAR 文件的名称是 hadoop-azure-2.6.0.jar。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>HDInsight 3.2 中的 Jar 文件名更新</td>
<td>HDInsight 版本 3.2 的这一更新包含几个 Bug 修补程序，并且作为 HDP 一部分打包的几个内部 jar 已升级。请注意，这些 JAR 文件是内部 HDP 包，而不是由客户应用程序直接使用。应用程序应打包自己的 Jar 版本，以便升级到 HDP 内部 Jar 不会中断客户应用程序。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 3 月 3 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.488.1375841（HDP 1.3.9.0-01351 - 保持不变）
* HDInsight 3.0.6.488.1375841（HDP 2.0.9.0-2097 - 保持不变）
* HDInsight 3.1.3.488.1375841（HDP 2.1.10.0-2290 - 保持不变）
* HDInsight 3.2.3.488.1375841（HDP-2.2.10.0-2340 - 保持不变）
* SDK 1.5.0（保持不变）

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA</th>
</tr>
<tr>
<td>可靠性改进</td>
<td>我们创建了修补程序，可让服务根据相对于群集创建增加的负载更好地扩展。</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 2 月 18 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.471.1342507（HDP 1.3.9.0-01351 - 保持不变）
* HDInsight 3.0.6.471.1342507（HDP 2.0.9.0-2097 - 保持不变）
* HDInsight 3.1.3.471.1342507（HDP 2.1.10.0-2290 - 保持不变）
* HDInsight 3.2.3.471.1342507 (HDP-2.2.10.0-2340)
* SDK 1.5.0

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>HDInsight 3.2 群集</td>
<td>Hadoop 2.6/HDP2.2，同时提供 HDInsight 3.2 群集。它包含对所有的开放源组件的重大更新。有关详细信息，请参阅“HDInsight 中的新增功能”和 <a href ="http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/bk_HDP_RelNotes/content/ch_relnotes_v220.html" target="_blank">HDP 2.2.0.0 发行说明</a>。</td>
<td>开源软件</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>Linux 上的 HDInsight（预览版）</td>
<td>可以在 Ubuntu Linux 上部署和运行群集。有关详细信息，请参阅“HDInsight on Linux 入门”。</td>
<td>服务</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>Storm 正式发布</td>
<td>Apache Storm 已推出正式版。有关详细信息，请参阅“HDInsight 中的 Storm 入门”。</td>
<td>服务</td>
<td>Storm</td>
<td>不适用</td>
</tr>
<tr>
<td>虚拟机大小</td>
<td>Azure HDInsight 支持更多的虚拟机类型和大小。HDInsight 可以利用 A2 到 A7 大小实现常规目的；搭载固态硬盘 (SSD) 和处理器速度提高 60% 的 D 系列节点；支持使用 InfiniBand 加快联网速度的 A8 和 A9 大小。</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>群集缩放</td>
<td>可以更改运行的 HDInsight 群集的数据节点数，而无需删除或重新创建它。目前，只有 Hadoop 查询和 Apache Storm 群集类型具有此功能，但 Apache HBase 群集类型即将提供此支持。有关详细信息，请参阅“管理 HDInsight 群集”。</td>
<td>服务</td>
<td>Hadoop、Storm</td>
<td>不适用</td>
</tr>
<tr>
<td>Visual Studio 工具</td>
<td>除了 Apache Storm 的完整工具，用于 Visual Studio 中 Apache Hive 的工具已更新，从而包括了对于语句完成、本地验证和改进的调试的支持。有关详细信息，请参阅 HDInsight Hadoop Tools for Visual Studio 入门。</td>
<td>工具</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<td>Hadoop Connector for DocumentDB</td>
<td>使用 Hadoop Connector for DocumentDB，你可以通过无架构 JSON 文档存储在 DocumentDB 集合之间或跨数据库帐户执行复杂的聚合、分析和操作。有关详细信息和教程，请参阅使用 DocumentDB 和 HDInsight 运行 Hadoop 作业。</td>
<td>服务</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>Bug 修复</td>
<td>我们为 HDInsight 服务创建了各种次要 Bug 修补程序。不需要进行任何面向客户的行为更改。</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 2 月 6 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.463.1325367（HDP 1.3.9.0-01351 - 保持不变）
* HDInsight 3.0.6.463.1325367（HDP 2.0.9.0-2097 - 保持不变）
* HDInsight 3.1.2.463.1325367 (HDP 2.1.10.0-2290)
* SDK 不适用

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>Bug 修复</td>
<td>我们为 HDInsight 服务创建了各种次要 Bug 修补程序。不需要进行任何面向客户的行为更改。</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>HDP 2.1 维护更新</td>
<td>HDInsight 3.1 更新来部署 HDP 2.1.10.0。有关详细信息，请参阅<a href ="http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.10/bk_releasenotes_hdp_2.1/content/ch_relnotes-HDP-2.1.10.html" target="_blank">发行说明 HDP-2.1.10</a>。</td>
<td>开源软件</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>HDP 二进制更新</td>
<td>在 HBase 中有几个 JAR 文件的文件名已更新。这些 JAR 文件由 HBase 在内部使用，因此不期望客户对这些 JAR 文件的名称有依赖关系。其中包括：
<ul>
<li>./lib/jetty-6.1.26.hwx.jar</li>
<li>./lib/jetty-sslengine-6.1.26.hwx.jar</li>
<li>./lib/jetty-util-6.1.26.hwx.jar</li>
</ul>
</td>
<td>开源软件</td>
<td>HBase</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 1 月 9 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.455.1309616（HDP 1.3.9.0-01351 - 保持不变）
* HDInsight 3.0.6.455.1309616（HDP 2.0.9.0-2097 - 保持不变）
* HDInsight 3.1.2.455.1309616（HDP 2.1.9.0-2196 - 保持不变）
* SDK 不适用

此版本包含以下更新。

<table border="1">

<tr>
<th>标题</th>
<th>说明</th>
<th>受影响区域（例如服务、组件或 SDK）</p></th>
<th>群集类型（例如 Hadoop、HBase 或 STORM）</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>Bug 修复</td>
<td>我们创建了几个重要的 Bug 修复，在 Azure 的升级过程中提高 HDInsight 群集的可靠性。</td>
<td>服务</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2015 年 1 月 5 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.420.1246118（HDP 1.3.9.0-01351 - 保持不变）
* HDInsight 3.0.6.420.1246118（HDP 2.0.9.0-2097 - 保持不变）
* HDInsight 3.1.2.420.1246118（HDP 2.1.9.0-2196 - 保持不变）

此版本包含以下更新。

<table border="1">

<tr>
<th>标题</th>
<th>说明</th>
<th>组件</th>
<th>群集类型</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>Twitter 趋势分析和 Mahout 电影推荐的示例</td>
<td><p>在此版本中，HDInsight 查询控制台具有两个额外示例：</p>

<p><b>Twitter 趋势分析</b><br>
Twitter 等网站所提供的公共 API 是一类用于分析和了解流行趋势的有用数据源。在此教程中，可以学习如何使用 Hive 获取符合以下条件的 Twitter 用户列表：发送包含特定文字的推文最多。</p>

<p><b>Mahout 电影推荐</b><br>
Apache Mahout 是 Apache Hadoop 的机器学习库。Mahout 包含用于处理数据的算法（例如筛选、分类和群集）。在本教程中，将使用一个推荐引擎根据好友观看的电影生成电影推荐。</p></td>
<td>查询控制台</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>更改为 Hive 配置的默认值：hive.auto.convert.join.noconditionaltask.size</td>
<td><p>此大小配置适用于自动转换后的映射联接。此值表示可以转换为适配内存的哈希映射的表的大小之和。在以前版本中，此值从默认值 10 MB 增加为 128 MB。但是，128 MB 的新值会导致作业由于内存不足而失败。此版本将恢复 10 MB 的默认值。在给定查询和表的大小的情况下，客户在群集创建过程给中仍然可以选择覆盖此值。有关此设置以及如何重写该方法的详细信息，请参阅 Hortonworks 文档中的<a href="http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.0.2/ds_Hive/optimize-joins.html#JoinOptimization-OptimizeAutoJoinConversion" target="_blank">优化自动联接转换</a>。</p></td>
<td>Hive</td>
<td>Hadoop、Hbase</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2014 年 12 月 23 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.10.420.1246783（HDP 版本保持不变）
* HDInsight 3.0.6.420.1246783（HDP 版本保持不变）
* HDInsight 3.1.1.420.1246783（HDP 版本保持不变）

此版本包含以下更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>组件</th>
<th>群集类型</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>由于负载过大造成间歇性的群集创建失败</td>
<td><p>改良群集创建期间用于下载 HDP 包的算法，可对因负载过大而造成的失败提供更稳健的处理。</p></td>
<td>服务</td>
<td>Hadoop、Hbase、Storm</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2014 年 12 月18 日发行说明
此版本包含以下组件更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>组件</th>
<th>群集类型</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td><a href = "/documentation/articles/hdinsight-hadoop-customize-cluster" target="_blank">群集自定义正式发布</a></td>
<td><p>自定义可让你自定义 Azure HDInsight 群集，以搭配使用 Apache Hadoop 生态系统的项目。使用这项新功能，现在可以试验并部署 Hadoop 项目到 Azure HDInsight。这通过启用“脚本操作”功能启用，该功能可以使用自定义脚本，以任意方式修改 Hadoop 群集。此自定义适用于所有类型的 HDInsight 群集，包括 Hadoop、HBase 和 Storm。为了演示这项强大功能，我们记录了过程以安装流行的 <a href = "/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql" target="_blank">Spark</a>、<a href = "/documentation/articles/hdinsight-hadoop-r-scripts" target="_blank">R</a><a href = "/documentation/articles/hdinsight-hadoop-solr-install" target="_blank">、Solr</a> 和 <a href = "/documentation/articles/hdinsight-hadoop-giraph-install" target="_blank">Giraph</a> 模块。这个版本还添加了让客户通过 Azure 门户预览指定其自定义脚本操作的功能、提供如何使用帮助器方法生成自定义脚本操作的指导和最佳作法，并提供有关如何测试脚本操作的指导。</p></td>
<td>功能正式发布</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2014 年 12 月 5 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.9.406.1221105 (HDP 1.3.9.0-01351)
* HDInsight 3.0.5.406.1221105 (HDP 2.0.9.0-2097)
* HDInsight 3.1.1.406.1221105 (HDP 2.1.9.0-2196)
* HDInsight SDK 不适用

此版本包含以下组件更新。

<table border="1">
<tr>
<th>标题</th>
<th>说明</th>
<th>组件</th>
<th>群集类型</th>
<th>JIRA（如果适用）</th>
</tr>
<tr>
<td>Bug 修复：将大量分区添加到 Hive DDL 中的表时发生的间歇性错误。 </td>
<td><p>将大量分区添加到 Hive 表时，如果 Hive 元存储数据库发生间歇性的连接错误，则 Hive DDL 可能失败。如果发生此错误，可在 Hive 错误日志中看到以下语句： </p><p>"ERROR [main]: ql.Driver (SessionState.java:printError(547)) - FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask.MetaException(message:java.lang.RuntimeException: commitTransaction was called but openTransactionCalls = 0.This probably indicates that there are unbalanced calls to openTransaction/commitTransaction)"</p></td>
<td>Hive</td>
<td>Hadoop、Hbase</td>
<td>HIVE-482 （这是内部 JIRA，因此不可在外部加上引号。在此记录供参考之用。）</td>
</tr>
<tr>
<td>Bug 修复：HDInsight 查询偶尔挂起</td>
<td>发生此情况时，可在 WebHCat 启动器作业的 WebHCat 日志中看到以下语句： <p>"org.apache.hive.hcatalog.templeton.CatchallExceptionMapper | org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.yarn.exceptions.YarnRuntimeException): Could not load history file {wasb url to the history file}"</p></td>
<td>WebHCat</td>
<td>Hadoop</td>
<td>HIVE-482 （这是内部 JIRA，因此不可在外部加上引号。在此记录供参考之用。）</td>
</tr>
<tr>
<td>Bug 修复：Hbase 查询延迟偶尔激增</td>
<td>如果发生这种情况，用户会发现 Hbase 查询延迟偶尔激增 3 秒。</td>
<td>HDInsight 群集网关</td>
<td>HBase</td>
<td>不适用</td>
</tr>
<tr>
<td>HDP JAR 文件名更改</td>
<td>对于 HDI 群集 3.0 版，HDP 安装的内部 jar 文件有一些更改。jetty-6.1.26.jar 已替换为 jetty-6.1.26.hwx.jar。jetty-util-6.1.26.jar 已替换为 jetty-util-6.1.26.hwx.jar。这些更改适用于 Hadoop、Mahout、WebHCat 和 Oozie 项目。</td>
<td>Hadoop、Mahout、WebHCat、Oozie</td>
<td>Hadoop、HBase</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2014 年 11 月 21 日发行说明
随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.9.382.1169709（从 2014/11/14 后未更改）
* HDInsight 3.0.5.382.1169709（从 2014/11/14 后未更改）
* HDInsight 3.1.1.382.1169709（从 2014/11/14 后未更改）
* HDINsight SDK 1.4.0

此版本包含以下组件更新。

<table border="1">
<tr><th>标题</th><th>说明</th><th>组件</th><th>群集类型</th><th>JIRA（如果适用）</th></tr>
<tr>
<td>访问应用程序日志</td>
<td>可以编程方式枚举群集上运行的应用程序，并下载相关的应用程序或容器特定日志以帮助调试有问题的应用程序。</td>
<td>SDK</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>在 IHdInsightClient.DeleteCluster 中指定地区名称的能力 </td>
<td>Azure HDInsight SDK 现在提供在使用“DeleteCluster”时指定区域名称的功能。这有助于解锁在不同的地区拥有 2 个同名资源，而且已无法删除任一资源的客户。</td>
<td>SDK</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>ClusterDetails.DeploymentId</td>
<td>“ClusterDetails”对象返回“Deployment”字段来代表群集的唯一标识符。这可在使用相同名称跨群集尝试创建时，保证有唯一的名称。</td>
<td>SDK</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

## HDInsight 2014 年 11 月 14 日发行说明

随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1.9.382.1169709
* HDInsight 3.0.5.382.1169709
* HDInsight 3.1.1.382.1169709

此版本包含以下新功能、组件更新和 Bug 修复。

<table border="1">
<tr><th>标题</th><th>说明</th><th>组件</th><th>群集类型</th><th>JIRA（如果适用）</th></tr>
<tr>
<td>脚本操作（预览）</td>
<td>预览版的群集自定义功能，可以任意方式使用自定义脚本来修改 Hadoop 群集。凭借此功能，用户可以体验如何将 Apache Hadoop 生态系统中的项目部署到 Azure HDInsight 群集。此自定义功能适用于所有类型的 HDInsight 群集，包括 Hadoop、HBase 和 Storm。</td>
<td>新功能</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>为 Azure 网站与存储日志分析预先生成作业</td>
<td>HDInsight 查询控制台有一个入门库，它支持处理数据或示例数据的解决方案。
<p>**处理数据的解决方案**：<br>
我们已经为最常见的部分数据分析方案创建作业，作为你创建自己的解决方案的起点。可以运行作业来使用自己的数据，以查看其运作方式。就绪后，使用已学得的知识来创建在预先生成作业后所制作的解决方案模型。</p>
<p>**处理示例数据的解决方案**：<br>
逐步运行部分基本方案（例如分析 Web 日志和传感器数据）来了解如何使用 HDInsight。将不只是学习如何使用 HDInsight 来分析此类数据，还会学习如何将其他应用程序和服务连接到此数据。通过连接到 Microsoft Excel 来提供此强大方案示例，以此可视化数据。</p></td>
<td>查询控制台</td>
<td>Hadoop</td>
<td>不适用</td>
</tr>
<tr>
<td>Templeton 中的内存泄漏修复</td>
<td>已解决 Templeton 中的内存泄漏，该问题影响已长期运行群集的客户，或每秒提交数百次作业请求的客户。此问题显示为 Templeton 5xx 错误，解决方法为重新启动服务。不再需要此解决方法。</td>
<td>Templeton</td>
<td>全部</td>
<td>https://issues.apache.org/jira/browse/HADOOP-11248</td>
</tr>
</table>

> [AZURE.NOTE]
为了演示群集自定义所提供的新功能，此过程使用脚本操作在所述的群集上安装 Spark 和 R 模块。有关详细信息，请参阅：

* [在 HDInsight 群集上安装和使用 Spark 1.0](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)
* [在 HDInsight Hadoop 群集上安装并使用 R](/documentation/articles/hdinsight-hadoop-r-scripts/)

## HDInsight 2014 年 11 月 7 日发行说明

随此版本一起部署的 HDInsight 群集的所有版本号包括：

* HDInsight 2.1 2.1.9.374.1153876
* HDInsight 3.0 3.0.5.374.1153876
* HDInsight 3.1 3.1.1.374.1153876

此版本包含以下组件更新。

<table border="1">
<tr><th>标题</th><th>说明</th><th>组件</th><th>群集类型</th><th>JIRA（如果适用）</th></tr>
<tr>
<td>HDP 2.1.7</td>
<td>此版本基于 Hortonworks 数据平台 (HDP) 2.1.7。有关详细信息，请参阅 <a href="http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html" target="_blank">HDP 2.1.7 发行说明</a>。</td>
<td>HDP</td>
<td>全部</td>
<td>不适用</td>
</tr>
<tr>
<td>YARN Timeline Server</td>
<td>YARN Timeline Server（也称为 Generic Application History Server）默认情况下已启用。Timeline Server 提供已完成应用程序的一般相关信息，例如应用程序 ID、应用程序名称、应用程序状态、应用程序提交时间和应用程序完成时间。可访问 URI http://headnodehost:8188 或运行 YARN 命令：yarn application -list -appStates ALL，以从头节点检索此应用程序信息。也可以通过 REST API（位于 https://{ClusterDnsName}. azurehdinsight.cn/ws/v1/applicationhistory/）远程检索此信息。有关详细信息，请参阅 <a href="http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html" target="_blank">YARN Timeline Server</a>。</td>
<td>服务、YARN</td>
<td>Hadoop、HBase</td>
<td>不适用</td>
</tr>
<tr>
<td>群集部署 ID</td>
<td>从最新的 SDK 1.3.3.1.5426.29232 版开始，用户可访问 HDInsight 为每个群集所发行的唯一 ID。在跨创建或删除方案重复使用 dnsname 时，这能让客户理解群集的唯一实例。</td>
<td>SDK</td>
<td>全部</td>
<td>不适用</td>
</tr>
</table>

> [AZURE.NOTE]
这个版本已修复防止门户显示、防止 SDK 或 Windows PowerShell 返回完整版本号的错误。

## 2014 年 10 月 15 日发行说明

此修补程序版本修复了 Templeton 中影响 Templeton 重度用户的内存泄漏。在某些情况下，Templeton 的重度用户看到以 500 错误码表示的错误，因为请求没有足够的内存用于运行。重新启动 Templeton 服务就能解决此问题。现在已修复此问题。

## 2014 年 10 月 7 日发行说明

* 使用 Ambari 终结点“https://{clusterDns}.azurehdinsight.cn/ambari/api/v1/clusters/{clusterDns}.azurehdinsight.cn/services/{servicename}/components/{componentname}”时，*host\_name* 字段会返回节点的完全限定域名 (FQDN)，而不只是主机名。例如，会看到 FQDN“**headnode0.{ClusterDNS}.azurehdinsight.cn**”，而不是返回“**headnode0**”。需要这种改变促进可以在一个虚拟网络中部署多个群集类型（如 HBase 和 Hadoop）的方案。例如，使用 HBase 作为 Hadoop 的后端平台时，会发生这种情况。

* 我们已为 HDInsight 群集的默认部署提供新内存设置。以前的默认内存设置没有充分考虑对于正在部署的 CPU 内核数的指导。根据 Hortonworks 建议，这些新的内存设置应该会提供更好的默认值。要更改，请参阅 SDK 参考文档来更改群集配置。下表中逐项列出了默认 4 CPU 内核（8 容器）HDInsight 群集使用的新内存设置。（还附带提供了在本次发布之前使用的值）。

<table border="1">
<tr><th>组件</th><th>内存分配</th></tr>
<tr><td> yarn.scheduler.minimum-allocation</td><td>768 MB（以前为 512 MB）</td></tr>
<tr><td> yarn.scheduler.maximum-allocation</td><td>6144 MB（无变化）</td></tr>
<tr><td>yarn.nodemanager.resource.memory</td><td>6144 MB（无变化）</td></tr>
<tr><td>mapreduce.map.memory</td><td>768 MB（以前为 512 MB）</td></tr>
<tr><td>mapreduce.map.java.opts</td><td>opts=-Xmx512m（以前为 -Xmx410m）</td></tr>
<tr><td>mapreduce.reduce.memory</td><td>1536 MB（以前为 1024 MB）</td></tr>
<tr><td>mapreduce.reduce.java.opts</td><td>opts=-Xmx1024m（以前为 -Xmx819m）</td></tr>
<tr><td>yarn.app.mapreduce.am.resource</td><td>768 MB（以前为 1024 MB）</td></tr>
<tr><td>yarn.app.mapreduce.am.command</td><td>opts=-Xmx512m（以前为 -Xmx819m）</td></tr>
<tr><td>mapreduce.task.io.sort</td><td>256 MB（以前为 200 MB）</td></tr>
<tr><td>tez.am.resource.memory</td><td>1536 MB（无变化）</td></tr>
</table>

关于 Azure PowerShell 和 HDInsight SDK 错误消息：“群集未配置 HTTP 服务访问”：

* 此错误是已知的[兼容性问题](https://social.msdn.microsoft.com/Forums/azure/a7de016d-8de1-4385-b89e-d2e7a1a9d927/hdinsight-powershellsdk-error-cluster-is-not-configured-for-http-services-access?forum=hdinsight)，源于 HDInsight SDK 或 Azure PowerShell 版本和群集版本的差异。8 月 15 日或之后创建的群集支持虚拟网络的新预配功能。但旧版的 HDInsight SDK 或 Azure PowerShell 无法正确解释此功能。结果造成某些作业提交操作失败。如果使用 HDInsight SDK API 或 Azure PowerShell cmdlet（**Use-AzureRmHDInsightCluster** 或 **Invoke-AzureRmHDInsightHiveJob**）提交作业，这些作业可能失败并返回错误消息“*未在群集 <群集名称> 上配置 HTTP 服务访问权限*”。 或者（根据具体的操作），可能会收到其他错误消息，例如“*无法连接到群集*”。
* 在最新版 HDInsight SDK 和 Azure PowerShell 中，这些兼容性问题均已解决。我们建议将 HDInsight SDK 更新至 1.3.1.6 版本或更高版本，将 Azure PowerShell 工具更新至 0.8.8 版本或更高版本。可以从 [Nuget](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 获取最新的 HDInsight SDK，从[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 获取 Azure PowerShell 工具。

## HDInsight 3.1 2014 9 月 12 日发行说明
* 此版本基于 Hortonworks 数据平台 (HDP) 2.1.5。有关此版本中修复的 Bug 列表，请参阅 Hortonworks 站点上的[此版本中修复的问题](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.5/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.5-fixed.html)页。
* 在 Pig 库文件夹中，“avro-mapred-1.7.4.jar”文件已改为“avro-mapred-1.7.4-hadoop2.jar”。 此文件的内容包含一个小 Bug 的不间断修复。建议客户不要直接依赖 JAR 文件名，以避免文件重命名时出现中断。

## 2014 年 8 月 24 日发行说明
* 我们正在添加以下 WebHCat 配置 (HIVE-7155)，该配置可将 Templeton 控制器作业的默认内存限制设置为 1 GB：（以前的默认值是 512 MB。）

        templeton.mapper.memory.mb (=1024)

    * 这项更改解决了某些 Hive 查询由于内存限制较低而遇到的以下错误：“容器即将超出物理内存限制”。
    * 要恢复到旧默认值，你可以在创建群集时使用以下命令通过 Azure PowerShell 将此配置值设置为 512：

            Add-AzureRmHDInsightConfigValues -Core @{"templeton.mapper.memory.mb"="512";}
* zookeeper 角色的主机名已更改为 *zookeeper*。这会影响群集内部的名称解析，但不会影响外部 REST API。如果你的组件使用了 *zookeepernode* 主机名，则需更新这些组件，让其使用新名称。三个 zookeeper 节点的新名称为：

    * zookeeper0
    * zookeeper1
    * zookeeper2
* 已更新 HBase 版本支持矩阵。仅 HDInsight 版本 3.1（HBase 版本 0.98）支持 HBase 工作负载生成。版本 3.0（用于预览）将不支持升级。

## 2014/8/15 之前创建的群集的注意事项
由于 Azure PowerShell 或 HDInsight SDK 与群集之间的版本不同，你可能会遇到 Azure PowerShell 或 HDInsight SDK 错误消息“未在群集 <群集名称> 上配置 HTTP 服务访问权限”（或者根据操作，遇到其他错误消息，如：“无法连接到群集”）。8 月 15 日或之后创建的群集支持虚拟网络的新预配功能。旧版本 Azure PowerShell 或 HDInsight SDK 无法正确解释此功能，导致提交作业操作失败。如果使用 HDInsight SDK API 或 Azure PowerShell cmdlet（例如 Use-AzureRmHDInsightCluster 或 Invoke-AzureRmHDInsightHiveJob）来提交作业，这些作业可能失败并返回上述其中一个错误消息。

在最新版 HDInsight SDK 和 Azure PowerShell 中，这些兼容性问题均已解决。我们建议将 HDInsight SDK 更新至 1.3.1.6 版本或更高版本，将 Azure PowerShell 工具更新至 0.8.8 版本或更高版本。你可以从 [NuGet][nuget-link] 访问最新的 HDInsight SDK。可以使用 [Microsoft Web 平台安装程序][webpi-link]访问 Azure PowerShell 工具。

## 2014 年 7 月 28 日发行说明
* **HDInsight 已在新区域推出：**我们已将 HDInsight 的地理覆盖范围扩展到三个新的区域。HDInsight 客户可以在这些区域创建群集。
    * 中国东部
    * 中国北部
* HDInsight 1.6 版（HDP1.1、Hadoop 1.0.3）和 HDInsight 2.1 版（HDP1.3、Hadoop 1.2）即将从 Azure 门户预览中删除。可以继续使用 Azure PowerShell cmdlet [New-AzureRmHDInsightCluster](http://msdn.microsoft.com/zh-cn/library/dn593744.aspx) 或 [HDInsight SDK](http://msdn.microsoft.com/zh-cn/library/azure/dn469975.aspx) 来创建这些版本的 Hadoop 群集。有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/)页。
* 此版本中发生的 Hortonworks 数据平台 (HDP) 更改：

<table border="1">
<tr><th>HDP</th><th>更改</th></tr>
<tr><td>HDP 1.3/HDI 2.1</td><td>无更改</td></tr>
<tr><td>HDP 2.0/HDI 3.0</td><td>无更改</td></tr>
<tr><td>HDP 2.1/HDI 3.1</td><td>zookeeper: ['3.4.5.2.1.3.0-1948'] -> ['3.4.5.2.1.3.2-0002']</td></tr>
</table>

## 2014 年 6 月 24 日发行说明
此版本包含 HDInsight 服务的几项新的增强功能：

* **HDP 2.1 可用性**：HDInsight 3.1（包含 HDP 2.1）已正式发布，并成为新群集的默认版本。
* **HBase - Azure 门户预览改进**：我们将在预览版中提供 HBase 群集。只需单击几下鼠标，就能从门户创建 HBase 群集：

借助 HBase，你可以在 HDInsight 上生成各种实时工作负载 - 从用于处理大型数据集的交互式网站，到用于存储来自数百万个终结点的传感器数据与遥测数据的服务。接下来要做的就是使用 Hadoop 作业分析这些工作负载中的数据，也可以通过 Azure PowerShell 和 Hive 群集仪表板在 HDInsight 中完成这种分析。

### Apache Mahout 已预装在 HDInsight 3.1 上
 [Mahout](http://hortonworks.com/hadoop/mahout/) 已预装在 HDInsight 3.1 Hadoop 群集上，从而无需任何其他群集配置，就能运行 Mahout 作业。例如，可以使用远程桌面协议 (RDP) 远程访问 Hadoop 群集，并且无需执行附加的步骤就能运行 Hello World Mahout 命令：

        mahout org.apache.mahout.classifier.df.tools.Describe -p /user/hdp/glass.data -f /user/hdp/glass.info -d I 9 N L

        mahout org.apache.mahout.classifier.df.BreimanExample -d /user/hdp/glass.data -ds /user/hdp/glass.info -i 10 -t 100

有关此过程的更完整说明，请参阅 Apache Mahout 网站上的 [Breiman 示例](https://mahout.apache.org/users/classification/breiman-example.html)文档。

### Hive 查询可以在 HDInsight 3.1 中使用 Tez
Hive 0.13 已在 HDInsight 3.1 中提供，并且能够使用 Tez 运行查询，这带来了极大的性能改善。默认情况下，没有为 Hive 查询启用 Tez。要使用 Tez，必须选择启用它。可以通过运行以下代码片段来启用 Tez：

        set hive.execution.engine=tez;
        select sc_status, count(*), histogram_numeric(sc_bytes,5) from website_logs_orc_local group by sc_status;

Hortonworks 发布了使用以标准基准版提供的 Tez 后，Hive 查询性能得到增强的明细。有关详细信息，请参阅[适用于 Enterprise Hadoop 的 Apache Hive 13 基准](http://hortonworks.com/blog/benchmarking-apache-hive-13-enterprise-hadoop/)。

有关将 Hive 与 Tez 配合使用的详细信息，请参阅 [Tez 上的 Hive](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)。

### 群集版本之间的注意事项
**HDInsight 3.1 群集使用的 Oozie 元存储与旧版的 HDInsight 2.1 群集不兼容，无法与此旧版本一起使用**。

与 HDInsight 3.1 群集一起部署的自定义 Oozie 元存储数据库无法在 HDInsight 2.1 群集上重复使用。即使是源自 HDInsight 2.1 群集的元存储也是如此。不支持此方案，因为元存储架构与 HDInsight 3.1 群集一起使用时升级，所以与 2.1 群集所需的元存储就不再兼容。尝试重复使用已在 HDInsight 3.1 群集上使用过的 Oozie 元存储会使 2.1 群集变得无用。

**群集之间无法共享 Oozie 元存储。**

Oozie 元存储连接到特定群集，无法在群集之间共享。

### 重大变化
**前缀语法**：HDInsight 3.1 和 3.0 群集仅支持“wasbs://”语法。较早的“asv://”语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.1 或 3.0 群集中不受支持。这意味着提交到 HDInsight 3.1 或 3.0 群集的任何显式使用“asv://”语法的作业都将会失败。应改用“wasbs://”语法。而且，提交到任何 HDInsight 3.1 或 3.0 群集的作业，如果是使用现有元存储创建，而该元存储包含对使用“asv://”语法的资源的显式引用，则这些作业也会失败。需要使用“wasbs://”语法重新创建这些元存储，确定资源地址。

**端口**：HDInsight 服务使用的端口已更改。以前使用的端口号在 Windows 操作系统瞬息端口范围内。端口从预定义的临时范围自动分配，该范围适用于基于 Internet 协议的短期通信。新的一组允许的 Hortonworks 数据平台 (HDP) 服务端口号在此范围外，目的是避免遭遇头节点上运行的服务所使用的端口时出现冲突。新端口号将不会导致任何重大变化。使用的端口号如下所示：

 **HDInsight 1.6 (HDP 1.1)**

<table border="1">
<tr><th>名称</th><th>值</th></tr>
<tr><td>dfs.http.address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.secondary.http.address</td><td>0.0.0.0:30090</td></tr>
<tr><td>mapred.job.tracker.http.address</td><td>jobtrackerhost:30030</td></tr>
<tr><td>mapred.task.tracker.http.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>mapreduce.history.server.http.address</td><td>0.0.0.0:31111</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table>

 **HDInsight 3.1 和 3.0（HDP 2.1 和 2.0）**

<table border="1">
<tr><th>名称</th><th>值</th></tr>
<tr><td>dfs.namenode.http-address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.namenode.https-address</td><td>headnodehost:30470</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.namenode.secondary.http-address</td><td>0.0.0.0:30090</td></tr>
<tr><td>yarn.nodemanager.webapp.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table>

### 依赖项
在 HDInsight 3.x (HDP2.x) 中添加了以下依赖项：

* guice-servlet
* optiq-core
* javax.inject
* activation
* jsr305
* geronimo-jaspic\_1.0\_spec
* jul-to-slf4j
* java-xmlbuilder
* ant
* commons-compiler
* jdo-api
* commons-math3
* paranamer
* jaxb-impl
* stringtemplate
* eigenbase-xom
* jersey-servlet
* commons-exec
* jaxb-api
* jetty-all-server
* janino
* xercesImpl
* optiq-avatica
* jta
* eigenbase-properties
* groovy-all
* hamcrest-core
* mail
* linq4j
* jpam
* jersey-client
* aopalliance
* geronimo-annotation\_1.0\_spec
* ant-launcher
* jersey-guice
* xml-apis
* stax-api
* asm-commons
* asm-tree
* wadl
* geronimo-jta\_1.1\_spec
* guice
* leveldbjni-all
* velocity
* jettison
* snappy-java
* jetty-all
* commons-dbcp

HDInsight 3.x (HDP2.x) 中不再存在以下依赖项：

* jdeb
* kfs
* sqlline
* ivy
* aspectjrt
* json
* core
* jdo2-api
* avro-mapred
* datanucleus-enhancer
* jsp
* commons-logging-api
* commons-math
* JavaEWAH
* aspectjtools
* javolution
* hdfsproxy
* hbase
* snappy

### 版本更改
在 HDInsight 2.x (HDP1.x) 与 HDInsight 3.x (HDP2.x) 之间发生了以下版本更改：

* metrics-core：['2.1.2'] -> ['3.0.0']
* derbynet：['10.4.2.0'] -> ['10.10.1.1']
* datanucleus：['rdbms-3.0.8'] -> ['rdbms-3.2.9']
* jasper-compiler：['5.5.12'] -> ['5.5.23']
* log4j：['1.2.15', '1.2.16'] -> ['1.2.16', '1.2.17']
* derbyclient：['10.4.2.0'] -> ['10.10.1.1']
* httpcore：['4.2.4'] -> ['4.2.5']
* hsqldb：['1.8.0.10'] -> ['2.0.0']
* jets3t：['0.6.1'] -> ['0.9.0']
* protobuf-java：['2.4.1'] -> ['2.5.0']
* derby：['10.4.2.0'] -> ['10.10.1.1']
* jasper：['runtime-5.5.12'] -> ['runtime-5.5.23']
* commons-daemon：['1.0.1'] -> ['1.0.13']
* datanucleus-core：['3.0.9'] -> ['3.2.10']
* datanucleus-api-jdo：['3.0.7'] -> ['3.2.6']
* zookeeper：['3.4.5.1.3.9.0-01320'] -> ['3.4.5.2.1.3.0-1948']
* bonecp：['0.7.1.RELEASE'] -> ['
* 0\.8.0.RELEASE']

### 驱动程序
SQL Server 的 Java 数据库连接 (JDBC) 驱动程序由 HDInsight 在内部使用，不用于外部操作。如果希望使用开放数据库连接 (ODBC) 连接到 HDInsight，请使用 Microsoft Hive ODBC 驱动程序。有关详细信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight](/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)。

### Bug 修复
随着此版本的发行，我们已完成了多项 Bug 修复，并更新了以下 HDInsight 版本：

* HDInsight 2.1 (HDP 1.3)
* HDInsight 3.0 (HDP 2.0)
* HDInsight 3.1 (HDP 2.1)

## Hortonworks 发行说明
以下位置提供了 HDInsight 版本群集使用的 Hortonworks 数据平台 (HDP) 的发行说明：

* HDInsight 版本 3.1 使用基于 [Hortonworks 数据平台 2.1.7][hdp-2-1-7] 的 Hadoop 分发版。这是使用 2014 年 11 月 7 日之后的 Azure 门户预览时创建的默认 Hadoop 群集。创建于 2014 年 11 月 7 日之前的 HDInsight 3.1 群集基于 [Hortonworks 数据平台 2.1.1][hdp-2-1-1]
* HDInsight 版本 3.0 使用基于 [Hortonworks 数据平台 2.0][hdp-2-0-8] 的 Hadoop 分发版。
* HDInsight 版本 2.1 使用基于 [Hortonworks 数据平台 1.3][hdp-1-3-0] 的 Hadoop 分发版。
* HDInsight 版本 1.6 使用基于 [Hortonworks 数据平台 1.1][hdp-1-1-0] 的 Hadoop 分发版。

[hdp-2-1-7]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html

[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.1.0.15_1.html

[nuget-link]: https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.HDInsight/

[webpi-link]: http://go.microsoft.com/?linkid=9811175&clcid=0x409

[hdinsight-install-spark]: /documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/
[hdinsight-r-scripts]: /documentation/articles/hdinsight-hadoop-r-scripts/

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->