<properties
	pageTitle="以编程方式访问 Hadoop YARN 应用程序日志 | Azure"
	description="以编程方式访问 HDInsight 中 Hadoop 群集上的应用程序日志。"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian" 
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="04/28/2016"
	wacn.date="06/29/2016"/>

# 在基于 Windows 的 HDInsight 上访问 YARN 应用程序日志

本主题说明如何访问 Azure HDInsight 中的 Hadoop 群集上已完成的 YARN (Yet Another Resource Negotiator) 应用程序的日志

### 先决条件

- 基于 Windows 的 HDInsight 群集。请参阅[在 HDInsight 中创建基于 Windows 的 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。


## YARN Timeline Server

<a href="http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html" target="_blank">YARN Timeline Server</a> 通过两个不同的接口提供已完成之应用程序的相关泛型信息，以及架构特定应用程序信息。具体而言：

* 存储及检索 HDInsight 群集上泛型应用程序信息的功能已在版本 3.1.1.374 或更新版本上启用。
* Timeline Server 的架构特定应用程序信息组件当前在 HDInsight 群集上并未提供。


应用程序的相关泛型信息包含以下类型的数据：

* 应用程序 ID（应用程序的唯一标识符）
* 启动应用程序的用户
* 为完成应用程序而进行的尝试的相关信息
* 任何给定应用程序尝试所用的容器

在 HDInsight 群集上，由 Azure 资源管理员将这项信息存储到默认存储帐户的默认容器中的历史记录存储中。可以通过 REST API 检索有关已完成的应用程序的此类泛型数据：

    GET on https://<cluster-dns-name>.azurehdinsight.cn/ws/v1/applicationhistory/apps


##<a name="YARNAppsAndLogs"></a> YARN 应用程序和日志

YARN 通过将资源管理与应用程序计划/监视相分离，来支持多种编程模型（MapReduce 就是其中之一）。这是通过全局 *ResourceManager* (RM)、按辅助节点 *NodeManagers* (NM) 和按应用程序 *ApplicationMasters* (AM) 来实现的。按应用程序 AM 与 RM 协商用于运行应用程序的资源（CPU、内存、磁盘、网络）。RM 与 NM 合作来授予这些资源（以 *容器* 的形式授予）。AM 负责跟踪 RM 分配给它的容器的进度。根据应用程序的性质，一个应用程序可能需要多个容器。

此外，每个应用程序可能包含多个 *应用程序尝试* ，为了在应用程序崩溃时，或因 AM 与 RM 之间通信中断时，完成应用程序。因此，容器是授予应用程序的特定尝试。在某种意义上，容器提供 YARN 应用程序所运行的基本工作单位的内容，而在容器的上下文中完成的所有工作都在分配给该容器的单个辅助节点上运行。请参阅 [YARN 的概念][YARN-concepts]，以获取更多参考信息。

应用程序日志（和关联的容器日志）在对有问题的 Hadoop 应用程序进行调试上相当重要。YARN 以[日志聚合][log-aggregation]功能提供一个良好的框架，以收集、聚合及存储应用程序日志。日志聚合功能让访问应用程序日志更具确定性，因为它会聚合辅助节点上所有容器的日志，并在应用程序完成之后，将它们按每个辅助节点一个聚合日志的方式存储在默认文件系统上。应用程序可能使用数百或数千个容器，但在单个辅助节点上运行的所有容器的日志将一律聚合成单个文件，也就是为应用程序所用的每个辅助节点生成一个日志。默认情况下，日志聚合已在 HDInsight 群集（3.0 和更高版本）上启用，在群集的默认容器中，可以找到聚合的日志，位置如下：

	wasb:///app-logs/<user>/logs/<applicationId>

在该位置中， *user* 是启动应用程序的用户名， *applicationId* 是 YARN RM 分配的应用程序唯一标识符。

你无法直接阅读聚合的日志，因为它们是以 [TFile][T-file]（由容器编制索引的[二进制格式][binary-format]）编写的。YARN 提供 CLI 工具，可针对你感兴趣的应用程序或容器，将这些日志转储成纯文本。你可以直接在群集节点上（通过 RDP 连接到节点之后）运行以下 YARN 命令之一，以纯文本格式查看这些日志：

	yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
	yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>

下一部分说明如何以编程方式访问应用程序或容器特定的日志，而无需通过 RDP 连接到你的 HDInsight 群集。

## <a name="enumerate-and-download"></a>以编程方式枚举应用程序和下载日志

若要使用以下代码示例，你必须下载最新版本的 HDInsight .NET SDK，以满足上述先决条件。请参阅相应位置提供的说明。

以下代码演示了如何使用新 API 来枚举应用程序，以及下载已完成应用程序的日志。

> [AZURE.NOTE] 以下 API 只能针对版本为 3.1.1.374 或更高的“运行中”Hadoop 群集执行。添加以下指令：

	using Microsoft.Hadoop.Client;
	using Microsoft.WindowsAzure.Management.HDInsight;

这些指令引用以下代码中新定义的 API。以下代码段将在订阅上的“运行中”群集上创建一个应用程序历史记录客户端。

	string subscriptionId = "<your-subscription-id>";
	string clusterName = "<your-cluster-name>";
	string certName = "<your-subscription-management-cert-name>";

	// Create an HDInsight client
	X509Store store = new X509Store(StoreName.My, StoreLocation.LocalMachine);
	store.Open(OpenFlags.ReadOnly);
	X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>()
	                            .Single(x => x.FriendlyName == certName);

	HDInsightCertificateCredential creds =
				new HDInsightCertificateCredential(new Guid(subscriptionId), cert);

	IHDInsightClient client = HDInsightClient.Connect(creds);

	// Get the cluster on which your applications were run
	// The cluster needs to be in the "Running" state
	ClusterDetails cluster = client.GetCluster(clusterName);

	// Create an Application History client against your cluster
	IHDInsightApplicationHistoryClient appHistoryClient =
				cluster.CreateHDInsightApplicationHistoryClient(TimeSpan.FromMinutes(5));


现在，你可以使用应用程序历史记录客户端来列出已完成的应用程序，根据条件筛选应用程序，以及下载相关的应用程序日志。以下代码段演示了如何以编程方式执行此操作。

	// Local download folder location where the logs will be placed
	string downloadLocation = "E:\\YarnApplicationLogs";

	// List completed applications on your cluster that were submitted in the last 24 hours but failed
	// Search for applications based on application name
	string appNamePrefix = "your-app-name-prefix";
	DateTime endTime = DateTime.UtcNow;
	DateTime startTime = endTime.AddHours(-24);
	IEnumerable<ApplicationDetails> applications = appHistoryClient
	                .ListCompletedApplications(startTime, endTime)
	                .Where(app =>
	                    app.GetApplicationFinalStatusAsEnum() == ApplicationFinalStatus.Failed
	                    && app.Name.StartsWith(appNamePrefix));

	// Download logs for failed or killed applications
	// This will generate one log file for each application
	foreach (ApplicationDetails application in applications)
	{
	    appHistoryClient.DownloadApplicationLogs(application, downloadLocation);
	}

上述代码使用应用程序历史记录客户端来列出/查找所需的应用程序，然后将这些应用程序的日志下载到本地文件夹。

或者，也可以使用以下代码段针对已知其应用程序 ID 的应用程序来下载日志。应用程序 ID 是 RM 分配给应用程序的全局唯一标识符。其构建方式是使用 RM 的开始时间，加上提交给它的应用程序的单调递增计数器。应用程序 ID 格式为“application\_&lt;RM-start-time&gt;\_&lt;Counter&gt;”。请注意，应用程序 ID 与作业 ID 完全不同。作业 ID 是特定于 MapReduce 框架的概念，而应用程序 ID 是不区分框架的 YARN 概念。在 YARN 中，作业 ID 标识特定的 MapReduce 作业，此作业由提交给 RM 的 MapReduce 应用程序的 AM 处理。

	// Download application logs for an application whose application ID is known
	string applicationId = "application_1416017767088_0028";
	ApplicationDetails someApplication = appHistoryClient.GetApplicationDetails(applicationId);
	appHistoryClient.DownloadApplicationLogs(someApplication, downloadLocation);

如果需要，你也可以下载应用程序使用的每个容器（或任何特定容器）的日志，如下所示。

	ApplicationDetails someApplication = appHistoryClient.GetApplicationDetails(applicationId);

	// Download logs separately for each container of application(s) of interest
	// This will generate one log file per container
	IEnumerable<ApplicationAttemptDetails> applicationAttempts =
				appHistoryClient.ListApplicationAttempts(someApplication);

	ApplicationAttemptDetails finalAttempt = applicationAttempts
	    		.Single(x => x.ApplicationAttemptId == someApplication.LatestApplicationAttemptId);

	IEnumerable<ApplicationContainerDetails> containers =
				appHistoryClient.ListApplicationContainers(finalAttempt);

	foreach (ApplicationContainerDetails container in containers)
	{
	    appHistoryClient.DownloadApplicationLogs(container, downloadLocation);
	}



[YARN-timeline-server]: http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html
[log-aggregation]: http://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/
[T-file]: https://issues.apache.org/jira/secure/attachment/12396286/TFile%20Specification%2020081217.pdf
[binary-format]: https://issues.apache.org/jira/browse/HADOOP-3315
[YARN-concepts]: http://hortonworks.com/blog/apache-hadoop-yarn-concepts-and-applications/

<!---HONumber=Mooncake_0307_2016-->