<!-- not suitable for Mooncake -->


<properties
   pageTitle="使用 .NET SDK 在 HDInsight 中创建基于 Windows 的 Hadoop 群集 | Azure"
   	description="了解如何使用 .NET SDK 创建 Azure HDInsight 的 HDInsight 群集。"
   services="hdinsight"
   documentationCenter=""
   tags="azure-portal"
   authors="mumian"
   manager="jhubbard"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/02/2016"
   wacn.date="02/06/2017"
   ms.author="jgao"/>

# 使用 .NET SDK 在 HDInsight 中创建基于 Windows 的 Hadoop 群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-selector-create-clusters.md)]


了解如何使用 .NET SDK 创建 HDInsight 群集。有关其他群集创建工具和功能，请单击本页顶部的相应选项卡，或参阅[群集创建方法](/documentation/articles/hdinsight-provision-clusters/#cluster-creation-methods)。


##先决条件：

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

在开始按照本文中的说明操作之前，你必须具有以下内容：

- Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
- Visual Studio 2013 或 2015。

### 访问控制要求

[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## 创建群集

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET Framework 应用程序使用 HDInsight 的操作。请遵照以下说明创建一个 Visual Studio 控制台应用程序，并粘贴用于创建群集的代码。

该应用程序需要 Azure 资源组和默认存储帐户。[附录 A](#appx-a-create-dependent-components) 提供了用于创建依赖组件的 PowerShell 脚本。

**创建 Visual Studio 控制台应用程序**

1. 在 Visual Studio 中新建 C# 控制台应用程序。
2. 在 Nuget 包管理器控制台中允许以下 Nuget 命令。

		Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
        Install-Package Microsoft.Azure.Management.ResourceManager -Pre
        Install-Package Microsoft.Azure.Management.HDInsight

6. 在解决方案资源管理器中，双击 **Program.cs** 将其打开、粘贴以下代码，并提供变量的值：

		using System;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Rest;
        using Microsoft.Rest.Azure.Authentication;
        using Microsoft.Azure;
        using Microsoft.Azure.Management.HDInsight;
        using Microsoft.Azure.Management.HDInsight.Models;
        using Microsoft.Azure.Management.ResourceManager;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using System.Net.Http;
		
		namespace CreateHDInsightCluster
		{
			class Program
			{
                // The client for managing HDInsight
				private static HDInsightManagementClient _hdiManagementClient;
		        // Replace with your AAD tenant ID if necessary
                private const string TenantId = UserTokenProvider.CommonTenantId; 
                private const string SubscriptionId = "<Your Azure Subscription ID>";
                // This is the GUID for the PowerShell client. Used for interactive logins in this example.
                private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";
				private const string ExistingResourceGroupName = "<Azure Resource Group Name>";
				private const string ExistingStorageName = "<Default Storage Account Name>.blob.core.chinacloudapi.cn";
				private const string ExistingStorageKey = "<Default Storage Account Key>";
				private const string ExistingBlobContainer = "<Default Blob Container Name>";
				private const string NewClusterName = "<HDInsight Cluster Name>";
				private const int NewClusterNumWorkerNodes = 2;
				private const string NewClusterLocation = "CHINA EAST";     // Must be the same as the default Storage account
				private const OSType NewClusterOsType = OSType.Windows;
				private const string NewClusterType = "Hadoop";
				private const string NewClusterVersion = "3.2";
				private const string NewClusterUsername = "admin";
				private const string NewClusterPassword = "<HTTP User password>";
                

		
				static void Main(string[] args)
				{
					System.Console.WriteLine("Creating a cluster.  The process takes 10 to 20 minutes ...");
		
					// Authenticate and get a token
                    var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
                    // Flag subscription for HDInsight, if it isn't already.
                    EnableHDInsight(authToken);
                    // Get an HDInsight management client
                    _hdiManagementClient = new HDInsightManagementClient(authToken);

                    // Set parameters for the new cluster
                    var parameters = new ClusterCreateParameters
                    {
                        ClusterSizeInNodes = NewClusterNumWorkerNodes,
                        UserName = NewClusterUsername,
                        Password = NewClusterPassword,
                        Location = NewClusterLocation,
                        DefaultStorageAccountName = ExistingStorageName,
                        DefaultStorageAccountKey = ExistingStorageKey,
                        DefaultStorageContainer = ExistingBlobContainer,
                        ClusterType = NewClusterType,
                        OSType = NewClusterOsType
                    };
                    // Create the cluster
                    _hdiManagementClient.Clusters.Create(ExistingResourceGroupName, NewClusterName, parameters);

					System.Console.WriteLine("The cluster has been created. Press ENTER to continue ...");
					System.Console.ReadLine();
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
                    var resourceManagementClient = new ResourceManagementClient(new TokenCredentials(authToken.Token));
                    resourceManagementClient.SubscriptionId = SubscriptionId;
                    // Register the HDInsight provider
                    var rpResult = resourceManagementClient.Providers.Register("Microsoft.HDInsight");
                }
			}
		}

7. 按 **F5** 运行应用程序。控制台窗口应打开并显示应用程序的状态。系统还会提示你输入 Azure 帐户凭据。创建 HDInsight 群集可能需要几分钟时间。



##后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

- [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/) - 了解如何开始使用你的 HDInsight 群集
- [使用 .NET SDK 在 HDInsight 中运行 Hive 作业](/documentation/articles/hdinsight-hadoop-use-hive-dotnet-sdk/)
- [使用 .NET SDK 在 HDInsight 中运行 Pig 作业](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk/)
- [使用 .NET SDK 在 HDInsight 中运行 Sqoop 作业](/documentation/articles/hdinsight-hadoop-use-sqoop-dotnet-sdk/)
- [在 HDInsight 中运行 Oozie 作业](/documentation/articles/hdinsight-use-oozie/)
- [Azure HDInsight SDK 文档][hdinsight-sdk-documentation] - 探索 HDInsight SDK

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx
[azure-preview-portal]: https://manage.windowsazure.cn
[connectionmanager]: http://msdn.microsoft.com/zh-cn/library/mt146773(v=sql.120).aspx
[ssispack]: http://msdn.microsoft.com/zh-cn/library/mt146770(v=sql.120).aspx
[ssisclustercreate]: http://msdn.microsoft.com/zh-cn/library/mt146774(v=sql.120).aspx
[ssisclusterdelete]: http://msdn.microsoft.com/zh-cn/library/mt146778(v=sql.120).aspx


## <a name="appx-a-create-dependent-components"></a>附录 A：创建依赖组件

以下 Azure PowerShell 脚本可用于创建本教程中的 .NET 应用程序所需的依赖组件。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

    ####################################
    # Set these variables
    ####################################
    #region - used for creating Azure service names
    $nameToken = "<Enter an Alias>" 
    #endregion

    ####################################
    # Service names and varialbes
    ####################################
    #region - service names
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $resourceGroupName = $namePrefix + "rg"
    $hdinsightClusterName = $namePrefix + "hdi"
    $defaultStorageAccountName = $namePrefix + "store"
    $defaultBlobContainerName = $hdinsightClusterName

    $location = "China East"
    #endregion

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    ####################################
    # Connect to Azure
    ####################################
    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Login-AzureRmAccount}
    #endregion

    #region - Create an HDInsight cluster
    ####################################
    # Create dependent components
    ####################################
    Write-Host "Creating a resource group ..." -ForegroundColor Green
    New-AzureRmResourceGroup `
        -Name  $resourceGroupName `
        -Location $location

    Write-Host "Creating the default storage account and default blob container ..."  -ForegroundColor Green
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_GRS

    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $defaultStorageContext = New-AzureStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $defaultStorageAccountKey
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageContext #use the cluster name as the container name

    Write-Host "Use the following names in your .NET application" -ForegroundColor Green
    Write-host "Resource Group Name: $resourceGroupName"
    Write-host "Default Storage Account Name: $defaultStorageAccountName"
    Write-host "Default Storage Account Key: $defaultStorageAccountKey"
    Write-host "Default Blob Container Name: $defaultBlobContainerName"

<!---HONumber=Mooncake_Quality_Review_1215_2016-->