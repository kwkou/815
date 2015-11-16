<properties
	pageTitle="在 HDInsight 中使用 R 来自定义群集 | Windows Azure"
	description="了解如何通过脚本操作安装 R，以及如何在 HDInsight 群集上使用 R。"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="10/02/2015"
	wacn.date="11/12/2015"/>

# 在 HDInsight Hadoop 群集上安装并使用 R


了解如何使用 R 通过脚本操作来自定义基于 Windows 的 HDInsight 群集，以及如何在 HDInsight 群集上使用 R。
 
你可以使用*脚本操作*，在 Azure HDInsight 的任何一种群集（Hadoop、Storm、HBase）上安装 R。用于在 HDInsight 群集上安装 R 的示例脚本可通过 [https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1](https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1) 上的只读 Azure 存储 Blob 获得。

**相关文章** 
- [在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters)：有关如何创建 HDInsight 群集的一般信息 
- [使用脚本操作自定义 HDInsight 群集][hdinsight-cluster-customize]：有关如何使用脚本操作自定义 HDInsight 群集的一般信息 
- [针对 HDInsight 开发脚本操作脚本](/documentation/articles/hdinsight-hadoop-script-actions)

## 什么是 R？

<a href="http://www.r-project.org/" target="_blank">统计计算的 R 项目</a>是一种用于统计计算的开放源代码语言和环境。R 提供了数百个内置统计函数及其自己的编程语言，该语言结合了各方面的函数编程和面向对象的编程。它还提供了各种图形功能。R 是面向各个领域最专业的统计学家和科学家的首选编程环境。

R 与 Azure Blob 存储 (WASB) 兼容，这样，存储在此的数据可以在 HDInsight 上使用 R 进行处理。

## 安装 R

用于在 HDInsight 群集上安装 R 的[示例脚本](https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1)可从 Azure 存储中的只读 Blob 获得。本部分提供有关如何在使用 Azure 管理门户预配群集时使用示例脚本的说明。

> [AZURE.NOTE]示例脚本是随同 HDInsight 群集版本 3.1 一起引入的。有关 HDInsight 群集版本的详细信息，请参阅 [HDInsight 群集版本](/documentation/articles/hdinsight-component-versioning)。

1. 根据[使用自定义选项设置群集](/documentation/articles/hdinsight-provision-clusters#portal)中的说明，使用“自定义创建”选项开始设置群集。 
2. 在向导的“脚本操作”页上，单击“添加脚本操作”，以提供有关脚本操作的详细信息，如下所述：


	![Use Script Action to customize a cluster](./media/hdinsight-hadoop-r-scripts/hdi-r-script-action.png "Use Script Action to customize a cluster")

	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>Name</td>
			<td>指定脚本操作的名称，例如 Install <b>R</b>。</td></tr>
		<tr><td>脚本 URI</td>
			<td>指定调用用于自定义群集的脚本的 URI，例如 <i>https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1</i></td></tr>
		<tr><td>节点类型</td>
			<td>指定在其上运行自定义脚本的节点。你可以选择“所有节点”、“仅限头节点”或“仅限从节点”。
		<tr><td>Parameters</td>
			<td>根据脚本的需要，指定参数。但是，用于安装 R 的脚本不需要任何参数，因此，你可以将此项保留为空。</td></tr>
	</table>

	你可以添加多个脚本操作，以在群集上安装多个组件。添加脚本后，请单击复选标记以开始创建群集。

你还可以通过 Azure PowerShell 或 HDInsight .NET SDK 使用脚本在 HDInsight 上安装 R。有关这些过程的说明在本文后面提供。

## 运行 R 脚本
本部分介绍如何在安装有 HDInsight 的 Hadoop 群集上运行 R 脚本。

1. **与群集建立远程桌面连接**：在管理门户中，对创建的安装有 R 的群集启用远程桌面，然后连接到该群集。有关说明，请参阅<a href="/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp" target="_blank">使用 RDP 连接到 HDInsight 群集</a>。

2. **打开 R 控制台**：R 安装将 R 控制台的链接放置在头节点的桌面上。单击它以打开 R 控制台。

3. **运行 R 脚本**：通过粘贴并选择 R 脚本，然后按 ENTER，可以从 R 控制台直接运行该脚本。下面是一个简单的示例脚本，该脚本将生成 1 到 100 的数字，然后将其乘以 2。

		library(rmr2)
		library(rhdfs)
		ints = to.dfs(1:100)
		calc = mapreduce(input = ints, map = function(k, v) cbind(v, 2*v))
		from.dfs(calc)

前两行调用随 R 一起安装的 RHadoop 库。最后一行将结果打印到控制台。输出应如下所示：

	[1,]  1 2
	[2,]  2 4
	.
	.
	.
	[98,]  98 196
	[99,]  99 198
	[100,] 100 200

## 使用 Azure PowerShell 安装 R

在本部分中，我们使用 **<a href = "http://msdn.microsoft.com/zh-cn/library/dn858088.aspx" target="_blank">Add-AzureHDInsightScriptAction</a>** cmdlet 通过脚本操作来调用脚本，以自定义群集。在继续前，确保你已安装并配置 Azure PowerShell。有关配置工作站以运行 HDInsight Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

执行以下步骤：

1. 打开 Azure PowerShell 控制台，并声明以下变量：

		# PROVIDE VALUES FOR THESE VARIABLES
		$subscriptionName = "<SubscriptionName>"		# Name of the Azure subscription
		$clusterName = "<HDInsightClusterName>"			# HDInsight cluster name
		$storageAccountName = "<StorageAccountName>"	# Azure storage account that hosts the default container
		$storageAccountKey = "<StorageAccountKey>"      # Key for the storage account
		$containerName = $clusterName
		$location = "<MicrosoftDataCenter>"				# Location of the HDInsight cluster. It must be in the same data center as the storage account.
		$clusterNodes = <ClusterSizeInNumbers>			# The number of nodes in the HDInsight cluster.
		$version = "<HDInsightClusterVersion>"          # HDInsight version, for example "3.1"

2. 指定配置值（如群集中的节点）和要使用的默认存储。

		# SPECIFY THE CONFIGURATION OPTIONS
		Select-AzureSubscription $subscriptionName
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes
		$config.DefaultStorageAccount.StorageAccountName="$storageAccountName.blob.core.chinacloudapi.cn"
		$config.DefaultStorageAccount.StorageAccountKey=$storageAccountKey
		$config.DefaultStorageAccount.StorageContainerName=$containerName

3. 使用 **Add-AzureHDInsightScriptAction** cmdlet 来调用用于安装 R 的示例脚本，例如：

		# INVOKE THE SCRIPT USING THE SCRIPT ACTION
		$config = Add-AzureHDInsightScriptAction -Config $config -Name "Install R"  -ClusterRoleCollection HeadNode,DataNode -Uri https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1


	**Add-AzureHDInsightScriptAction** cmdlet 采用以下参数：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
<tr>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">参数</th>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:550px; padding-left:5px; padding-right:5px;">定义</th></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Config</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">脚本操作信息添加到的配置对象</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Name</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">脚本操作的名称</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">ClusterRoleCollection</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">指定在其上运行自定义脚本的节点。有效值是 **HeadNode**（在头节点上安装）或 **DataNode**（在所有数据节点上安装）。你可以使用任一值或两个值。</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Parameters</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">脚本需要的参数
</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Uri</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">指定执行的脚本的 URI</td></tr>
</table>

4. 最后，设置你通过自定义安装 R 的群集。

		# PROVISION A CLUSTER WITH R INSTALLED
		New-AzureHDInsightCluster -Config $config -Name $clusterName -Location $location -Version $version

出现提示时，请输入群集的凭据。创建群集可能需要几分钟时间。


## 使用 .NET SDK 安装 R

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 应用程序中使用 HDInsight 的操作。

执行以下过程以使用 SDK 设置 HDInsight 群集：

- [安装 HDInsight .NET SDK](#installSDK)
- [创建自签名证书](#createCert)
- [在 Visual Studio 中创建 .NET 应用程序](#createApp)
- [运行应用程序](#runApp)

以下各部分说明如何执行这些过程。

**安装 HDInsight .NET SDK**

可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装该 SDK 的最新发行版。下一过程中将显示说明。

**创建自签名证书**

创建自签名证书，将其安装到工作站上，然后将其上传到你的 Azure 订阅。有关说明，请参阅[创建自签名证书](/documentation/articles/hdinsight-administer-use-management-portal-v1#cert)。


**在 Visual Studio 中创建 .NET 应用程序**

1. 打开 Visual Studio 2013。

2. 在“文件”菜单中，单击“新建”，然后单击“项目”。

3. 在“新建项目”中，键入或选择以下值：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
<tr>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">属性</th>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">值</th></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">类别</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">模板/Visual C#/Windows</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">模板</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">控制台应用程序</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Name</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">CreateRCluster</td></tr>
</table>

4. 单击“确定”以创建该项目。

5. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。

6. 在控制台中运行下列命令以安装程序包。

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

	此命令将从当前 Visual Studio 项目添加 .NET 库以及对这些库的引用。

7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。

8. 将下列 **using** 语句添加到文件顶部：

		using System.Security.Cryptography.X509Certificates;
		using Microsoft.WindowsAzure.Management.HDInsight;
		using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;
		using Microsoft.WindowsAzure.Management.HDInsight.Framework.Logging;

9. 在 **Main()** 函数中，粘贴以下代码，并为变量提供值：

        var clusterName = args[0];

        // PROVIDE VALUES FOR THE VARIABLES
        string thumbprint = "<CertificateThumbprint>";  
        string subscriptionId = "<AzureSubscriptionID>";
        string location = "<MicrosoftDataCenterLocation>";
        string storageaccountname = "<AzureStorageAccountName>.blob.core.chinacloudapi.cn";
        string storageaccountkey = "<AzureStorageAccountKey>";
        string username = "<HDInsightUsername>";
        string password = "<HDInsightUserPassword>";
        int clustersize = <NumberOfNodesInTheCluster>;

        // PROVIDE THE CERTIFICATE THUMBPRINT TO RETRIEVE THE CERTIFICATE FROM THE CERTIFICATE STORE
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.Thumbprint == thumbprint);

        // CREATE AN HDINSIGHT CLIENT OBJECT
        HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionId), cert);
        var client = HDInsightClient.Connect(creds);
		client.IgnoreSslErrors = true;

        // PROVIDE THE CLUSTER INFORMATION
		var clusterInfo = new ClusterCreateParameters()
        {
            Name = clusterName,
            Location = location,
            DefaultStorageAccountName = storageaccountname,
            DefaultStorageAccountKey = storageaccountkey,
            DefaultStorageContainer = clusterName,
            UserName = username,
            Password = password,
            ClusterSizeInNodes = clustersize,
            Version = "3.1"
        };

10. 将以下代码追加到 **Main()** 函数，以使用 [ScriptAction](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.management.hdinsight.clusterprovisioning.data.scriptaction.aspx) 类通过调用自定义脚本来安装 R。

		// ADD THE SCRIPT ACTION TO INSTALL R

        clusterInfo.ConfigActions.Add(new ScriptAction(
            "Install R",
            new ClusterNodeType[] { ClusterNodeType.HeadNode, ClusterNodeType.DataNode },
            new Uri("https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1"), null
            ));

11. 最后，创建群集：

		client.CreateCluster(clusterInfo);

11. 将更改保存到应用程序并构建解决方案。

**运行应用程序**

打开 Azure PowerShell 控制台，导航到项目保存到的位置，导航到项目中的 \\bin\\debug 目录，然后运行以下命令：

	.\CreateRCluster <cluster-name>

提供群集名称，然后按 ENTER 以预配安装有 R 的群集。

## 另请参阅

- [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install)。使用群集自定义在 HDInsight Hadoop 群集上安装 Giraph。Giraph 可让你使用 Hadoop 执行图形处理，并可以在 Azure HDInsight 上使用。
- [在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters)：有关如何创建 HDInsight 群集的一般信息
- [使用脚本操作自定义 HDInsight 群集][hdinsight-cluster-customize]：有关如何使用脚本操作自定义 HDInsight 群集的一般信息
- [为 HDInsight 开发脚本操作脚本](/documentation/articles/hdinsight-hadoop-script-actions)
- [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install)：有关如何安装 Giraph 的脚本操作示例


[powershell-install-configure]: /documentation/articles/install-configure-powershell
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters
[hdinsight-cluster-customize]: /documentation/articles/hdinsight-hadoop-customize-cluster

<!---HONumber=79-->