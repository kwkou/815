<properties
	pageTitle="在 HDInsight 中提交 Hadoop 作业 | Windows Azure"
	description="了解如何将 Hadoop 作业提交到 Azure HDInsight Hadoop。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="10/02/2015"
	wacn.date="11/27/2015"/>

# 在 HDInsight 中提交 Hadoop 作业

学习如何使用 Azure PowerShell 提交 MapReduce 和 Hive 作业，以及如何使用 HDInsight .NET SDK 提交 MapReduce、Hadoop 流式处理和 Hive 作业。

##先决条件

在开始阅读本文前，你必须具有：

* **一个 Azure HDInsight 群集**。有关说明，请参阅 [HDInsight 入门][hdinsight-get-started]或[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]。
- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell](/documentation/articles/install-configure-powershell)。



##使用 Azure PowerShell 提交 MapReduce 作业
Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关将 Azure PowerShell 用于 HDInsight 的详细信息，请参阅[使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]。

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的应用程序。HDInsight 群集附带一个 JAR 文件（位于 *\\example\\jars\\hadoop-mapreduce-examples.jar* 中），其中包含几个 MapReduce 示例。

一个示例是计算源文件中的单词频率。在本课中，你将了解如何从工作站使用 Azure PowerShell 来运行单词计数示例。有关开发和运行 MapReduce 作业的详细信息，请参阅[将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]。

**使用 Azure PowerShell 运行单词计数 MapReduce 程序**

1.	打开 **Azure PowerShell**。有关如何打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

3. 通过运行这些 Azure PowerShell 命令来设置以下变量：

		$subscriptionName = "<SubscriptionName>"
		$clusterName = "<HDInsightClusterName>"

	订阅名称是你用于创建 HDInsight 群集的订阅。HDInsight 群集是你要用于运行 MapReduce 作业的群集。

5. 运行以下命令创建 MapReduce 作业定义：

		# Define the word count MapReduce job
		$wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

	有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。有关 wasb:// 前缀的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

6. 运行以下命令来运行 MapReduce 作业：

		# Submit the MapReduce job
		Select-AzureSubscription $subscriptionName
		$wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition

	除了 MapReduce 作业定义外，你还提供要运行 MapReduce 作业的 HDInsight 群集名称。

7. 运行以下命令来检查 MapReduce 作业的完成：

		# Wait for the job to complete
		Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600


8. 运行以下命令来检查运行 MapReduce 作业时的错误：

		# Get the job standard error output
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError

	以下屏幕快照显示了成功运行的输出。否则，你将看到一些错误消息。

	![HDI.GettingStarted.RunMRJob][image-hdi-gettingstarted-runmrjob]


**检索 MapReduce 作业的结果**

1. 打开 **Azure PowerShell**。
2. 通过运行这些 Azure PowerShell 命令来设置以下变量：

		$subscriptionName = "<SubscriptionName>"
		$storageAccountName = "<StorageAccountName>"
		$containerName = "<ContainerName>"

	存储帐户名称是你在创建 HDInsight 群集期间指定的 Azure 存储帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。容器名称通常与 HDInsight 群集共享相同的名称，除非你在创建群集时指定其他名称。

3. 运行以下命令以创建 Azure Blob 存储上下文对象：

		# Create the storage account context object
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$storageContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

	**Select-AzureSubscription** 用于在你具有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

4. 运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

		# Get the blob content
		Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

	*example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹中的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。文件将下载到：

**C:\\example\\data\\WordCountOutput*

5. 运行以下命令来打印 MapReduce 作业输出文件：

		cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

	![HDI.GettingStarted.MRJobOutput][image-hdi-gettingstarted-mrjoboutput]

	MapReduce 作业将生成一个名为 *part-r-00000* 的文件，其中包含单词和计数。该脚本使用 **findstr** 命令来列出包含“there”的所有单词。


> [AZURE.NOTE]如果你在记事本中打开 ./example/data/WordCountOutput/part-r-00000（这是来自 MapReduce 作业的多行输出），将会注意到断行未正确呈现。这是正常情况。









































































































































##使用 Azure PowerShell 提交 Hive 作业
[Apache Hive][apache-hive] 提供了通过类似 SQL 的脚本语言（称作 *HiveQL*）运行 MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。

HDInsight 群集提供了一个名为 *hivesampletable* 的示例 Hive 表。在本部分中，你将使用 Azure PowerShell 来运行 Hive 作业，以便列出 Hive 表中的一些数据。

**使用 Azure PowerShell 运行 Hive 作业**

1.	打开 **Azure PowerShell**。有关如何打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

2. 在以下命令中设置前两个变量，然后运行它们：

		$subscriptionName = "<SubscriptionName>"
		$clusterName = "<HDInsightClusterName>"
		$querystring = "SELECT * FROM hivesampletable WHERE Country='China';"

	$querystring 是 HiveQL 查询。

3. 运行以下命令以选择 Azure 订阅和要运行 Hive 作业的群集：

		Select-AzureSubscription -SubscriptionName $subscriptionName

4. 运行以下命令来提交 Hive 作业：

		Use-AzureHDInsightCluster $clusterName
		Invoke-Hive -Query $queryString

	可以使用 **-File** 开关来指定 Hadoop 分布式文件系统 (HDFS) 中的 HiveQL 脚本文件。

有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。


## 使用 Visual Studio 提交 Hive 作业

请参阅[开始使用适用于 Visual Studio 的 HDInsight 工具][hdinsight-visual-studio-tools]。

##使用 Azure PowerShell 提交 Sqoop 作业

请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

##使用 HDInsight .NET SDK 提交 MapReduce 作业
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。HDInsight 群集附带一个 JAR 文件（位于 *\\example\\jars\\hadoop-mapreduce-examples.jar* 中），其中包含几个 MapReduce 示例。一个示例是计算源文件中的单词频率。在本课中，你将学习如何创建 .NET 应用程序以运行单词计数示例。有关开发和运行 MapReduce 作业的详细信息，请参阅[将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]。

**提交 WordCount MapReduce 作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令。

		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

2. 在 Program.cs 文件中使用以下 using 语句：

		using System;
		using System.Collections.Generic;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;

3. 将以下代码添加到 Main() 函数中。

		var ExistingClusterName = "<HDInsightClusterName>";
		var ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
		var ExistingClusterUsername = "<HDInsightClusterHttpUsername>";
		var ExistingClusterPassword = "<HDInsightClusterHttpUserPassword>";
	    HDInsightJobManagementClient _hdiJobManagementClient;

	    List<string> arguments = new List<string> { { "wasb:///example/data/gutenberg/davinci.txt" }, { "wasb:///example/data/WordCountOutput" } };
	
	    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
	    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
	
	    var parameters = new MapReduceJobSubmissionParameters
	    {
	        UserName = ExistingClusterUsername,
	        JarFile = "wasb:///example/jars/hadoop-mapreduce-examples.jar",
	        JarClass = "wordcount",
	        Arguments = ConvertArgsToString(arguments)
	    };
	
	    System.Console.WriteLine("Submitting the MapReduce job to the cluster...");
	    var response = _hdiJobManagementClient.JobManagement.SubmitMapReduceJob(parameters);
	    System.Console.WriteLine("Validating that the response is as expected...");
	    System.Console.WriteLine("Response status code is " + response.StatusCode);
	    System.Console.WriteLine("Validating the response object...");
	    System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
	    Console.WriteLine("Press ENTER to continue ...");
	    Console.ReadLine();

4. 将以下 Helper 函数添加到类。

        private static string ConvertArgsToString(List<string> args)
        {
            if (args.Count == 0)
            {
                return null;
            }

            return string.Join("&arg=", args.ToArray());
        }

5. 按 **F5** 运行应用程序。










##使用 HDInsight .NET SDK 提交 Hadoop 流作业
HDInsight 群集附带了一个用 C# 开发的单词计数 Hadoop 流程序。映射器程序为 */example/apps/cat.exe*，化简程序为 */example/apps/wc.exe*。在本课中，你将学习如何创建 .NET 应用程序以运行单词计数示例。

有关创建 .NET 应用程序来提交 MapReduce 作业的详细信息，请参阅[使用 HDInsight .NET SDK 提交 MapReduce 作业](#mapreduce-sdk)。

有关开发和部署 Hadoop 流作业的详细信息，请参阅[为 HDInsight 开发 C# Hadoop 流程序][hdinsight-develop-streaming-jobs]。

**提交 WordCount MapReduce 作业**

1. 在 Visual Studio 包管理器控制台中，运行以下 Nuget 命令将包导入。

		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

2. 在 Program.cs 文件中使用以下 using 语句：

		using System;
		using System.Collections.Generic;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;

3. 将以下代码添加到 Main() 函数中。

		var ExistingClusterName = "<HDInsightClusterName>";
		var ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
		var ExistingClusterUsername = "<HDInsightClusterHttpUsername>";
		var ExistingClusterPassword = "<HDInsightClusterHttpUserPassword>";

        List<string> arguments = new List<string> { { "/example/apps/cat.exe" }, { "/example/apps/wc.exe" } };

        HDInsightJobManagementClient _hdiJobManagementClient;
        var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
        _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);

        var parameters = new MapReduceStreamingJobSubmissionParameters
        {
            UserName = ExistingClusterUsername,
            File = ConvertArgsToString(arguments),
            Mapper = "cat.exe",
            Reducer = "wc.exe",
            Input = "/example/data/gutenberg/davinci.txt",
            Output = "/tutorials/wordcountstreaming/output"
        };

        System.Console.WriteLine("Submitting the MapReduce job to the cluster...");
        var response = _hdiJobManagementClient.JobManagement.SubmitMapReduceStreamingJob(parameters);
        System.Console.WriteLine("Validating that the response is as expected...");
        System.Console.WriteLine("Response status code is " + response.StatusCode);
        System.Console.WriteLine("Validating the response object...");
        System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
        Console.WriteLine("Press ENTER to continue ...");
        Console.ReadLine();

4. 向类添加 Help 函数。

        private static string ConvertArgsToString(List<string> args)
        {
            if (args.Count == 0)
            {
                return null;
            }

            return string.Join("&arg=", args.ToArray());
        }

5. 按 **F5** 运行应用程序。

##使用 HDInsight .NET SDK 提交 Hive 作业
HDInsight 群集提供了一个名为 *hivesampletable* 的示例 Hive 表。在本课中，你将创建一个 .NET 应用程序来运行 Hive 作业，以列出在 HDInsight 群集中创建的 Hive 表。有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。

使用 SDK 创建 HDInsight 群集的步骤如下：

- 安装 HDInsight .NET SDK
- 创建控制台应用程序
- 运行应用程序


**安装 HDInsight .NET SDK**。可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装最新发布的 SDK 版本。下一过程中将显示说明。

**创建 Visual Studio 控制台应用程序**

1. 打开 Visual Studio 2013 或 2015。

2. 创建一个具有以下设置的新项目：

	|属性|值|
	|--------|-----|
	|模板|模板/Visual C#/Windows/控制台应用程序|
	|Name|SubmitHiveJob|

3. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。
4. 在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.Azure.Common.Authentication -pre
		Install-Package Microsoft.Azure.Management.HDInsight -Pre
		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

	这些命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

5. 在解决方案资源管理器中双击 **Program.cs** 将它打开，粘贴以下代码，并提供变量的值：

		using System.Collections.Generic;
		using System.Linq;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;
		
		namespace SubmitHiveJob
		{
		    class Program
		    {
		        private static HDInsightJobManagementClient _hdiJobManagementClient;
		
		        private const string ExistingClusterName = "<HDINSIGHT CLUSTER NAME>";
		        private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
		
		        private const string ExistingClusterUsername = "<HDINSIGHT HTTP USER NAME>";  //The default name is admin.
		        private const string ExistingClusterPassword = "<HDINSIGHT HTTP USER PASSWORD>";
		
		        private static void Main(string[] args)
		        {
		
		            var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
		            _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
		
		            SubmitHiveJob();
		        }
		
		        private static void SubmitHiveJob()
		        {
		            Dictionary<string, string> defines = new Dictionary<string, string> { { "hive.execution.engine", "ravi" }, { "hive.exec.reducers.max", "1" } };
		            List<string> args = new List<string> { { "argA" }, { "argB" } };
		            var parameters = new HiveJobSubmissionParameters
		            {
		                UserName = ExistingClusterUsername,
		                Query = "SHOW TABLES",
		                Defines = ConvertDefinesToString(defines),
		                Arguments = ConvertArgsToString(args)
		            };
		
		            System.Console.WriteLine("Submitting the Hive job to the cluster...");
		            var response = _hdiJobManagementClient.JobManagement.SubmitHiveJob(parameters);
		            System.Console.WriteLine("Validating that the response is as expected...");
		            System.Console.WriteLine("Response status code is " + response.StatusCode);
		            System.Console.WriteLine("Validating the response object...");
		            System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
		            System.Console.WriteLine("Press ENTER to continue ...");
		            System.Console.ReadLine();
		        }
		
		        private static string ConvertDefinesToString(Dictionary<string, string> defines)
		        {
		            if (defines.Count == 0)
		            {
		                return null;
		            }
		
		            return string.Join("&define=", defines.Select(x => x.Key + "%3D" + x.Value).ToArray());
		        }
		        private static string ConvertArgsToString(List<string> args)
		        {
		            if (args.Count == 0)
		            {
		                return null;
		            }
		
		            return string.Join("&arg=", args.ToArray());
		        }
		    }
		}

6. 按 **F5** 运行应用程序。

##使用适用于 Visual Studio 的 HDInsight 工具提交作业

你可以使用适用于 Visual Studio 的 HDInsight 工具来运行 Hive 查询和 Pig 脚本。请参阅[开始使用适用于 HDInsight 的 Visual Studio Hadoop 工具](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started)。


##后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门][hdinsight-get-started]
* [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]
* [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [HDInsight Cmdlet 参考文档][hdinsight-powershell-reference]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]


[azure-certificate]: /documentation/articles/cloud-services-certs-create/
[azure-management-portal]: https://manage.windowsazure.cn/

[hdinsight-visual-studio-tools]: /documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell
[hdinsight-develop-streaming-jobs]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs

[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx

[powershell-install-configure]: /documentation/articles/install-configure-powershell

[image-hdi-gettingstarted-runmrjob]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.RunMRJob.png
[image-hdi-gettingstarted-mrjoboutput]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.MRJobOutput.png

[apache-hive]: http://hive.apache.org/

<!---HONumber=82-->