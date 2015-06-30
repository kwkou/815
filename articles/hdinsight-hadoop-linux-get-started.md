<properties 
   pageTitle="在 Linux 上的 HDInsight 中开始将 Hadoop 与 Hive 配合使用 | Azure" 
   description="在 Linux 上的 HDInsight 中开始使用 Hadoop。了解如何设置在 Linux 上运行的 HDInsight Hadoop 群集，并使用 Hive 查询数据" 
   services="hdinsight" 
   documentationCenter="" 
   authors="nitinme" 
   manager="paulettm" 
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="03/13/2015"
    wacn.date="04/15/2015"
    />


# 在 Linux 上的 HDInsight 中开始将 Hadoop 与 Hive 配合使用（预览版）

本教程演示如何在 Linux 上设置 HDInsight Hadoop 群集并运行 Hive 查询以从非结构化数据中提取有用的信息，让你快速地在 Linux 上开始使用 HDInsight。然后，你将在商业智能 (BI) 工具（例如 Tableau）中分析结果。


> [AZURE.NOTE] 如果你是 Hadoop 和大数据的新手，可以进一步了解这些术语：<a href="http://hadoop.apache.org/" target="_blank">Apache Hadoop</a>、<a href="http://wiki.apache.org/hadoop/MapReduce" target="_blank">MapReduce</a>、<a href="http://hadoop.apache.org/docs/r1.0.4/hdfs_design.html" target="_blank">HDFS</a> 和 <a href="https://cwiki.apache.org/confluence/display/Hive/Home%3bjsessionid=AF5B37E667D7DBA633313BB2280C9072" target="_blank">Hive</a>。若要了解 HDInsight 如何在 Azure 中启用 Hadoop，请参阅 [HDInsight 中的 Hadoop 简介](/documentation/articles/hdinsight-hadoop-introduction)。


## 本教程的目标是什么？ ##

假设你具有一个大型非结构化数据集，想要对其运行查询以提取一些有意义的信息。下面说明了如何实现此目标：

   !["Get started using Hadoop with Hive in HDInsight" tutorial steps illustrated: create an account; provision a cluster; query data; and analyze in Tableau.](./media/hdinsight-hadoop-linux-get-started/HDI.Linux.GetStartedFlow.png)


**先决条件：**

在开始阅读本教程前，你必须具有：


- Azure 订阅。有关获取订阅的详细信息，请参阅<a href="/pricing/overview/" target="_blank">购买选项</a>或<a href="/pricing/1rmb-trial/" target="_blank">试用</a>。

**预计完成时间：**30 分钟

## 本教程的内容

* [创建 Azure 存储帐户](#storage)
* [设置 HDInsight Linux 群集](#provision)
* [提交 Hive 作业](#hivequery)
* [后续步骤](#nextsteps)

## <a name="storage"></a>创建 Azure 存储帐户

HDInsight 使用 Azure Blob 存储来存储数据。该 Blob 称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-use-blob-storage)。

你设置 HDInsight 群集时，指定 Azure 存储帐户。将该帐户的一个特定 Blob 存储容器指定为默认文件系统，如同在 HDFS 中一样。默认情况下，HDInsight 群集与你指定的存储帐户设置在同一数据中心内。

除此存储帐户外，你在自定义配置 HDInsight 群集时还可以添加其他存储帐户。附加的此存储帐户可以来自同一 Azure 订阅，也可以来自不同的 Azure 订阅。有关说明，请参阅[使用自定义选项设置 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters)。 

为简化本教程，仅使用了默认 blob 容器和默认存储帐户。而在实践中，数据文件通常存储在指定的存储帐户中。

**创建 Azure 存储帐户**

1. 登录到 <a href="https://manage.windowsazure.cn/" target="_blank">Azure 管理门户</a>。
2. 单击左下角的"新建"，指向"数据服务"，指向"存储"，然后单击"快速创建"。

	![Azure portal where you can use Quick Create to set up a new storage account.](./media/hdinsight-hadoop-linux-get-started/HDI.StorageAccount.QuickCreate.png)

3. 输入"URL"、"位置"和"复制"，然后单击"创建存储帐户"。不支持地缘组。你将在存储列表中看到新的存储帐户。

	>[AZURE.NOTE]  用于设置 HDInsight Linux 群集的快速创建选项与我们在本教程中使用的一样，在设置群集时并不要求提供位置。默认情况下，它将群集与存储帐户一起放在同一数据中心。因此，确保在群集支持的位置中创建存储帐户，这些位置包括：**中国东部**、**中国北部**。

4. 等到新存储帐户的"状态"更改为"联机"。
5. 从列表中选择新存储帐户，然后单击页面底部的"管理访问密钥"。
7. 记下"存储帐户名称"和"主访问密钥"（或"辅助访问密钥"。任一密钥都有效）。本教程后面的步骤中将会用到它们。


有关详细信息，请参阅[如何创建存储帐户](/documentation/articles/storage-create-storage-account) 和[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-use-blob-storage)。
	
## <a name="provision"></a>在 Linux 上设置 HDInsight 群集

当你设置 HDInsight 群集时，便设置了包含 Hadoop 和相关应用程序的 Azure 计算资源。在本部分中，你将使用快速创建选项在 Linux 上设置 HDInsight 群集。此选项使用默认用户名和 Azure 存储容器，并使用在 Ubuntu 12.04 LTS 上运行的 HDInsight 3.2 版（Hadoop 2.5 版、HDP 2.2 版）配置群集。有关不同 HDInsight 版本及其 SLA 的信息，请参阅 [HDInsight 组件版本](/documentation/articles/hdinsight-component-versioning) 页面。

>[AZURE.NOTE]  你还可以创建运行 Windows Server 操作系统的 Hadoop 群集。有关说明，请参阅 [Windows 上的 HDInsight 入门](/documentation/articles/hdinsight-get-started)。


**设置 HDInsight 群集**

1. 登录到 <a href="https://manage.windowsazure.cn/" target="_blank">Azure 管理门户</a>。 

2. 单击左下角的"新建"，然后依次单击"数据服务"、"HDInsight"和"Linux 上的 Hadoop"。

	![Creation of a Hadoop cluster in HDInsight.](./media/hdinsight-hadoop-linux-get-started/HDI.QuickCreateCluster.png)

4. 输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td>群集名称</td><td>群集的名称</td></tr>
	<tr><td>群集大小</td><td>要部署的数据节点的数目。默认值为 4。但是，下拉菜单还提供了使用 1 或 2 个数据节点的选项。可以使用"自定义创建"选项指定任意数量的群集节点<strong></strong>。提供了有关各种群集大小的计帐费率的定价详细信息。单击下拉框正上方的 <strong>?</strong> 符号在弹出窗口中打开链接。</td></tr>
	<tr><td>密码</td><td><i>HTTP</i> 帐户（默认用户名：admin）和 <i>SSH</i> 帐户（默认用户名：hdiuser）的密码。请注意，这不是设置群集所在的 VM 的管理员帐户。 </td></tr>
	
	<tr><td>存储帐户</td><td>从下拉框中选择你创建的存储帐户。 <br/>

	一旦选择存储帐户，就不能更改它。如果删除了存储帐户，则群集将不再可用。

	HDInsight 群集与存储帐户共同位于同一数据中心中。 
	</td></tr>
	</table>

	保留群集名称的副本。本教程后面的步骤中将会用到它。

	
5. 单击"创建 HDInsight 群集"。完成设置后，状态列将显示"正在运行"。

	>[AZURE.NOTE] 上述过程会使用快速创建选项创建 Linux 群集，该选项使用默认的 SSH 用户名和 Azure 存储容器。若要使用自定义选项创建群集，例如使用 SSH 密钥进行身份验证或使用其他存储账户，请参阅[使用自定义选项设置 HDInsight Linux 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters)。


## <a name="hivequery"></a>在群集上提交 Hive 作业
现在，你已设置 HDInsight Linux 群集，下一步是运行示例 Hive 作业，以查询 HDInsight 群集随附的示例数据 (sample.log)。示例数据包含日志信息，例如跟踪、警告、信息、错误，等等。我们将查询此数据，以检索具有特定严重性的所有错误日志。你必须执行以下步骤，以在 HDInsight Linux 群集上运行 Hive 查询。

- 连接到 Linux 群集
- 运行 HIVE 作业



### 连接到群集

可以从 Linux 计算机或 Windows 计算机使用 SSH 连接到 Linux 上的 HDInsight 群集。

**从 Linux 计算机连接**

1. 打开终端并输入以下命令：

		ssh <username>@<clustername>-ssh.azurehdinsight.cn

	由于群集是使用快速创建选项设置的，默认的 SSH 用户名为 **hdiuser**。因此，该命令必须是

		ssh hdiuser@myhdinsightcluster-ssh.azurehdinsight.cn

2. 出现提示时，输入你在设置群集时所提供的密码。成功连接后，提示将更改为：

		hdiuser@headnode-0:~$


**从 Windows 计算机连接**

1. 下载适用于 Windows 客户端的 **PuTTY**。可以从 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html</a> 获取

2. 打开 **PuTTY**。在"类别"中，单击"会话"。在"PuTTY 会话的基本选项"屏幕中，将 HDInsight 服务器的 SSH 地址输入到"主机名(或 IP 地址)"字段中。SSH 地址是群集名称后接**-ssh.azurehdinsight.cn**。例如 **myhdinsightcluster-ssh.azurehdinsight.cn**。

	![Connect to an HDInsight cluster on Linux using PuTTY](./media/hdinsight-hadoop-linux-get-started/HDI.linux.connect.putty.png)

3. 若要保存连接信息供日后使用，请在"保存的会话"下面输入此连接的名称，然后单击"保存"。该连接将会添加到已保存会话的列表中。

4. 单击"打开"以连接到群集。在提示输入用户名时，请输入  *hdiuser*。至于密码，请输入你在设置群集时指定的密码。成功连接后，提示将更改为：

		hdiuser@headnode-0:~$

### 运行 Hive 作业

使用 SSH 连接到群集后，请使用以下命令来运行 Hive 查询。

1. 在提示符下使用以下命令启动 Hive 命令行界面 (CLI)：

		hive

2. 在 CLI 中输入以下语句，以使用群集上已有的示例数据创建名为 log4jLogs 的新表。

		DROP TABLE log4jLogs;
		CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) 
		ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' 
		STORED AS TEXTFILE LOCATION 'wasb:///example/data/';
		SELECT t4 AS sev, COUNT(*) AS cnt FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;

	这些语句将执行以下操作。

	- **DROP TABLE** - 删除表和数据文件（如果表已存在）。
	- **CREATE EXTERNAL TABLE** - 在 Hive 中创建新的"外部"表。外部表只会在 Hive 中存储表定义；数据会保留在原始位置。
	- **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。
	- **STORED AS TEXTFILE LOCATION** - 让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。
	- **SELECT** - 选择其列 t4 包含值 [ERROR] 的所有行计数。

	>[AZURE.NOTE] 当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。删除外部表**不会**删除数据，只会删除表定义。

	这将返回以下输出。

		Query ID = hdiuser_20150116000202_cceb9c6b-4356-4931-b9a7-2c373ebba493
		Total jobs = 1
		Launching Job 1 out of 1
		Number of reduce tasks not specified. Estimated from input data size: 1
		In order to change the average load for a reducer (in bytes):
		  set hive.exec.reducers.bytes.per.reducer=<number>
		In order to limit the maximum number of reducers:
		  set hive.exec.reducers.max=<number>
		In order to set a constant number of reducers:
		  set mapreduce.job.reduces=<number>
		Starting Job = job_1421200049012_0006, Tracking URL = <URL>:8088/proxy/application_1421200049012_0006/
		Kill Command = /usr/hdp/2.2.1.0-2165/hadoop/bin/hadoop job  -kill job_1421200049012_0006
		Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
		2015-01-16 00:02:40,823 Stage-1 map = 0%,  reduce = 0%
		2015-01-16 00:02:55,488 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 3.32 sec
		2015-01-16 00:03:05,298 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 5.62 sec
		MapReduce Total cumulative CPU time: 5 seconds 620 msec
		Ended Job = job_1421200049012_0006
		MapReduce Jobs Launched: 
		Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 5.62 sec   HDFS Read: 0 HDFS Write: 0 SUCCESS
		Total MapReduce CPU Time Spent: 5 seconds 620 msec
		OK
		[ERROR]    3
		Time taken: 60.991 seconds, Fetched: 1 row(s)

	请注意，输出包含 **[ERROR]  3**，因为有三个行包含此值。

3. 使用以下语句创建名为 **errorLogs** 的新"内部"表。

		CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
		INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';


	这些语句将执行以下操作。

	- **CREATE TABLE IF NOT EXISTS** - 创建表（如果尚不存在）。由于未使用 EXTERNAL 关键字，因此这是一个 "内部"表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。与 **EXTERNAL** 表不同，删除内部表会同时删除基础数据。
	- **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
	- **INSERT OVERWRITE ...SELECT** - 从包含 [ERROR] 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表

4. 若要验证是否只将列 t4 中包含 **[ERROR]** 的行存储到了 **errorLogs** 表，请使用以下语句从 errorLogs 返回所有行。

		SELECT * from errorLogs;

	控制台上应显示以下输出。

		2012-02-03	18:35:34	SampleClass0	[ERROR]	 incorrect		id
		2012-02-03	18:55:54	SampleClass1	[ERROR]	 incorrect		id
		2012-02-03	19:25:27	SampleClass4	[ERROR]	 incorrect		id
		Time taken: 0.987 seconds, Fetched: 3 row(s)

	返回的数据应该全都对应于 [ERROR] 日志。


## <a name="nextsteps"></a>后续步骤
在本教程中，你已了解如何使用 HDInsight 设置 Hadoop Linux 群集并使用 SSH 在群集上运行 Hive 查询。若要了解更多信息，请参阅以下文章：

- [使用自定义选项在 Linux 上设置 HDInsight](/documentation/articles/hdinsight-hadoop-provision-linux-clusters)
- [使用 Linux 上的 HDInsight](/documentation/articles/hdinsight-hadoop-linux-information)
- [使用 Ambari 管理 HDInsight 群集](/documentation/articles/hdinsight-hadoop-manage-ambari)
- [将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-use-blob-storage)
- [将数据上载到 HDInsight][hdinsight-upload-data]


[1]: /documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

[powershell-download]: http://go.microsoft.com/?linkid=9811175&clcid=0x409
[powershell-install-configure]: /documentation/articles/install-configure-powershell/
[powershell-open]: /documentation/articles/install-configure-powershell/#Install

[img-hdi-dashboard]: ./media/hdinsight-get-started/HDI.dashboard.png
[img-hdi-dashboard-query-select]: ./media/hdinsight-get-started/HDI.dashboard.query.select.png
[img-hdi-dashboard-query-select-result]: ./media/hdinsight-get-started/HDI.dashboard.query.select.result.png
[img-hdi-dashboard-query-select-result-output]: ./media/hdinsight-get-started/HDI.dashboard.query.select.result.output.png
[img-hdi-dashboard-query-browse-output]: ./media/hdinsight-get-started/HDI.dashboard.query.browse.output.png
[image-hdi-clusterstatus]: ./media/hdinsight-get-started/HDI.ClusterStatus.png
[image-hdi-gettingstarted-powerquery-importdata]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData.png
[image-hdi-gettingstarted-powerquery-importdata2]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData2.png

<!--HONumber=50-->