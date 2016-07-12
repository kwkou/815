<properties
   pageTitle="在 HDInsight 中将 Hadoop Pig 与 .NET 配合使用 | Azure"
   description="了解如何使用 .NET SDK for Hadoop 将 Pig 作业提交到 HDInsight 上的 Hadoop。"
   services="hdinsight"
   documentationCenter=".net"
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>

#使用 HDInsight 中的 .NET SDK for Hadoop 运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-selector-use-pig.md)]

本文档提供使用 .NET SDK for Hadoop 将 Pig 作业提交到 HDInsight 上的 Hadoop 群集的示例。

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 HDInsight 群集的操作。Pig 可让你通过为一系列数据转换建模，来创建 MapReduce 操作。你将学习如何使用基本 C# 应用程序将 Pig 作业提交到 HDInsight 群集。

> [AZURE.IMPORTANT] 目前，Azure 中国区的 HDInsight 只能通过 Azure 服务管理器 (ASM) 进行管理。适用于 HDInsight 的 Azure Resource Manager (ARM) 模型尚不可用。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* Azure HDInsight（HDInsight 上的 Hadoop）群集 (Windows) 

* Visual Studio 2012 或 2013

##<a id="certificate"></a>创建管理证书

若要在 Azure HDInsight 上对应用程序进行身份验证，必须创建自签名证书，将它安装在开发工作站上，同时将它上载到你的 Azure 订阅。

有关如何执行此操作的说明，请参阅[创建自签名证书](/documentation/articles/hdinsight-administer-use-management-portal-v1/#cert)。

> [AZURE.NOTE] 创建证书时，请务必记下使用的友好名称供以后使用。

##<a id="subscriptionid"></a>查找你的订阅 ID

每个 Azure 订阅都是以 GUID 值（称为订阅 ID）标识的。请使用以下步骤来查找此值。

1. 访问 [Azure 管理控制台](https://manage.windowsazure.cn/)。

2. 在经典管理门户左侧的栏中，选择“设置”。

3. 在页面右侧显示的信息中，找到你要使用的订阅，并记下“订阅 ID”列中的值。

保存该订阅 ID，因为稍后你要用到它。

##<a id="create"></a>创建应用程序

1. 打开 Visual Studio 2012、2013 或 2015。

2. 在“文件”菜单中，选择“新建”，然后选择“项目”。

3. 对于新项目，请键入或选择以下值。

	<table>
	<tr>
	<th>属性</th>
	<th>值</th>
	</tr>
	<tr>
	<th>类别</th>
	<th>模板/Visual C#/Windows</th>
	</tr>
	<tr>
	<th>模板</th>
	<th>控制台应用程序</th>
	</tr>
	<tr>
	<th>Name</th>
	<th>SubmitPigJob</th>
	</tr>
	</table>

4. 单击“确定”以创建该项目。

5. 从“工具”菜单中选择“库包管理器”或“Nuget 包管理器”，然后选择“包管理器控制台”。

6. 在控制台中运行以下命令，以安装 .NET SDK 包。

		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre

7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。将现有代码替换为以下内容。

		using System;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;
		
		namespace HDInsightSubmitPigJobsDotNet
		{
		    class Program
		    {
		        static void Main(string[] args)
		        {
					var ExistingClusterName = "<HDInsightClusterName>";
					var ExistingClusterUri = ExistingClusterName + ".azurehdinsight.cn";
					var ExistingClusterUsername = "<HDInsightClusterHttpUsername>";
					var ExistingClusterPassword = "<HDInsightClusterHttpUserPassword>";
		
		            // The Pig Latin statements to run
		            string queryString = "LOGS = LOAD 'wasb:///example/data/sample.log';" +
		                "LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;" +
		                "FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;" +
		                "GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;" +
		                "FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;" +
		                "RESULT = order FREQUENCIES by COUNT desc;" +
		                "DUMP RESULT;";
		
		
		            HDInsightJobManagementClient _hdiJobManagementClient;
		            var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
		            _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
		
		            // Define the Pig job
		            var parameters = new PigJobSubmissionParameters()
		            {
		                Query = queryString,
		            };
		
		            System.Console.WriteLine("Submitting the Pig job to the cluster...");
		            var response = _hdiJobManagementClient.JobManagement.SubmitPigJob(parameters);
		            System.Console.WriteLine("Validating that the response is as expected...");
		            System.Console.WriteLine("Response status code is " + response.StatusCode);
		            System.Console.WriteLine("Validating the response object...");
		            System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
		            Console.WriteLine("Press ENTER to continue ...");
		            Console.ReadLine();
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

<!---HONumber=Mooncake_0530_2016-->