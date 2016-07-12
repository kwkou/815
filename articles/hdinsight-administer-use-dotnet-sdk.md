<properties
	pageTitle="使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集 | Azure"
	description="了解如何使用 HDInsight .NET SDK 针对 HDInsight 中的 Hadoop 群集执行管理任务。"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="05/02/2016"
	wacn.date="06/06/2016"/>

# 使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [HDInsight.NET SDK](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx) 管理 HDInsight 群集。


**先决条件**

在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。


##连接到 Azure HDInsight

需要以下 Nuget 包：

	Install-Package Microsoft.WindowsAzure.Management.HDInsight

以下代码示例演示如何先连接到 Azure，然后管理 Azure 订阅下面的 HDInsight 群集。

	using System;
	using Microsoft.WindowsAzure.Management.HDInsight;
	using System.Security.Cryptography.X509Certificates;

	namespace HDInsightManagement
	{
		class Program
		{
			private static IHDInsightClient _hdinsightClient;
			private static String SubscriptionId = "<Your Azure Subscription ID>";
			private static Uri baseUri = new Uri("https://management.core.chinacloudapi.cn");
			private static X509Certificate2 cert = new X509Certificate2("c:/path/to/cert.cer");

			static void Main(string[] args)
			{
				_hdinsightClient = HDInsightClient.Connect(new HDInsightCertificateCredential(new Guid(SubscriptionId), cert, baseUri));

				// insert code here

				System.Console.WriteLine("Press ENTER to continue");
				System.Console.ReadLine();
			}
		}
	}

若要使用此代码段，需要创建证书文件并将其上载到 Azure。有关详细信息，请参阅 [Upload an Azure Management API Management Certificate（上载 Azure Management API 管理证书）](/documentation/articles/azure-api-management-certs/)。

##列出群集

以下代码段列出了群集和一些属性：

    var results = _hdinsightClient.ListClusters();
    foreach (var name in results) {
        Console.WriteLine("Cluster Name: " + name.Name);
        Console.WriteLine("\t Cluster type: " + name.ClusterType);
        Console.WriteLine("\t Cluster location: " + name.Location);
        Console.WriteLine("\t Cluster version: " + name.Version);
    }

##删除群集

使用以下代码段以同步或异步方式删除群集：

    _hdinsightClient.DeleteCluster("<Cluster Name>");
    _hdinsightClient.DeleteClusterAsync("<Cluster Name>");

##<a name="grant/revoke-access"></a>授予/撤消访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

- ODBC
- JDBC
- Oozie
- Templeton


默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。若要撤消：

	var cluster = _hdinsightClient.GetCluster("<Cluster Name>");
    _hdinsightClient.DisableHttp(cluster.Name, cluster.Location);

若要授予：

	_hdinsightClient.EnableHttp(cluster.Name, cluster.Location, "admin","<password>");

>[AZURE.NOTE] 授予/撤消访问权限时，你将重设群集用户的用户名和密码。

也可以使用经典管理门户完成此操作。请参阅 [Administer HDInsight by using the Azure Portal（使用 Azure 经典管理门户管理 HDInsight）][hdinsight-admin-portal]。

##更新 HTTP 用户凭据

这与[授予/撤消 HTTP 访问权限](#grant/revoke-access)是同一过程。如果已授予群集 HTTP 访问权限，必须先撤消该访问权限。然后再使用新的 HTTP 用户凭据授予访问权限。


##查找默认存储帐户

以下代码段演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

	var cluster = _hdinsightClient.GetCluster("<Cluster Name>");
	var storageKey = cluster.DefaultStorageAccount.Key;


##提交作业

**提交 Hive 作业**

请参阅 [Run Hive queries using .NET SDK（使用 .NET SDK 运行 Hive 查询）](/documentation/articles/hdinsight-hadoop-use-hive-dotnet-sdk/)。

**提交 Pig 作业**

请参阅 [Run Pig jobs using .NET SDK（使用 .NET SDK 运行 Pig 作业）](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk-v1/)。

**提交 Sqoop 作业**

请参阅[将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-sqoop-dotnet-sdk/)。

**提交 Oozie 作业**

请参阅[在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流](/documentation/articles/hdinsight-use-oozie/)。

##将数据上载到 Azure Blob 存储
请参阅[将数据上载到 HDInsight][hdinsight-upload-data]。


## 另请参阅
* [HDInsight .NET SDK 参考文档](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)
* [Administer HDInsight by using the Azure Portal（使用 Azure 经典管理门户管理 HDInsight）][hdinsight-admin-portal]
* [使用命令行界面管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上载到 HDInsight][hdinsight-upload-data]
* [Azure HDInsight 入门][hdinsight-get-started]


[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-provision-custom-options]: /documentation/articles/hdinsight-provision-clusters-v1/#configuration
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-flight]: /documentation/articles/hdinsight-analyze-flight-delay-data/



<!---HONumber=Mooncake_0530_2016-->