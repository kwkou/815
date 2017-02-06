<properties
    pageTitle="使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集 | Azure"
    description="了解如何使用 HDInsight .NET SDK 针对 HDInsight 中的 Hadoop 群集执行管理任务。"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    tags="azure-portal"
    author="mumian"
    documentationcenter="" />
<tags
    ms.assetid="fd134765-c2a0-488a-bca6-184d814d78e9"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集
[AZURE.INCLUDE [选择器](../../includes/hdinsight-portal-management-selector.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 [HDInsight.NET SDK](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx) 管理 HDInsight 群集。

**先决条件**

在开始本文前，你必须具有以下项：

* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

## <a name="connect-to-azure-hdinsight"></a> 连接到 Azure HDInsight

需要以下 Nuget 包：

    Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
    Install-Package Microsoft.Azure.Management.ResourceManager -Pre
    Install-Package Microsoft.Azure.Management.HDInsight

以下代码示例演示如何先连接到 Azure，然后管理 Azure 订阅下面的 HDInsight 群集。

    using System;
    using Microsoft.Azure;
    using Microsoft.Azure.Management.HDInsight;
    using Microsoft.Azure.Management.HDInsight.Models;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;

    namespace HDInsightManagement
    {
        class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;
            // Replace with your AAD tenant ID if necessary
            private const string TenantId = UserTokenProvider.CommonTenantId; 
            private const string SubscriptionId = "<Your Azure Subscription ID>";
            // This is the GUID for the PowerShell client. Used for interactive logins in this example.
            private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";
            private static Uri BaseUri = new Uri("https://management.chinacloudapi.cn/");

            static void Main(string[] args)
            {
                // Authenticate and get a token
                var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
                // Flag subscription for HDInsight, if it isn't already.
                EnableHDInsight(authToken);
                // Get an HDInsight management client
                _hdiManagementClient = new HDInsightManagementClient(authToken, BaseUri);

                // insert code here

                System.Console.WriteLine("Press ENTER to continue");
                System.Console.ReadLine();
            }

            /// <summary>
            /// Authenticate to an Azure subscription and retrieve an authentication token
            /// </summary>
            static TokenCloudCredentials Authenticate(string TenantId, string ClientId, string SubscriptionId)
            {
                var authContext = new AuthenticationContext("https://login.chinacloudapi.cn/" + TenantId);
                var tokenAuthResult = authContext.AcquireToken("https://management.core.chinacloudapi.cn/", 
                    ClientId, 
                    new Uri("urn:ietf:wg:oauth:2.0:oob"), 
                    PromptBehavior.Always, 
                    UserIdentifier.AnyUser);
                return new TokenCloudCredentials(SubscriptionId, tokenAuthResult.AccessToken);
            }
            /// <summary>
            /// Marks your subscription as one that can use HDInsight, if it has not already been marked as such.
            /// </summary>
            /// <remarks>This is essentially a one-time action; if you have already done something with HDInsight
            /// on your subscription, then this isn't needed at all and will do nothing.</remarks>
            /// <param name="authToken">An authentication token for your Azure subscription</param>
            static void EnableHDInsight(TokenCloudCredentials authToken)
            {
                // Create a client for the Resource manager and set the subscription ID
                var resourceManagementClient = new ResourceManagementClient(BaseUri, new TokenCredentials(authToken.Token));
                resourceManagementClient.SubscriptionId = SubscriptionId;
                // Register the HDInsight provider
                var rpResult = resourceManagementClient.Providers.Register("Microsoft.HDInsight");
            }
        }
    }

运行此程序时，将看到提示。如果不想看到提示，请参阅[创建非交互式身份验证 .NET HDInsight 应用程序](/documentation/articles/hdinsight-create-non-interactive-authentication-dotnet-applications/)。

## 创建群集
请参阅[使用 .NET SDK 在 HDInsight 中创建基于 Linux 的群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-dotnet-sdk/)。

## <a name="list-clusters"></a> 列出群集
以下代码片段列出了群集和一些属性：

    var results = _hdiManagementClient.Clusters.List();
    foreach (var name in results.Clusters) {
        Console.WriteLine("Cluster Name: " + name.Name);
        Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
        Console.WriteLine("\t Cluster location: " + name.Location);
        Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
    }

## <a name="delete-clusters"></a> 删除群集
使用以下代码片段以同步或异步方式删除群集：

    _hdiManagementClient.Clusters.Delete("<Resource Group Name>", "<Cluster Name>");
    _hdiManagementClient.Clusters.DeleteAsync("<Resource Group Name>", "<Cluster Name>");

## <a name="scale-clusters"></a> 缩放群集
使用群集缩放功能，可更改 Azure HDInsight 中运行的群集使用的辅助节点数，而无需重新创建群集。

> [AZURE.NOTE]
只支持使用 HDInsight 3.1.3 或更高版本的群集。如果不确定群集的版本，可以查看“属性”页面。请参阅[列出并显示群集](/documentation/articles/hdinsight-administer-use-portal-linux/#list-and-show-clusters)。
> 
> 

更改 HDInsight 支持的每种类型的群集所用数据节点数的影响：

* Hadoop
  
    可顺利增加正在运行的 Hadoop 群集中的辅助节点数，而不会影响任何挂起或运行中的作业。也可在操作进行中提交新作业。系统会正常处理失败的缩放操作，让群集始终保持正常运行状态。
  
    减少数据节点数目以缩减 Hadoop 群集时，系统会重新启动群集中的某些服务。这会导致所有正在运行和挂起的作业在缩放操作完成时失败。但是，可在操作完成后重新提交这些作业。
* HBase
  
    可在 HBase 群集运行时顺利添加或删除节点。完成缩放操作后的几分钟内，区域服务器将自动平衡。但也可手动平衡区域服务器，方法是登录到群集的头节点，然后在命令提示符窗口中运行以下命令：
  
        >pushd %HBASE_HOME%\bin
        >hbase shell
        >balancer
* Storm
  
    可在 Storm 群集运行时顺利添加或删除数据节点。但是，缩放操作成功完成后，需要重新平衡拓扑。
  
    可以使用两种方法来完成重新平衡操作：
  
  * Storm Web UI
  * 命令行界面 (CLI) 工具
    
    有关更多详细信息，请参阅 [Apache Storm 文档](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。
    
    HDInsight 群集上提供了 Storm Web UI：
    
    ![HDInsight Storm 缩放重新平衡](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)
    
    以下是有关如何使用 CLI 命令重新平衡 Storm 拓扑的示例：
    
        ## Reconfigure the topology "mytopology" to use 5 worker processes,
        ## the spout "blue-spout" to use 3 executors, and
        ## the bolt "yellow-bolt" to use 10 executors
        $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

以下代码片段显示如何以同步或异步方式调整群集的大小：

    _hdiManagementClient.Clusters.Resize("<Resource Group Name>", "<Cluster Name>", <New Size>);   
    _hdiManagementClient.Clusters.ResizeAsync("<Resource Group Name>", "<Cluster Name>", <New Size>);   

## <a name="grantrevoke-access" id="grant/revoke-access"></a> 授予/撤消访问权限
HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。若要撤消：

    var httpParams = new HttpSettingsParameters
    {
        HttpUserEnabled = false,
        HttpUsername = "admin",
        HttpPassword = "*******",
    };
    _hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);

若要授予：

    var httpParams = new HttpSettingsParameters
    {
        HttpUserEnabled = enable,
        HttpUsername = "admin",
        HttpPassword = "*******",
    };
    _hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);

> [AZURE.NOTE]
通过授予/撤消访问权限，你将重置群集用户名和密码。
> 
> 

也可以通过门户完成此操作。请参阅[使用 Azure 门户预览管理 HDInsight][hdinsight-admin-portal]。

## <a name="update-http-user-credentials"></a> 更新 HTTP 用户凭据
这与[授予/撤消 HTTP 访问权限](#grant/revoke-access)是同一过程。如果已授予群集 HTTP 访问权限，必须先撤消该访问权限。然后再使用新的 HTTP 用户凭据授予访问权限。

## <a name="find-the-default-storage-account"></a> 查找默认存储帐户
以下代码片段演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

    var results = _hdiManagementClient.Clusters.GetClusterConfigurations(<Resource Group Name>, <Cluster Name>, "core-site");
    foreach (var key in results.Configuration.Keys)
    {
        Console.WriteLine(String.Format("{0} => {1}", key, results.Configuration[key]));
    }

## 提交作业
**提交 MapReduce 作业**

请参阅[在 HDInsight 中运行 Hadoop MapReduce 示例](/documentation/articles/hdinsight-hadoop-run-samples-linux/)。

**提交 Hive 作业**

请参阅[使用 .NET SDK 运行 Hive 查询）](/documentation/articles/hdinsight-hadoop-use-hive-dotnet-sdk/)。

**提交 Pig 作业**

请参阅[使用 .NET SDK 运行 Pig 作业](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk/)。

**提交 Sqoop 作业**

请参阅[将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-sqoop-dotnet-sdk/)。

**提交 Oozie 作业**

请参阅[在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流](/documentation/articles/hdinsight-use-oozie-linux-mac/)。

## 将数据上传到 Azure Blob 存储
请参阅[将数据上传到 HDInsight][hdinsight-upload-data]。

## 另请参阅
* [HDInsight .NET SDK 参考文档](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)
* [使用 Azure 门户预览管理 HDInsight][hdinsight-admin-portal]
* [使用命令行借口管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上传到 HDInsight][hdinsight-upload-data]
* [Azure HDInsight 入门][hdinsight-get-started]

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-provision-custom-options]: /documentation/articles/hdinsight-provision-clusters/#configuration
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-portal-linux/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-flight]: /documentation/articles/hdinsight-analyze-flight-delay-data/

<!---HONumber=Mooncake_0120_2017-->