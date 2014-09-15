<properties linkid="manage-services-hdinsight-get-started-hdinsight" urlDisplayName="Get Started" pageTitle="Get started using HDInsight | Azure" metaKeywords="" description="Get started with HDInsight, a big data solution. Learn how to provision clusters, run MapReduce jobs, and output data to Excel for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using Azure HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 开始使用 Azure HDInsight

HDInsight 使 [Apache Hadoop][] 可在云中作为服务使用，并使 MapReduce 软件框架可用于更简单、缩放性更高且经济实用的 Azure 环境。HDInsight 还提供了使用 Azure Blob 存储管理和存储数据的经济实用方法。

在本教程中，你将使用 Azure 管理门户对 HDInsight 群集进行设置，使用 PowerShell 提交一个对文本文件中的单词进行计数的 Hadoop MapReduce 作业，然后将该 MapReduce 作业的输出数据导入到 Excel 中进行检查。

> [WACOM.NOTE] 本教程介绍如何在 HDInsight 上使用 Hadoop 1.2 群集。有关在 HDInsight 上使用 Hadoop 2.2 群集的教程，请参阅[将 Hadoop 2.2 群集与 HDInsight 配合使用入门][]。有关版本信息，请参阅 [HDInsight 提供的群集版本有哪些新功能？][]

Microsoft 还发布了 HDInsight Emulator for Azure（以前称作 *Microsoft HDInsight 开发者预览版*），与 Azure HDInsight 的通用版本结合使用。该产品针对开发人员方案并因此仅支持单节点部署。有关如何使用 HDInsight Emulator 的信息，请参阅 [HDInsight Emulator 入门][]。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   Azure 订阅。有关获取订阅的详细信息，请参阅[购买选项][]、[免费试用][]。
-   运行 Windows 8.1、Windows 8.8、Windows 7、Windows Server 2012 或 Windows Server 2008 R2 的计算机。此计算机将用于提交 MapReduce 作业。
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

3.  在窗口中，键入与你的帐户相关联的电子邮件地址和密码。Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。此连接将在几个小时后过期。

连接到你的订阅的另一个方法是使用证书方法。有关说明，请参阅[安装和配置 Azure PowerShell][]。

> [WACOM.NOTE] 若要在将 Azure AD 方法与 Add-AzureAccount cmdlet 配合使用后返回到证书方法，请运行 Remove-AzureAccount cmdlet。

## 设置 HDInsight 群集

HDInsight 将 Azure Blob 存储用于存储数据。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。该存储帐户必须与 HDInsight 计算资源位于同一数据中心。目前，只能在以下数据中心内设置 HDInsight 群集：

-   中国东部
-   中国北部

除了此存储帐户外，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][1]。

为了简化本教程，我们仅使用默认存储帐户，并且所有文件均存储在默认文件系统容器（位于 */tutorials/getstarted/*）中。

**创建 Azure 存储帐户**

1.  登录到 [Azure 管理门户][]。
2.  单击左下角的“新建” ，指向“数据服务” ，指向“存储” ，然后单击“快速创建” 。

    ![HDI.StorageAccount.QuickCreate][]

3.  输入“URL” 、“位置” 和“复制” ，然后单击“创建存储帐户” 。不支持地缘组。你将在存储列表中看到新的存储帐户。
4.  等到新存储帐户的**“状态”**更改为“联机” 。
5.  从列表中单击新存储帐户以选中它。
6.  单击页面底部的“管理访问密钥” 。
7.  记下**存储帐户名称**和**主访问密钥**（或**辅助访问密钥**，这两个密钥中的任一个均有效）。本教程后面的步骤中将会用到它们。

有关详细信息，请参阅
[如何创建存储帐户][]和[将 Azure Blob 存储与 HDInsight 配合使用][]。

**设置 HDInsight 群集**

1.  登录到 [Azure 管理门户][]。

2.  单击左侧的“HDInsight” 以便列出你的帐户中群集的状态。在下面的屏幕截图中，没有现有的 HDInsight 群集。

    ![HDI.ClusterStatus][]

3.  单击左下角的“新建” ，然后依次单击“数据服务”、 “HDInsight” 和“快速创建” 。

    ![HDI.QuickCreateCluster][]

4.  输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td>群集名称</td><td>群集的名称</td></tr>
	<tr><td>群集大小</td><td>要部署的数据节点的数目。默认值为 4。不过，下拉菜单上还提供了以下数据节点群集数：8、16 和 32。使用&ldquo;自定义创建&rdquo;<strong></strong>选项时可指定任意数量的数据节点。提供了有关各种群集大小的计帐费率的定价详细信息。单击下拉框正上方的 <strong>?</strong> 符号并打开弹出窗口上的链接。</td></tr>
	<tr><td>密码（群集管理员）</td><td>帐户 <i>admin</i> 的密码。默认情况下，在使用&ldquo;快速创建&rdquo;选项时，会将群集用户名指定为&ldquo;admin&rdquo;。只能通过使用&ldquo;自定义创建&rdquo;<strong></strong>向导更改此设置。密码字段必须至少为 10 个字符且必须包含 1 个大写字母、1 个小写字母、1 个数字和 1 个特殊字符。</td></tr>
	<tr><td>存储帐户</td><td>从下拉框中选择你创建的存储帐户。 <br/>
	
	一旦选择存储帐户，就不能更改它。如果删除了存储帐户，则群集将不再可用。
	
	HDInsight 群集位置将与存储帐户相同。
	</td></tr>
	</table>

    记下群集名称。本教程后面的步骤中将会用到它。

    > [WACOM.NOTE] 快速创建方法将创建 HDInsight 2.1 版群集。若要创建 1.6 或 3.0 版群集，请使用管理门户中的自定义创建方法，或使用 Azure PowerShell。

5.  单击右下角的“创建 HDInsight 群集” 。完成设置过程后，状态列将显示“正在运行” 。

有关使用“自定义创建” 选项的信息，请参阅[设置 HDInsight 群集][1]。

## 运行 WordCount MapReduce 作业

现在你已设置了一个 HDInsight 群集。下一步是运行一个 MapReduce 作业，以便对文本文件中单词出现的次数进行计数。

运行 MapReduce 作业要求以下元素：

-   MapReduce 程序。在本教程中，你将使用 HDInsight 群集附带的 WordCount 示例，因此，你无需编写自己的 MapReduce 程序。该示例位于 */example/jars/hadoop-examples.jar*。有关编写你自己的 MapReduce 作业的说明，请参阅[为 HDInsight 开发 Java MapReduce 程序][]和[为 HDInsight 开发 C\# Hadoop 流程序][]。

    > [WACOM.NOTE] 在 HDInsight 3.0 版群集中，jar 文件名是 */example/jars/hadoop-mapreduce-examples.jar*。

-   输入文件文件夹。在本教程中，你将指定一个文件，即 */example/data/gutenberg/davinci.txt*。有关上载自己的数据文件的信息，请参阅[将数据上载到 HDInsight][]。
-   输出文件文件夹。你将使用 */tutorials/getstarted/WordCountOutput* 作为输出文件文件夹。此文件夹不能是现有文件夹。否则，你将收到异常。

用于访问 Blob 存储中的文件的 URI 方案为：

    wasb[s]://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/<路径>

> [WACOM.NOTE] 默认情况下，设置过程将创建一个与 HDInsight 群集同名的默认 Blob 容器。如果已存在同名的容器，则设置过程会将默认容器命名为 -x（x 是序列号），例如 mycluster-1。

URI 方案提供了使用 *wasb:*前缀的未加密访问和使用 WASBS 的 SSL 加密访问。建议尽量使用 wasbs，即使在访问位于同一 Azure 数据中心内的数据时也是如此。

由于 HDInsight 使用 Blob 存储容器作为默认文件系统，因此你可以使用相对或绝对路径引用默认文件系统中的文件和目录。

例如，若要访问 hadoop-examples.jar，你可以使用以下选项之一：

    ● wasb://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/example/jars/hadoop-examples.jar
    ● wasb:///example/jars/hadoop-examples.jar
    ● /example/jars/hadoop-examples.jar
                

有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

**运行 WordCount 示例**

1.  打开 **Azure PowerShell** 或 **PowerShell ISE**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  如果你尚未连接到 Azure 订阅，请运行以下命令：

        Add-AzureAccount

    有关详细信息，请参阅本文中的[为运行 PowerShell 设置本地环境][为运行的 PowerShell 设置本地环境]。

3.  如果你有多个 Azure 订阅，则可以使用以下命令选择此 PowerShell 会话中使用的当前订阅：

        $subscriptionName = "<SubscriptionName>" 
        Select-AzureSubscription $subscriptionName

    若要获取订阅名称，请登录到[管理门户][]。单击左窗格中的“设置” 。

4.  设置以下脚本中的第一个变量，并运行该脚本：

        $clusterName = "<HDInsightClusterName>"

        $jarFile = "wasb:///example/jars/hadoop-examples.jar"
        $className = "wordcount"
        $statusFolder = "wasb:///tutorials/getstarted/wordCountStatus"
        $inputFolder = "wasb:///example/data/gutenberg/davinci.txt"
        $outputFolder = "wasb:///tutorials/getstarted/WordCountOutput"

	<table>
	<thead>
	<tr class="header">
	<th align="left">变量</th>
	<th align="left">说明</th>
	</tr>
	</thead>
	<tbody>
	<tr class="odd">
	<td align="left">$clusterName</td>
	<td align="left">此群集名称必须与你在本教程前面部分使用 Azure 管理门户创建的群集名称匹配。</td>
	</tr>
	<tr class="even">
	<td align="left">$jarFile</td>
	<td align="left">这是你将运行的 MapReduce jar 文件。该文件随 HDInsight 群集提供。</td>
	</tr>
	<tr class="odd">
	<td align="left">$className</td>
	<td align="left">此类名在 MapReduce 程序中进行硬编码。</td>
	</tr>
	<tr class="even">
	<td align="left">$statusFolder</td>
	<td align="left">-StatusFolder 参数是可选的。如果指定此参数，则标准错误和标准输出文件将放入该文件夹。</td>
	</tr>
	<tr class="odd">
	<td align="left">$inputFolder</td>
	<td align="left">这是 MapReduce 作业的源数据文件文件夹。用于单词计数的 MapReduce 作业将对此文件夹的文件中的单词进行计数。你可以指定文件夹名称或文件名。</td>
	</tr>
	<tr class="even">
	<td align="left">$outputFolder</td>
	<td align="left">MapReduce 作业将在此文件夹中生成输出文件。输出文件将包含单词及其计数的列表。默认输出文件名为 part-r-00000。</td>
	</tr>
	</tbody>
	</table>

5.  运行以下命令创建 MapReduce 作业定义：

        # 定义 MapReduce 作业
        $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "wordcount" -StatusFolder $statusFolder -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///tutorials/getstarted/WordCountOutput"

    作业定义使用了你在上一步中定义的变量。

6.  运行以下命令来提交 MapReduce 作业：

        # 提交作业
        $wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition 

    除了 MapReduce 作业定义外，你还必须提供要运行 MapReduce 作业的 HDInsight 群集名称。

    *Start-AzureHDInsightJob* 是异步调用。若要检查作业是否完成，请使用 *Wait-AzureHDInsightJob* cmdlet。

7.  运行以下命令来检查 MapReduce 作业的完成：

        Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600 

8.  运行以下命令来检查运行 MapReduce 作业时的错误：

        # 获取作业输出
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError

    以下屏幕截图显示了成功运行的输出。否则，你将看到一些错误消息。

    ![HDI.GettingStarted.RunMRJob][]

    对此 cmdlet 使用 *-StandardOuput* 开关，还可以获取标准输出日志。

**检索 MapReduce 作业的结果**

1.  打开 **Azure PowerShell**。
2.  运行以下命令创建 C:\\Tutorials 文件夹，并将目录更改为该文件夹：

        mkdir \Tutorials
        cd \Tutorials

    默认 Azure Powershell 目录是 *C:\\Windows\\System32\\WindowsPowerShell\\v1.0*。默认情况下，你对此文件夹没有写入权限。在脚本的后面部分，你会将输出文件的副本下载到当前目录。必须将目录更改为你有写入权限的文件夹。

3.  在以下命令中设置三个变量，然后运行它们：

        $storageAccountName = "<StorageAccountName>"   
        $containerName = "<ContainerName>"      

        $blobName = "tutorials/getstarted/WordCountOutput/part-r-00000"   

    Azure 存储帐户是你在本教程前面部分创建的帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。Blob 存储容器通常与 HDInsight 群集共享相同的名称，除非你在设置群集时指定其他名称。

4.  运行以下命令以创建 Azure 存储上下文对象：

        # 创建存储帐户上下文对象
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    *Select-AzureSubscription* 用于在你有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

5.  运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

        # 将作业输出下载到工作站
        Get-AzureStorageBlobContent -Container $ContainerName -Blob $blobName -Context $storageContext -Force

    *example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹上的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。此文件将下载到 *C:\\example\\data\\WordCountOutput\\* 文件夹。

6.  运行以下命令来打印 MapReduce 作业输出文件：

        cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

    ![HDI.GettingStarted.MRJobOutput][]

    MapReduce 作业将生成一个名为 part-r-00000** 的文件，其中包含单词和计数。此脚本使用 findstr 命令来列出包含“there”**的所有单词。

> [WACOM.NOTE] 如果你在记事本中打开 *./example/data/WordCountOutput/part-r-00000*（MapReduce 作业的多行输出），将会注意到断行未正确呈现。这是正常情况。

输出文件夹不能是现有文件夹。否则，MapReduce 作业将失败。如果要再次运行 MapReduce 作业，则必须删除输出文件夹和输出文件。下面是用于执行此作业的 PowerShell 脚本：

        # 设置变量
    $storageAccountName = "<StorageAccountName>"   
    $containerName = "<ContainerName>"      

    $blobName = "tutorials/getstarted/WordCountOutput/part-r-00000" 
    $blobFolderName = "tutorials/getstarted/WordCountOutput" 
        
    # 创建存储帐户上下文对象
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  
        
    # 将输出下载到本地计算机
    Remove-AzureStorageBlob -Container $ContainerName -Blob $blobName -Context $storageContext -Force
    Remove-AzureStorageBlob -Container $ContainerName -Blob $blobFolderName -Context $storageContext -Force

## 连接到 Microsoft 商业智能工具

可以使用用于 Excel 的 Power Query 外接程序将 HDInsight 的输出导出到 Excel 中，在 Excel 中可以使用 Microsoft 商业智能 (BI) 工具进一步处理或显示结果。

要完成本部分教程，必须已安装 Excel 2010 或 2013。

**下载 Microsoft Power Query for Excel**

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

-   [将 Hadoop 2.2 群集与 HDInsight 配合使用入门][]
-   [HDInsight Emulator 入门][]
-   [将 Azure Blob 存储与 HDInsight 配合使用][]
-   [使用 PowerShell 管理 HDInsight][]
-   [将数据上载到 HDInsight][]
-   [将 MapReduce 与 HDInsight 配合使用][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 Pig 与 HDInsight 配合使用][]
-   [将 Oozie 与 HDInsight 配合使用][]
-   [为 HDInsight 开发 C\# Hadoop 流程序][]
-   [为 HDInsight 开发 Java MapReduce 程序][]

  [Apache Hadoop]: http://hadoop.apache.org/
  [将 Hadoop 2.2 群集与 HDInsight 配合使用入门]: ../hdinsight-get-started-30/
  [HDInsight 提供的群集版本有哪些新功能？]: ../hdinsight-component-versioning/
  [HDInsight Emulator 入门]: ../hdinsight-get-started-emulator/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: https://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [为运行的 PowerShell 设置本地环境]: #setup
  [设置 HDInsight 群集]: #provision
  [运行 WordCount MapReduce 程序]: #sample
  [连接到 Microsoft 商业智能工具]: #powerquery
  [后续步骤]: #nextsteps
  [Microsoft Web 平台安装程序]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [如何：安装 Azure PowerShell]: ../install-configure-powershell/#Install
  [将 Azure Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/
  [1]: ../hdinsight-provision-clusters/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [HDI.StorageAccount.QuickCreate]: ./media/hdinsight-get-started/HDI.StorageAccount.QuickCreate.png
  [如何创建存储帐户]: ../storage-create-storage-account/
  [HDI.ClusterStatus]: ./media/hdinsight-get-started/HDI.ClusterStatus.png
  [HDI.QuickCreateCluster]: ./media/hdinsight-get-started/HDI.QuickCreateCluster.png
  [为 HDInsight 开发 Java MapReduce 程序]: ../hdinsight-develop-deploy-java-mapreduce/
  [为 HDInsight 开发 C\# Hadoop 流程序]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
  [将数据上载到 HDInsight]: ../hdinsight-upload-data/
  [管理门户]: https://manage.windowsazure.cn
  [HDI.GettingStarted.RunMRJob]: ./media/hdinsight-get-started/HDI.GettingStarted.RunMRJob.png
  [HDI.GettingStarted.MRJobOutput]: ./media/hdinsight-get-started/HDI.GettingStarted.MRJobOutput.png
  [Microsoft 下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?id=39379
  [HDI.GettingStarted.PowerQuery.ImportData]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData.png
  [HDI.GettingStarted.PowerQuery.ImportData2]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData2.png
  [HDI.GettingStarted.PowerQuery.ImportData3]: ./media/hdinsight-get-started/HDI.GettingStarted.PowerQuery.ImportData3.png
  [使用 PowerShell 管理 HDInsight]: ../hdinsight-administer-use-powershell/
  [将 MapReduce 与 HDInsight 配合使用]: ../hdinsight-use-mapreduce
  [将 Hive 与 HDInsight 配合使用]: ../hdinsight-use-hive/
  [将 Pig 与 HDInsight 配合使用]: ../hdinsight-use-pig/
  [将 Oozie 与 HDInsight 配合使用]: ../hdinsight-use-oozie/
