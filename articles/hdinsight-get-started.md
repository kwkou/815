<properties linkid="manage-services-hdinsight-get-started-hdinsight-hadoop" urlDisplayName="Get Started" pageTitle="开始在 HDInsight 中使用 Hadoop | Azure" metaKeywords="" description="Get started using Hadoop in HDInsight, a big data solution. Learn how to provision clusters, run hive jobs, and output data to Excel for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using Hadoop in HDInsight" authors="nitinme" solutions="big-data" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/14/2014" ms.author="nitinme" />


# 开始在 HDInsight 中使用 Hadoop

<!--div class="dev-center-tutorial-selector sublanding">
<a href="../hdinsight-get-started" title="开始在 HDInsight 中使用 Hadoop 2.4" class="current">Hadoop 2.4</a>
<a href="../hdinsight-get-started-30" title="Get started using Hadoop 2.2 in HDInsight">Hadoop 2.2</a>
<!--a href="../hdinsight-get-started-21" title="Get started using Hadoop 1.2 in HDInsight">Hadoop 1.2</a>
</div-->

HDInsight 使 Apache Hadoop（一种 MapReduce 软件框架）可用于更简单、缩放性更高且更经济高效的 Azure 环境。HDInsight 还提供一种经济高效的方法使用 Azure Blob 存储来管理和存储数据。

> [WACOM.NOTE]如果你不了解 Hadoop 和大数据，那你可能需要阅读有关术语 [Apache Hadoop][apache-hadoop]、[MapReduce][apache-mapreduce]、[HDFS][apache-hdfs]和 [Hive][apache-hive]的更多信息。若要了解 HDInsight 如何在 Azure 中启用 Hadoop，请参阅 [HDInsight 中的 Hadoop 简介][hadoop-hdinsight-intro]。

Microsoft 还提供了 HDInsight Emulator for Azure（以前称作 *Microsoft HDInsight 开发者预览版*），与 Azure HDInsight 的通用版本结合使用。Emulator 针对开发人员方案，仅支持单节点部署。有关如何使用 HDInsight Emulator 的信息，请参见 [HDInsight Emulator 入门][hdinsight-emulator]。

> [WACOM.NOTE]有关如何设置 HBase 群集的说明，请参阅[在 HDInsight 中设置 HBase 群集][hdinsight-hbase-custom-provision]。请参阅 <a href="http://go.microsoft.com/fwlink/?LinkId=510237">Hadoop 与 HBase 之间的差别</a>，以了解为何要选择其中的某一种群集。   

## 本教程实现的目标是什么？ ##

假设你拥有一个大型非结构化数据集，想要对其运行查询以提取一些有意义的信息。这正是我们在本教程中将要实现的目标。以下内容说明如何实现此目标：

   ![HDI.GetStartedFlow][image-hdi-getstarted-flow]

<!--
You can also watch a demo video of this tutorial:

<center><iframe width="560" height="315" src="http://www.youtube.com/embed/v=Y4aNjnoeaHA?list=PLDrz-Fkcb9WWdY-Yp6D4fTC1ll_3lU-QS" frameborder="0" allowfullscreen></iframe></center>-->

<!--center><a href="https://www.youtube.com/watch?v=Y4aNjnoeaHA&list=PLDrz-Fkcb9WWdY-Yp6D4fTC1ll_3lU-QS" target = "_blank">![HDI.getstarted.video][img-hdi-getstarted-video]</a></center-->



**先决条件：**

在开始阅读本教程前，你必须具有：


- Azure 订阅。有关获取订阅的详细信息，请参阅[购买选项][azure-purchase-options]或[免费试用][azure-free-trial]。
- 装有 Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus 的计算机。

**估计完成时间：** 30 分钟

## 本教程的内容

* [创建 Azure 存储帐户](#storage)
* [设置 HDInsight 群集](#provision)
* [从门户运行示例](#sample)
* [运行 HIVE 作业](#hivequery)
* [后续步骤](#nextsteps)

## <a name="storage"></a>创建 Azure 存储帐户

HDInsight 使用 Azure Blob 存储来存储数据。它称为 *WASB* 或 *Azure 存储 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关更多信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

在设置 HDInsight 群集时，指定 Azure 存储帐户。将该帐户的一个特定 Blob 存储容器指定为默认文件系统，如同在 HDFS 中一样。默认情况下，HDInsight 群集与你指定的存储帐户设置在同一数据中心内。

除此存储帐户外，你在自定义配置 HDInsight 群集时还可以添加其他存储帐户。附加的此存储帐户可以来自同一 Azure 订阅，也可以来自不同的 Azure 订阅。有关说明，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。 

为简化本教程，仅使用了默认 blob 容器和默认存储帐户。在实践中，数据文件通常存储在指定的存储帐户中。

**创建 Azure 存储帐户**

1. 登录到 [Azure 管理门户][azure-management-portal]。
2. 单击左下角的**新建**、指向**数据服务**、指向**存储**，然后单击**快速创建**。

	![HDI.StorageAccount.QuickCreate][image-hdi-storageaccount-quickcreate]

3. 进入 **URL**、**位置**和**复制**，然后单击**创建存储帐户**。不支持地缘组。你将在存储列表中看到新的存储帐户。

	>[WACOM.NOTE]  用于设置 HDInsight 群集的快速创建选项与我们在本教程中使用的一样，在设置群集时并不要求提供位置。相反，它默认将群集与存储帐户共同放在同一数据中心中。因此，确保在群集支持的位置中创建存储帐户，这些位置包括：**中国东部**、**中国北部**、**亚洲东部**、**亚洲东南部**、**欧洲北部**、**欧洲西部**、**美国东部**、**美国西部**、**美国中北部**和**美国中南部**。

4. 等待，直到新存储帐户的**状态**更改为**联机**。
5. 从列表中选择新存储帐户，然后单击页面底部的**管理访问密钥**。
7. 记下**存储帐户名称**和**主访问密钥**（或**辅助访问密钥**，任一密钥都有效）。本教程后面的步骤中将会用到它们。


有关更多信息，请参见
[如何创建存储帐户][azure-create-storageaccount]和[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。
	
## <a name="provision"></a>设置 HDInsight 群集

当你设置 HDInsight 群集时，便设置了包含 Hadoop 和相关应用程序的 Azure 计算资源。在此部分中，设置基于 Hadoop 版本 2.4 的 HDInsight 群集版本 3.1。你还可以使用 Azure 门户、HDInsight PowerShell cmdlet 或 HDInsight .NET SDK 为其他版本创建 Hadoop 群集。有关说明，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。有关不同 HDInsight 版本及其 SLA 的信息，请参阅 [HDInsight 组件版本](http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-component-versioning/)页面。

[WACOM.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]


**设置 HDInsight 群集** 

1. 登录到 [Azure 管理门户][azure-management-portal]。 

2. 单击左侧的 **HDInsight** 以列出帐户中群集的状态。在下面的屏幕快照中，没有现有的 HDInsight 群集。

	![HDI.ClusterStatus][image-hdi-clusterstatus]

3. 单击左下方的**新建**、单击**数据服务**、单击 **HDInsight**，然后单击 **Hadoop**。

	![HDI.QuickCreateCluster][image-hdi-quickcreatecluster]

4. 输入或选择下列值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td>群集名称</td><td>群集的名称</td></tr>
	<tr><td>群集大小</td><td>要部署的数据节点数。默认值为 4。但是，下拉菜单还提供了使用 1 或 2 个数据节点的选项。通过使用<strong>自定义创建</strong>选项，可以指定任何数量的群集节点。提供了有关各种群集大小的计帐费率的定价详细信息。单击下拉框正上方的 <strong>?</strong> 符号并访问弹出框上的链接。</td></tr>
	<tr><td>密码</td><td><i>admin</i>帐户的密码。如果使用的不是<strong>自定义创建</strong>选项，则指定群集用户名"admin"。请注意，这不是设置其群集的 VM 的 Windows Administrator 帐户。帐户名称可以通过使用<strong>自定义创建</strong>向导来更改。</td></tr>
	<tr><td>存储帐户</td><td>从下拉框中选择你所创建的存储帐户。 <br/>

	一旦选择了存储帐户，就不能更改它。如果删除了存储帐户，则群集将不再可用。

	HDInsight 群集与存储帐户共同位于同一数据中心中。 
	</td></tr>
	</table>

	保留群集名称的副本。本教程后面的步骤中将会用到它。

	
5. 单击**创建 HDInsight 群集**。完成设置后，状态列将显示**正在运行**。

	>[WACOM.NOTE] 上述过程通过 HDInsight 群集版本 3.1 来创建群集。若要创建其他群集版本，请使用管理门户中的自定义创建方法，或使用 Azure PowerShell。有关各群集版本之间的差异的信息，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]有关使用**自定义创建**选项的信息，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。


## <a name="sample"></a>从门户运行示例

成功设置的 HDInsight 群集提供查询控制台以直接从门户运行示例。通过浏览一些基本方案，你可以使用示例了解如何使用 HDInsight。这些示例提供所有必要组件，比如要分析的数据和要对数据运行的查询。 

**若要运行示例**，请从 Azure 管理门户单击你想要运行示例的群集名称，然后单击页面底部的**查询控制台**。从打开的网页中，单击**入门库**选项卡，然后在**示例**类别下，单击要运行的示例。按照网页上的说明完成示例。若要详细了解各示例的功能，请单击下面的链接。

示例 | 示例的作用是什么？
------ | ---------------
[传感器数据分析][hdinsight-sensor-data-sample] | 了解如何使用 HDInsight 处理加热、通风和空调 (HVAC) 系统产生的历史数据，以识别无法可靠地保持设定温度的系统
[网站日志分析][hdinsight-weblogs-sample] | 了解如何使用 HDInsight 分析网站日志文件，以了解一天中从外部网站对该网站的访问次数，以及用户遇到的网站错误汇总


## <a name="hivequery"></a>从门户运行 HIVE 查询
现在，你的 HDInsight 群集已设置完毕，下一步是运行 Hive 作业以查询 HDInsight 群集随附的示例 Hive 表 *hivesampletable*。该表包含有关移动设备制造商、平台和型号的数据。我们查询该表以按特定制造商检索移动设备的数据。

**从群集仪表板运行 Hive 作业**

1. 登录到 [Azure 管理门户][azure-management-portal]。 
2. 从左窗格单击 **HDINSIGHT**。你将会看到所创建的群集的列表，包括你刚刚在上一部分中创建的群集。
3. 单击要运行 Hive 作业的群集名称，然后单击页面底部的**查询控制台**。 
4. 这会在另一个浏览器选项卡中打开一个网页。输入 Hadoop 用户帐户和密码。默认用户名为 **admin**；密码是你在设置群集时所输入的密码。仪表板类似于：

	![hdi.dashboard][img-hdi-dashboard]

	顶部有多个选项卡。默认选项卡为 **Hive 编辑器**，而其他选项卡为**作业历史记录**和**文件浏览器**。使用仪表板可以提交 Hive 查询、检查 Hadoop 作业日志，以及浏览 WASB 文件。

	> [WACOM.NOTE] 请注意，网页的 URL 为 *&lt;ClusterName&gt;.azurehdinsight.cn*。因此，如果不从管理门户打开仪表板，也可以在 Web 浏览器中使用 URL 打开仪表板。

6. 在 **Hive 编辑器**选项卡上，为**查询名称**输入 **HTC20**。查询名称为作业标题。

7. 在查询窗格中输入以下查询： 

		SELECT * FROM hivesampletable
			WHERE devicemake LIKE "HTC%"
			LIMIT 20;

	![hdi.dashboard.query.select][img-hdi-dashboard-query-select]

4. 单击**提交**。片刻之后即可返回结果。屏幕将每隔 30 秒刷新一次。你也可以单击**刷新**来刷新屏幕。

    完成后，屏幕将如下所示：

	![hdi.dashboard.query.select.result][img-hdi-dashboard-query-select-result]

5. 单击屏幕上的查询名称以查看输出。记下**作业开始时间 (UTC)**。稍后需要用到此值。 

    ![hdi.dashboard.query.select.result.output][img-hdi-dashboard-query-select-result-output]

    该页面还显示**作业输出**和**作业日志**。你也可以选择下载输出文件 (\_stdout) 和日志文件 \(_stderr)。


	> [WACOM.NOTE] 只要停留在 **Hive 编辑器**选项卡上，该选项卡上的**作业会话**表便会列出已完成或正在运行的作业。如果离开该页面，该表不会列出任何作业。**作业历史记录**选项卡保留所有作业（已完成或正在运行）的列表。
 

**浏览到输出文件**

1. 从群集仪表板中，单击顶部的**文件浏览器**。 
2. 单击你的存储帐户名称、单击容器名称（与群集名称相同），然后单击**用户**。
3. 单击 **admin**，然后单击其上次修改时间比你前面记下的作业开始时间稍晚的 GUID。记下此 GUID。你在下一部分中需要该 GUID。


   	![hdi.dashboard.query.browse.output][img-hdi-dashboard-query-browse-output]


### <a name="powerquery"></a>连接到 Microsoft 商业智能工具 

你可以使用 Microsoft Excel 的 Power Query 附加组件将作业输出从 HDInsight 导入到 Excel 中，从中可以使用 Microsoft 商业智能 (BI) 工具进一步分析结果。 

为完成本部分教程，您必须安装了 Excel 2010 或 2013。 

**下载 Microsoft Power Query for Excel**

从 - Microsoft 下载中心[下载 Microsoft Power Query for Microsoft Excel ](http://www.microsoft.com/zh-cn/download/details.aspx?id=39379) 并进行安装。

**导入 HDInsight 数据**

1. 打开 Excel，然后创建新的空白工作簿。
3. 依次单击 **Power Query** 菜单，**来自其他源**和**来自 Azure HDInsight**。

	![HDI.GettingStarted.PowerQuery.ImportData][image-hdi-gettingstarted-powerquery-importdata]

3. 输入与群集关联的 Azure Blob 存储帐户的**帐户名称**，然后单击**确定**。这是先前在教程中创建的存储帐户。
4. 输入 Azure Blob 存储帐户的**帐户密钥**，然后单击**保存**。 
5. 在右侧的"导航器"窗格中，双击 Blob 存储容器名称。默认情况下，该容器名称与群集名称相同。 

6. 在**名称**列中找到 **stdout**。确认对应**文件夹路径**列中的 GUID 是否与你前面记下的 GUID 匹配。若匹配，则表明输出数据与所提交的作业相对应。单击 **stdout** 左侧的**二进制**。

	![HDI.GettingStarted.PowerQuery.ImportData2][image-hdi-gettingstarted-powerquery-importdata2]

9. 单击左上角的**关闭并加载**以将 Hive 作业输出导入到 Excel 中。


## <a name="nextsteps"></a>后续步骤
在本教程中，您已了解了如何使用 HDInsight 设置群集、对其运行 MapReduce 作业，以及如何将结果导入到 Excel 中，在 Excel 中，可以使用商业智能工具进一步处理结果以及以图形方式显示结果。若要了解更多信息，请参阅下列文章：

- [HDInsight Emulator 入门][hdinsight-emulator]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上载到 HDInsight][hdinsight-upload-data]
- [将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]
- [为 HDInsight 开发 C# Hadoop 流程序][hdinsight-develop-streaming]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]

[hdinsight-versions]: ../hdinsight-component-versioning/

[hdinsight-get-started-30]: ../hdinsight-get-started-30/
[hdinsight-provision]: ../hdinsight-provision-clusters/
[hdinsight-admin-powershell]: ../hdinsight-administer-use-powershell/
[hdinsight-upload-data]: ../hdinsight-upload-data/
[hdinsight-use-mapreduce]: ../hdinsight-use-mapreduce
[hdinsight-use-hive]: ../hdinsight-use-hive/
[hdinsight-use-pig]: ../hdinsight-use-pig/
[hdinsight-use-oozie]: ../hdinsight-use-oozie/
[hdinsight-storage]: ../hdinsight-use-blob-storage/
[hdinsight-emulator]: ../hdinsight-get-started-emulator/
[hdinsight-develop-streaming]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-mapreduce]: ../hdinsight-develop-deploy-java-mapreduce/
[hadoop-hdinsight-intro]: ../hdinsight-hadoop-introduction/
[hdinsight-weblogs-sample]: ../hdinsight-hive-analyze-website-log/
[hdinsight-sensor-data-sample]: ../hdinsight-hive-analyze-sensor-data/

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-free-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: ../storage-create-storage-account/ 

[apache-hadoop]: http://go.microsoft.com/fwlink/?LinkId=510084
[apache-hive]: http://go.microsoft.com/fwlink/?LinkId=510085
[apache-mapreduce]: http://go.microsoft.com/fwlink/?LinkId=510086
[apache-hdfs]: http://go.microsoft.com/fwlink/?LinkId=510087
[hdinsight-hbase-custom-provision]: http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-hbase-get-started/


[powershell-download]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[powershell-install-configure]: ../install-configure-powershell/
[powershell-open]: ../install-configure-powershell/#Install


[img-hdi-dashboard]: ./media/hdinsight-get-started/HDI.dashboard.png
[img-hdi-dashboard-query-select]: ./media/hdinsight-get-started/HDI.dashboard.query.select.png
[img-hdi-dashboard-query-select-result]: ./media/hdinsight-get-started/HDI.dashboard.query.select.result.png
[img-hdi-dashboard-query-select-result-output]: ./media/hdinsight-get-started/HDI.dashboard.query.select.result.output.png
[img-hdi-dashboard-query-browse-output]: ./media/hdinsight-get-started/HDI.dashboard.query.browse.output.png

[img-hdi-getstarted-video]: ./media/hdinsight-get-started/HDI.GetStarted.Video.png


[image-hdi-storageaccount-quickcreate]: ./media/hdinsight-get-started/HDI.StorageAccount.QuickCreate.png
[image-hdi-clusterstatus]: ./media/hdinsight-get-started/HDI.ClusterStatus.png
[image-hdi-quickcreatecluster]: ./media/hdinsight-get-started/HDI.QuickCreateCluster.png
[image-hdi-getstarted-flow]: ./media/hdinsight-get-started/HDI.GetStartedFlow.png

[image-hdi-gettingstarted-powerquery-importdata]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData.png
[image-hdi-gettingstarted-powerquery-importdata2]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData2.png

