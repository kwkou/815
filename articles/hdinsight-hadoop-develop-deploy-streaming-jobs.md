<properties linkid="manage-services-hdinsight-develop-hadoop-streaming-programs-for-hdinsight" urlDisplayName="" pageTitle="Develop C# Hadoop streaming programs for HDInsight | Azure" metaKeywords="hdinsight hdinsight development, hadoop development, dhinsight deployment, development, deployment, tutorial, MapReduce" description="Learn how to develop Hadoop streaming MapReduce programs in C#, and how to deploy them to Azure HDInsight." metaCanonical="" services="hdinsight" documentationCenter="" title="Develop C# Hadoop streaming programs for HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 为 HDInsight 开发 C\# Hadoop 流程序

Hadoop 向 MapReduce 提供了一个流式 API，利用它，你可以采用 Java 之外的其他语言来编写映射函数和化简函数。本教程指导你完成端到端方案：从在 HDInsight Emulator 上使用 C\# 开发/测试 Hadoop 流 MapReduce 程序，到在 Azure HDInsight 上运行 MapReduce 作业并检索结果。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   安装 Azure HDInsight Emulator。有关说明，请参阅[开始使用 HDInsight Emulator][]。
-   在模拟器计算机上安装 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]
-   获取 Azure 订阅。有关说明，请参阅[购买选项][]、[免费试用][]。

## 本文内容

-   [使用 C\# 开发单词计数 Hadoop 流程序][]
-   [在模拟器中测试该程序][]
-   [将数据和应用程序上载到 Azure Blob 存储][]
-   [在 Azure HDInsight 中运行 MapReduce 程序][]
-   [检索 MapReduce 结果][]
-   [后续步骤][]

## 使用 C\# 开发单词计数 Hadoop 流程序

单词计数解决方案包含两个控制台应用程序项目：映射器和化简器。映射器应用程序将每个单词流式传输到控制台，化简器应用程序对从文档流式传输的单词进行计数。映射器和化简器都从标准输入流 (stdin) 逐行读取字符，并写入到标准输出流 (stdout)。

**创建 C\# 控制台应用程序**

1.  打开 Visual Studio 2013。
2.  依次单击“文件” 、“新建” 和“项目” 。
3.  键入或选择以下值：

	<table border="1">
	<tr><td>字段</td><td>值</td></tr>
	<tr><td>模板</td><td>Visual C#/Windows/控制台应用程序</td></tr>
	<tr><td>名称</td><td>WordCountMapper</td></tr>
	<tr><td>位置</td><td>C:\Tutorials</td></tr>
	<tr><td>解决方案名称</td><td>WordCount</td></tr>
	</table>

4.  单击“确定” 以创建该项目。

**创建映射器程序**

1.  在解决方案资源管理器中，右键单击“Program.cs” ，然后单击“重命名” 。
2.  将该文件重命名为 **WordCountMapper.cs**，然后按 **ENTER**。
3.  单击“是” 以确认重命名所有引用。
4.  双击 **WordCountMapper.cs** 以打开它。
5.  添加以下 using 语句：

        using System.IO;

6.  将 Main() 函数替换为以下内容：

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

7.  单击“生成” ，然后单击“生成解决方案” 以编译该映射器程序。

**创建化简器程序**

1.  在 Visual Studio 2013 中，依次单击“文件” 、“添加” 和“新项目” 。
2.  键入或选择以下值：

	<table border="1">
	<tr><td>字段</td><td>值</td></tr>
	<tr><td>模板</td><td>Visual C#/Windows/控制台应用程序</td></tr>
	<tr><td>名称</td><td>WordCountReducer</td></tr>
	<tr><td>位置</td><td>C:\Tutorials\WordCount</td></tr>
	</table>

3.  单击“确定” 以创建项目。
4.  在解决方案资源管理器中，右键单击“Program.cs” ，然后单击“重命名” 。
5.  将该文件重命名为 **WordCountReducer.cs**，然后按 **ENTER**。
6.  单击“是” 以确认重命名所有引用。
7.  双击 **WordCountReducer.cs** 以打开它。
8.  添加以下 using 语句：

        using System.IO;

9.  将 Main() 函数替换为以下内容：

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

10. 单击“生成” ，然后单击“生成解决方案” 以编译该解决方案。

映射器和化简器可执行文件位于：

-   C:\\Tutorials\\WordCount\\WordCountMapper\\bin\\Debug\\WordCountMapper.exe
-   C:\\Tutorials\\WordCount\\WordCountReducer\\bin\\Debug\\WordCountReducer.exe

## 在模拟器中测试该程序

本节包含以下过程：

1.  将数据上载到模拟器 HDFS
2.  将映射器和化简器上载到模拟器 HDFS
3.  提交单词计数 MapReduce 作业
4.  检查作业状态
5.  检索作业结果

默认情况下，HDInsight Emulator 使用 HDFS 作为默认文件系统。（可选）你可以将HDInsight Emulator 配置为使用 Azure Blob 存储。有关详细信息，请参阅 [HDInsight Emulator 入门][]。在本节中，你将使用 HDFS copyFromLocal 命令上载文件。下一节说明如何使用 Azure PowerShell 上载文件。有关其他方法，请参阅[将数据上载到 HDInsight][]。

本教程使用以下文件夹结构：

	<table border="1">
	<tr><td>文件夹</td><td>说明</td></tr>
	<tr><td>\WordCount</td><td>单词计数项目的根文件夹。 </td></tr>
	<tr><td>\WordCount\Apps</td><td>映射器和化简器可执行文件所在的文件夹。</td></tr>
	<tr><td>\WordCount\Input</td><td>MapReduce 源文件文件夹。</td></tr>
	<tr><td>\WordCount\Output</td><td>MapReduce 输出文件文件夹。</td></tr>
	<tr><td>\WordCount\MRStatusOutput</td><td>作业输出文件夹。</td></tr>
	</table>

本教程使用位于 %hadoop\_home% 目录中的 .txt 文件。

> [WACOM.NOTE] Hadoop HDFS 命令区分大小写。

**将文本文件复制到模拟器 HDFS**

1.  在 Hadoop 命令行窗口中，运行以下命令以生成输入文件的目录：

        hadoop fs -mkdir /WordCount/Input

    此处使用的路径是相对路径。它等效于以下命令：

        hadoop fs -mkdir hdfs://localhost:8020/WordCount/Input

2.  运行以下命令，将一些文本文件复制到 HDFS 中的输入文件夹：

        hadoop fs -copyFromLocal %hadoop_home%\*.txt \WordCount\Input

3.  运行以下命令以列出已上载的文件：

        hadoop fs -ls \WordCount\Input

    你会看到大约八个 .txt 文件。

**将映射器和化简器部署到模拟器 HDFS**

1.  从桌面打开 Hadoop 命令行。
2.  运行以下命令：

        hadoop fs -copyFromLocal C:\Tutorials\WordCount\WordCountMapper\bin\Debug\WordCountMapper.exe /WordCount/Apps/WordCountMapper.exe
        hadoop fs -copyFromLocal C:\Tutorials\WordCount\WordCountReducer\bin\Debug\WordCountReducer.exe /WordCount/Apps/WordCountReducer.exe

3.  运行以下命令以列出已上载的文件

        hadoop fs -lsr /WordCount/Apps

    你应看到两个 .exe 文件。

**使用 HDInsight PowerShell 运行 MapReduce 作业**

1.  打开 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。
2.  运行以下命令以设置变量：

        $clusterName = "http://localhost:50111"

        $mrMapper = "WordCountMapper.exe"
        $mrReducer = "WordCountReducer.exe"
        $mrMapperFile = "/WordCount/Apps/WordCountMapper.exe"
        $mrReducerFile = "/WordCount/Apps/WordCountReducer.exe"
        $mrInput = "/WordCount/Input/"
        $mrOutput = "/WordCount/Output"
        $mrStatusOutput = "/WordCount/MRStatusOutput"

    HDInsight Emulator 群集名称是 <http://localhost:50111>。

3.  运行以下命令以定义流式处理作业：

        $mrJobDef = New-AzureHDInsightStreamingMapReduceJobDefinition -JobName mrWordCountStreamingJob -StatusFolder $mrStatusOutput -Mapper $mrMapper -Reducer $mrReducer -InputPath $mrInput -OutputPath $mrOutput
        $mrJobDef.Files.Add($mrMapperFile)
        $mrJobDef.Files.Add($mrReducerFile)

4.  运行以下命令以创建凭据对象：

        $creds = Get-Credential -Message "Enter password" -UserName "hadoop"

    系统将提示你输入密码。密码可以是任意字符串。用户名必须是“hadoop”。

5.  运行以下命令以提交 MapReduce 作业并等待作业完成：

        $mrJob = Start-AzureHDInsightJob -Cluster $clusterName -Credential $creds -JobDefinition $mrJobDef
        Wait-AzureHDInsightJob -Credential $creds -job $mrJob -WaitTimeoutInSeconds 3600

    完成后，你将获得如下输出：

        StatusDirectory :/WordCount/MRStatusOutput
        ExitCode        : 
        Name            :mrWordCountStreamingJob
        Query           : 
        State           :Completed
        SubmissionTime  :11/15/2013 7:18:16 PM
        Cluster         :http://localhost:50111
        PercentComplete :map 100%  reduce 100%
        JobId           :job_201311132317_0034

    因此，你知道作业 ID 是 job-201311132317-0034（此处为举例）。

**检查作业状态**

1.  从桌面单击“Hadoop MapReduce 状态” ，或者浏览到 **<http://localhost:50030/jobtracker.jsp>**。
2.  使用作业 ID 在以下三部分之一下查找该作业：“已完成的作业” 、“正在运行的作业” 、“已停用的作业” 。
3.  如果作业失败，你可以打开作业详细信息，找到一些有用的调试信息。

**显示 HDFS 的输出**

1.  打开 Hadoop 命令行。
2.  运行以下命令以显示输出：

        hadoop fs -ls /WordCount/Output/
        hadoop fs -cat /WordCount/Output/part-00000

    你可以在命令结尾处追加“|more”以获取页面视图。

## 将数据上载到 Azure Blob 存储

Azure HDInsight 将 Azure Blob 存储用作默认文件系统。你可以将 HDInsight 群集配置为将其他 Blob 存储用于数据文件。在本节中，你将创建存储帐户并将数据文件上载到 Blob 存储。数据文件是 %hadoop\_home% 目录中的 txt 文件。

**创建 Blob 存储和容器**

1.  打开 Azure PowerShell。
2.  设置变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<AzureStorageAccountName>"  
        $containerName = "<ContainerName>"
        $location = "<MicrosoftDataCenter>"  # 例如，“East US”

3.  运行以下命令以创建存储帐户，并在该帐户上创建 Blob 存储容器

        # 选择 Azure 订阅
        Select-AzureSubscription $subscriptionName

        # 创建存储帐户
        New-AzureStorageAccount -StorageAccountName $storageAccountName -location $location

        # 创建 Blob 存储容器
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $destContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  
        New-AzureStorageContainer -Name $containerName -Context $destContext

4.  运行以下命令以验证存储帐户和容器：

        Get-AzureStorageAccount -StorageAccountName $storageAccountName
        Get-AzureStorageContainer -Context $destContext

**上载数据文件**

1.  打开 Azure PowerShell。
2.  设置前三个变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<AzureStorageAccountName>"  
        $containerName = "<ContainerName>"

        $localFolder = "c:\Hadoop\hadoop-1.1.0-SNAPSHOT"
        $destFolder = "WordCount/Input"

    请注意，源文件文件夹是 **c:\\Hadoop\\hadoop-1.1.0-SNAPSHOT**，目标文件夹是 **WordCount/Input**。

3.  运行以下命令以获取源文件文件夹中的 txt 文件的列表：

        # 获取 txt 文件的列表
        $filesAll = Get-ChildItem $localFolder
        $filesTxt = $filesAll | where {$_.Extension -eq ".txt"}

4.  运行以下命令以创建存储上下文对象：

        # 创建存储上下文对象
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

5.  运行以下命令以复制文件：

        # 将本地工作站中的文件复制到 Blob 容器        
        foreach ($file in $filesTxt){

        $fileName = "$localFolder\$file"
        $blobName = "$destFolder/$file"

        write-host "Copying $fileName to $blobName"

        Set-AzureStorageBlobContent -File $fileName -Container $containerName -Blob $blobName -Context $destContext
        }

6.  运行以下命令以列出已上载的文件：

        # 列出 Blob 存储容器中已上载的文件
        Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $destFolder

**上载单词计数应用程序**

1.  打开 Azure PowerShell。
2.  设置前三个变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<AzureStorageAccountName>"  
        $containerName = "<ContainerName>"

        $mapperFile = "C:\Tutorials\WordCount\WordCountMapper\bin\Debug\WordCountMapper.exe"
        $reducerFile = "C:\Tutorials\WordCount\WordCountReducer\bin\Debug\WordCountReducer.exe"
        $blobFolder = "WordCount/Apps"

    请注意，目标文件夹是 **WordCount/Apps**。

3.  运行以下命令以创建存储上下文对象：

        # 创建存储上下文对象
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

4.  运行以下命令以复制应用程序：

        Set-AzureStorageBlobContent -File $mapperFile -Container $containerName -Blob "$blobFolder/WordCountMapper.exe" -Context $destContext
        Set-AzureStorageBlobContent -File $reducerFile -Container $containerName -Blob "$blobFolder/WordCountReducer.exe" -Context $destContext

5.  运行以下命令以列出已上载的文件：

        # 列出 Blob 存储容器中已上载的文件
        Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $blobFolder

    你应看到这两个文件在该处列出。

## 在 Azure HDInsight 中运行 MapReduce 作业

下面的 PowerShell 脚本执行以下任务：

1.  设置 HDInsight 群集

    1.  创建将用作默认 HDInsight 群集文件系统的存储帐户
    2.  创建 Blob 存储容器
    3.  创建 HDInsight 群集

2.  提交 MapReduce 作业

    1.  创建流式处理 MapReduce 作业定义
    2.  提交 MapReduce 作业
    3.  等待作业完成
    4.  显示标准错误
    5.  显示标准输出

3.  删除群集

    1.  删除 HDInsight 群集
    2.  删除用作默认 HDInsight 群集文件系统的存储帐户

**运行 PowerShell 脚本**

1.  打开记事本。
2.  复制并粘贴以下代码：

        # 存储帐户和 HDInsight 群集变量
        $subscriptionName = "<AzureSubscriptionName>"
        $serviceNameToken = "<ServiceNameTokenString>"
        $storageAccountName_Data = "<TheDataStorageAccountName>"
        $containerName_Data = "<TheDataBlobStorageContainerName>"
        $location = "East US"     ### 必须与数据存储帐户位置匹配
        $clusterNodes = 1

        $clusterName = $serviceNameToken + "hdicluster"

        $storageAccountName_Default = $serviceNameToken + "hdistore"
        $containerName_Default =  $serviceNameToken + "hdicluster"

        # 流式处理 MapReduce 作业变量
        $mrMapper = "WordCountMapper.exe"
        $mrReducer = "WordCountReducer.exe"
        $mrMapperFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Apps/WordCountMapper.exe"
        $mrReducerFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/Apps/WordCountReducer.exe"
        $mrInput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Input/"
        $mrOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Output/"
        $mrStatusOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/MRStatusOutput/"

        Select-AzureSubscription $subscriptionName

        #=============================
        # 创建存储帐户
        Write-Host "Create a storage account" -ForegroundColor Green
        New-AzureStorageAccount -StorageAccountName $storageAccountName_Default -location $location

        #=============================
        # 创建 Blob 存储容器
        Write-Host "Create a Blob storage container" -ForegroundColor Green
        $storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
        $destContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Default –StorageAccountKey $storageAccountKey_Default

        New-AzureStorageContainer -Name $containerName_Default -Context $destContext

        #=============================
        # 创建 HDInsight 群集
        Write-Host "Create an HDInsight cluster" -ForegroundColor Green
        $storageAccountKey_Data = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }

        $config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
        Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
        Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Data.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Data

        New-AzureHDInsightCluster -Subscription $subscriptionName  -Name $clusterName -Location $location -Config $config

        #=============================
        # 创建流式处理 MapReduce 作业定义
        Write-Host "Create a streaming MapReduce job definition" -ForegroundColor Green

        $mrJobDef = New-AzureHDInsightStreamingMapReduceJobDefinition -JobName mrWordCountStreamingJob -StatusFolder $mrStatusOutput -Mapper $mrMapper -Reducer $mrReducer -InputPath $mrInput -OutputPath $mrOutput
        $mrJobDef.Files.Add($mrMapperFile)
        $mrJobDef.Files.Add($mrReducerFile)

        #=============================
        # 运行流式处理 MapReduce 作业
        Write-Host "Run a streaming MapReduce job" -ForegroundColor Green
        $mrJob = Start-AzureHDInsightJob -Cluster $clusterName -Subscription $subscriptionName -JobDefinition $mrJobDef 
        Wait-AzureHDInsightJob -Subscription $subscriptionName -Job $mrJob -WaitTimeoutInSeconds 3600 

        Get-AzureHDInsightJobOutput -Cluster $clusterName -Subscription $subscriptionName -JobId $mrJob.JobId -StandardError 
        Get-AzureHDInsightJobOutput -Cluster $clusterName -Subscription $subscriptionName -JobId $mrJob.JobId -StandardOutput

        #=============================
        # 删除 HDInsight 群集
        Write-Host "Delete the HDInsight cluster" -ForegroundColor Green
        Remove-AzureHDInsightCluster -Name $clusterName -Subscription $subscriptionName 

        # 删除存储帐户
        Write-Host "Delete the storage account" -ForegroundColor Green
        Remove-AzureStorageAccount -StorageAccountName $storageAccountName_Default

3.  设置此脚本中的前四个变量。\$serviceNameToken 将用于 HDInsight 群集名称、存储帐户名称和 Blob 存储容器名称。由于服务名称必须介于 3 到 24 个字符之间，并且此脚本将包含多达 10 字符的字符串追加到名称，因此你必须将字符串限制为 14 个字符或更少字符。必须对 \$serviceNameToken 使用全小写字母。\$storageAccountName\_Data 和 \$containerName\_Data 是用于存储数据文件和应用程序的存储帐户和容器。\$location 必须与数据存储帐户位置匹配。
4.  复查其余变量。
5.  保存脚本文件。
6.  打开 Azure PowerShell。
7.  运行以下命令，将执行策略设为 RemoteSigned：

        PowerShell -File <FileName> -ExecutionPolicy RemoteSigned

8.  出现提示时，输入 HDInsight 群集的用户名和密码。由于你将在脚本末尾删除群集，并且将不再需要用户名和密码，因此用户名和密码可以是任何字符串。如果你不想让系统提示你输入凭据，请参阅[在 Windows PowerShell 中使用密码、安全字符串和凭据][]

有关提交 Hadoop 流式处理作业的 HDInsight .NET SDK 示例，请参阅[以编程方式提交 Hadoop 作业][]。

## 检索 MapReduce 作业输出

本节演示如何下载和显示输出。有关在 Excel 中显示结果的信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][] 和[利用 Power Query 将 Excel 连接到 HDInsight][]。

**检索输出**

1.  打开 Azure PowerShell 窗口。
2.  设置值，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<TheDataStorageAccountName>"
        $containerName = "<TheDataBlobStorageContainerName>"
        $blobName = "WordCount/Output/part-00000"

3.  运行以下命令以创建 Azure 存储上下文对象：

        Select-AzureSubscription $subscriptionName
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  

4.  运行以下命令以下载并显示输出：

        Get-AzureStorageBlobContent -Container $ContainerName -Blob $blobName -Context $storageContext -Force
        cat "./$blobName" | findstr "there"

## 后续步骤

在本教程中，你已学习如何执行以下操作：开发 Hadoop 流式处理 MapReduce 作业、在 HDInsight Emulator 中测试应用程序、编写 PowerShell 脚本以设置 HDInsight 群集以及在群集上运行 MapReduce。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][]
-   [HDInsight Emulator 入门][开始使用 HDInsight Emulator]
-   [为 HDInsight 开发 Java MapReduce 程序][]
-   [将 Azure Blob 存储与 HDInsight 配合使用][]
-   [使用 PowerShell 管理 HDInsight][]
-   [将数据上载到 HDInsight][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 Pig 与 HDInsight 配合使用][]

  [开始使用 HDInsight Emulator]: /en-us/manage/services/hdinsight/get-started-with-windows-azure-hdinsight-emulator/
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [使用 C\# 开发单词计数 Hadoop 流程序]: #develop
  [在模拟器中测试该程序]: #test
  [将数据和应用程序上载到 Azure Blob 存储]: #upload
  [在 Azure HDInsight 中运行 MapReduce 程序]: #run
  [检索 MapReduce 结果]: #retrieve
  [后续步骤]: #nextsteps
  [HDInsight Emulator 入门]: /en-us/manage/services/hdinsight/get-started-with-windows-azure-hdinsight-emulator/#blobstorage
  [将数据上载到 HDInsight]: /en-us/manage/services/hdinsight/howto-upload-data-to-hdinsight/
  [在 Windows PowerShell 中使用密码、安全字符串和凭据]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx
  [以编程方式提交 Hadoop 作业]: ../hdinsight-submit-hadoop-jobs-programmatically/
  [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]: /en-us/manage/services/hdinsight/connect-excel-with-hive-ODBC/
  [利用 Power Query 将 Excel 连接到 HDInsight]: /en-us/manage/services/hdinsight/connect-excel-with-power-query/
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [为 HDInsight 开发 Java MapReduce 程序]: ../hdinsight-develop-deploy-java-mapreduce/
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [使用 PowerShell 管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [将 Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [将 Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
