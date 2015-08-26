<properties 
	pageTitle="为 HDInsight 开发 C# Hadoop 流式处理程序 | Azure" 
	description="了解如何以 C# 语言开发 Hadoop 流式处理 MapReduce 程序，以及如何将这些程序部署到 Azure HDInsight。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="mumian" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
   ms.service="hdinsight"
   ms.date="07/08/2015" 
   wacn.date=""/>



# 为 HDInsight 开发 C# Hadoop 流式处理程序

Hadoop 为 MapReduce 提供了一个流式处理 API，使你能够以 Java 之外的其他语言来编写映射和化简函数。本教程将逐步引导你创建一个 C# 单词计数程序，从你提供的输入数据中统计给定单词的出现次数。下图显示了 MapReduce 框架如何执行单词计数：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

> [AZURE.NOTE]本文中的步骤仅适用于基于 Windows 的 Azure HDInsight 群集。有关基于 Linux 的 HDInsight 流式处理的示例，请参阅[开发适用于 HDInsight 的 Python 流式处理程序](/documentation/articles/hdinsight-hadoop-streaming-python)。

本教程演示如何：

- 在 HDInsight Emulator for Azure 上使用 C# 开发和测试 Hadoop 流式处理 MapReduce 程序
- 在 Azure HDInsight 中运行同一个 MapReduce 作业 
- 检索 MapReduce 作业的结果

##<a name="prerequisites"></a>先决条件

在开始本教程之前，必须先完成以下操作：

- 安装 HDInsight Emulator。有关说明，请参阅[开始使用 HDInsight Emulator][hdinsight-get-started-emulator]。
- 在模拟器计算机上安装 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
- 获取 Azure 订阅。有关说明，请参阅[购买选项][azure-purchase-options]、[试用][azure-trial]。


##<a name="develop"></a>使用 C&#35; 开发单词计数 Hadoop 流式处理程序

单词计数解决方案包含两个控制台应用程序项目：映射器和化简器。映射器应用程序将每个单词流式传输到控制台，化简器应用程序对从文档流式传输的单词进行计数。映射器和化简器都从标准输入流 (stdin) 逐行读取字符，并写入到标准输出流 (stdout)。

**创建 C# 控制台应用程序**

1. 打开 Visual Studio 2013。
2. 依次单击“文件”、“新建”和“项目”。
3. 键入或选择以下值：

	<table border="1">
<tr><td>字段</td><td>值</td></tr>
<tr><td>模板</td><td>Visual C#/Windows/控制台应用程序</td></tr>
<tr><td>Name</td><td>WordCountMapper</td></tr>
<tr><td>位置</td><td>C:\Tutorials</td></tr>
<tr><td>解决方案名称</td><td>WordCount</td></tr>
</table>
	
4. 单击“确定”以创建该项目。

**创建映射器程序**

5. 在解决方案资源管理器中，右键单击“Program.cs”，然后单击“重命名”。
6. 将该文件重命名为 **WordCountMapper.cs**，然后按 **ENTER**。
7. 单击“是”以确认重命名所有引用。
8. 双击 **WordCountMapper.cs** 将它打开。
9. 添加以下 **using** 语句：

		using System.IO;

10. 将 **Main()** 函数替换为以下内容：

		static void Main(string[] args)
		{
		    if (args.Length > 0)
		    {
		        Console.SetIn(new StreamReader(args[0]));
		    }
		
		    string line;
		    string[] words;
		
		    while ((line = Console.ReadLine()) != null)
		    {
		        words = line.Split(' ');
		
		        foreach (string word in words)
		            Console.WriteLine(word.ToLower());
		    }
		}

11. 单击“生成”，然后单击“生成解决方案”以编译该映射器程序。


**创建化简器程序**

1. 在 Visual Studio 2013 中，依次单击“文件”、“添加”、“新项目”。
2. 键入或选择以下值：

	<table border="1">
<tr><td>字段</td><td>值</td></tr>
<tr><td>模板</td><td>Visual C#/Windows/控制台应用程序</td></tr>
<tr><td>Name</td><td>WordCountReducer</td></tr>
<tr><td>位置</td><td>C:\Tutorials\WordCount</td></tr>
</table>
3. 清除“创建解决方案的目录”复选框，然后单击“确定”以创建项目。
4. 在解决方案资源管理器中，右键单击“Program.cs”，然后单击“重命名”。
5. 将该文件重命名为 **WordCountReducer.cs**，然后按 **ENTER**。
7. 单击“是”以确认重命名所有引用。
8. 双击 **WordCountReducer.cs** 将它打开。
9. 添加以下 **using** 语句：

		using System.IO;

10. 将 **Main()** 函数替换为以下内容：

		static void Main(string[] args)
		{
		    string word, lastWord = null;
		    int count = 0;
		
		    if (args.Length > 0)
		    {
		        Console.SetIn(new StreamReader(args[0]));
		    }
		
		    while ((word = Console.ReadLine()) != null)
		    {
		        if (word != lastWord)
		        {
		            if(lastWord != null)
		                Console.WriteLine("{0}[{1}]", lastWord, count);
		
		            count = 1;
		            lastWord = word;
		        }
		        else
		        {
		            count += 1; 
		        }
		    }
		    Console.WriteLine(count);
		}

11. 单击“生成”，然后单击“生成解决方案”以编译该化简器程序。

映射器和化简器可执行文件位于：

- C:\\Tutorials\\WordCount\\WordCountMapper\\bin\\Debug\\WordCountMapper.exe
- C:\\Tutorials\\WordCount\\WordCountReducer\\bin\\Debug\\WordCountReducer.exe


##<a name="test"></a>在模拟器中测试该程序

执行以下操作以在 HDInsight Emulator 中测试该程序：

1. 将数据上载到模拟器的文件系统
2. 将映射器和化简器应用程序上载到模拟器的文件系统
3. 提交单词计数 MapReduce 作业
4. 检查作业状态
5. 检索作业结果

默认情况下，HDInsight Emulator 使用 HDFS 作为文件系统。（可选）你可以将 HDInsight Emulator 配置为使用 Azure Blob 存储。有关详细信息，请参阅 [HDInsight Emulator 入门][hdinsight-emulator-wasb]。在本部分中，你将使用 HDFS **copyFromLocal** 命令上载文件。下一部分说明如何使用 Azure PowerShell 上载文件。有关其他方法，请参阅[将数据上载到 HDInsight][hdinsight-upload-data]。

本教程使用以下文件夹结构：

<table border="1"><tr><td>文件夹</td><td>注意</td></tr><tr><td>\\WordCount</td><td>单词计数项目所在的根文件夹。</td></tr><tr><td>\\WordCount\\Apps</td><td>映射器和化简器可执行文件所在的文件夹。</td></tr><tr><td>\\WordCount\\Input</td><td>MapReduce 源文件所在的夹。</td></tr><tr><td>\\WordCount\\Output</td><td>MapReduce 输出文件所在的文件夹。</td></tr><tr><td>\\WordCount\\MRStatusOutput</td><td>作业输出文件夹。</td></tr></table></br>

本教程使用位于 %hadoop_home% 目录中的 .txt 文件。

> [AZURE.NOTE]Hadoop HDFS 命令区分大小写。

**将文本文件复制到模拟器的文件系统**

1. 在 Hadoop 命令行窗口中，运行以下命令以生成输入文件的目录：

		hadoop fs -mkdir /WordCount/
		hadoop fs -mkdir /WordCount/Input

	此处使用的路径是相对路径。它等效于以下命令：

		hadoop fs -mkdir hdfs://localhost:8020/WordCount/Input

2. 运行以下命令，将一些文本文件复制到 HDFS 中的输入文件夹：

		hadoop fs -copyFromLocal %hadoop_home%\share\doc\hadoop\common*.txt \WordCount\Input

3. 运行以下命令以列出已上载的文件：

		hadoop fs -ls \WordCount\Input

	


**将映射器和化简器部署到模拟器的文件系统**

1. 从桌面打开 Hadoop 命令行，并在 HDFS 中创建 /Apps 文件夹：

		hadoop fs -mkdir /WordCount/Apps

2. 运行以下命令：

		hadoop fs -copyFromLocal C:\Tutorials\WordCount\WordCountMapper\bin\Debug\WordCountMapper.exe /WordCount/Apps/WordCountMapper.exe
		hadoop fs -copyFromLocal C:\Tutorials\WordCount\WordCountReducer\bin\Debug\WordCountReducer.exe /WordCount/Apps/WordCountReducer.exe

3. 运行以下命令以列出已上载的文件：

		hadoop fs -ls /WordCount/Apps

	你应看到两个 .exe 文件。


**使用 Azure PowerShell 运行 MapReduce 作业**

1. 打开 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。 
3. 运行以下命令以设置变量：

		$clusterName = "http://localhost:50111"
		
		$mrMapper = "WordCountMapper.exe"
		$mrReducer = "WordCountReducer.exe"
		$mrMapperFile = "/WordCount/Apps/WordCountMapper.exe"
		$mrReducerFile = "/WordCount/Apps/WordCountReducer.exe"
		$mrInput = "/WordCount/Input/"
		$mrOutput = "/WordCount/Output"
		$mrStatusOutput = "/WordCount/MRStatusOutput"

	HDInsight Emulator 群集名称是 http://localhost:50111。

4. 运行以下命令以定义流式处理作业：

		$mrJobDef = New-AzureHDInsightStreamingMapReduceJobDefinition -JobName mrWordCountStreamingJob -StatusFolder $mrStatusOutput -Mapper $mrMapper -Reducer $mrReducer -InputPath $mrInput -OutputPath $mrOutput
		$mrJobDef.Files.Add($mrMapperFile)
		$mrJobDef.Files.Add($mrReducerFile)

5. 运行以下命令以创建凭据对象：

		$creds = Get-Credential -Message "Enter password" -UserName "hadoop"

	系统将提示你输入密码。密码可以是任意字符串。用户名必须是“hadoop”。

6. 运行以下命令以提交 MapReduce 作业并等待作业完成：
		
		$mrJob = Start-AzureHDInsightJob -Cluster $clusterName -Credential $creds -JobDefinition $mrJobDef
		Wait-AzureHDInsightJob -Credential $creds -job $mrJob -WaitTimeoutInSeconds 3600

	作业完成后，你将获得如下输出：

		StatusDirectory : /WordCount/MRStatusOutput
		ExitCode        : 
		Name            : mrWordCountStreamingJob
		Query           : 
		State           : Completed
		SubmissionTime  : 11/15/2013 7:18:16 PM
		Cluster         : http://localhost:50111
		PercentComplete : map 100%  reduce 100%
		JobId           : job_201311132317_0034
		
	你可以在输出中看到作业 ID，例如 *job-201311132317-0034*。

**检查作业状态**

1. 从桌面单击“Hadoop YARN 状态”，或者浏览到 **http://localhost:50030/jobtracker.jsp**。2. 使用作业 ID 在“正在运行”或“已完成”类别下查找该作业。 
3. 如果作业失败，你可以在“失败”类别下找到该作业。你也可以打开作业详细信息，找到一些有用的调试信息。


**显示 HDFS 的输出**

1. 打开 Hadoop 命令行。
2. 运行以下命令以显示输出：

		hadoop fs -ls /WordCount/Output/
		hadoop fs -cat /WordCount/Output/part-00000

	你可以在命令结尾处追加“|more”以获取页面视图。

##<a id="upload"></a>将数据上载到 Azure Blob 存储
Azure HDInsight 将 Azure Blob 存储用作默认文件系统。你可以将 HDInsight 群集配置为将其他 Blob 存储用于数据文件。在本部分中，你将创建 Azure 存储帐户并将数据文件上载到 Blob 存储。数据文件是 %hadoop_home%\\share\\doc\\hadoop\\common 目录中的 .txt 文件。


**创建存储帐户和容器**

1. 打开 Azure PowerShell。
2. 设置变量，然后运行命令：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName = "<AzureStorageAccountName>"  
		$containerName = "<ContainerName>"
		$location = "<MicrosoftDataCenter>"  # For example, "East US"

3. 运行以下命令以创建存储帐户，并在该帐户上创建 Blob 存储容器

		# Select an Azure subscription
		Select-AzureSubscription $subscriptionName
		
		# Create a Storage account
		New-AzureStorageAccount -StorageAccountName $storageAccountName -location $location
				
		# Create a Blob storage container
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$destContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  
		New-AzureStorageContainer -Name $containerName -Context $destContext

4. 运行以下命令以验证存储帐户和容器：

		Get-AzureStorageAccount -StorageAccountName $storageAccountName
		Get-AzureStorageContainer -Context $destContext

**上载数据文件**

1. 在 Azure PowerShell 窗口中，为本地和目标文件夹设置值：

		$localFolder = "C:\hdp\hadoop-2.4.0.2.1.3.0-1981\share\doc\hadoop\common"
		$destFolder = "WordCount/Input"

	请注意，本地源文件文件夹是 **C:\\hdp\\hadoop-2.4.0.2.1.3.0-1981\\share\\doc\\hadoop\\common**，目标文件夹是 **WordCount/Input**。源位置是在 HDInsight Emulator 上的 .txt 文件的位置。目标是反映在 Azure Blob 容器下的文件夹结构。

3. 运行以下命令以获取源文件文件夹中的 .txt 文件的列表：

		# Get a list of the .txt files
		$filesAll = Get-ChildItem $localFolder
		$filesTxt = $filesAll | where {$_.Extension -eq ".txt"}
		
5. 运行以下代码段以复制文件：

		# Copy the files from the local workstation to the Blob container        
		foreach ($file in $filesTxt){
		 
		    $fileName = "$localFolder\$file"
		    $blobName = "$destFolder/$file"
		
		    write-host "Copying $fileName to $blobName"
		
		    Set-AzureStorageBlobContent -File $fileName -Container $containerName -Blob $blobName -Context $destContext
		}

6. 运行以下命令以列出已上载的文件：

		# List the uploaded files in the Blob storage container
		Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $destFolder


**上载单词计数应用程序**

1. 在 Azure PowerShell 窗口中设置以下变量：

		$mapperFile = "C:\Tutorials\WordCount\WordCountMapper\bin\Debug\WordCountMapper.exe"
		$reducerFile = "C:\Tutorials\WordCount\WordCountReducer\bin\Debug\WordCountReducer.exe"
		$blobFolder = "WordCount/Apps"

	请注意，目标文件夹是 **WordCount/Apps**，这是将反映在 Azure Blob 容器中的结构。

2. 运行以下命令以复制应用程序：

		Set-AzureStorageBlobContent -File $mapperFile -Container $containerName -Blob "$blobFolder/WordCountMapper.exe" -Context $destContext
		Set-AzureStorageBlobContent -File $reducerFile -Container $containerName -Blob "$blobFolder/WordCountReducer.exe" -Context $destContext

3. 运行以下命令以列出已上载的文件：

		# List the uploaded files in the Blob storage container
		Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $blobFolder

	你应看到这两个应用程序文件在该处列出。


##<a name="run"></a>在 Azure HDInsight 中运行 MapReduce 作业

本部分提供的 Azure PowerShell 脚本将执行与 MapReduce 作业运行有关的所有任务。任务列表包括：

1. 设置 HDInsight 群集
	
	1. 创建将用作默认 HDInsight 群集文件系统的存储帐户
	2. 创建 Blob 存储容器 
	3. 创建 HDInsight 群集

2. 提交 MapReduce 作业

	1. 创建流式处理 MapReduce 作业定义
	2. 提交 MapReduce 作业
	3. 等待作业完成
	4. 显示标准错误
	5. 显示标准输出

3. 删除群集

	1. 删除 HDInsight 群集
	2. 删除用作默认 HDInsight 群集文件系统的存储帐户


**运行 Azure PowerShell 脚本**

1. 打开记事本。
2. 复制并粘贴以下代码：
		
		# ====== STORAGE ACCOUNT AND HDINSIGHT CLUSTER VARIABLES ======
		$subscriptionName = "<AzureSubscriptionName>"
		$stringPrefix = "<StringForPrefix>"     ### Prefix to cluster, Storage account, and container names
		$storageAccountName_Data = "<TheDataStorageAccountName>"
		$containerName_Data = "<TheDataBlobStorageContainerName>"
		$location = "<MicrosoftDataCenter>"     ### Must match the data storage account location
		$clusterNodes = 1
		
		$clusterName = $stringPrefix + "hdicluster"
		
		$storageAccountName_Default = $stringPrefix + "hdistore"
		$containerName_Default =  $stringPrefix + "hdicluster"

		# ====== THE STREAMING MAPREDUCE JOB VARIABLES ======
		$mrMapper = "WordCountMapper.exe"
		$mrReducer = "WordCountReducer.exe"
		$mrMapperFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Apps/WordCountMapper.exe"
		$mrReducerFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Apps/WordCountReducer.exe"
		$mrInput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Input/"
		$mrOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Output/"
		$mrStatusOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/MRStatusOutput/"
		
		Select-AzureSubscription $subscriptionName
		
		#====== CREATE A STORAGE ACCOUNT ======
		Write-Host "Create a storage account" -ForegroundColor Green
		New-AzureStorageAccount -StorageAccountName $storageAccountName_Default -location $location
		
		#====== CREATE A BLOB STORAGE CONTAINER ======
		Write-Host "Create a Blob storage container" -ForegroundColor Green
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$destContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Default –StorageAccountKey $storageAccountKey_Default
		
		New-AzureStorageContainer -Name $containerName_Default -Context $destContext
		
		#====== CREATE AN HDINSIGHT CLUSTER ======
		Write-Host "Create an HDInsight cluster" -ForegroundColor Green
		$storageAccountKey_Data = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
		
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
		    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
		    Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Data.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Data
		
		Select-AzureSubscription $subscriptionName
		New-AzureHDInsightCluster -Name $clusterName -Location $location -Config $config
		
		#====== CREATE A STREAMING MAPREDUCE JOB DEFINITION ======
		Write-Host "Create a streaming MapReduce job definition" -ForegroundColor Green
		
		$mrJobDef = New-AzureHDInsightStreamingMapReduceJobDefinition -JobName mrWordCountStreamingJob -StatusFolder $mrStatusOutput -Mapper $mrMapper -Reducer $mrReducer -InputPath $mrInput -OutputPath $mrOutput
		$mrJobDef.Files.Add($mrMapperFile)
		$mrJobDef.Files.Add($mrReducerFile)
		
		#====== RUN A STREAMING MAPREDUCE JOB ======
		Write-Host "Run a streaming MapReduce job" -ForegroundColor Green
		Select-AzureSubscription $subscriptionName
		$mrJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $mrJobDef 
		Wait-AzureHDInsightJob -Job $mrJob -WaitTimeoutInSeconds 3600 
		
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardError 
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardOutput
		
		#====== DELETE THE HDINSIGHT CLUSTER ======
		Write-Host "Delete the HDInsight cluster" -ForegroundColor Green
		Select-AzureSubscription $subscriptionName
		Remove-AzureHDInsightCluster -Name $clusterName 
		
		#====== DELETE THE STORAGE ACCOUNT ======
		Write-Host "Delete the storage account" -ForegroundColor Green
		Remove-AzureStorageAccount -StorageAccountName $storageAccountName_Default

3. 设置此脚本中的前四个变量。**$stringPrefix** 变量用于对 HDInsight 群集名称、存储帐户名称和 Blob 存储容器名称的指定字符串加上前缀。因为这些项目的名称必须为 3 到 24 个字符，请确保你指定的字符串和此脚本使用的名称，合计不超过名称的字符数限制。**$stringPrefix** 必须全部为小写。

	**$storageAccountName_Data** 和 **$containerName_Data** 变量是你已在前面步骤中创建的存储帐户和容器。因此，你必须提供这些项的名称。这些项用于存储数据文件和应用程序。**$location** 变量必须与数据存储帐户位置匹配。

4. 复查其余变量。
5. 保存脚本文件。
6. 打开 Azure PowerShell。
7. 运行以下命令，将执行策略设为 RemoteSigned：

		PowerShell -File <FileName> -ExecutionPolicy RemoteSigned

8. 出现提示时，输入 HDInsight 群集的用户名和密码。请确保密码字段至少为 10 个字符且必须至少包含一个大写字母、一个小写字母、一个数字和一个特殊字符。如果你不想让系统提示你输入凭据，请参阅[在 Windows PowerShell 中使用密码、安全字符串和凭据][powershell-PSCredential]。

有关提交 Hadoop 流式处理作业的 HDInsight .NET SDK 示例，请参阅[以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]。


##<a name="retrieve"></a>检索 MapReduce 作业输出
本节演示如何下载和显示输出。有关在 Excel 中显示结果的信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][hdinsight-ODBC] 和[利用 Power Query 将 Excel 连接到 HDInsight][hdinsight-power-query]。


**检索输出**

1. 打开 Azure PowerShell 窗口。
2. 设置值，然后运行以下命令：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName = "<TheDataStorageAccountName>"
		$containerName = "<TheDataBlobStorageContainerName>"
		$blobName = "WordCount/Output/part-00000"
	
3. 运行以下命令以创建 Azure 存储上下文对象：
		
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$storageContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  

4. 运行以下命令以下载并显示输出：

		Get-AzureStorageBlobContent -Container $ContainerName -Blob $blobName -Context $storageContext -Force
		cat "./$blobName" | findstr "there"

	

##<a id="nextsteps"></a>后续步骤
在本教程中，你已学习如何执行以下操作：开发 Hadoop 流式处理 MapReduce 作业、在 HDInsight Emulator 中测试应用程序、编写 Azure PowerShell 脚本以设置 HDInsight 群集以及在群集上运行 MapReduce 作业。若要了解更多信息，请参阅下列文章：

- [Azure HDInsight 入门](/documentation/articles/hdinsight-get-started)
- [HDInsight Emulator 入门][hdinsight-get-started-emulator]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上载到 HDInsight][hdinsight-upload-data]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/


[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically

[hdinsight-get-started-emulator]: /documentation/articles/hdinsight-get-started-emulator
[hdinsight-emulator-wasb]: /documentation/articles/hdinsight-get-started-emulator/#blobstorage
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig
[hdinsight-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query

[powershell-PSCredential]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx
[Powershell-install-configure]: /documentation/articles/install-configure-powershell

[image-hdi-wordcountdiagram]: ./media/hdinsight-hadoop-develop-deploy-streaming-jobs/HDI.WordCountDiagram.gif "MapReduce 单词计数应用程序流"

<!---HONumber=66-->