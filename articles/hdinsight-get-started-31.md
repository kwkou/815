<properties linkid="manage-services-hdinsight-get-started-hdinsight-hadoop-3.0" urlDisplayName="Get Started" pageTitle="Get started using Hadoop in HDInsight | Azure" metaKeywords="" description="Get started using Hadoop in HDInsight, a big data solution. Learn how to provision clusters, run hive jobs, and output data to Excel for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using Hadoop in HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="jgao"></tags>

# 开始在 HDInsight（预览版）中使用 Hadoop 2.4

<div class="dev-center-tutorial-selector sublanding">
<a href="../hdinsight-get-started" title="开始在 HDInsight 中使用 Hadoop 2.2">Hadoop 2.2</a>
<a href="../hdinsight-get-started-31" title="开始在 HDInsight 中使用 Hadoop 2.4" class="current">Hadoop 2.4</a>
<a href="../hdinsight-get-started-21" title="开始在 HDInsight 中使用 Hadoop 1.2">Hadoop 1.2</a>
</div>

HDInsight 使 [Apache Hadoop][apache-hadoop] 可在云中作为服务使用，并使 MapReduce 软件框架可用于更简单、缩放性更高且经济实用的 Azure 环境。HDInsight 还提供了使用 Azure Blob 存储管理和存储数据的经济实用方法。

在本教程中，你将使用 Azure 管理门户在 HDInsight 中设置一个 Hadoop 群集，使用 HDInsight 群集仪表板提交一个针对示例 Hive 表执行查询的 Hive 作业，然后将该 Hive 作业的输出数据导入到 Excel 中进行检查。

> [WACOM.NOTE] 本教程介绍如何在 HDInsight（预览版）上使用 Hadoop 2.4 群集。有关其他受支持的版本，请单击页顶部的选择器。有关版本信息，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]

<!-- The live demo of this article:  <center><a href="https://www.youtube.com/watch?v=Y4aNjnoeaHA&list=PLDrz-Fkcb9WWdY-Yp6D4fTC1ll_3lU-QS" target = "_blank">![HDI.getstarted.video][img-hdi-getstarted-video]</a></center> -->

Microsoft 还发布了 HDInsight Emulator for Azure（以前称作 *Microsoft HDInsight 开发者预览版*），与 Azure HDInsight 的通用版本结合使用。该产品针对开发人员方案并因此仅支持单节点部署。有关如何使用 HDInsight Emulator 的信息，请参阅 [HDInsight Emulator 入门][hdinsight-emulator]。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   Azure 订阅。有关获取订阅的详细信息，请参阅[购买选项][azure-purchase-options]或[免费试用][azure-free-trial]。

<!--  [Member Offers][azure-member-offers]  -->

-   装有 Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus 的计算机。
-   有关最新 HDInsight 版本的说明，请参阅 [HDInsight 发行说明](http://azure.microsoft.com/en-us/documentation/articles/hdinsight-release-notes/)。

**估计完成时间：**30 分钟

## 本教程的内容

-   [设置 HDInsight 群集](#provision)
-   [运行 Hive 作业](#sample)
-   [连接到 Microsoft 商业智能工具](#powerquery)
-   [后续步骤](#nextsteps)

## <a name="provision"></a>设置 HDInsight 群集

HDInsight 将 Azure Blob 存储用于存储数据。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。该存储帐户必须与 HDInsight 计算资源位于同一数据中心。目前，只能在以下数据中心内设置 HDInsight 群集：

-   亚洲东南部
-   欧洲北部
-   欧洲西部
-   美国东部
-   美国西部
-   中国东部
-   中国北部

除了此存储帐户外，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][hdinsight-provision]。

为了简化本教程，我们仅使用了默认的 Blob 容器和默认的存储帐户，并且所有文件均存储在默认文件系统容器（位于 */tutorials/getstarted/*）中。而在实践中，数据文件通常存储在指定的存储帐户中。

**创建 Azure 存储帐户**

1.  登录到 [Azure 管理门户][azure-management-portal]。
2.  单击左下角的“新建”，指向“数据服务”，指向“存储”，然后单击“快速创建”。

    ![HDI.StorageAccount.QuickCreate][image-hdi-storageaccount-quickcreate]

3.  输入“URL”、“位置”和“复制”，然后单击“创建存储帐户”。不支持地缘组。你将在存储列表中看到新的存储帐户。
4.  等到新存储帐户的“状态”更改为“联机”。
5.  从列表中单击该新存储帐户以便选择它。
6.  单击页底部的“管理访问密钥”。
7.  记下**存储帐户名称**和**主访问密钥**（或**辅助访问密钥**，这两个密钥中的任一个均有效）。本教程后面的步骤中将会用到它们。

有关详细信息，请参阅
[如何创建存储帐户][azure-create-storageaccount]和[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

**设置 HDInsight 群集**

1.  登录到 [Azure 管理门户][azure-management-portal]。

2.  单击左侧的“HDINSIGHT”以便列出你的帐户下的 HDInsight 群集。在下面的屏幕截图中，没有现有的 HDInsight 群集。

    ![HDI.ClusterStatus][image-hdi-clusterstatus]

3.  单击左下角的“新建”，然后依次单击“数据服务”、“HDINSIGHT”和“自定义创建”。

    ![HDI.CustomCreateCluster][image-hdi-customcreatecluster]

4.  在“群集详细信息”选项卡上，输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td><strong>群集名称</strong></td><td>群集的名称。</td></tr>
	<tr><td><strong>数据节点</strong></td><td>要部署的数据节点的数目。出于测试目的，请创建单节点群集。<br />群集大小限制因 Azure 订阅而异。若要提高限制的大小，请联系 Azure 计费支持。</td></tr>
	<tr><td><strong>HDInsight 版本</strong></td><td>选择 <strong>3.1</strong> 以在 HDInsight 上创建 Hadoop 2.4 群集。</td></tr>
	<tr><td><strong>区域</strong></td><td>选择与上一个过程中创建的存储帐户相同的区域。HDInsight 要求存储帐户位于同一区域。在以后的配置中，你只能选择你在此处指定的区域中的存储帐户。
	</td></tr>
	</table>

5.  单击右下角的右箭头以配置群集用户。
6.  在“配置群集用户”选项卡中，输入 HDInsight 群集用户帐户的**用户名**和**密码**。除了此帐户外，你还可以在设置群集后创建一个 RDP 用户帐户，以便通过远程桌面访问群集。有关说明，请参阅 [使用管理门户管理 HDInsight][hdinsight-admin-portal]
7.  单击右下角的右箭头以配置存储帐户。
8.  在“存储帐户”标签上，输入或选择以下值：

    | 名称         | 值                                                                                             |
    |--------------|------------------------------------------------------------------------------------------------|
    | 存储帐户     | 选择“使用现有存储”。你也可以选择让管理门户创建一个新的存储帐户（如果你尚未创建）。             |
    | 帐户名称     | 指定你在本教程的上一个过程中创建的存储帐户。请注意，只有同一区域中的存储帐户才显示在列表框中。 |
    | 默认容器     | 选择“创建默认容器”。选择了此选项时，默认容器名称将与群集名称相同。                             |
    | 其他存储帐户 | 选择 **0**。你可以选择将群集连接到最多 7 个其他存储帐户。                                      |

9.  单击右下角的复选图标以创建群集。完成设置过程后，状态列将显示“正在运行”。

## <a name="sample"></a>运行 Hive 作业

现在你已设置了一个 HDInsight 群集。下一步是运行一个 Hive 作业，以查询 HDInsight 群集随附的示例 Hive 表。该表的名称为 *hivesampletable*。

**打开 HDInsight 群集仪表板**

1.  登录到 [Azure 管理门户][azure-management-portal]。
2.  单击左窗格中的“HDINSIGHT”。你将会看到所创建的群集的列表，包括你刚刚在上一部分中创建的群集。
3.  单击你要在其中运行该 Hive 作业的群集名称。
4.  单击页底部的“管理群集”以打开 HDInsight 群集仪表板。这会在另一个浏览器选项卡中打开一个网页。
5.  输入 Hadoop 用户帐户的用户名和密码。默认的用户名为 **admin**，密码是你在设置过程中输入的密码。仪表板类似于：

    ![hdi.dashboard][img-hdi-dashboard]

    顶部有多个选项卡。默认选项卡为“Hive 编辑器”，其他选项卡包括“作业”和“文件”。使用仪表板可以提交 Hive 查询、检查 Hadoop 作业日志，以及浏览 WASB 文件。

> [WACOM.NOTE] 请注意，URL 为 *\<群集名称\>.hdinsightservices.cn*。如果不从管理门户打开仪表板，也可以在 Web 浏览器中使用 URL 打开仪表板。

**运行 Hive 查询**

1.  在 HDInsight 群集仪表板中，单击顶部的“Hive 编辑器”。
2.  在“查询名称”中输入 **HTC20**。查询名称为作业标题。
3.  在查询窗格中输入以下查询：

        SELECT * FROM hivesampletable
            WHERE devicemake LIKE "HTC%"
            LIMIT 20;

    ![hdi.dashboard.query.select][img-hdi-dashboard-query-select]

4.  单击“提交”。片刻之后即可返回结果。屏幕将每隔 30 秒刷新一次。你也可以单击“刷新”来刷新屏幕。

    完成后，屏幕将如下所示：

    ![hdi.dashboard.query.select.result][img-hdi-dashboard-query-select-result]

    记下“作业开始时间(UTC)”。稍后需要用到此值。

    稍微向下滚动，你将会看到“作业日志”。作业输出为 stdout，作业日志为 stderr。

5.  如果你将来想要重新打开日志文件，可以单击屏幕顶部的“作业”，然后单击作业标题（查询名称），在本例中为 **HTC20**。

**浏览输出文件**

1.  在 HDInsight 群集仪表板中，单击顶部的“文件”。
2.  单击“Templeton-Job-Status”。
3.  单击其上次修改时间比你前面记下的作业开始时间稍晚的 GUID 编号。记下此 GUID。在后一个部分将要用到它。
4.  **stdout** 文件包含后一个部分需要的数据。如果需要，你可以单击“stdout”来下载该数据文件的副本。

## <a name="powerquery"></a>连接到 Microsoft 商业智能工具

可以使用用于 Excel 的 Power Query 外接程序将 HDInsight 的输出导出到 Excel 中，在 Excel 中可以使用 Microsoft 商业智能 (BI) 工具进一步处理或显示结果。

要完成本部分教程，必须已安装 Excel 2010 或 2013。

**下载 Microsoft Power Query for Excel**

-   从 [Microsoft 下载中心](http://www.microsoft.com/en-us/download/details.aspx?id=39379)下载 Microsoft Power Query for Excel 并安装它。

**导入 HDInsight 数据**

1.  打开 Excel，然后创建一个新的空白工作簿。
2.  依次单击“Power Query”菜单、“从其他源”和“从 Azure HDInsight”。

    ![HDI.GettingStarted.PowerQuery.ImportData][image-hdi-gettingstarted-powerquery-importdata]

3.  输入与你的群集关联的 Azure Blob 存储帐户的**帐户名称**，然后单击“确定”。这是先前在教程中创建的存储帐户。
4.  输入 Azure Blob 存储帐户的**帐户密钥**，然后单击“保存”。
5.  在右侧的“导航器”窗格中，双击 Blob 存储容器名称。默认情况下，该容器名称与群集名称相同。

6.  在“名称”列中找到“stdout”（路径为 *.../Templeton-Job-Status/<guid>*），然后单击“stdout”左侧的“二进制”。<guid> 必须与你在上一部分中记下的 GUID 匹配。

    ![HDI.GettingStarted.PowerQuery.ImportData2][image-hdi-gettingstarted-powerquery-importdata2]

7.  单击左上角的“应用并关闭”。该查询随后会将 Hive 作业输出导入到 Excel 中。

## <a name="nextsteps"></a>后续步骤

在本教程中，你已了解了如何使用 HDInsight 设置群集、如何对其运行 MapReduce 作业，以及如何将结果导入到 Excel 中，在 Excel 中，可以使用 BI 工具进一步处理结果以及以图形方式显示结果。若要了解更多信息，请参阅下列文章：

-   [HDInsight Emulator 入门][hdinsight-emulator]
-   [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
-   [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
-   [将数据上载到 HDInsight][hdinsight-upload-data]
-   [将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]
-   [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
-   [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
-   [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]
-   [为 HDInsight 开发 C\# Hadoop 流程序][hdinsight-develop-streaming]
-   [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]

<!-- [azure-member-offers]: http://azure.microsoft.com/en-us/pricing/member-offers/ -->

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

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/

<!--
[azure-member-offers]: http://azure.microsoft.com/en-us/pricing/member-offers/
-->

[azure-free-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: ../storage-create-storage-account/ 

[apache-hadoop]: http://hadoop.apache.org/

[powershell-download]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[powershell-install-configure]: ../install-configure-powershell/
[powershell-open]: ../install-configure-powershell/#Install


[img-hdi-dashboard]: ./media/hdinsight-get-started/HDI.dashboard.png
[img-hdi-dashboard-query-select]: ./media/hdinsight-get-started/HDI.dashboard.query.select.png
[img-hdi-dashboard-query-select-result]: ./media/hdinsight-get-started/HDI.dashboard.query.select.result.png
[image-hdi-quickcreatecluster]: ./media/hdinsight-get-started/HDI.QuickCreateCluster.png
[img-hdi-getstarted-video]: ./media/hdinsight-get-started/HDI.GetStarted.Video.png

[image-hdi-storageaccount-quickcreate]: ./media/hdinsight-get-started/HDI.StorageAccount.QuickCreate.png
[image-hdi-clusterstatus]: ./media/hdinsight-get-started/HDI.ClusterStatus.png
[image-hdi-quickcreatecluster]: ./media/hdinsight-get-started/HDI.QuickCreateCluster.png
[image-hdi-customcreatecluster]: ./media/hdinsight-get-started/HDI.CustomCreateCluster.png
[image-hdi-gettingstarted-powerquery-importdata]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData.png
[image-hdi-gettingstarted-powerquery-importdata2]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData2.png

