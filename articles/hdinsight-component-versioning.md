<properties linkid="manage-services-hdinsight-version" urlDisplayName="HDInsight Hadoop Version" pageTitle="What's new in the cluster versions provided by HDInsight? | Azure" metaKeywords="hdinsight, hadoop, hdinsight hadoop, hadoop azure" description="HDInsight supports multiple Hadoop cluster versions deployable at any time. See the Hadoop and HortonWorks Data Platform (HDP) distribution versions supported." services="HDInsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" title="What's new in the cluster versions provided by HDInsight?" authors="bradsev" />

# HDInsight 提供的群集版本有哪些新功能？

Azure HDInsight 现在通过 HDInsight 群集版本 3.0 支持 Hadoop 2.2，并充分利用这些平台为客户提供一系列显著好处。这些好处包括（最主要是）：

-   **Microsoft Avro Library**：此库针对 Microsoft.NET 环境实现了 Apache Avro 数据序列化系统。Apache Avro 为序列化提供了一种紧凑的二进制数据交换格式。它使用 JSON 定义与语言无关的架构，以支持语言互操作性。以一种语言序列化的数据可以用另一种语言读取。目前支持 C、C++、C\#、Java、PHP、Python 和 Ruby。Apache Avro 序列化格式在 Azure HDInsight 中广泛使用，用于表示 Hadoop MapReduce 作业内的复杂数据结构。

-   **YARN**：一种全新的通用分布式应用程序管理框架，已取代了经典 Apache Hadoop MapReduce 框架，用于处理 Hadoop 群集中的数据。它有效地充当 Hadoop 操作系统，并将 Hadoop 从用于批处理的单一用途数据平台转换为实现批处理、交互式处理、联机处理和流式处理的多用途平台。这个新管理框架根据容量保障、公平和服务级别协议等标准提高了可伸缩性和群集利用率。

-   **高可用性**：已将第二个头节点添加到 HDInsight 所部署的 Hadoop 群集，以增加服务的可用性。Hadoop 群集的标准实现通常具有单个头节点。HDInsight 通过添加辅助头节点去除了此单点故障。切换到新 HA 群集配置不会更改群集的价格，除非客户使用超大头节点设置群集。

-   **Hive 性能**：使用**优化行纵栏表** (ORC) 格式对 Hive 查询响应时间（最多 40 倍）和数据压缩（最多 80%）的数量级改善。

-   **Pig、Sqoop、Qozie、Ambari**：对 HDInsight 群集版本 3.0 (HDP 2.0/Hadoop 2.2) 进行组件版本升级，以提供 HDInsight 群集版本 2.1 (HDP 1.3/Hadoop 1.2) 的奇偶校验。有关具体细节，请参阅下面的版本表。请注意，不包括 HBase、Mahout、Flume。

**部署**
在 Hadoop 2.2 上创建 HDInsight 3.0 群集由 Azure 门户、HDInsight SDK 和 Azure PowerShell 提供支持。请注意，默认情况下在 Hadoop 1.2 上创建 HDInsight 2.1 群集，因此用户必须指定 HDInsight 3.0 群集版本才能创建 Hadoop 2.2 群集。

**全球可用性**
随着在 Hadoop 2.2 上发布 Azure HDInsight，Microsoft 已使 HDInsight 在所有主要 Azure 地区（大中华除外）可用。具体来说，欧洲西部和东南亚数据中心已联机。这使客户能够在距离近且可能位于具有类似合规要求的区域的数据中心内找到群集。

**重大更改**
HDInsight 3.0 群集只支持“wasb://”语法。较早的“asv://”语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不支持，以后的版本将不会支持该语法。这意味着提交到 HDInsight 3.0 群集的任何显式使用“asv://”语法的作业都将会失败。应改用 wasb:// 语法。而且，提交到任何 HDInsight 3.0 群集的作业，如果是使用现有元存储创建的，而该元存储包含对使用 asv:// 语法的资源的显式引用，则这些作业也会失败。这些元存储将需要使用 wasb:// 重新创建以确定资源地址。

## HDInsight 版本

HDInsight 支持多个可随时部署的 Hadoop 群集版本。每个版本选项设置 Hortonworks 数据平台 (HDP) 分发的特定版本和该分发内包含的一组组件。下表中逐项列出了与每个 HDInsight 群集版本关联的组件版本。请注意，[Azure HDInsight][] 使用的默认群集版本当前是 2.1（基于 HDP 1.3）。

<table border="1">
<tr><th>组件</th><th>版本 3.0</th><th>版本 2.1（默认）</th><th>版本 1.6</th></tr>
<tr><td>Hortonworks 数据平台 (HDP)</td><td>2.2.</td><td>1.3</td><td>1.1</td></tr>
<tr><td>Apache Hadoop</td><td>2.2.0</td><td>1.2.0</td><td>1.0.3</td></tr>
<tr><td>Apache Hive</td><td>0.12.0</td><td>0.11.0</td><td>0.9.0</td></tr>
<tr><td>Apache Pig</td><td>0.12.0</td><td>0.11.0</td><td>0.9.3</td></tr>
<tr><td>Apache Sqoop</td><td>1.4.4</td><td>1.4.3</td><td>1.4.2</td></tr>
<tr><td>Apache Oozie</td><td>4.0.0</td><td>3.2.2</td><td>3.2.0</td></tr>
<tr><td>Apache HCatalog</td><td>已与 Hive 合并</td><td>已与 Hive 合并</td><td>0.4.1</td></tr>
<tr><td>Apache Templeton</td><td>已与 Hive 合并</td><td>已与 Hive 合并</td><td>0.1.4</td></tr>
<tr><td>Ambari</td><td>1.4.1</td><td>API 1.0 版</td><td>无版本
</td></tr>
</table>

### 设置 HDInsight 群集时选择一个版本

通过 HDInsight PowerShell Cmdlet 或 HDInsight .NET SDK 创建群集时，你可以使用“Version”参数选择 HDInsight Hadoop 群集的版本。

如果你使用“快速创建” 选项，则在默认情况下将得到 HDInsight Hadoop 群集的 2.1 版本。如果在 Azure 门户中使用“自定义创建” 选项，可以从“群集详细信息” 页上的“HDInsight 版本” 下拉列表中选择要部署的群集版本。3.0 版的 HDInsight Hadoop 群集只在“自定义创建” 向导中作为选项提供。

![HDI.Versioning.VersionScreen][]

## 支持的版本

下表列出当前可用的 HDInsight 版本以及它们使用的相应 Hortonworks 数据平台 (HDP) 版本和发布日期。如果知道，还提供它们的弃用日期。默认情况下，会针对 HDInsight 2.1 和 3.0 群集部署具有两个头节点的高度可用群集。它们不适用于 HDInsight 1.6 群集。

<table border="1">
<tr><th>HDInsight 版本</th><th>HDP 版本</a><th>高可用性</th></th><th>发布日期</th><th>支持到期日期</th><th>弃用日期</th></tr>
<tr><td>HDI 3.0</td><td>HDP 2.0</td><td>是</td><td>02/11/2014</td><td>08/11/2014</td><td></td></tr>
<tr><td>HDI 2.1</td><td>HDP 1.3</td><td>是</td><td>10/28/2013</td><td>05/12/2014</td><td>05/01/2015</td></tr>
<tr><td>HDI 1.6</td><td>HDP 1.1</td><td>否</td><td>10/28/2013</td><td>04/28/2014</td><td>05/01/2014</td></tr>
</table>

### HDInsight 群集版本的服务级别协议 (SLA)

SLA 用“支持窗口”来定义。“支持窗口”是指 HDInsight 群集版本受 Microsoft 客户支持部门支持的时间段。如果 HDInsight 群集版本的**支持到期日期**早于当前日期，则表示该 HDInsight 群集在支持窗口外。有关支持的 HDInsight 群集版本的列表，请参见上表。给定 HDInsight 版本（用版本 X 表示）的支持到期日期为按以下公式计算所得时间的较晚者：

-   公式 1：发布 HDInsight 群集版本 X 的日期加 180 天
-   公式 2：HDInsight 群集版本 X+1（X 之后的后续版本）在 Azure 管理门户中可用的日期加 90 天。

**弃用日期**是在该日期后，不能在 HDInsight 上创建此群集版本的日期。

> [WACOM.NOTE] HDInsight 2.1 和 3.0 群集均运行在 Azure 来宾 OS [系列 4][]上，该系列使用 64 位版本的 Windows Server 2012 R2 并支持 .NET Framework 4.0、4.5 和 4.5.1。

### 有关版本控制的其他说明和信息

-   SQL Server JDBC 驱动程序由 HDInsight 在内部使用，不用于外部操作。如果你希望使用 ODBC 连接到 HDInsight，请使用 Microsoft Hive ODBC 驱动程序。有关使用 Hive ODBC 的详细信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][]。

-   HDInsight 服务使用的端口已更改。以前所用的端口号在 Windows OS 临时端口范围内。端口是从预定义的临时范围自动分配的，该范围适用于基于 Internet 协议的短期通信。新的一组允许的 HDP 服务端口号现已在此范围外，目的是避免遇到头节点上运行的服务所使用的端口时出现冲突。新端口号不会导致任何重大更改。现在使用的端口号如下所示：

**HDP1.1**

<table border="1"><br /><tr><th>名称</th><th>值</th></tr><br /><tr><td>dfs.http.address</td><td>namenodehost:30070</td></tr><br /><tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr><br /><tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr><br /><tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr><br /><tr><td>dfs.secondary.http.address</td><td>0.0.0.0:30090</td></tr><br /><tr><td>mapred.job.tracker.http.address</td><td>jobtrackerhost:30030</td></tr><br /><tr><td>mapred.task.tracker.http.address</td><td>0.0.0.0:30060</td></tr><br /><tr><td>mapreduce.history.server.http.address</td><td>0.0.0.0:31111</td></tr><br /><tr><td>templeton.port</td><td>30111</td></tr><br /></table></p>
<p><strong>HDP2.0 和 2.1</strong><br /><table border="1"><br /><tr><th>名称</th><th>值</th></tr><br /><tr><td>dfs.namenode.http-address</td><td>namenodehost:30070</td></tr><br /><tr><td>dfs.namenode.https-address</td><td>headnodehost:30470</td></tr><br /><tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr><br /><tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr><br /><tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr><br /><tr><td>dfs.namenode.secondary.http-address</td><td>0.0.0.0:30090</td></tr><br /><tr><td>yarn.nodemanager.webapp.address</td><td>0.0.0.0:30060</td></tr><br /><tr><td>templeton.port</td><td>30111</td></tr><br /></table></p>

-   HDInsight 群集版本 3.0 使用基于 [Hortonworks 数据平台 2.0][] 的 Hadoop 分发。

-   HDInsight 群集版本 2.1 使用基于 [Hortonworks 数据平台 1.3][] 的 Hadoop 分发。这是在使用 Azure HDInsight 门户时创建的默认 Hadoop 群集。

-   HDInsight 群集版本 1.6 使用基于 [Hortonworks 数据平台 1.1][] 的 Hadoop 分发。

-   与 HDInsight 群集版本关联的组件版本可能在将来的 HDInsight 更新中更改。确定可用组件并验证正在使用哪些群集版本的一种方法是使用 Ambari REST API。GetComponentInformation 命令可用于检索有关服务组件的信息。有关详细信息，请参阅 [Ambari 文档][]。获取此信息的另一个方法是使用远程桌面登录到群集并直接检查“C:\\apps\\dist”目录的内容。

  [Azure HDInsight]: http://go.microsoft.com/fwlink/?LinkID=285601
  [HDI.Versioning.VersionScreen]: ./media/hdinsight-component-versioning/hdi-versioning-version-screen.png
  [系列 4]: http://msdn.microsoft.com/zh-cn/library/azure/ee924680.aspx#explanation
  [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]: /en-us/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver
  [Hortonworks 数据平台 2.0]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html
  [Hortonworks 数据平台 1.3]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html
  [Hortonworks 数据平台 1.1]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html
  [Ambari 文档]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md
