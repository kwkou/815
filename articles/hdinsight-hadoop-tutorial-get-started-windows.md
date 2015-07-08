<properties
   pageTitle="Hadoop 教程：Windows 上的 Hadoop 入门 | Windows Azure"
   description="HDInsight 中的 Hadoop 入门。学习如何在 Windows 上设置 Hadoop 群集、对数据运行 Hive 查询，以及在 Excel 中分析输出。"
   keywords="hadoop tutorial,hadoop on windows,hadoop cluster,learn hadoop, hive query"
   services="hdinsight"
   documentationCenter=""
   authors="nitinme"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.date="03/16/2015"
   wacn.date="06/26/2015"/>


# Hadoop 教程：在 Windows 上的 HDInsight 中开始将 Hadoop 与 Hive 查询配合使用

> [AZURE.SELECTOR]
- [Windows](/documentation/articles/hdinsight-get-started)
- [Linux](/documentation/articles/hdinsight-hadoop-linux-get-started)

为了帮助你了解 Windows 上的 HDInsight 并快速开始使用 HDInsight，本教程说明了如何对 Hadoop 群集中的非结构化数据运行 Hive 查询。你提取有关手机使用情况的有用信息，然后在 Microsoft Excel 中分析结果。


> [AZURE.NOTE]如果你是 Hadoop 和大数据的新手，可以进一步了解这些术语：[Apache Hadoop][apache-hadoop]、[MapReduce][apache-mapreduce]、[HDFS][apache-hdfs] 和 [Hive][apache-hive]。若要了解 HDInsight 如何在 Azure 中启用 Hadoop，请参阅 [HDInsight 中的 Hadoop 简介][hadoop-hdinsight-intro]。

Microsoft 还提供了 HDInsight Emulator for Azure（以前称作 *Microsoft HDInsight 开发者预览版*），与 Azure HDInsight 的通用版本结合使用。Emulator 针对开发人员方案，仅支持单节点部署。有关如何使用 HDInsight Emulator 的信息，请参阅 [HDInsight Emulator 入门][hdinsight-emulator]。

> [AZURE.NOTE]有关如何设置 HBase 群集的说明，请参阅[在 HDInsight 中设置 HBase 群集][hdinsight-hbase-custom-provision]。请参阅 <a href="http://go.microsoft.com/fwlink/?LinkId=510237">Hadoop 与 HBase 之间的差别</a>，以了解为何要选择其中的某一种数据库。

## 本 Hadoop 教程的目标是什么？

假设你具有一个大型非结构化数据集，想要对其运行 Hive 查询以提取一些有意义的信息。这正是我们在本教程中将要实现的目标。下面说明了如何实现此目标：

   ![Hadoop 教程：创建帐户；设置 Hadoop 群集；提交 Hive 查询；在 Excel 中分析数据。][image-hdi-getstarted-flow]




## 先决条件

在开始学习这篇针对 Windows 上的 Hadoop 的教程之前，你必须具有：


- Azure 订阅。有关获得订阅的详细信息，请参阅[购买选项][azure-purchase-options][试用版][azure-trial]。
- 装有 Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus 的计算机。

**本教程预计完成时间：**30 分钟



## <a name="storage"></a>创建 Azure 存储帐户

HDInsight 使用 Azure Blob 存储来存储数据。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

在设置 HDInsight 中的 Hadoop 群集时，将要指定 Azure 存储帐户。将该帐户的一个特定 Blob 存储容器指定为默认文件系统，如同在 Hadoop 分布式文件系统 (HDFS) 中一样。默认情况下，HDInsight 群集与你指定的存储帐户设置在同一数据中心内。

>[AZURE.NOTE]请不要在多个 Hadoop 群集之间共享默认的 Blob 存储容器。

除此存储帐户外，你在自定义配置群集时还可以添加其他存储帐户。附加的此存储帐户可以来自同一 Azure 订阅，也可以来自不同的 Azure 订阅。有关说明，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。

为简化本教程，仅使用了默认 Blob 和默认存储帐户。而在实践中，数据文件通常存储在指定的存储帐户中。

**创建 Azure 存储帐户**

1. 登录到 [Azure 门户][azure-management-portal]。
2. 单击左下角的“新建”，依次指向“数据服务”和“存储”，然后单击“快速创建”。

	![在 Azure 门户中，可以使用“快速创建”来设置新的存储帐户。][image-hdi-storageaccount-quickcreate]

3. 输入“URL”、“位置”和“复制”信息，然后单击“创建存储帐户”。（不支持地缘组。） 你将在存储列表中看到新的存储帐户。

	>[AZURE.NOTE]用于设置 HDInsight 群集的“快速创建”选项（与我们在本教程中使用的一样）在设置群集时并不要求提供位置。默认情况下，它将群集与存储帐户一起放在同一数据中心。因此，确保在群集支持的位置中创建存储帐户。位置包括：**中国东部**、**中国北部**。

4. 等到新存储帐户的“状态”更改为“联机”。
5. 从列表中选择新存储帐户，然后单击页面底部的“管理访问密钥”。
7. 记下“存储帐户名称”和“主访问密钥”（或“辅助访问密钥”- 任一密钥都有效）。本教程后面的步骤中将会用到它们。


有关详细信息，请参阅[如何创建存储帐户][azure-create-storageaccount]和[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

## <a name="provision"></a>设置 Hadoop 群集

当你设置群集时，便设置了包含 Hadoop 和相关应用程序的 Azure 计算资源。在此部分中，设置基于 Hadoop 版本 2.4 的 HDInsight 版本 3.1 群集。你还可以使用 Azure 门户、HDInsight PowerShell cmdlet 或 HDInsight .NET SDK 为其他版本创建 Hadoop 群集。有关说明，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。有关 HDInsight 版本及其 SLA 的信息，请参阅 [HDInsight 组件版本](hdinsight-component-versioning.md)。

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]


**设置 Hadoop 群集**

1. 登录到 [Azure 门户][azure-management-portal]。

2. 单击左窗格中的“HDInsight”以便列出你的帐户中群集的状态。在下面的屏幕快照中，没有现有的 HDInsight 群集。

	![HDInsight 群集在 Azure 门户中的状态。][image-hdi-clusterstatus]

3. 单击左下角的“新建”，然后依次单击“数据服务”、“HDInsight”和“Hadoop”。

	![在 HDInsight 中创建 Hadoop 群集。][image-hdi-quickcreatecluster]

4. 输入或选择下列值：

	<table border="1">
	<tr><th>Name</th><th>值</th></tr>
	<tr><td>群集名称</td><td>群集的名称。</td></tr>
	<tr><td>群集大小</td><td>要部署的数据节点的数目。默认值为 4。但是，下拉列表中还提供了使用 1 或 2 个数据节点的选项。可以使用“自定义创建”选项指定任意数量的群集节点<strong></strong>。提供了有关各种群集大小的计帐费率的定价详细信息。单击下拉框正上方的 <strong>?</strong> 符号并访问弹出框上的链接。</td></tr>
	<tr><td>密码</td><td><i>admin</i> 帐户的密码。如果使用的不是“自定义创建”选项，则指定群集用户名“admin”<strong></strong>。请注意，这不是设置群集所在的 VM 的 Windows Administrator 帐户。帐户名称可以通过使用“自定义创建”向导来更改<strong></strong>。</td></tr>
	<tr><td>存储帐户</td><td>从下拉框中选择你所创建的存储帐户。<br/>

	选择存储帐户后，就不能更改它。如果删除了存储帐户，则群集将不再可用。HDInsight 群集与存储帐户位于同一数据中心中。
	</td></tr>
	</table>保留群集名称的副本。本教程后面的步骤中将会用到它。


5. 单击“创建 HDINSIGHT 群集”。完成设置后，状态列将显示“正在运行”。

	>[AZURE.NOTE]上述过程通过 HDInsight 版本 3.1 来创建 Hadoop 群集。若要使用其他版本创建群集，请使用门户中的“自定义”创建方法，或使用 Azure PowerShell。有关各版本之间的差异的信息，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。有关使用“自定义创建”选项的信息，请参阅[使用自定义选项设置 HDInsight 群集][hdinsight-provision]。


## <a name="sample"></a>从门户运行示例数据

成功设置的 HDInsight 群集提供包括入门库的查询控制台以直接从门户运行示例。通过浏览一些基本方案，你可以使用示例了解如何使用 HDInsight。这些示例提供所有必要组件，比如要分析的数据和要对数据运行的查询。若要了解有关入门库中的示例的详细信息，请参阅[使用 HDInsight 入门库了解 HDInsight 中的 Hadoop](hdinsight-learn-hadoop-use-sample-gallery)。

**若要运行示例**，请从 Azure 门户中单击你想要运行示例的群集名称，然后单击页面底部的“查询控制台”。从打开的网页中，单击“入门库”选项卡，然后在“示例”类别下，单击要运行的示例。按照网页上的说明完成示例。下表列出了几个示例，并提供了有关每个示例的作用的详细信息。

示例 | 它有什么作用？
------ | ---------------
[传感器数据分析][hdinsight-sensor-data-sample] | 了解如何使用 HDInsight 处理加热、通风和空调 (HVAC) 系统产生的历史数据，以识别无法可靠地保持设定温度的系统
[网站日志分析][hdinsight-weblogs-sample] | 了解如何使用 HDInsight 分析网站日志文件，以了解一天中从外部网站对该网站的访问次数，以及用户遇到的网站错误汇总




## <a name="hivequery"></a>从门户运行 Hive 查询
现在，你的 HDInsight 群集已设置完毕，下一步是运行 Hive 作业以查询示例 Hive 表。我们将使用 HDInsight 群集随附的 *hivesampletable*。该表包含有关移动设备制造商、平台和型号的数据。对此表运行 Hive 查询可按特定制造商检索移动设备的数据。

> [AZURE.NOTE]HDInsight Tools for Visual Studio 随附了 Azure SDK for .NET 2.5 或更高版本。使用 Visual Studio 中的工具可以连接到 HDInsight 群集、创建 Hive 表和运行 Hive 查询。有关详细信息，请参阅 [HDInsight Hadoop Tools for Visual Studio 入门][1]。

**从群集仪表板运行 Hive 作业**

1. 登录到 [Azure 门户][azure-management-portal]。
2. 单击左窗格中的“HDINSIGHT”。你将会看到所创建的群集的列表，包括你刚刚在上一部分中创建的群集。
3. 单击要用于运行 Hive 作业的群集名称，然后单击页面底部的“查询控制台”。
4. 这会在另一个浏览器选项卡中打开一个网页。输入 Hadoop 用户帐户和密码。默认用户名为 **admin**；密码是你在设置群集时所输入的密码。仪表板类似于：

	![HDInsight 群集仪表板中的“Hive 编辑器”选项卡。][img-hdi-dashboard]

	页面顶部有多个选项卡。默认选项卡为“Hive 编辑器”，而其他选项卡为“作业历史记录”和“文件浏览器”。使用仪表板可以提交 Hive 查询、检查 Hadoop 作业日志，以及浏览 WASB 文件。

	> [AZURE.NOTE]请注意，网页的 URL 为 *&lt;群集名称&gt;.azurehdinsight.cn*。因此，如果不从门户打开仪表板，也可以在 Web 浏览器中使用 URL 打开仪表板。

6. 在“Hive 编辑器”选项卡上，为“查询名称”输入 **HTC20**。查询名称为作业标题。

7. 在查询窗格中输入以下 Hive 查询：

		SELECT * FROM hivesampletable
			WHERE devicemake LIKE "HTC%"
			LIMIT 20;

	![在 Hive 编辑器的查询窗格中输入的 Hive 查询。][img-hdi-dashboard-query-select]

4. 单击“提交”。片刻之后即可返回结果。屏幕将每隔 30 秒刷新一次。你也可以单击“刷新”来刷新屏幕。

    完成后，屏幕将如下所示：

	![在群集仪表板底部列出的 Hive 查询结果。][img-hdi-dashboard-query-select-result]

5. 单击屏幕上的查询名称以查看输出。记下“作业开始时间(UTC)”。稍后需要用到此值。

    ![在 HDInsight 群集仪表板的“作业历史记录”选项卡中列出的“作业开始时间”。][img-hdi-dashboard-query-select-result-output]

    该页面还显示“作业输出”和“作业日志”。你也可以选择下载输出文件 (_stdout) 和日志文件 (_stderr)。


	> [AZURE.NOTE]只要停留在“Hive 编辑器”选项卡上，该选项卡上的“作业会话”表便会列出已完成或正在运行的作业。如果导航离开页面，该表就不列出任何作业。“作业历史记录”选项卡保留所有作业（已完成或正在运行）的列表。


**浏览到输出文件**

1. 在群集仪表板中，单击“文件浏览器”。
2. 单击你的存储帐户名称，单击容器名称（与群集名称相同），然后单击“用户”。
3. 单击“admin”，然后单击其上次修改时间比你前面记下的作业开始时间稍晚的 GUID。复制此 GUID。在后一个部分将要用到它。


  ![在“文件浏览器”选项卡中列出的 Hive 查询输出文件 GUID。][img-hdi-dashboard-query-browse-output]


## <a name="powerquery"></a>连接到 Excel 的 Microsoft 商业智能工具

你可以使用 Microsoft Excel 的 Power Query 附加组件将作业输出从 HDInsight 导入到 Excel 中，从中可以使用 Microsoft 商业智能工具进一步分析结果。

要完成本部分教程，必须已安装 Excel 2013 或 2010。

**下载 Microsoft Power Query for Excel**

- 从 [Microsoft 下载中心](http://www.microsoft.com/zh-CN/download/details.aspx?id=39379)下载 Microsoft Power Query for Microsoft Excel 并将其安装。

**导入 HDInsight 数据**

1. 打开 Excel，然后创建一个新的工作簿。
3. 依次单击“Power Query”菜单、“从其他源”和“从 Azure HDInsight”。

	![为 Azure HDInsight 打开的 Excel PowerQuery 导入菜单。][image-hdi-gettingstarted-powerquery-importdata]

3. 输入与你的群集关联的 Azure Blob 存储帐户的“帐户名称”，然后单击“确定”。（这是先前在教程中创建的存储帐户。）
4. 输入 Azure Blob 存储帐户的“帐户密钥”，然后单击“保存”。
5. 在右窗格中，双击 Blob 名称。默认情况下，该 Blob 名称与群集名称相同。

6. 在“名称”列中找到 **stdout**。确认对应“文件夹路径”列中的 GUID 是否与你前面记下的 GUID 匹配。若匹配，则表明输出数据与所提交的作业相对应。单击 **stdout** 列左侧的“二进制”。

	![按内容列表中的 GUID 查找数据输出。][image-hdi-gettingstarted-powerquery-importdata2]

9. 单击左上角的“关闭并加载”以将 Hive 作业输出导入到 Excel 中。


## <a name="nextsteps"></a>后续步骤
在本 Hadoop 教程中，你已学习如何在 Windows 上的HDInsight 中设置 Hadoop 群集、如何对数据运行 Hive 查询，以及如何将结果导入到 Excel 中，在 Excel 中，可以使用商业智能工具进一步处理结果以及以图形方式显示结果。若要了解更多信息，请参阅以下教程：

- [开始使用适用于 Visual Studio 的 HDInsight 工具][1]
- [HDInsight Emulator 入门][hdinsight-emulator]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上载到 HDInsight][hdinsight-upload-data]
- [将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]
- [为 HDInsight 开发 C# Hadoop 流式处理程序][hdinsight-develop-streaming]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]


[1]: hdinsight-hadoop-visual-studio-tools-get-started

[hdinsight-versions]: hdinsight-component-versioning


[hdinsight-provision]: hdinsight-provision-clusters
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell
[hdinsight-upload-data]: hdinsight-upload-data
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce
[hdinsight-use-hive]: hdinsight-use-hive
[hdinsight-use-pig]: hdinsight-use-pig
[hdinsight-use-oozie]: hdinsight-use-oozie
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage
[hdinsight-emulator]: hdinsight-hadoop-emulator-get-started
[hdinsight-develop-streaming]: hdinsight-hadoop-develop-deploy-streaming-jobs
[hdinsight-develop-mapreduce]: hdinsight-develop-deploy-java-mapreduce
[hadoop-hdinsight-intro]: hdinsight-hadoop-introduction
[hdinsight-weblogs-sample]: hdinsight-hive-analyze-website-log
[hdinsight-sensor-data-sample]: hdinsight-hive-analyze-sensor-data

[azure-purchase-options]: /pricing/overview/
[azure-trial]: /pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: storage-create-storage-account

[apache-hadoop]: http://go.microsoft.com/fwlink/?LinkId=510084
[apache-hive]: http://go.microsoft.com/fwlink/?LinkId=510085
[apache-mapreduce]: http://go.microsoft.com/fwlink/?LinkId=510086
[apache-hdfs]: http://go.microsoft.com/fwlink/?LinkId=510087
[hdinsight-hbase-custom-provision]: hdinsight-hbase-tutorial-get-started


[powershell-download]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[powershell-install-configure]: install-configure-powershell
[powershell-open]: install-configure-powershell#Install


[img-hdi-dashboard]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.dashboard.png
[img-hdi-dashboard-query-select]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.dashboard.query.select.png
[img-hdi-dashboard-query-select-result]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.dashboard.query.select.result.png
[img-hdi-dashboard-query-select-result-output]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.dashboard.query.select.result.output.png
[img-hdi-dashboard-query-browse-output]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.dashboard.query.browse.output.png

[img-hdi-getstarted-video]: ./media/hdinsight-hadoop-tutorial-get-started-windows/hdi-get-started-video.png


[image-hdi-storageaccount-quickcreate]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.StorageAccount.QuickCreate.png
[image-hdi-clusterstatus]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.ClusterStatus.png
[image-hdi-quickcreatecluster]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.QuickCreateCluster.png
[image-hdi-getstarted-flow]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.GetStartedFlow.png

[image-hdi-gettingstarted-powerquery-importdata]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.GettingStarted.PowerQuery.ImportData.png
[image-hdi-gettingstarted-powerquery-importdata2]: ./media/hdinsight-hadoop-tutorial-get-started-windows/HDI.GettingStarted.PowerQuery.ImportData2.png

<!---HONumber=61-->