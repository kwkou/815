<properties
    pageTitle="在 HDInsight 中使用 Hadoop Sqoop | Azure"
    description="了解如何使用 HDInsight .NET SDK 在 Hadoop 群集和 Azure SQL 数据库之间运行 Sqoop 导入和导出。"
    editor="cgronlun"
    manager="jhubbard"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian" />
<tags
    ms.assetid="87bacd13-7775-4b71-91da-161cb6224a96"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/14/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 HDInsight 中的 .NET SDK for Hadoop 运行 Sqoop 作业
[AZURE.INCLUDE [sqoop-selector](../../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 HDInsight .NET SDK 在 HDInsight 中运行 Sqoop 作业，以在 HDInsight 群集和 Azure SQL 数据库或 SQL Server 数据库之间进行导入和导出。

> [AZURE.NOTE]
可以对基于 Linux 或 Windows 的 HDInsight 群集使用本文中的步骤，但是，只能从 Windows 客户端执行这些步骤。使用本文顶部的选项卡选择器选择其他方法。
> 
> 

### 先决条件
在开始本教程前，你必须具有以下项：

* **HDInsight 中的 Hadoop 群集**。请参阅[创建群集和 SQL 数据库](/documentation/articles/hdinsight-use-sqoop/#create-cluster-and-sql-database)。

## 使用 .NET SDK 运行 Sqoop
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。在本部分中，你将创建一个 C# 控制台应用程序，以便将 hivesampletable 导出到在本教程前面创建的 SQL 数据库表。

**提交 Sqoop 作业**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 在 Visual Studio 包管理器控制台中，运行以下 Nuget 命令以导入包。
   
        Install-Package Microsoft.Azure.Management.HDInsight.Job
3. 在 Program.cs 文件中使用以下代码：
   
        using System.Collections.Generic;
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
   
                static void Main(string[] args)
                {
                    System.Console.WriteLine("The application is running ...");
   
                    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
   
                    SubmitSqoopJob();
   
                    System.Console.WriteLine("Press ENTER to continue ...");
                    System.Console.ReadLine();
                }
   
                private static void SubmitSqoopJob()
                {
                    var sqlDatabaseServerName = "<SQLDatabaseServerName>";
                    var sqlDatabaseLogin = "<SQLDatabaseLogin>";
                    var sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>";
                    var sqlDatabaseDatabaseName = "<DatabaseName>";
   
                    var tableName = "<TableName>";
                    var exportDir = "/tutorials/usesqoop/data";
   
                    // Connection string for using Azure SQL Database.
                    // Comment if using SQL Server
                    var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ".database.chinacloudapi.cn;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
                    // Connection string for using SQL Server.
                    // Uncomment if using SQL Server
                    //var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
   
                    var parameters = new SqoopJobSubmissionParameters
                    {
                        Files = new List<string> { "/user/oozie/share/lib/sqoop/sqljdbc41.jar" }, // This line is required for Linux-based cluster.
                        Command = "export --connect " + connectionString + " --table " + tableName + "_mobile --export-dir " + exportDir + "_mobile --fields-terminated-by \\t -m 1"
                    };
   
                    System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
                    var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
                    System.Console.WriteLine("Validating that the response is as expected...");
                    System.Console.WriteLine("Response status code is " + response.StatusCode);
                    System.Console.WriteLine("Validating the response object...");
                    System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
                }
            }
        }
4. 按 **F5** 运行程序。

## 限制
* 批量导出 - 在基于 Linux 的 HDInsight 上，用于将数据导出到 Microsoft SQL Server 或 Azure SQL 数据库的 Sqoop 连接器目前不支持批量插入。
* 批处理 - 在基于 Linux 的 HDInsight 上，如果执行插入时使用 `-batch` 开关，Sqoop 将执行多次插入而不是批处理插入操作。

## 后续步骤
现在你已了解如何使用 Sqoop。若要了解详细信息，请参阅以下文章：

* [将 Oozie 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-oozie/)：在 Oozie 工作流中使用 Sqoop 操作。
* [使用 HDInsight 分析外部测试版延迟数据](/documentation/articles/hdinsight-analyze-flight-delay-data/)：使用 Hive 分析外部测试版延迟数据，然后使用 Sqoop 将数据导出到 Azure SQL 数据库。
* [将数据上传到 HDInsight](/documentation/articles/hdinsight-upload-data/)：了解将数据上传到 HDInsight/Azure Blob 存储的其他方法。

<!---HONumber=Mooncake_0120_2017-->