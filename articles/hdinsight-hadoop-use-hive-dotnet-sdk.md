<properties
	pageTitle="使用 HDInsight .NET SDK 运行 Hive 查询 | Azure"
	description="了解如何使用 HDInsight .NET SDK 将 Hadoop 作业提交到 Azure HDInsight Hadoop。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="04/06/2016"
	wacn.date="05/23/2016"/>

# 使用 HDInsight .NET SDK 运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]


了解如何使用 HDInsight .NET SDK 提交 Hive 查询。

##先决条件

在开始阅读本文前，你必须具有：

- **HDInsight 中的 Hadoop 群集**。请参阅[创建群集和 SQL 数据库](/documentation/articles/hdinsight-use-sqoop/#create-cluster-and-sql-database)。
- **Visual Studio 2012/2013/2015**。

##使用 HDInsight .NET SDK 提交 Hive 查询

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。

**提交作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令。

		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

2. 使用以下代码：

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
		                DefaultStorageContainerName);
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

5. 按 **F5** 运行应用程序。


## 后续步骤

在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门][hdinsight-get-started]
* [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]
* [Manage Hadoop clusters in HDInsight by using the Azure Portal（使用 Azure 经典管理门户管理 HDInsight 中的 Hadoop 群集）](/documentation/articles/hdinsight-administer-use-management-portal-v1/)
* [HDInsight .NET SDK 参考](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/



<!---HONumber=Mooncake_0516_2016-->