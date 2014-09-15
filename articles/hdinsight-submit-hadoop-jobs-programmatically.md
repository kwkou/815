<properties linkid="manage-services-hdinsight-submit-hadoop-jobs-programmatically" urlDisplayName="HDInsight Administration" pageTitle="Submit Hadoop jobs programmatically | Azure" metaKeywords="hdinsight, hdinsight administration, hdinsight administration azure, hive, mapreduce, HDInsight .NET SDK, powershell, submit mapreduce jobs, submit hive jobs, development, hadoop, apache" description="Learn how to programmatically submit Hadoop jobs to Azure HDInsight." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="Submit Hadoop jobs programmatically" authors="jgao" />

# 以编程方式提交 Hadoop 作业

在本文中，你将了解如何使用 PowerShell 和 HDInsight .NET SDK 提交 MapReduce 和 Hive 作业。

**必备条件：**

在开始阅读本文前，你必须具有：

-   Azure HDInsight 群集。有关说明，请参见 [HDInsight 入门][]或[设置 HDInsight 群集][]。
-   安装和配置 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。

## 本文内容

-   [使用 PowerShell 提交 MapReduce 作业][]
-   [使用 PowerShell 提交 Hive 作业][]
-   [使用 PowerShell 提交 Sqoop 作业][]
-   [使用 HDInsight .NET SDK 提交 MapReduce 作业][]
-   [使用 HDInsight .NET SDK 提交 Hadoop 流 MapReduce 作业][]
-   [使用 HDInsight .NET SDK 提交 Hive 作业][]
-   [后续步骤][]

## 使用 PowerShell 提交 MapReduce 作业

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关将 PowerShell 用于 HDInsight 的更多信息，请参见[使用 PowerShell 管理 HDInsight][]。

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的应用程序。HDInsight 群集附带一个 jar 文件，位于 *\\example\\jars\\hadoop-examples.jar*，它包含几个 MapReduce 示例。此文件在版本 3.0 的 HDInsight 群集上已重命名为 hadoop-mapreduce-examples.jar。一个示例是计算源文件中的单词频率。在本课中，你将了解如何从工作站使用 PowerShell 来运行单词计数示例。有关开发和运行 MapReduce 作业的详细信息，请参阅[将 MapReduce 与 HDInsight 配合使用][]。

**使用 PowerShell 运行单词计数 MapReduce 程序**

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  通过运行以下 PowerShell 命令设置这两个变量：

        $subscriptionName = "<SubscriptionName>"   
        $clusterName = "<HDInsightClusterName>"

    该订阅是你用于创建 HDInsight 群集的订阅。HDInsight 群集是你要用于运行 MapReduce 作业的群集。

3.  运行以下命令创建 MapReduce 作业定义：

        # 定义字数统计 MapReduce 作业
        $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput" 

    有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。有关 wasb 前缀的详细信息，请参阅 [将 Azure Blob 存储与 HDInsight 配合使用][]。

4.  运行以下命令来运行 MapReduce 作业：

        # 提交 MapReduce 作业
        Select-AzureSubscription $subscriptionName
        $wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition 

    除了 MapReduce 作业定义外，你还提供要运行 MapReduce 作业的 HDInsight 群集名称。

5.  运行以下命令来检查 MapReduce 作业的完成：

        # 等待作业完成
        Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600 

6.  运行以下命令来检查运行 MapReduce 作业时的错误：

        # 获取作业标准错误输出
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError 

    以下屏幕截图显示了成功运行的输出。否则，你将看到一些错误消息。

    ![HDI.GettingStarted.RunMRJob][]

**检索 MapReduce 作业的结果**

1.  打开 **Azure PowerShell**。
2.  通过运行以下 PowerShell 命令设置这三个变量：

        $subscriptionName = "<SubscriptionName>"       
        $storageAccountName = "<StorageAccountName>"
        $containerName = "<ContainerName>"          

    Azure 存储帐户是你在 HDInsight 群集设置过程中指定的。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。Blob 存储容器通常与 HDInsight 群集共享相同的名称，除非你在设置群集时指定其他名称。

3.  运行以下命令以创建 Azure 存储上下文对象：

        # 创建存储帐户上下文对象
        Select-AzureSubscription $subscriptionName
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Select-AzureSubscription 用于在你具有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

4.  运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

        # 获取 Blob 内容
        Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    *example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹上的相同文件夹结构。例如，在以下屏幕快照中，当前文件夹是 C 根文件夹。此文件将下载到 \*C:\\example\\data\\WordCountOutput\* 文件夹。

5.  运行以下命令来打印 MapReduce 作业输出文件：

        cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

    ![HDI.GettingStarted.MRJobOutput][]

    MapReduce 作业将生成一个名为 part-r-00000** 的文件，其中包含单词和计数。此脚本使用 findstr 命令来列出包含“there”的所有单词。

> [WACON.NOTE] 如果你在记事本中打开 ./example/data/WordCountOutput/part-r-00000（MapReduce 作业的多行输出），将会注意到断行未正确呈现。这是正常情况。

## 使用 PowerShell 提交 Hive 作业

Apache [Hive][] 提供了通过类似 SQL 的脚本语言（称作 *HiveQL*）运行 MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。

HDInsight 群集提供了一个名为 *hivesampletable* 的示例 Hive 表。在本节中，你将使用 PowerShell 来运行 Hive 作业，以便列出 Hive 表中的一些数据。

**使用 PowerShell 运行 Hive 作业**

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  在以下命令中设置前两个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   
        $clusterName = "<HDInsightClusterName>"             
        $querystring = "SELECT * FROM hivesampletable WHERE Country='China';"

    \$querystring 是 HiveQL 查询。

3.  运行以下命令以选择 Azure 订阅和要运行 Hive 作业的群集：

        Select-AzureSubscription -SubscriptionName $subscriptionName

4.  提交 hive 作业：

        Use-AzureHDInsightCluster $clusterName
        Invoke-Hive -Query $queryString

    可以使用 -File 开关来指定 HDFS 上的 HiveQL 脚本文件。

有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][]。

## 使用 PowerShell 提交 Sqoop 作业

请参阅[将 Sqoop 与 HDInsight 配合使用][]。

## 使用 HDInsight .NET SDK 提交 MapReduce 作业

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。HDInsight 群集附带一个 jar 文件，位于 *\\example\\jars\\hadoop-examples.jar*，它包含几个 MapReduce 示例。一个示例是计算源文件中的单词频率。在本课中，你将了解如何创建 .NET 应用程序来运行单词计数示例。有关开发和运行 MapReduce 作业的详细信息，请参阅[将 MapReduce 与 HDInsight 配合使用][]。

使用 SDK 设置 HDInsight 群集的步骤如下：

-   安装 HDInsight .NET SDK
-   创建控制台应用程序
-   运行应用程序

**安装 HDInsight .NET SDK**
 可以从 [NuGet][] 安装最新发布的 SDK 版本。下一过程中将显示说明。

**创建 Visual Studio 控制台应用程序**

1.  打开 Visual Studio 2012。

2.  在“文件”菜单中，单击“新建” ，然后单击“项目” 。

3.  从“新建项目”中，键入或选择以下值：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
	<tr>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">属性</th>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">值</th></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">类别</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">模板/Visual C#/Windows</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">模板</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">控制台应用程序</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">名称</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">SubmitMapReduceJob</td></tr>
	</table>

4.  单击“确定” 以创建该项目。

5.  从“工具”菜单中 ，单击“库程序包管理器” ，然后单击“程序包管理器控制台” 。

6.  在控制台中运行下列命令以安装程序包。

        Install-Package Microsoft.WindowsAzure.Management.HDInsight

    此命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。版本应该是 0.11.0.1 或更高。

7.  从解决方案资源管理器中，双击 **Program.cs** 将其打开。

8.  将下列 using 语句添加到文件顶部：

        using System.IO;
        using System.Threading;
        using System.Security.Cryptography.X509Certificates;

        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

        using Microsoft.WindowsAzure.Management.HDInsight;
        using Microsoft.Hadoop.Client;

9.  将以下函数定义添加到类。此函数用于等待 Hadoop 作业完成。

        private static void WaitForJobCompletion(JobCreationResults jobResults, IJobSubmissionClient client)
        {
        JobDetails jobInProgress = client.GetJob(jobResults.JobId);
        while (jobInProgress.StatusCode != JobStatusCode.Completed && jobInProgress.StatusCode != JobStatusCode.Failed)
            {
        jobInProgress = client.GetJob(jobInProgress.JobId);
        Thread.Sleep(TimeSpan.FromSeconds(10));
            }
        }

10. 在 Main() 函数中，复制并粘贴以下代码：

        // 设置变量
        string subscriptionID = "<Azure subscription ID>";
        string certFriendlyName = "<certificate friendly name>";

        string clusterName = "<HDInsight cluster name>";

        string storageAccountName = "<Azure storage account name>";
        string storageAccountKey = "<Azure storage account key>";
        string containerName = "<Blob container name>";

    这些是你需要为该程序设置的所有变量。你可以从 [Azure 管理门户][]获取 Azure 订阅名称。

    有关证书的信息，请参阅[创建并上载 Azure 管理证书][]。配置证书的简单方法是运行 *Get-AzurePublishSettingsFile* 和 *Import-AzurePublishSettingsFile* PowerShell cmdlet。它们将自动创建和上载管理证书。在运行 PowerShell cmdlet 后，可以从工作站打开 *certmgr.msc*，并通过展开“个人/证书”**来查找该证书。由 PowerShell cmdlet 创建的证书的“颁发给”**和“颁发者”**两个字段均为“Azure 工具”**。

    Azure 存储帐户名称是你在设置 HDInsight 群集时指定的帐户。默认容器名与 HDInsight 群集名相同。

11. 在 Main() 函数中，追加以下代码以定义 MapReduce 作业：

        // 定义 MapReduce 作业
        MapReduceJobCreateParameters mrJobDefinition = new MapReduceJobCreateParameters()
        {
        JarFile = "wasb:///example/jars/hadoop-examples.jar",
        ClassName = "wordcount"
        };

        mrJobDefinition.Arguments.Add("wasb:///example/data/gutenberg/davinci.txt");
        mrJobDefinition.Arguments.Add("wasb:///example/data/WordCountOutput");

    有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。有关 wasb 前缀的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

12. 在 Main() 函数中，追加以下代码来创建 JobSubmissionCertificateCredential 对象：

        // 通过使用友好名称进行标识从证书存储区获取证书对象
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certFriendlyName);
        JobSubmissionCertificateCredential creds = new JobSubmissionCertificateCredential(new Guid(subscriptionID), cert, clusterName);

13. 在 Main() 函数中，追加以下代码来运行作业并等待作业完成：

        // 创建 hadoop 客户端以连接到 HDInsight
        var jobClient = JobSubmissionClientFactory.Connect(creds);

        // 运行 MapReduce 作业
        JobCreationResults mrJobResults = jobClient.CreateMapReduceJob(mrJobDefinition);

        // 等待作业完成
        WaitForJobCompletion(mrJobResults, jobClient);

14. 在 Main() 函数中，追加以下代码来打印 MapReduce 作业的输出：

        // 打印 MapReduce 作业的输出结果
        Stream stream = new MemoryStream();

        CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=" + storageAccountName + ";AccountKey=" + storageAccountKey);
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
        CloudBlobContainer blobContainer = blobClient.GetContainerReference(containerName);
        CloudBlockBlob blockBlob = blobContainer.GetBlockBlobReference("example/data/WordCountOutput/part-r-00000");

        blockBlob.DownloadToStream(stream);
        stream.Position = 0;

        StreamReader reader = new StreamReader(stream);
        Console.WriteLine(reader.ReadToEnd());

        Console.WriteLine("Press ENTER to continue.");
        Console.ReadLine();

    输出文件夹是你在定义 MapReduce 作业时指定的。默认文件名为 *part-r-00000*。

**运行应用程序**

该应用程序在 Visual Studio 中打开时，按 **F5** 键以运行该应用程序。控制台窗口应该会打开并显示该应用程序的状态和应用程序输出结果。

## 使用 HDInsight .NET SDK 提交 Hadoop 流作业

HDInsight 群集提供了一个用 C\# 开发的字数统计 Hadoop 流程序。映射器程序为 */example/apps/cat.exe*，化简程序为 */example/apps/wc.exe*。在本课中，你将学习如何创建 .NET 应用程序以运行字数统计示例。

有关创建 .Net 应用程序来提交 MapReduce 作业的详细信息，请参阅[使用 HDInsight .NET SDK 提交 MapReduce 作业][]。

有关开发和部署 Hadoop 流作业的详细信息，请参阅[为 HDInsight 开发 C\# Hadoop 流程序][]。

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;

    using System.IO;
    using System.Threading;
    using System.Security.Cryptography.X509Certificates;

    using Microsoft.WindowsAzure.Management.HDInsight;
    using Microsoft.Hadoop.Client;

    namespace SubmitStreamingJob
    {
    class Program
        {
    static void Main(string[] args)
            {

    // 设置变量
    string subscriptionID = "<Azure subscription ID>";
    string certFriendlyName = "<certificate friendly name>";
        
    string clusterName = "<HDInsight cluster name>";
    string statusFolderName = @"/tutorials/wordcountstreaming/status";

    // 定义 Hadoop 流 MapReduce 作业
    StreamingMapReduceJobCreateParameters myJobDefinition = new StreamingMapReduceJobCreateParameters()
                {
    JobName = "my word counting job",
    StatusFolder = statusFolderName,
    Input = "/example/data/gutenberg/davinci.txt",
    Output = "/tutorials/wordcountstreaming/output",
    Reducer = "wc.exe",
    Mapper = "cat.exe"
                };

    myJobDefinition.Files.Add("/example/apps/wc.exe");
    myJobDefinition.Files.Add("/example/apps/cat.exe");

    // 通过使用友好名称进行标识从证书存储区获取证书对象
    X509Store store = new X509Store();
    store.Open(OpenFlags.ReadOnly);
    X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certFriendlyName);

    JobSubmissionCertificateCredential creds = new JobSubmissionCertificateCredential(new Guid(subscriptionID), cert, clusterName);

    // 创建 hadoop 客户端以连接到 HDInsight
    var jobClient = JobSubmissionClientFactory.Connect(creds);

    // 运行 MapReduce 作业
    Console.WriteLine("----- Submit the Hadoop streaming job ...");
    JobCreationResults mrJobResults = jobClient.CreateStreamingJob(myJobDefinition);

    // 等待作业完成
    Console.WriteLine("----- Wait for the Hadoop streaming job to complete ...");
    WaitForJobCompletion(mrJobResults, jobClient);

    // 显示错误日志
    Console.WriteLine("----- The hadoop streaming job error log.");
    using (Stream stream = jobClient.GetJobErrorLogs(mrJobResults.JobId))
                {
    var reader = new StreamReader(stream);
    Console.WriteLine(reader.ReadToEnd());
                }

    // 显示输出日志
    Console.WriteLine("----- The hadoop streaming job output log.");
    using (Stream stream = jobClient.GetJobOutput(mrJobResults.JobId))
                {
    var reader = new StreamReader(stream);
    Console.WriteLine(reader.ReadToEnd());
                }

    Console.WriteLine("----- Press ENTER to continue.");
    Console.ReadLine();
            }

    private static void WaitForJobCompletion(JobCreationResults jobResults, IJobSubmissionClient client)
            {
    JobDetails jobInProgress = client.GetJob(jobResults.JobId);
    while (jobInProgress.StatusCode != JobStatusCode.Completed && jobInProgress.StatusCode != JobStatusCode.Failed)
                {
    jobInProgress = client.GetJob(jobInProgress.JobId);
    Thread.Sleep(TimeSpan.FromSeconds(10));
                }
            }
        }
    }

## 使用 HDInsight .NET SDK 提交 Hive 作业

HDInsight 群集提供了一个名为 *hivesampletable* 的示例 Hive 表。在本课中，你将创建一个 .NET 应用程序来运行 Hive 作业，以列出在 HDInsight 群集上创建的 Hive 表。有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][]。

使用 SDK 设置 HDInsight 群集的步骤如下：

-   安装 HDInsight .NET SDK
-   创建控制台应用程序
-   运行应用程序

**安装 HDInsight .NET SDK**
 可以从 [NuGet][] 安装最新发布的 SDK 版本。下一过程中将显示说明。

**创建 Visual Studio 控制台应用程序**

1.  打开 Visual Studio 2012。

2.  在“文件”菜单中，单击“新建” ，然后单击“项目” 。

3.  从“新建项目”中，键入或选择以下值：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
	<tr>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">属性</th>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">值</th></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">类别</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">模板/Visual C#/Windows</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">模板</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">控制台应用程序</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">名称</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">SubmitHiveJob</td></tr>
	</table>

4.  单击“确定” 以创建该项目。

5.  从“工具”菜单中 ，单击“库程序包管理器” ，然后单击“程序包管理器控制台” 。

6.  在控制台中运行下列命令以安装程序包。

        Install-Package Microsoft.WindowsAzure.Management.HDInsight

    此命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7.  从解决方案资源管理器中，双击 **Program.cs** 将其打开。

8.  将下列 using 语句添加到文件顶部：

        using System.IO;
        using System.Threading;
        using System.Security.Cryptography.X509Certificates;

        using Microsoft.WindowsAzure.Management.HDInsight;
        using Microsoft.Hadoop.Client;

9.  将以下函数定义添加到类。此函数用于等待 Hadoop 作业完成。

        private static void WaitForJobCompletion(JobCreationResults jobResults, IJobSubmissionClient client)
        {
        JobDetails jobInProgress = client.GetJob(jobResults.JobId);
        while (jobInProgress.StatusCode != JobStatusCode.Completed && jobInProgress.StatusCode != JobStatusCode.Failed)
            {
        jobInProgress = client.GetJob(jobInProgress.JobId);
        Thread.Sleep(TimeSpan.FromSeconds(10));
            }
        }

10. 在 Main() 函数中，复制并粘贴以下代码：

        // 设置变量
        string subscriptionID = "<Azure subscription ID>";
        string clusterName = "<HDInsight cluster name>";
        string certFriendlyName = "<certificate friendly name>";        

    这些是你需要为该程序设置的所有变量。你可以从你的系统管理员那里获取 Azure 订阅 ID。

    有关证书的信息，请参阅[创建并上载 Azure 管理证书][]。配置证书的简单方法是运行 *Get-AzurePublishSettingsFile* 和 *Import-AzurePublishSettingsFile* PowerShell cmdlet。它们将自动创建和上载管理证书。在运行 PowerShell cmdlet 后，可以从工作站打开 *certmgr.msc*，并通过展开“个人/证书”**来查找该证书。由 PowerShell cmdlet 创建的证书的“颁发给”**和“颁发者”**两个字段均为“Azure 工具”**。

11. 在 Main() 函数中，追加以下代码以定义 Hive 作业：

        // 定义 Hive 作业
        HiveJobCreateParameters hiveJobDefinition = new HiveJobCreateParameters()
        {
        JobName = "show tables job",
        StatusFolder = "/ShowTableStatusFolder",
        Query = "show tables;"
        };

    还可以使用 File 参数来指定 HDFS 上的 HiveQL 脚本文件。例如

        // 定义 Hive 作业
        HiveJobCreateParameters hiveJobDefinition = new HiveJobCreateParameters()
        {
        JobName = "show tables job",
        StatusFolder = "/ShowTableStatusFolder",
        File = "/user/admin/showtables.hql"
        };

12. 在 Main() 函数中，追加以下代码来创建 JobSubmissionCertificateCredential 对象：

        // 通过使用友好名称进行标识从证书存储区获取证书对象
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certFriendlyName);
        JobSubmissionCertificateCredential creds = new JobSubmissionCertificateCredential(new Guid(subscriptionID), cert, clusterName);

13. 在 Main() 函数中，追加以下代码来运行作业并等待作业完成：

        // 提交 Hive 作业
        var jobClient = JobSubmissionClientFactory.Connect(creds);
        JobCreationResults jobResults = jobClient.CreateHiveJob(hiveJobDefinition);

        // 等待作业完成
        WaitForJobCompletion(jobResults, jobClient);

14. 在 Main() 函数中，追加以下代码来打印 Hive 作业的输出：

        // 打印 Hive 作业的输出结果
        System.IO.Stream stream = jobClient.GetJobOutput(jobResults.JobId);

        StreamReader reader = new StreamReader(stream);
        Console.WriteLine(reader.ReadToEnd());

        Console.WriteLine("Press ENTER to continue.");
        Console.ReadLine();

**运行应用程序**

该应用程序在 Visual Studio 中打开时，按 **F5** 键以运行该应用程序。控制台窗口应打开并显示应用程序的状态。输出结果应该是：

    hivesampletable

## 后续步骤

在本文中，你已经学习了几种设置 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][HDInsight 入门]
-   [配置 HDInsight 群集][设置 HDInsight 群集]
-   [使用 PowerShell 管理 HDInsight][]
-   [HDInsight Cmdlet 参考文档][]
-   [Hive 与 HDInsight 配合使用][将 Hive 与 HDInsight 配合使用]
-   [Pig 与 HDInsight 配合使用][]

  [HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [使用 PowerShell 提交 MapReduce 作业]: #mapreduce-powershell
  [使用 PowerShell 提交 Hive 作业]: #hive-powershell
  [使用 PowerShell 提交 Sqoop 作业]: #sqoop-powershell
  [使用 HDInsight .NET SDK 提交 MapReduce 作业]: #mapreduce-sdk
  [使用 HDInsight .NET SDK 提交 Hadoop 流 MapReduce 作业]: #streaming-sdk
  [使用 HDInsight .NET SDK 提交 Hive 作业]: #hive-sdk
  [后续步骤]: #nextsteps
  [使用 PowerShell 管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [将 MapReduce 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-mapreduce-with-hdinsight/
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [HDI.GettingStarted.RunMRJob]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.RunMRJob.png
  [HDI.GettingStarted.MRJobOutput]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.MRJobOutput.png
  [Hive]: http://hive.apache.org/
  [将 Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [将 Sqoop 与 HDInsight 配合使用]: ../hdinsight-use-sqoop/
  [NuGet]: http://nuget.codeplex.com/wikipage?title=Getting%20Started
  [Azure 管理门户]: http://manage.windowsazure.cn/
  [创建并上载 Azure 管理证书]: http://msdn.microsoft.com/zh-cn/library/azure/gg551722.aspx
  [为 HDInsight 开发 C\# Hadoop 流程序]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
  [HDInsight Cmdlet 参考文档]: http://msdn.microsoft.com/zh-cn/library/azure/dn479228.aspx
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
