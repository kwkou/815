<properties
    pageTitle="使用 HDInsight .NET SDK 提交 MapReduce 作业 | Azure"
    description="了解如何使用 HDInsight .NET SDK 将 MapReduce 作业提交到 Azure HDInsight Hadoop。"
    editor="cgronlun"
    manager="jhubbard"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian" />  

<tags
    ms.assetid="c85e44b0-85fd-4185-ad1c-c34a9fe5ef44"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="12/26/2016"
    ms.author="jgao" />  


# 使用 HDInsight .NET SDK 运行 MapReduce 作业
[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

了解如何使用 HDInsight .NET SDK 提交 MapReduce 作业。HDInsight 群集附带了一个 jar 文件，其中包含一些 MarReduce 示例。该 jar 文件是 */example/jars/hadoop-mapreduce-examples.jar*。其中一个示例是 *wordcount*。你开发一个 C# 控制台应用程序以提交 wordcount 作业。该作业读取*/example/data/gutenberg/davinci.txt* 文件，并将结果输出到 */example/data/davinciwordcount*。如果要重新运行该应用程序，必须清理输出文件夹。

> [AZURE.NOTE]
必须从 Windows 客户端执行本文中的步骤。有关使用 Linux、OS X 或 Unix 客户端处理 Hive 的信息，请使用本文顶部显示的选项卡选择器。
> 
> 

## 先决条件
开始阅读本文之前，必须具备以下条件：

* **HDInsight 中的 Hadoop 群集**。请参阅[开始在 HDInsight 中使用 Hadoop](/documentation/articles/hdinsight-use-sqoop/#create-cluster-and-sql-database)。
* **Visual Studio 2012/2013/2015**。

## 使用 HDInsight .NET SDK 提交 MapReduce 作业
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。

**提交作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令。
   
        Install-Package Microsoft.Azure.Management.HDInsight.Job
3. 使用以下代码：
   
        using System.Collections.Generic;
        using System.IO;
        using System.Text;
        using System.Threading;
        using Microsoft.Azure.Management.HDInsight.Job;
        using Microsoft.Azure.Management.HDInsight.Job.Models;
        using Hyak.Common;
   
        namespace SubmitHDInsightJobDotNet
        {
            class Program
            {
                private static HDInsightJobManagementClient _hdiJobManagementClient;
   
                private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
                private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
                private const string ExistingClusterUsername = "<Cluster Username>";
                private const string ExistingClusterPassword = "<Cluster User Password>";
   
                private const string DefaultStorageAccountName = "<Default Storage Account Name>";
                private const string StorageAccountSuffix = "core.chinacloudapi.cn";
                private const string DefaultStorageAccountKey = "<Default Storage Account Key>";
                private const string DefaultStorageContainerName = "<Default Blob Container Name>";
   
                static void Main(string[] args)
                {
                    System.Console.WriteLine("The application is running ...");
   
                    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
   
                    SubmitMRJob();
   
                    System.Console.WriteLine("Press ENTER to continue ...");
                    System.Console.ReadLine();
                }
   
                private static void SubmitMRJob()
                {
                    List<string> args = new List<string> { { "/example/data/gutenberg/davinci.txt" }, { "/example/data/davinciwordcount" } };
   
                    var paras = new MapReduceJobSubmissionParameters
                    {
                        JarFile = @"/example/jars/hadoop-mapreduce-examples.jar",
                        JarClass = "wordcount",
                        Arguments = args
                    };
   
                    System.Console.WriteLine("Submitting the MR job to the cluster...");
                    var jobResponse = _hdiJobManagementClient.JobManagement.SubmitHiveJob(parameters);
                    var jobId = jobResponse.JobSubmissionJsonResponse.Id;
                    System.Console.WriteLine("Response status code is " + jobResponse.StatusCode);
                    System.Console.WriteLine("JobId is " + jobId);
   
                    System.Console.WriteLine("Waiting for the job completion ...");
   
                    // Wait for job completion
                    var jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                    while (!jobDetail.Status.JobComplete)
                    {
                        Thread.Sleep(1000);
                        jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                    }
   
                    // Get job output
                    var storageAccess = new AzureStorageAccess(DefaultStorageAccountName, DefaultStorageAccountKey,
                        DefaultStorageContainerName, StorageAccountSuffix);
                    var output = (jobDetail.ExitValue == 0)
                        ? _hdiJobManagementClient.JobManagement.GetJobOutput(jobId, storageAccess) // fetch stdout output in case of success
                        : _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess); // fetch stderr output in case of failure
   
                    System.Console.WriteLine("Job output is: ");
   
                    using (var reader = new StreamReader(output, Encoding.UTF8))
                    {
                        string value = reader.ReadToEnd();
                        System.Console.WriteLine(value);
                    }
                }
            }
        }
4. 按 **F5** 运行应用程序。

## 后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* 有关创建群集和提交 Hive 作业，请参阅 [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/)。
* 有关创建 HDInsight 群集，请参阅[在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。
* 有关管理 HDInsight 群集，请参阅[在 HDInsight 中管理 Hadoop 群集](/documentation/articles/hdinsight-administer-use-management-portal-v1/)。
* 有关学习 HDInsight .NET SDK，请参阅 [HDInsight .NET SDK 参考](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)。

<!---HONumber=Mooncake_1219_2016-->