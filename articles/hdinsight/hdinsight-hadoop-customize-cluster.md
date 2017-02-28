<properties
    pageTitle="使用脚本操作自定义 HDInsight 群集 | Azure"
    description="了解如何使用脚本操作自定义 HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="3a63e216-4163-40c1-aa04-6b42fd0162ad"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/05/2016"
    wacn.date="02/20/2017"
    ms.author="nitinme" />

# 使用脚本操作自定义基于 Windows 的 HDInsight 群集
创建群集期间，可以使用**脚本操作**调用[自定义脚本](/documentation/articles/hdinsight-hadoop-script-actions/)，以便在群集上安装其他软件。

本文中的信息特定于基于 Windows 的 HDInsight 群集。

也可以使用其他各种方法自定义 HDInsight 群集，例如包括其他 Azure 存储帐户、更改 hadoop 配置文件（core-site.xml、hive-site.xml 等），或者将共享库（如 Hive、Oozie）添加到群集中的共同位置。这些自定义可通过使用 Azure PowerShell、Azure HDInsight .NET SDK 或 Azure 门户预览来完成。有关详细信息，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell-cli-and-dotnet-sdk.md)]

## 群集创建过程中的脚本操作
只能在创建群集期间使用脚本操作。下图说明了在创建期间执行脚本操作的时间：

![群集创建期间的 HDInsight 群集自定义和阶段][img-hdi-cluster-states]

当脚本运行时，群集进入 **ClusterCustomization** 阶段。在此阶段，脚本在系统管理员帐户下，以并行方式在群集中所有指定的节点上运行，并在节点上提供完全系统管理员权限。

> [AZURE.NOTE]
由于在 **ClusterCustomization** 阶段期间在群集节点上拥有系统管理员权限，因此可以使用脚本运行作业，例如停止和启动服务，包括 Hadoop 相关服务。因此，作为脚本的一部分，必须在脚本运行完成前，确保 Hadoop 相关服务已启动且正在运行。创建时成功确定群集的运行状况和状态需要这些服务。如果更改群集上影响这些服务的任何配置，必须使用提供的帮助函数。有关帮助函数的详细信息，请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。
>
>

脚本的输出和错误日志文件存储在为群集指定的默认存储帐户中。这些日志存储在名为 **u<\\cluster-name-fragment><\\time-stamp>setuplog** 的表中。这是从群集中所有节点（头节点和辅助节点）上运行的脚本聚合的日志文件。

每个群集可接受按指定顺序调用的多个脚本操作。脚本可在头节点和/或辅助节点上运行。

HDInsight 提供了多个脚本用于在 HDInsight 群集上安装以下组件：

| Name | 脚本 |
| --- | --- |
| **安装 Spark** |https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1。请参阅[在 HDInsight 群集上安装并使用 Spark][hdinsight-install-spark]。 |
| **安装 R** |https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1。请参阅[在 HDInsight 群集上安装并使用 R][hdinsight-install-r]。 |
| **安装 Solr** |https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1。请参阅[在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)。 |
| - **安装 Giraph** |https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1。请参阅[在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)。 |
| **预加载 Hive 库** |https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1。请参阅 [Add Hive libraries on HDInsight clusters](/documentation/articles/hdinsight-hadoop-add-hive-libraries/)（在 HDInsight 群集上添加 Hive 库） |

## 使用 Azure 门户预览调用脚本
**在 Azure 门户预览中**

1. 根据 [Create Hadoop clusters in HDInsight](/documentation/articles/hdinsight-provision-clusters/)（在 HDInsight 中创建 Hadoop 群集）中所述开始创建群集。
2. 在“脚本操作”边栏选项卡的“可选配置”下，单击“添加脚本操作”，提供有关脚本操作的详细信息，如下所示：

    ![使用脚本操作自定义群集](./media/hdinsight-hadoop-customize-cluster/HDI.CreateCluster.8.png "使用脚本操作自定义群集")  


    <table border='1'>
        <tr><th>属性</th><th>值</th></tr>
        <tr><td>名称</td>
            <td>指定脚本操作的名称。</td></tr>
        <tr><td>脚本 URI</td>
            <td>指定调用以自定义群集的脚本的 URI。</td></tr>
        <tr><td>头节点/辅助节点</td>
            <td>指定运行自定义脚本的节点（**Head** 或 **Worker**）。</b>
        <tr><td>Parameters</td>
            <td>根据脚本的需要，请指定参数。</td></tr>
    </table>

    按 ENTER 可添加多个脚本操作，以便在群集上安装多个组件。
3. 单击“选择”可保存脚本操作配置，并继续创建群集。

## <a name="call-scripts-using-azure-powershell"></a> 使用 Azure PowerShell 调用脚本
以下 PowerShell 脚本演示如何在基于 Windows 的 HDInsight 群集上安装 Spark。

    # Provide values for these variables
    $subscriptionID = "<Azure Suscription ID>" # After "Login-AzureRmAccount -EnvironmentName AzureChinaCloud", use "Get-AzureRmSubscription" to list IDs.

    $nameToken = "<Enter A Name Token>"  # The token is use to create Azure service names.
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $resourceGroupName = $namePrefix + "rg"
    $location = "CHINA EAST" # used for creating resource group, storage account, and HDInsight cluster.

    $hdinsightClusterName = $namePrefix + "spark"
    $httpUserName = "admin"
    $httpPassword = "<Enter a Password>"

    $defaultStorageAccountName = "$namePrefix" + "store"
    $defaultBlobContainerName = $hdinsightClusterName

    #############################################################
    # Connect to Azure
    #############################################################

    Try{
        Get-AzureRmSubscription
    }
    Catch{
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    }
    Select-AzureRmSubscription -SubscriptionId $subscriptionID

    #############################################################
    # Prepare the dependent components
    #############################################################

    # Create resource group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

    # Create storage account
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_GRS
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $storageAccountKey  
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageAccountContext

    #############################################################
    # Create cluster with ApacheSpark
    #############################################################

    # Specify the configuration options
    $config = New-AzureRmHDInsightClusterConfig `
                -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
                -DefaultStorageAccountKey $defaultStorageAccountKey

    # Add a script action to the cluster configuration
    $config = Add-AzureRmHDInsightScriptAction `
                -Config $config `
                -Name "Install Spark" `
                -NodeType HeadNode `
                -Uri https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1 `

    # Start creating a cluster with Spark installed
    New-AzureRmHDInsightCluster `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $hdinsightClusterName `
            -Location $location `
            -ClusterSizeInNodes 2 `
            -ClusterType Hadoop `
            -OSType Windows `
            -DefaultStorageContainer $defaultBlobContainerName `
            -Config $config

若要安装其他软件，需要替换脚本中的脚本文件：

出现提示时，请输入群集的凭据。创建群集可能需要几分钟时间。

## <a name="call-scripts-using-net-sdk"></a> 使用 .NET SDK 调用脚本
以下示例演示如何在基于 Windows 的 HDInsight 群集上安装 Spark。若要安装其他软件，需要替换代码中的脚本文件。

**创建具有 Spark 的 HDInsight 群集**

1. 在 Visual Studio 中创建 C# 控制台应用程序。
2. 通过 Nuget 包管理器控制台运行以下命令。

        Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
        Install-Package Microsoft.Azure.Management.ResourceManager -Pre
        Install-Package Microsoft.Azure.Management.HDInsight
3. 在 Program.cs 文件中使用以下 using 语句：

        using System;
        using System.Security;
        using Microsoft.Azure;
        using Microsoft.Azure.Management.HDInsight;
        using Microsoft.Azure.Management.HDInsight.Models;
        using Microsoft.Azure.Management.ResourceManager;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Rest;
        using Microsoft.Rest.Azure.Authentication;
4. 将类中的代码替换为以下内容：

        private static HDInsightManagementClient _hdiManagementClient;

        // Replace with your AAD tenant ID if necessary
        private const string TenantId = UserTokenProvider.CommonTenantId;
        private const string SubscriptionId = "<Your Azure Subscription ID>";
        // This is the GUID for the PowerShell client. Used for interactive logins in this example.
        private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";
        private static Uri BaseUri = new Uri("https://management.chinacloudapi.cn/");
        private const string ResourceGroupName = "<ExistingAzureResourceGroupName>";
        private const string NewClusterName = "<NewAzureHDInsightClusterName>";
        private const int NewClusterNumWorkerNodes = 2;
        private const string NewClusterLocation = "China East";
        private const string NewClusterVersion = "3.2";
        private const string ExistingStorageName = "<ExistingAzureStorageAccountName>";
        private const string ExistingStorageKey = "<ExistingAzureStorageAccountKey>";
        private const string ExistingContainer = "<ExistingAzureBlobStorageContainer>";
        private const string NewClusterType = "Hadoop";
        private const OSType NewClusterOSType = OSType.Windows;
        private const string NewClusterUsername = "<HttpUserName>";
        private const string NewClusterPassword = "<HttpUserPassword>";

        static void Main(string[] args)
        {
            System.Console.WriteLine("Running");

            // Authenticate and get a token
            var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
            // Flag subscription for HDInsight, if it isn't already.
            EnableHDInsight(authToken);
            // Get an HDInsight management client
            _hdiManagementClient = new HDInsightManagementClient(authToken, BaseUri);

            CreateCluster();
        }

        private static void CreateCluster()
        {
            var parameters = new ClusterCreateParameters
            {
                ClusterSizeInNodes = NewClusterNumWorkerNodes,
                Location = NewClusterLocation,
                ClusterType = NewClusterType,
                OSType = NewClusterOSType,
                Version = NewClusterVersion,

                DefaultStorageAccountName = ExistingStorageName,
                DefaultStorageAccountKey = ExistingStorageKey,
                DefaultStorageContainer = ExistingContainer,

                UserName = NewClusterUsername,
                Password = NewClusterPassword,
            };

            ScriptAction sparkScriptAction = new ScriptAction("Install Spark",
                new Uri("https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1"), "");

            parameters.ScriptActions.Add(ClusterNodeType.HeadNode, new System.Collections.Generic.List<ScriptAction> { sparkScriptAction });
            parameters.ScriptActions.Add(ClusterNodeType.WorkerNode, new System.Collections.Generic.List<ScriptAction> { sparkScriptAction });

            _hdiManagementClient.Clusters.Create(ResourceGroupName, NewClusterName, parameters);
        }

        /// <summary>
        /// Authenticate to an Azure subscription and retrieve an authentication token
        /// </summary>
        /// <param name="TenantId">The AAD tenant ID</param>
        /// <param name="ClientId">The AAD client ID</param>
        /// <param name="SubscriptionId">The Azure subscription ID</param>
        /// <returns></returns>
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
5. 按 **F5** 运行应用程序。

## 支持 HDInsight 群集上使用的开源软件
Azure HDInsight 服务是一个弹性平台，可让你使用围绕着 Hadoop 形成的开放源代码技术生态系统，在云中生成大数据应用程序。Azure 为开放源代码技术提供一般级别的支持，如 <a href="/support/faq/" target="_blank">Azure 支持常见问题网站</a>上的**支持范围**部分中所述。HDInsight 服务为如下所述的某些组件提供附加的支持级别。

HDInsight 服务提供两种类型的开源组件：

* **内置组件** - 这些组件预先安装在 HDInsight 群集上，并提供群集的核心功能。例如，Yarn ResourceManager、Hive 查询语言 (HiveQL) 和 Mahout 库均属于此类别。[HDInsight 提供的 Hadoop 群集版本有哪些新增功能](/documentation/articles/hdinsight-component-versioning/)</a>中提供了群集组件的完整列表。
* **自定义组件** - 作为群集用户，可以安装，或者在工作负荷中使用由社区提供或自己创建的任何组件。

完全支持内置组件，Microsoft 支持部门将帮助找出并解决与这些组件相关的问题。

> [AZURE.WARNING]
完全支持通过 HDInsight 群集提供的组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。
><p>
> 自定义组件可获得合理范围的支持，有助于进一步解决问题。这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=hdinsight)、[Azure CSDN](http://azure.csdn.net)。此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)、[Spark](http://spark.apache.org/)。
>
>

HDInsight 服务可提供多种方法使用自定义组件。无论在群集上使用或安装组件的方式如何，均适用相同级别的支持。以下是可在 HDInsight 群集上使用自定义组件的最常见方法列表：

1. 作业提交 - 可将 Hadoop 或其他类型的作业提交到执行或使用自定义组件的群集。
2. 群集自定义 - 在群集创建期间，可指定将安装在群集节点的其他设置和自定义组件。
3. 示例 - 对于常见的自定义组件，Microsoft 和其他用户可能会提供演示如何在 HDInsight 群集上使用这些组件的示例。不会针对这些示例提供支持。

## 开发脚本操作脚本
请参阅[为 HDInsight 开发脚本操作脚本][hdinsight-write-script]。

## 另请参阅
* [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]说明了如何使用其他自定义选项创建 HDInsight 群集。
* [为 HDInsight 开发脚本操作脚本][hdinsight-write-script]
* [在 HDInsight 群集上安装并使用 Spark][hdinsight-install-spark]
* [在 HDInsight 群集上安装并使用 R][hdinsight-install-r]
* [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)
* [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)

[hdinsight-install-spark]: /documentation/articles/hdinsight-hadoop-spark-install/
[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-write-script]: /documentation/articles/hdinsight-hadoop-script-actions/
[hdinsight-provision-cluster]: /documentation/articles/hdinsight-provision-clusters/
[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster/HDI-Cluster-state.png "群集创建期间的阶段"

<!---HONumber=Mooncake_0213_2017-->