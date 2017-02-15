<!-- not suitable for Mooncake -->

<properties
    pageTitle="可与 HDInsight 群集使用的不同组件有哪些？|Azure"
    description="HDInsight 支持多个可部署的 Hadoop 群集组件和版本。请参见“支持的 Hadoop 和 HortonWorks 数据平台 (HDP) 版本”。"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    author="saurinsh"
    tags="azure-portal"
    documentationcenter="" />
<tags 
    ms.assetid="367b3f4a-f7d3-4e59-abd0-5dc59576f1ff"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/13/2016"
    wacn.date="02/06/2017"
    ms.author="saurinsh" />

# 可与 HDInsight 使用的不同 Hadoop 组件有哪些？
了解 HDInsight 提供的不同服务级别，以及 HDInsight 所包括的不同 Hadoop 组件版本。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

## 可与不同 HDInsight 版本使用的 Hadoop 组件
Azure HDInsight 支持多个可随时部署的 Hadoop 群集版本。每个版本选项创建 Hortonworks 数据平台 (HDP) 分发的特定版本和该分发内包含的一组组件。下表中逐项列出了与 HDInsight 群集版本关联的组件版本。请注意，Azure HDInsight 使用的默认群集版本当前是 3.4（到 09/14/2016 为止）并基于 HDP 2.4。

> [AZURE.NOTE]
服务默认版本随时可更改，恕不另行通知。如果依赖某个版本，我们建议在创建群集时使用 .NET SDK/Azure PowerShell 和 Azure CLI 指定版本。
> 
> 

| 组件 | HDInsight 版本 3.5 | HDInsight 版本 3.4（默认） | HDInsight 版本 3.3 | HDInsight 版本 3.2 | HDInsight 版本 3.1 | HDInsight 版本 3.0 |
| --- | --- | --- | --- | --- | --- | --- |
| Hortonworks 数据平台 |2\.5 |2\.4 |2\.3 |2\.2 |2\.1.7 |2\.0 |
| Apache Hadoop 和 YARN |2\.7.3 |2\.7.1 |2\.7.1 |2\.6.0 |2\.4.0 |2\.2.0 |
| Apache Tez |0\.7.0 |0\.7.0 |0\.7.0 |0\.5.2 |0\.4.0 | |
| Apache Pig |0\.16.0 |0\.15.0 |0\.15.0 |0\.14.0 |0\.12.1 |0\.12.0 |
| Apache Hive 和 HCatalog |1\.2.1.2.5 |1\.2.1 |1\.2.1 |0\.14.0 |0\.13.1 |0\.12.0 |
| Apache HBase |1\.1.2 |1\.1.2 |1\.1.1 |0\.98.4 |0\.98.0 | |
| Apache Sqoop |1\.4.6 |1\.4.6 |1\.4.6 |1\.4.5 |1\.4.4 |1\.4.4 |
| Apache Oozie |4\.2.0 |4\.2.0 |4\.2.0 |4\.1.0 |4\.0.0 |4\.0.0 |
| Apache Zookeeper |3\.4.6 |3\.4.6 |3\.4.6 |3\.4.6 |3\.4.5 |3\.4.5 |
| Apache Storm |1\.0.1 |0\.10.0 |0\.10.0 |0\.9.3 |0\.9.1 | |
| Apache Mahout |0\.9.0+ |0\.9.0+ |0\.9.0+ |0\.9.0 |0\.9.0 | |
| Apache Phoenix |4\.7.0 |4\.4.0 |4\.4.0 |4\.2.0 |4\.0.0.2.1.7.0-2162 | |
| Apache Spark |1\.6.2 + 2.0（仅限 Linux） |1\.6.0（仅限 Linux） |1\.5.2（仅限 Linux/实验性生成） |1\.3.1（仅限 Windows） | | |

**获取当前组件版本信息**

与 HDInsight 群集版本关联的组件版本可能在将来的 HDInsight 更新中更改。确定可用组件并验证正在使用哪些群集版本的一种方法是使用 Ambari REST API。**GetComponentInformation** 命令可用于检索有关服务组件的信息。有关详细信息，请参阅 [Ambari 文档][ambari-docs]。获取此信息的另一个方法是使用远程桌面登录到群集并直接检查“C:\\apps\\dist”目录的内容。

**发行说明**

请参阅 [HDInsight 发行说明](/documentation/articles/hdinsight-release-notes/)，了解 HDInsight 最新版本的更多发行说明。

## <a name="supported-hdinsight-versions"></a> 支持的 HDInsight 版本
下表列出当前可用的 HDInsight 版本以及它们使用的相应 Hortonworks 数据平台版本和发布日期。如果知道，还提供其支持到期日期和弃用日期。请注意以下事项：

* 默认情况下，会针对 HDInsight 2.1 和更高版本的群集部署具有两个头节点的高度可用群集。它们不适用于 HDInsight 1.6 群集。
* 某版本的支持到期后，可能不再通过 Azure 门户预览提供它。下表列出 Azure 经典管理门户上提供的版本。可继续使用 Windows PowerShell [New-AzureRmHDInsightCluster](https://msdn.microsoft.com/zh-cn/library/mt619331.aspx) 命令中的 `Version` 参数和 .NET SDK 获取群集版本，直到其弃用日期为止。

| HDInsight 版本 | HDP 版本 | VM OS | 高可用性 | 发布日期 | 在 Azure 门户预览上提供 | 支持到期日期 | 弃用日期 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HDI 3.5 |HDP 2.5 |Ubuntu 16 |是 |2016/9/30 |是 | | |
| HDI 3.4 |HDP 2.4 |Ubuntu 14.0.4 LTS |是 |03/29/2016 |是 |2016/12/29 |2018/1/9 |
| HDI 3.3 |HDP 2.3 |Ubuntu 14.0.4 LTS 或 Windows Server 2012R2 |是 |12/02/2015 |是 |06/27/2016 |07/31/2017 |
| HDI 3.2 |HDP 2.2 |Ubuntu 12.04 LTS 或 Windows Server 2012R2 |是 |2/18/2015 |是 |3/1/2016 |04/01/2017 |
| HDI 3.1 |HDP 2.1 |Windows Server 2012R2 |是 |6/24/2014 |否 |05/18/2015 |06/30/2016 |
| HDI 3.0 |HDP 2.0 |Windows Server 2012R2 |是 |02/11/2014 |否 |09/17/2014 |06/30/2015 |
| HDI 2.1 |HDP 1.3 |Windows Server 2012R2 |是 |10/28/2013 |否 |05/12/2014 |05/31/2015 |
| HDI 1.6 |HDP 1.1 | |否 |10/28/2013 |否 |04/26/2014 |05/31/2015 |

## <a name="hdi-version-32-and-33-nearing-deprecation-date"></a>HDI 版本 3.2 和 3.3 接近弃用日期
对 HDI 3.2 群集的支持于 03/01/2016 到期，并将于 04/01/2017 弃用。对 HDI 3.3 群集的支持于 06/27/2016 到期，并将于 07/31/2017 弃用。如果你有 HDI 3.2 或 3.3 HDI 群集，则请立刻将群集升级到 HDI 3.5（最新版本）。

### HDInsight 群集版本的服务级别协议
SLA 用“支持窗口”来定义。“支持窗口”是指 HDInsight 群集版本受 Microsoft 客户服务和支持部门支持的时间段。如果 HDInsight 群集版本具有早于当前日期的**支持过期日期**，则表示它处于支持窗口外。有关支持的 HDInsight 群集版本的列表，请参见上表。给定 HDInsight 版本 X（一旦提供更新的 X+1 版本）的支持到期日期为按以下公式计算所得时间的较晚者：

* 公式 1：发布 HDInsight 群集版本 X 的日期加 180 天。
* 公式 2：HDInsight 群集版本 X+1（X 之后的后续版本）在门户中可用的日期加 90 天。

**弃用日期**是在该日期后，不能在 HDInsight 上创建此群集版本的日期。

> [AZURE.NOTE]
基于 Windows 的 HDInsight 群集（包括版本 2.1、3.0、3.1、3.2 和 3.3）在 Azure 来宾 OS 系列 4 上运行，该系列使用 64 位版本的 Windows Server 2012 R2 并支持 .NET Framework 4.0、4.5、4.5.1 和 4.5.2。
> 
> 

##HDInsight 在 Windows 上弃用
从 HDI 版本 3.4 开始，我们仅发布 Linux 操作系统上的 HDInsight。HDInsight 的一些产品/服务仅适用于 Linux - Apache Ranger、HDInsight 应用程序等。这对于客户来说有多个优势

* 我们可以通过 HDInsight 服务更快地为市场带来开源大数据技术
* 有大型社区和生态系统可以支持
* 开源社区可以主动开发实现 Hadoop 和更新的大数据技术
* HDInsight 服务可以更专注于大数据开放源代码技术

为了对开放源代码大数据技术持续投资，将来的 HDInsight 版本将仅在 Linux 操作系统上可用。Windows 操作系统上的 HDInsight 将不会有任何将来版本。Windows 上的最后一个 HDInsight 版本是 HDI 3.3。对 HDI 3.3 的支持于 06/27/2016 到期，并将于 07/31/2017 弃用。若要从基于 Windows 的 HDInsight 群集迁移到基于 Linux 的群集，请参阅[此文](/documentation/articles/hdinsight-migrate-from-windows-to-linux/)。


## 与 HDInsight 版本相关的 Hortonworks 发行说明
* HDInsight 群集版本 3.4 使用基于 [Hortonworks 数据平台 2.4](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_HDP_RelNotes/content/ch_relnotes_v240.html) 的 Hadoop 分发版。这是使用门户时创建的**默认** Hadoop 群集。
* HDInsight 群集版本 3.3 使用基于 [Hortonworks 数据平台 2.3](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_HDP_RelNotes/content/ch_relnotes_v230.html) 的 Hadoop 分发版。
  
    * [此处](https://storm.apache.org/2015/11/05/storm0100-released.html)提供了 Apache Storm 发行说明。
    * [此处](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12332384&styleName=Text&projectId=12310843)提供了 Apache Hive 发行说明。
* HDInsight 群集版本 3.2 使用基于 [Hortonworks 数据平台 2.2][hdp-2-2] 的 Hadoop 分发版。
  
    * 特定 Apache 组件的发行说明 - [Hive 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310843&version=12326450)、[Pig 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310730&version=12326954)、[HBase 0.98.4](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310753&version=12326810)、[Phoenix 4.2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12315120&version=12327581)、[M/R 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310941&version=12327180)、[HDFS 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310942&version=12327181)、[YARN 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12313722&version=12327197)、[Common](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310240&version=12327179)、[Tez 0.5.2](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314426&version=12328742)、[Ambari 2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12312020&version=12327486)、[Storm 0.9.3](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314820&version=12327112)、[Oozie 4.1.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12324960&projectId=12311620)。
* HDInsight 群集版本 3.1 使用基于 [Hortonworks 数据平台 2.1.7][hdp-2-1-7] 的 Hadoop 分发版。创建于 11/7/2014 之前的 HDInsight 3.1 群集基于 [Hortonworks 数据平台 2.1.1][hdp-2-1-1]
* HDInsight 群集版本 3.0 使用基于 [Hortonworks 数据平台 2.0][hdp-2-0-8] 的 Hadoop 分发版。
* HDInsight 群集版本 2.1 使用基于 [Hortonworks 数据平台 1.3][hdp-1-3-0] 的 Hadoop 分发版。
* HDInsight 群集版本 1.6 使用基于 [Hortonworks 数据平台 1.1][hdp-1-1-0] 的 Hadoop 分发版。

[image-hdi-versioning-versionscreen]: ./media/hdinsight-component-versioning-v1/hdi-versioning-version-screen.png

[wa-forums]: /support/forums/

[connect-excel-with-hive-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/

[hdp-2-2]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/bk_HDP_RelNotes/content/ch_relnotes_v220.html

[hdp-2-1-7]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html

[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html

[ambari-docs]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[zookeeper]: http://zookeeper.apache.org/

<!---HONumber=Mooncake_0103_2017-->