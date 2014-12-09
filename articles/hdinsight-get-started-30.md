<properties linkid="manage-services-hdinsight-get-started-hdinsight" urlDisplayName="Get Started" pageTitle="Get started using Hadoop 2.2 clusters with HDInsight | Azure" metaKeywords="" description="Get started using Hadoop 2.2 clusters with HDInsight, a big data solution. Learn how to provision clusters, run MapReduce jobs, and output data to Excel for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using Azure HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 将 Hadoop 2.2 群集与 HDInsight 配合使用入门

HDInsight 使 [Apache Hadoop][] 可在云中作为服务使用，并使 MapReduce 软件框架可用于更简单、缩放性更高且经济实用的 Azure 环境。HDInsight 还提供了使用 Azure Blob 存储管理和存储数据的经济实用方法。

在本教程中，你将使用 Azure 管理门户对 HDInsight 群集进行设置，使用 PowerShell 提交一个 Hadoop MapReduce 作业，然后将该 MapReduce 作业的输出数据导入到 Excel 中进行检查。

> [WACOM.NOTE] 本教程介绍如何在 HDInsight 上使用 Hadoop 2.2 群集。有关在 HDInsight 上使用 Hadoop 1.2 群集的教程，请参阅[开始使用 Azure HDInsight][]。

> [WACOM.NOTE] HDInsight 群集版本 3.0 及将来的更高版本都不支持 *asv://* 语法。应改用 *wasb://* 语法。

Microsoft 还发布了 HDInsight Emulator for Azure（以前称作 Microsoft HDInsight 开发者预览版），与 Azure HDInsight 的通用版本结合使用。该产品针对开发人员方案并因此仅支持单节点部署。有关如何使用 HDInsight Emulator 的信息，请参阅 [HDInsight Emulator 入门][]。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   Azure 订阅。有关获取订阅的详细信息，请参阅[购买选项][]、[免费试用][]。
-   运行 Windows 8、Windows 7、Windows Server 2012 或 Windows Server 2008 R2 的计算机。此计算机将用于提交 MapReduce 作业。
-   Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus。

**估计完成时间：**30 分钟

## 在本教程中

-   [为运行的 PowerShell 设置本地环境][]
-   [设置 HDInsight 群集][]
-   [运行 WordCount MapReduce 程序][]
-   [连接到 Microsoft 商业智能工具][]
-   [后续步骤][]

## 为运行 PowerShell 设置本地环境

有多种方法可将 MapReduce 作业提交到 HDInsight。在本教程中，你将使用 Azure PowerShell。若要安装 Azure PowerShell，请运行 [Microsoft Web 平台安装程序][]。出现提示时，依次单击“运行” 、“安装” ，然后按照说明进行操作。有关详细信息，请参阅[安装和配置 Azure PowerShell][]。

PowerShell cmdlet 需要使用你的订阅信息来管理你的服务。

**使用 Azure AD 连接到你的订阅**

1.  打开 Azure PowerShell 控制台，按照[如何：安装 Azure PowerShell][] 中的说明进行操作。
2.  运行以下命令：

        Add-AzureAccount

3.  在窗口中，键入与你的帐户相关联的电子邮件地址和密码。Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

连接到你的订阅的另一个方法是使用证书方法。有关说明，请参阅[安装和配置 Azure PowerShell][]。

## 设置 HDInsight 群集

HDInsight 设置过程要求某一 Azure 存储帐户用作默认文件系统。该存储帐户必须与 HDInsight 计算资源位于同一数据中心。目前，只能在以下数据中心内设置 HDInsight 群集：

-   中国东部
-   中国北部

你必须为你的 Azure 存储帐户选择五个数据中心中的一个。

**创建 Azure 存储帐户**

1.  登录到 [Azure 管理门户][]。
2.  单击左下角的“新建” ，指向“数据服务” ，指向“存储” ，然后单击“快速创建” 。

    ![HDI.StorageAccount.QuickCreate][]

3.  输入“URL” 、“位置” 和“复制” ，然后单击“创建存储帐户” 。不支持地缘组。你将在存储列表中看到新的存储帐户。
4.  等到新存储帐户的**“状态”**更改为“联机” 。
5.  从列表中单击新存储帐户以选中它。
6.  单击页面底部的“管理访问密钥” 。
7.  记下**存储帐户名称**和**主访问密钥**。本教程后面的步骤中将会用到它们。

有关详细说明，请参阅
[如何创建存储帐户][]和[将 Azure Blob 存储与 HDInsight 配合使用][]。

目前仅支持使用“自定义创建”选项设置 HDInsight 3.0 群集。

**设置 HDInsight 群集**

1.  登录到 [Azure 管理门户][]。

2.  单击左侧的“HDINSIGHT” 以便列出你的帐户下的 HDInsight 群集。在下面的屏幕截图中，没有现有的 HDInsight 群集。

    ![HDI.ClusterStatus][]

3.  单击左下角的“新建” ，然后依次单击“数据服务”、 “HDINSIGHT” 和“自定义创建” 。

    ![HDI.CustomCreateCluster][]

4.  在“群集详细信息”选项卡上，输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td><strong>群集名称</strong></td><td>群集的名称。</td></tr>
	<tr><td><strong>数据节点</strong></td><td>要部署的数据节点的数目。出于测试目的，请创建单节点群集。<br />群集大小限制因 Azure 订阅而异。若要提高限制的大小，请联系 Azure 计费支持。</td></tr>
	<tr><td><strong>HDInsight 版本</strong></td><td>选择 <strong>3.0</strong> 以在 HDInsight 上创建 Hadoop 2.2 群集。</td></tr>
	<tr><td><strong>区域</strong></td><td>选择与上一个过程中创建的存储帐户相同的区域。HDInsight 要求存储帐户位于同一区域。在以后的配置中，你只能选择你在此处指定的区域中的存储帐户。
	</td></tr>
	</table>

5.  单击右下角的右箭头以配置群集用户。
6.  在“配置群集用户”选项卡中，输入 HDInsight 群集用户帐户的**用户名**和**密码**。除了此帐户外，你还可以在设置群集后创建一个 RDP 用户帐户，以便通过远程桌面访问群集。有关说明，请参阅[使用管理门户管理 HDInsight][]
7.  单击右下角的右箭头以配置存储帐户。
8.  在“存储帐户”标签上，输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td>存储帐户</td><td>选择&ldquo;使用现有存储&rdquo;<strong></strong>。你也可以选择让管理门户创建一个新的存储帐户（如果你尚未创建）。</td></tr>
	<tr><td>帐户名称</td><td>指定你在本教程的上一个过程中创建的存储帐户。请注意，只有同一区域中的存储帐户才显示在列表框中。</td></tr>
	<tr><td>默认容器</td><td>选择&ldquo;创建默认容器&rdquo;<strong></strong>。选择了此选项时，默认容器名称将与群集名称相同。</td></tr>
	<tr><td>其他存储帐户</td><td>选择 <strong>0</strong>。你可以选择将群集连接到最多 7 个其他存储帐户。</td></tr>
	</table>

9.  单击右下角的复选图标以创建群集。完成设置过程后，状态列将显示“正在运行” 。

## 运行 WordCount MapReduce 作业

现在你已设置了一个 HDInsight 群集。下一步是运行一个 MapReduce 作业，以便对文本文件中的单词进行计数。

运行 MapReduce 作业要求以下元素：

-   MapReduce 程序。在本教程中，你将使用 HDInsight 群集分发随附的 WordCount 示例，因此，你无需编写自己的 MapReduce 程序。该示例位于 */example/jars/hadoop-mapreduce-examples.jar*。有关编写自己的 MapReduce 作业的说明，请参阅[为 HDInsight 开发 Java MapReduce 程序][]。

-   一个输入文件。你将使用 */example/data/gutenberg/davinci.txt* 作为输入文件。有关上载文件的信息，请参见[将数据上载到 HDInsight][]。
-   输出文件文件夹。你将使用 */example/data/WordCountOutput* 作为输出文件文件夹。如果该文件夹不存在，系统将会创建。

用于访问 Blob 存储中的文件的 URI 方案为：

    wasb[s]://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/<路径>

> [WACOM.NOTE] 默认情况下，用于默认文件系统的 Blob 容器与 HDInsight 群集同名。

URI 方案提供了使用 *wasb:*前缀的未加密访问和使用 wasbs 的 SSL 加密访问。建议尽量使用 wasbs，即使在访问位于同一 Azure 数据中心内的数据时也是如此。

由于 HDInsight 使用 Blob 存储容器作为默认文件系统，因此你可以使用相对或绝对路径引用默认文件系统中的文件和目录。

例如，若要访问 hadoop-mapreduce-examples.jar，你可以使用以下选项之一：

    ● wasb://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/example/jars/hadoop-mapreduce-examples.jar
    ● wasb:///example/jars/hadoop-mapreduce-examples.jar
    ● /example/jars/hadoop-mapreduce-examples.jar
                

前缀 *wasb://* 在这些文件的路径中的使用。需要指出 Azure Blob 存储正用于输入和输出文件。输出目录假定一个相对于 *wasb:///user/\<username\>* 文件夹的默认路径。

有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

**运行 WordCount 示例**

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  运行以下命令以设置变量。

        $subscriptionName = "<SubscriptionName>" 
        $clusterName = "<HDInsightClusterName>"        

3.  运行以下命令创建 MapReduce 作业定义：

        # 定义 MapReduce 作业
        $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

    hadoop-mapreduce-examples.jar 文件随 HDInsight 群集分发提供。此 MapReduce 作业有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。源文件随 HDInsight 群集分发提供，输出文件路径将会在运行时创建。

4.  运行以下命令来提交 MapReduce 作业：

        # 提交作业
        Select-AzureSubscription $subscriptionName
        $wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition 

    除了 MapReduce 作业定义外，你还必须提供要运行 MapReduce 作业的 HDInsight 群集名称。

    *Start-AzureHDInsightJob* 是异步调用。若要检查作业是否完成，请使用 *Wait-AzureHDInsightJob* cmdlet。

5.  运行以下命令来检查 MapReduce 作业的完成：

        Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600 

6.  运行以下命令来检查运行 MapReduce 作业时的错误：

        # 获取作业输出
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError

    以下屏幕截图显示了成功运行的输出。否则，你将看到一些错误消息。

    ![HDI.GettingStarted.RunMRJob][]

**检索 MapReduce 作业的结果**

1.  打开 **Azure PowerShell**。
2.  运行以下命令创建 C:\\Tutorials 文件夹，并将目录更改为该文件夹：

        mkdir \Tutorials
        cd \Tutorials

    默认 Azure Powershell 目录是 *C:\\Windows\\System32\\WindowsPowerShell\\v1.0*。默认情况下，你对此文件夹没有写入权限。必须将目录更改为你有写入权限的文件夹。

3.  在以下命令中设置三个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"       
        $storageAccountName = "<StorageAccountName>"   
        $containerName = "<ContainerName>"             

    Azure 存储帐户是你在本教程前面部分创建的帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。Blob 存储容器通常与 HDInsight 群集共享相同的名称，除非你在设置群集时指定其他名称。

4.  运行以下命令以创建 Azure 存储上下文对象：

        # 创建存储帐户上下文对象
        Select-AzureSubscription $subscriptionName
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    *Select-AzureSubscription* 用于在你有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

5.  运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

        # 将作业输出下载到工作站
        Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    *example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹上的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。此文件将下载到 *C:\\example\\data\\WordCountOutput\\* 文件夹。

6.  运行以下命令来打印 MapReduce 作业输出文件：

        cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

    ![HDI.GettingStarted.MRJobOutput][]

    MapReduce 作业将生成一个名为 part-r-00000** 的文件，其中包含单词和计数。此脚本使用 findstr 命令来列出包含“there”**的所有单词。

> [WACOM.NOTE] 如果你在记事本中打开 *./example/data/WordCountOutput/part-r-00000*（MapReduce 作业的多行输出），将会注意到断行未正确呈现。这是正常情况。

## 连接到 Microsoft 商业智能工具

可以使用用于 Excel 的 Power Query 外接程序将 HDInsight 的输出导出到 Excel 中，在 Excel 中可以使用 Microsoft 商业智能 (BI) 工具进一步处理或显示结果。

要完成本部分教程，必须已安装 Excel 2010 或 2013。这里我们将导入在 HDInsight 中随附的默认 Hive 表。

**下载 Microsoft PowerQuery for Excel**

-   从 [Microsoft 下载中心][]下载 Microsoft Power Query for Excel 并安装它。

**导入 HDInsight 数据**

1.  打开 Excel，然后创建一个新的空白工作簿。
2.  依次单击“Power Query” 菜单、“从其他源” 和“从 Azure HDInsight” 。

    ![HDI.GettingStarted.PowerQuery.ImportData][]

3.  输入与你的群集关联的 Azure Blob 存储帐户的**帐户名称**，然后单击“确定” 。这是先前在教程中创建的存储帐户。
4.  输入 Azure Blob 存储帐户的**帐户密钥**，然后单击“保存” 。
5.  在右侧的“导航器”窗格中，双击 Blob 存储容器名称。默认情况下，该容器名称与群集名称相同。

6.  在“名称”列 中找到 **part-r-00000**（路径是 *.../example/data/WordCountOutput*），然后单击 **part-r-00000** 左侧的“二进制” 。

    ![HDI.GettingStarted.PowerQuery.ImportData2][]

7.  右键单击“Column1.1” ，然后选择“重命名” 。
8.  将名称更改为“Word” 。
9.  重复该过程以便将“Column1.2” 重命名为“Count” 。

    ![HDI.GettingStarted.PowerQuery.ImportData3][]

10. 单击左上角的“应用并关闭” 。然后，该查询将单词计数 MapReduce 作业输出导入到 Excel 中。

## 后续步骤

在本教程中，你已了解了如何使用 HDInsight 设置群集、如何对其运行 MapReduce 作业，以及如何将结果导入到 Excel 中，在 Excel 中，可以使用 BI 工具进一步处理结果以及以图形方式显示结果。若要了解更多信息，请参阅下列文章：

-   [HDInsight 入门][开始使用 Azure HDInsight]
-   [HDInsight Emulator 入门][]
-   [将 Azure Blob 存储与 HDInsight 配合使用][]
-   [使用 PowerShell 管理 HDInsight][]
-   [将数据上载到 HDInsight][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 Pig 与 HDInsight 配合使用][]
-   [将 Oozie 与 HDInsight 配合使用][]
-   [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序][]
-   [为 HDInsight 开发 Java MapReduce 程序][]

  [Apache Hadoop]: http://hadoop.apache.org/
  [开始使用 Azure HDInsight]: /en-us/documentation/articles/hdinsight-get-started/
  [HDInsight Emulator 入门]: /en-us/documentation/articles/hdinsight-get-started-emulator/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [为运行的 PowerShell 设置本地环境]: #setup
  [设置 HDInsight 群集]: #provision
  [运行 WordCount MapReduce 程序]: #sample
  [连接到 Microsoft 商业智能工具]: #powerquery
  [后续步骤]: #nextsteps
  [Microsoft Web 平台安装程序]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [如何：安装 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/#install
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [HDI.StorageAccount.QuickCreate]: ./media/hdinsight-get-started-3.0/HDI.StorageAccount.QuickCreate.png
  [如何创建存储帐户]: /en-us/documentation/articles/storage-create-storage-account/
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-blob-storage/
  [HDI.ClusterStatus]: ./media/hdinsight-get-started-3.0/HDI.ClusterStatus.png
  [HDI.CustomCreateCluster]: ./media/hdinsight-get-started-3.0/HDI.CustomCreateCluster.png
  [使用管理门户管理 HDInsight]: /en-us/documentation/articles/hdinsight-administer-use-management-portal/
  [为 HDInsight 开发 Java MapReduce 程序]: /en-us/documentation/articles/hdinsight-develop-deploy-java-mapreduce/
  [将数据上载到 HDInsight]: /en-us/documentation/articles/hdinsight-upload-data/
  [HDI.GettingStarted.RunMRJob]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.RunMRJob.png
  [HDI.GettingStarted.MRJobOutput]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.MRJobOutput.png
  [Microsoft 下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?id=39379
  [HDI.GettingStarted.PowerQuery.ImportData]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData.png
  [HDI.GettingStarted.PowerQuery.ImportData2]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData2.png
  [HDI.GettingStarted.PowerQuery.ImportData3]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData3.png
  [使用 PowerShell 管理 HDInsight]: /en-us/documentation/articles/hdinsight-administer-use-powershell/
  [将 Hive 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-hive/
  [将 Pig 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-pig/
  [将 Oozie 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-oozie/
  [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序]: /en-us/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
