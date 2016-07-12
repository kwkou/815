<properties
	pageTitle="HDInsight 的 Hadoop 群集版本有哪些新功能？| Azure"
	description="HDInsight 支持多个可部署的 Hadoop 群集版本。请参见“支持的 Hadoop 和 HortonWorks 数据平台 (HDP) 版本”。"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="04/28/2016"
	wacn.date="06/29/2016"/>


#HDInsight 提供的 Hadoop 群集版本有哪些新功能？

##HDInsight 版本和 Hadoop 组件
Azure HDInsight 支持多个可随时部署的 Hadoop 群集版本。每个版本选项设置 Hortonworks 数据平台 (HDP) 分发的特定版本和该分发内包含的一组组件。下表中逐项列出了与 HDInsight 群集版本关联的组件版本。请注意，Azure HDInsight 使用的默认群集版本当前是 3.1（到 11/7/2014 为止）并基于 HDP 2.1.7。


<table border="1">
<tr><th>组件</th><th>HDInsight 版本 3.2</th><th>HDInsight 版本 3.1（默认）</th><th>HDInsight 版本 3.0</th><th>HDInsight 版本 2.1</th></tr>
<tr><td>Hortonworks 数据平台</td><td>2.2</td><td>2.1.7</td><td>2.0</td><td>1.3</td></tr>
<tr><td>Apache Hadoop 和 YARN</td><td>2.6.0</td><td>2.4.0</td><td>2.2.0</td><td>1.2.0</td></tr>
<tr><td>Apache Tez</td><td>0.5.2</td><td>0.4.0</td><td></td><td></td></tr>
<tr><td>Apache Pig</td><td>0.14.0</td><td>0.12.1</td><td>0.12.0</td><td>0.11.0</td></tr>
<tr><td>Apache Hive 和 HCatalog</td><td>0.14.0</td><td>0.13.1</td><td>0.12.0</td><td>0.11.0</td></tr>
<tr><td>Apazhe HBase </td><td>0.98.4</td><td>0.98.0</td><td></td><td></td></tr>
<tr><td>Apache Sqoop</td><td>1.4.5</td><td>1.4.4</td><td>1.4.4</td><td>1.4.3</td></tr>
<tr><td>Apache Oozie</td><td>4.1.0</td><td>4.0.0</td><td>4.0.0</td><td>3.3.2</td></tr>
<tr><td>Apache Zookeeper</td><td>3.4.6</td><td>3.4.5</td><td>3.4.5</td><td></td></tr>
<tr><td>Apache Storm</td><td>0.9.3</td><td>0.9.1</td><td></td><td></td></tr>
<tr><td>Apache Mahout</td><td>0.9.0</td><td>0.9.0</td><td></td><td></td></tr>
<tr><td>Apache Phoenix</td><td>4.2.0</td><td>4.0.0.2.1.7.0-2162</td><td></td><td></td></tr>
</table>


**获取当前组件版本信息**

与 HDInsight 群集版本关联的组件版本可能在将来的 HDInsight 更新中更改。确定可用组件并验证正在使用哪些群集版本的一个方法是使用远程桌面登录到群集并直接检查“C:\\apps\\dist”目录的内容。


**发行说明**

请参阅 [HDInsight 发行说明](/documentation/articles/hdinsight-release-notes/)，了解 HDInsight 最新版本的更多发行说明。

### 设置 HDInsight 群集时选择一个版本

通过 HDInsight Windows PowerShell cmdlet 或 HDInsight .NET SDK 创建群集时，你可以使用“Version”参数选择 HDInsight Hadoop 群集的版本。

如果你使用“快速创建”选项，则在默认情况下将获得 HDInsight 的 3.1 版本用于创建 Hadoop 群集。如果在 Azure 经典管理门户中使用“自定义创建”选项，可以从“群集详细信息”页上的“HDInsight 版本”下拉列表中选择要部署的群集版本。

##功能特点
HDInsight 平台的一些突出功能包括：


- **Storm** - Storm on Azure HDInsight 现已正式发布，它你只需单击几下鼠标，就能在数分钟内快速轻松地完成实时部署。Apache Storm on Azure HDInsight 是 Apache Hadoop 生态系统中的开放源代码项目，它提供分析平台，能够可靠地处理数以百万计的事件。现在，Hadoop 用户可以分析发生的事件，以及从过去的事件中获得见解。Microsoft 还提供与 Visual Studio 的内置集成，方便开发人员与 Storm 交互。现在，你可以从 Visual Studio 内部开发、部署和调试 Storm 拓扑。

- **更多的 VM 大小** - 现在，HDInsight 群集支持更多的 VM 类型和大小。HDInsight 群集现在可以利用 A2 到 A7 大小实现常规目的；搭载固态硬盘 (SSD) 和处理器速度提高 60% 的 D 系列节点；支持使用 InfiniBand 加快网络速度的 A8 和 A9 大小。Azure HDInsight 上的 Apache HBase 客户可以受益于 D 系列的更高内存配置和性能。Azure HDInsight 上的 Apache Storm 客户还受益于更大的内存，因此可以加载更大的引用数据集，此外，更快的 CPU 可以提高吞吐量。

- **群集缩放** - 群集缩放使你能够更改正在运行的 HDInsight 群集的节点数，而无需删除或重新创建群集。目前，只有 Hadoop 查询和 Apache Storm 具有此功能，但 Apache HBase 正紧随其后。

- **脚本操作** - 此群集自定义功能可让你使用自定义脚本任意修改 Hadoop 群集。凭借此新功能，用户可以体验如何将 Apache Hadoop 生态系统中的项目部署到 Azure HDInsight 群集。此自定义功能适用于所有类型的 HDInsight 群集，包括 Hadoop、HBase 和 Storm。

- **HBase** - HBase 是一种低延迟的 NoSQL 数据库，可用于对大数据进行联机事务处理。HBase 以集成到 Azure 环境中的托管群集形式提供。这些群集配置为在 Azure Blob 存储中直接存储数据，这样就降低了延迟，使客户在性能与价格方面做出选择时拥有更大的弹性。这样，客户便可构建用于处理大型数据集的交互式 Web 应用，构建用于存储数百万个终结点的传感器数据与遥测数据的服务，并通过 Hadoop 作业来分析这些数据。

- **Apache Phoenix** - Apache Phoenix 是基于 HBase 的结构化查询语言 (SQL) 查询层。它支持有限的 SQL 查询语言规范子集，包括辅助索引支持。它是以客户端中嵌入的 Java 数据库连接 (JDBC) 驱动程序形式交付，面向 HBase 数据的低延迟查询。Apache Phoenix 将获取 SQL 查询，将它编译成一系列 HBase 扫描和协处理器调用，然后生成正则 JDBC 结果集。Apache Phoenix 是基于 HBase 的关系数据库。它是以客户端中嵌入的 JDBC 驱动程序形式交付，面向 HBase 数据的低延迟查询。Apache Phoenix 将获取 SQL 查询，将它编译成一系列 HBase 扫描，并协调这些扫描的运行来生成正则 JDBC 结果集。

- **群集仪表板** - 部署到 HDInsight 群集中的新 Web 应用程序。用于运行 Hive 查询、检查作业日志以及浏览 Azure Blob 存储。用于访问 Web 应用程序的 URL 是 <*ClusterName*>.azurehdinsight.cn。

- **Microsoft Avro Library** - 此库针对 Microsoft.NET 环境实现 Apache Avro 数据序列化系统。Apache Avro 为序列化提供了一种紧凑的二进制数据交换格式。它使用 JavaScript 对象表示法 (JSON) 定义与语言无关的架构，以支持语言互操作性。以一种语言序列化的数据可以用另一种语言读取。目前支持 C、C++、C#、Java、PHP、Python 和 Ruby。Apache Avro 序列化格式在 Azure HDInsight 中广泛使用，用于表示 Hadoop MapReduce 作业内的复杂数据结构。

- **YARN** - 全新的通用分布式应用程序管理框架，已取代了经典 Apache Hadoop MapReduce 框架，用于处理 Hadoop 群集中的数据。它有效地充当 Hadoop 操作系统，并将 Hadoop 从用于批处理的单一用途数据平台转换为实现批处理、交互式处理、联机处理和流式处理的多用途平台。这个新管理框架根据容量保障、公平和服务级别协议 (SLA) 等标准提高了可伸缩性和群集利用率。

- **Tez（仅限 HDInsight 3.1 和更高版本）**- 这是一套通用且可自定义的框架，可在 Hadoop 的大小规模工作负荷中创建简化的数据处理任务。其能够为一项作业的任务执行复杂的定向非循环图形 (DAG)，以便 Apache Hive 和 Apache Pig 等 Apache Hadoop 生态系统中的项目符合人机交互响应时间和极大的 PB 字节规模吞吐量的要求。请注意，Hive 0.13 允许 Hive 查询在 Tez 之上运行，而不是在 MapReduce 上。

- **高可用性 (HA)** - 已将第二个头节点添加到 HDInsight 所部署的 Hadoop 群集，以增加服务的可用性。Hadoop 群集的标准实现通常具有单个头节点。HDInsight 通过添加辅助头节点去除了此单点故障。切换到新 HA 群集配置不会更改群集的价格，除非客户不使用默认大节点而是使用超大头节点设置群集。

- **Hive 性能** - 使用**优化行纵栏表** (ORC) 格式对 Hive 查询响应时间（最多 40 倍）和数据压缩（最多 80%）的数量级改善。

- **Pig、Sqoop、Oozie** - 对 HDInsight 群集版本 3.0 (HDP 2.0/Hadoop 2.2) 进行组件版本升级，以提供 HDInsight 群集版本 2.1 (HDP 1.3/Hadoop 1.2) 的奇偶校验。有关具体细节，请参阅下面的版本表。

- **Mahout** - 这个可缩放的机器学习算法库已预装在 HDInsight 3.1（和更高版本）的 Hadoop 群集上。因此，你无需进行其他任何群集配置，就能运行 Mahout 作业。

- **虚拟网络支持** - HDInsight 群集可用于 Azure 虚拟网络，以支持隔离云资源或将云资源与数据中心资源相链接的混合方案。


## 支持的版本
下表列出当前可用的 HDInsight 版本以及它们使用的相应 Hortonworks 数据平台版本和发布日期。如果知道，还提供其支持到期日期和弃用日期。请注意以下事项：

* 默认情况下，会针对 HDInsight 2.1 和更高版本的群集部署具有两个头节点的高度可用群集。它们不适用于 HDInsight 1.6 群集。
* 某版本的支持到期后，其就不能通过 Azure 经典管理门户提供了。下表列出 Azure 经典管理门户上提供的版本。可继续使用 Windows PowerShell [New-AzureHDInsightCluster](https://msdn.microsoft.com/zh-cn/library/dn947292.aspx) 命令中的 `Version` 参数和 .NET SDK 获取群集版本，直到其弃用日期为止。

<table border="1"> 
<tr><th>HDInsight 版本</th><th>HDP 版本</a><th>高可用性</th></th><th>发行日期</th><th>可在 Azure 经典管理门户中使用</th><th>支持过期日期</th><th>弃用日期</th></tr> 
<tr><td>HDI 3.2</td><td>HDP 2.2</td><td>是</td><td>2/18/2015</td><td>是</td><td></td><td></td></tr> 
<tr><td>HDI 3.1</td><td>HDP 2.1</td><td>是</td><td>6/24/2014</td><td>是</td><td></td><td></td></tr> 
<tr><td>HDI 3.0</td><td>HDP 2.0</td><td>是</td><td>02/11/2014</td><td>是</td><td>09/17/2014</td><td>06/30/2015</td></tr> 
<tr><td>HDI 2.1</td><td>HDP 1.3</td><td>是</td><td>10/28/2013</td><td>否</td><td>05/12/2014</td><td>05/31/2015</td></tr> 
<tr><td>HDI 1.6</td><td>HDP 1.1</td><td>否</td><td>10/28/2013</td><td>否</td><td>04/26/2014</td><td>05/31/2015</td></tr> 
</table><br>

**非默认群集的部署**

默认情况下在 Hadoop 2.4 上创建 HDInsight 3.1 群集，因此用户必须使用经典管理门户中的“自定义创建”选项以指定需要创建的其他 HDInsight 群集版本。

### HDInsight 群集版本的服务级别协议

SLA 用“支持窗口”来定义。“支持窗口”是指 HDInsight 群集版本受 Microsoft 客户服务和支持部门支持的时间段。如果 HDInsight 群集版本具有早于当前日期的**支持过期日期**，则表示它处于支持窗口外。有关支持的 HDInsight 群集版本的列表，请参见上表。给定 HDInsight 版本 X（一旦提供更新的 X+1 版本）的支持到期日期为按以下公式计算所得时间的较晚者：

- 公式 1：发布 HDInsight 群集版本 X 的日期加 180 天。
- 公式 2：HDInsight 群集版本 X+1（X 之后的后续版本）在 Azure 经典管理门户中可用的日期加 90 天。

**弃用日期**是在该日期后，不能在 HDInsight 上创建此群集版本的日期。

> [AZURE.NOTE]HDInsight 2.1 和 3.0 群集均运行在 Azure 来宾 OS [系列 4](/documentation/articles/cloud-services-guestos-update-matrix/) 上，该系列使用 64 位版本的 Windows Server 2012 R2 并支持 .NET Framework 4.0、4.5 和 4.5.1。

## 与 HDInsight 版本相关的 Hortonworks 发行说明##

* HDInsight 群集版本 3.2 使用基于 [Hortonworks 数据平台 2.2][hdp-2-2] 的 Hadoop 分发版。

	* 特定 Apache 组件的发行说明 - [Hive 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310843&version=12326450)、[Pig 0.14](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310730&version=12326954)、[HBase 0.98.4](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310753&version=12326810)、[Phoenix 4.2.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12315120&version=12327581)、[M/R 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310941&version=12327180)、[HDFS 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310942&version=12327181)、[YARN 2.6](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12313722&version=12327197)、[Common](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12310240&version=12327179)、[Tez 0.5.2](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314426&version=12328742)、[Storm 0.9.3](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314820&version=12327112)、[Oozie 4.1.0](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12324960&projectId=12311620)。


* HDInsight 群集版本 3.1 使用基于 [Hortonworks 数据平台 2.1.7][hdp-2-1-7] 的 Hadoop 分发版。这是使用 11/7/2014 之后的 Azure HDInsight 门户时创建的**默认** Hadoop 群集。创建于 11/7/2014 之前的 HDInsight 3.1 群集基于[ Hortonworks 数据平台 2.1.1][hdp-2-1-1]。

* HDInsight 群集版本 3.0 使用基于 [Hortonworks 数据平台 2.0][hdp-2-0-8] 的 Hadoop 分发版。

* HDInsight 群集版本 2.1 使用基于 [Hortonworks 数据平台 1.3][hdp-1-3-0] 的 Hadoop 分发版。

* HDInsight 群集版本 1.6 使用基于 [Hortonworks 数据平台 1.1][hdp-1-1-0] 的 Hadoop 分发版。


[image-hdi-versioning-versionscreen]: ./media/hdinsight-component-versioning/hdi-versioning-version-screen.png

[wa-forums]: /support/forums/

[connect-excel-with-hive-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/

[hdp-2-2]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/bk_HDP_RelNotes/content/ch_relnotes_v220.html

[hdp-2-1-7]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.7-Win/bk_releasenotes_HDP-Win/content/ch_relnotes-HDP-2.1.7.html

[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html

[zookeeper]: http://zookeeper.apache.org/

<!---HONumber=Mooncake_1207_2015-->