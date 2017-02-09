<!-- not suitable for Mooncake -->


<properties
   pageTitle="在 HDInsight 中将 Hadoop Pig 与 .NET 配合使用 | Azure"
   description="了解如何使用 .NET SDK for Hadoop 将 Pig 作业提交到 HDInsight 上的 Hadoop。"
   services="hdinsight"
   documentationCenter=".net"
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
   tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/06/2016"
	wacn.date="02/06/2017"/>

#使用 HDInsight 中的 .NET SDK for Hadoop 运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

本文档提供使用 .NET SDK for Hadoop 将 Pig 作业提交到 HDInsight 上的 Hadoop 群集的示例。

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。Pig 可让你通过为一系列数据转换建模，创建 MapReduce 操作。你将了解如何使用基本 C# 应用程序将 Pig 作业提交到 HDInsight 群集。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要以下各项：


* Azure HDInsight（HDInsight 上的 Hadoop）群集 (Windows)

* Visual Studio 2012、2013 或 2015

##<a id="subscriptionid"></a>查找你的订阅 ID

每个 Azure 订阅都以 GUID 值（称为订阅 ID）标识。请使用以下步骤查找此值。

1. 访问 [Azure 门户预览][preview-portal]。

2. 从门户左侧的栏中，选择“浏览全部”，然后从“浏览”边栏选项卡中选择“订阅”。

3. 在“订阅”边栏选项卡上显示的信息中，找到你要使用的订阅，并记下“订阅 ID”列中的值。

保存该订阅 ID，因为稍后你要用到它。

##<a id="create"></a>创建应用程序

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。

下面的示例使用用户交互式身份验证。若要使用非交互式身份验证，请参阅[创建非交互式身份验证 .NET HDInsight 应用程序](/documentation/articles/hdinsight-create-non-interactive-authentication-dotnet-applications/)。

1. 打开 Visual Studio 2012 或 2013
2. 在“文件”菜单中，选择“新建”，然后选择“项目”。
3. 对于新项目，请键入或选择以下值。

	<table>
	<tr>
	<th>属性</th>
	<th>值</th>
	</tr>
	<tr>
	<th>类别</th>
	<th>Templates/Visual C#/Windows</th>
	</tr>
	<tr>
	<th>模板</th>
	<th>Console Application</th>
	</tr>
	<tr>
	<th>名称</th>
	<th>SubmitPigJob</th>
	</tr>
	</table>
4. 单击“确定”以创建该项目。
5. 从“工具”菜单中，选择“库包管理器”或“Nuget 包管理器”，然后选择“包管理器控制台”。
6. 在控制台中运行以下命令，安装 .NET SDK 包。

        	Install-Package Microsoft.Azure.Common.Authentication -Pre
        	Install-Package Microsoft.Azure.Management.HDInsight -Pre
		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。将现有代码替换为以下内容。

        using System;
        using System.Collections.Generic;
        using System.Security;
        using System.Threading;
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
                    var tokenCreds = GetTokenCloudCredentials();
                    var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
                    _hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);

        
        
        
                    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
        

                    SubmitPigJob();
                    System.Console.WriteLine("Press ENTER to continue ...");
                    System.Console.ReadLine();
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
            }
        }

7. 按 **F5** 启动应用程序。
8. 按 **ENTER** 退出应用程序。

##<a id="summary"></a>摘要

如你所见，.NET SDK for Hadoop 可让你创建 .NET 应用程序，以将 Pig 作业提交到 HDInsight 群集、监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Pig 的一般信息。

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)
[preview-portal]: https://portal.azure.cn/

<!---HONumber=Mooncake_Quality_Review_1215_2016-->