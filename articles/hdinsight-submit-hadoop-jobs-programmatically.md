<properties
	pageTitle="在 HDInsight 中提交 Hadoop 作业 | Microsoft Azure"
	description="了解如何将 Hadoop 作业提交到 Azure HDInsight Hadoop。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="10/30/2015"
	wacn.date="12/17/2015"/>

# 在 HDInsight 中提交 Hadoop 作业

学习如何使用 Azure PowerShell 提交 MapReduce 和 Hive 作业，以及如何使用 HDInsight .NET SDK 提交 MapReduce、Hadoop 流式处理和 Hive 作业。

##先决条件

在开始阅读本文前，你必须具有：

* **一个 Azure HDInsight 群集**。有关说明，请参阅 [HDInsight 入门][hdinsight-get-started]或[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]。
- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell](/documentation/articles/powershell-install-configure)。

##使用 PowerShell 提交 MapReduce 作业

请参阅[在基于 Windows 的 HDInsight 中运行 Hadoop MapReduce 示例](/documentation/articles/hdinsight-run-samples)。

##使用 PowerShell 提交 Hive 作业

请参阅[使用 PowerShell 运行 Hive 查询](/documentation/articles/hdinsight-hadoop-use-hive-powershell)

## 使用 Visual Studio 提交 Hive 作业

请参阅[适用于 Visual Studio 的 HDInsight Hadoop 工具入门][hdinsight-visual-studio-tools]。

##使用 PowerShell 提交 Sqoop 作业

请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

##使用 HDInsight .NET SDK 提交 Hive/Pig/Sqoop 作业
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。

**提交作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令。

		Install-Package Microsoft.Azure.Common.Authentication -pre
		Install-Package Microsoft.Azure.Management.HDInsight -Pre
		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre
2. 使用以下代码：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Security;
		
		using Microsoft.Azure;
		using Microsoft.Azure.Common.Authentication;
		using Microsoft.Azure.Common.Authentication.Factories;
		using Microsoft.Azure.Common.Authentication.Models;
		using Microsoft.Azure.Management.HDInsight;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;
		
		namespace SubmitHDInsightJobDotNet
		{
			class Program
			{
				private static HDInsightManagementClient _hdiManagementClient;
				private static HDInsightJobManagementClient _hdiJobManagementClient;
		
				private static Guid SubscriptionId = new Guid("<Your Subscription ID>");
				private const string ResourceGroupName = "<Your Resource Group Name>";
		
				private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
				private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
				private const string ExistingClusterUsername = "admin";
				private const string ExistingClusterPassword = "**********";
		
				static void Main(string[] args)
				{
					System.Console.WriteLine("Running");
		
					var tokenCreds = GetTokenCloudCredentials();
					var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
		
					_hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);
		
					var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
					_hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
		
					SubmitHiveJob();
					SubmitPigJob();
					SubmitSqoopJob();
				}
		
				public static TokenCloudCredentials GetTokenCloudCredentials(string username = null, SecureString password = null)
				{
					var authFactory = new AuthenticationFactory();
		
					var account = new AzureAccount { Type = AzureAccount.AccountType.User };
		
					if (username != null && password != null)
						account.Id = username;
		
					var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
		
					var accessToken =
						authFactory.Authenticate(account, env, AuthenticationFactory.CommonAdTenant, password, ShowDialog.Auto)
							.AccessToken;
		
					return new TokenCloudCredentials(accessToken);
				}
		
				public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(TokenCloudCredentials creds, Guid subId)
				{
					return new TokenCloudCredentials(subId.ToString(), creds.Token);
				}
		
				private static void SubmitPigJob()
				{
					var parameters = new PigJobSubmissionParameters
					{
						UserName = ExistingClusterUsername,
						Query = @"LOGS = LOAD 'wasb:///example/data/sample.log';
							LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
							FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
							GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
							FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
							RESULT = order FREQUENCIES by COUNT desc;
							DUMP RESULT;"
					};
		
					System.Console.WriteLine("Submitting the Pig job to the cluster...");
					var response = _hdiJobManagementClient.JobManagement.SubmitPigJob(parameters);
					System.Console.WriteLine("Validating that the response is as expected...");
					System.Console.WriteLine("Response status code is " + response.StatusCode);
					System.Console.WriteLine("Validating the response object...");
					System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
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
				}
		
				private static void SubmitSqoopJob()
				{
					var sqlDatabaseServerName = "<SQLDatabaseServerName>";
					var sqlDatabaseLogin = "<SQLDatabaseLogin>";
					var sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>";
					var sqlDatabaseDatabaseName = "hdisqoop";
		
					var tableName = "log4jlogs";
					var exportDir = "/tutorials/usesqoop/data";
		
					// Connection string for using Azure SQL Database.
					// Comment if using SQL Server
					var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ".database.chinacloudapi.cn;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
					// Connection string for using SQL Server.
					// Uncomment if using SQL Server
					//var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
		
					var parameters = new SqoopJobSubmissionParameters
					{
						UserName = ExistingClusterUsername,
						Command = "export --connect " + connectionString + " --table " + tableName + "_mobile --export-dir " + exportDir + "_mobile --fields-terminated-by \\t -m 1"
					};
		
					System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
					var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
					System.Console.WriteLine("Validating that the response is as expected...");
					System.Console.WriteLine("Response status code is " + response.StatusCode);
					System.Console.WriteLine("Validating the response object...");
					System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
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

5. 按 **F5** 运行应用程序。

##使用适用于 Visual Studio 的 HDInsight 工具提交作业

你可以使用适用于 Visual Studio 的 HDInsight 工具来运行 Hive 查询和 Pig 脚本。请参阅[适用于 HDInsight 的 Visual Studio Hadoop 工具入门](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started)。

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
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell
[hdinsight-develop-streaming-jobs]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs

[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx

[powershell-install-configure]: /documentation/articles/powershell-install-configure

[image-hdi-gettingstarted-runmrjob]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.RunMRJob.png
[image-hdi-gettingstarted-mrjoboutput]: ./media/hdinsight-submit-hadoop-jobs-programmatically/HDI.GettingStarted.MRJobOutput.png

[apache-hive]: http://hive.apache.org/

<!---HONumber=Mooncake_1207_2015-->