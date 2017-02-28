<properties
    pageTitle="使用 HDInsight .NET SDK 运行 Hive 查询 | Azure"
    description="了解如何使用 HDInsight .NET SDK 将 Hadoop 作业提交到 Azure HDInsight Hadoop。"
    editor="cgronlun"
    manager="jhubbard"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian" />
<tags
    ms.assetid="4e291890-f8b4-426c-b5e8-d4fd512ff042"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/16/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 HDInsight .NET SDK 运行 Hive 查询
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 HDInsight .NET SDK 提交 Hive 查询。

> [AZURE.NOTE]
必须从 Windows 客户端执行本文中的步骤。有关使用 Linux、OS X 或 Unix 客户端处理 Hive 的信息，请使用本文顶部显示的选项卡选择器。
> 
> 

## 先决条件
在开始阅读本文前，必须具有以下项：

* **HDInsight 中的 Hadoop 群集**。请参阅[在 HDInsight 中使用基于 Windows 的 Hadoop 入门](/documentation/articles/hdinsight-use-sqoop/#create-cluster-and-sql-database)。
* **Visual Studio 2012/2013/2015**。

## 使用 HDInsight .NET SDK 提交 Hive 查询
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。

**提交作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令：
   
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
   
                        SubmitHiveJob();
   
                        System.Console.WriteLine("Press ENTER to continue ...");
                        System.Console.ReadLine();
                    }
   
                    private static void SubmitHiveJob()
                    {
                        Dictionary<string, string> defines = new Dictionary<string, string> { { "hive.execution.engine", "ravi" }, { "hive.exec.reducers.max", "1" } };
                        List<string> args = new List<string> { { "argA" }, { "argB" } };
                        var parameters = new HiveJobSubmissionParameters
                        {
                            Query = "SHOW TABLES",
                            Defines = defines,
                            Arguments = args
                        };
   
                        System.Console.WriteLine("Submitting the Hive job to the cluster...");
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

应用程序的输出应如下：

![HDInsight Hadoop Hive 作业输出](./media/hdinsight-hadoop-use-hive-dotnet-sdk/hdinsight-hadoop-use-hive-net-sdk-output.png)  


## 后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门][hdinsight-get-started]
* [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]
* [使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-management-portal/)
* [HDInsight .NET SDK 参考](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)
* [创建非交互式身份验证 .NET HDInsight 应用程序](/documentation/articles/hdinsight-create-non-interactive-authentication-dotnet-applications/)

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->