<properties 
	pageTitle="使用脚本操作在 Hadoop 群集上安装 Solr | Azure" 
	description="了解如何使用 Solr 自定义 HDInsight 群集。你将使用脚本操作配置选项来通过脚本安装 Solr。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="02/12/2016"
	wacn.date="03/28/2016"/>

# 在 HDInsight Hadoop 群集上安装并使用 Solr


了解如何使用 Solr 通过脚本操作来自定义基于 Windows 的 HDInsight 群集，以及如何使用 R 来搜索数据。
你可以使用*脚本操作*，在 Azure HDInsight 的任何一种群集（Hadoop、Storm、HBase）上安装 Solr。用于在 HDInsight 群集上安装 Solr 的示例脚本可通过 [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1) 上的只读 Azure 存储 Blob 获得。

示例脚本仅适用于 HDInsight 群集版本 3.1。有关 HDInsight 群集版本的详细信息，请参阅 [HDInsight 群集版本](/documentation/articles/hdinsight-component-versioning-v1/)。

本主题中使用的示例脚本将使用特定配置创建基于 Windows 的 Solr 群集。如果你要使用不同集合、分片、架构、副本等配置 Solr 群集，则必须相应地修改脚本和 Solr 二进制文件。

**相关文章** 

- [在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)：有关如何创建 HDInsight 群集的一般信息 
- [使用脚本操作自定义 HDInsight 群集][hdinsight-cluster-customize]：有关如何使用脚本操作自定义 HDInsight 群集的一般信息 
- [针对 HDInsight 开发脚本操作脚本](/documentation/articles/hdinsight-hadoop-script-actions/)
<a name="whatis"></a> 
## 什么是 Solr？
<a href="http://lucene.apache.org/solr/features.html" target="_blank">Apache Solr</a> 是一种企业搜索平台，用于对数据实现功能强大的全文搜索。虽然 Hadoop 可用于存储和管理大量数据，但是，Apache Solr 提供了快速检索数据的搜索功能。

## 如何安装 Solr？

用于在 HDInsight 群集上安装 Solr 的示例脚本可通过 [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1) 上的只读 Azure 存储 Blob 获得。本部分说明了如何在通过 Azure 经典管理门户预配群集时使用示例脚本。 
* [在 HDInsight 群集上安装 Solr](/documentation/articles/hdinsight-hadoop-solr-install-v1/)

1. 根据[在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/#portal)中的说明，使用“自定义创建”选项开始创建群集。 
2. 在向导的“脚本操作”页上，单击“添加脚本操作”，以提供有关脚本操作的详细信息，如下所示：

	![使用脚本操作自定义群集](./media/hdinsight-hadoop-solr-install-v1/hdi-script-action-solr.png "使用脚本操作自定义群集")
	
	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>Name</td>
			<td>指定脚本操作的名称。例如 <b>Install Solr</b>。</td></tr>
		<tr><td>脚本 URI</td>
			<td>指定调用以自定义群集的脚本的统一资源标识符 (URI)。例如 <i>https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1</i></td></tr>
		<tr><td>节点类型</td>
			<td>指定在其上运行自定义脚本的节点。你可以选择“所有节点”、“仅限头节点”或“仅限从节点”<b></b><b></b><b></b>。
		<tr><td>Parameters</td>
			<td>根据脚本的需要，指定参数。用于安装 Solr 的脚本不需要任何参数，因此，你可以将此项保留为空。</td></tr>
	</table>

	你可以添加多个脚本操作，以在群集上安装多个组件。在添加了脚本后，单击复选标记以开始创建群集。

你还可以通过 Azure PowerShell 或 the HDInsight .NET SDK 使用脚本在 HDInsight 上安装 Solr。本主题后面将提供有关这些过程的说明。

<a name="usesolr"></a> 
## 使用 Solr

你必须从使用一些数据文件为 Solr 编制索引开始。然后，可以使用 Solr 对索引数据运行搜索查询。执行以下步骤，以在 HDInsight 群集中使用 Solr：

1. **使用远程桌面协议 (RDP) 远程连接到安装有 Solr 的 HDInsight 群集**。在 Azure 经典管理门户中，对你创建的安装有 Solr 的群集启用远程桌面，然后远程连接到该群集。有关说明，请参阅<a href="/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp" target="_blank">使用 RDP 连接到 HDInsight 群集</a>。

2. **通过上载数据文件为 Solr 编制索引**。在为 Solr 编制索引时，你应将可能需要搜索的文档放置在其中。若要为 Solr 编制索引，请使用 RDP 远程连接到群集，导航到桌面，打开 Hadoop 命令行，然后导航到 **C:\\apps\\dist\\solr-4.7.2\\example\\exampledocs**。运行以下命令：
	
		java -jar post.jar solr.xml monitor.xml

	你将会在控制台上看到以下输出：

		POSTing file solr.xml
		POSTing file monitor.xml
		2 files indexed.
		COMMITting Solr index changes to http://localhost:8983/solr/update..
		Time spent: 0:00:01.624

	post.jar 实用程序通过以下两个示例文档为 Solr 编制索引：**solr.xml** 和 **monitor.xml**。post.jar 实用程序和示例文档是随 Solr 安装一起提供的。

3. **使用 Solr 仪表板在索引文档中搜索**。在连接到 HDInsight 群集的 RDP 会话中，打开 Internet Explorer，然后启动位于 **http://headnodehost:8983/solr/#/** 的 Solr 仪表板。在左窗格的**“核心选择器”**下拉列表中，选择**“collection1”**，然后在其中单击**“查询”**。作为示例，若要在 Solr 中选择并返回所有文档，请提供以下值：
	1. 在 **q** 文本框中，输入 ***:***。此时将返回所有已在 Solr 中编制索引的文档。如果你要在文档中搜索特定字符串，则可以在此处输入该字符串。
	2. 在 **wt** 文本框中，选择输出格式。默认值为 **json**。单击**“执行查询”**。

		![使用脚本操作自定义群集](./media/hdinsight-hadoop-solr-install-v1/hdi-solr-dashboard-query.png "在 Solr 仪表板上运行查询")
	
	输出返回两个我们用于为 Solr 编制索引的文档。输出如下所示：

			"response": {
			    "numFound": 2,
			    "start": 0,
			    "maxScore": 1,
			    "docs": [
			      {
			        "id": "SOLR1000",
			        "name": "Solr, the Enterprise Search Server",
			        "manu": "Apache Software Foundation",
			        "cat": [
			          "software",
			          "search"
			        ],
			        "features": [
			          "Advanced Full-Text Search Capabilities using Lucene",
			          "Optimized for High Volume Web Traffic",
			          "Standards Based Open Interfaces - XML and HTTP",
			          "Comprehensive HTML Administration Interfaces",
			          "Scalability - Efficient Replication to other Solr Search Servers",
			          "Flexible and Adaptable with XML configuration and Schema",
			          "Good unicode support: héllo (hello with an accent over the e)"
			        ],
			        "price": 0,
			        "price_c": "0,USD",
			        "popularity": 10,
			        "inStock": true,
			        "incubationdate_dt": "2006-01-17T00:00:00Z",
			        "_version_": 1486960636996878300
			      },
			      {
			        "id": "3007WFP",
			        "name": "Dell Widescreen UltraSharp 3007WFP",
			        "manu": "Dell, Inc.",
			        "manu_id_s": "dell",
			        "cat": [
			          "electronics and computer1"
			        ],
			        "features": [
			          "30" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
			        ],
			        "includes": "USB cable",
			        "weight": 401.6,
			        "price": 2199,
			        "price_c": "2199,USD",
			        "popularity": 6,
			        "inStock": true,
			        "store": "43.17614,-90.57341",
			        "_version_": 1486960637584081000
			      }
			    ]
			  }
   

4. **建议：将索引数据从 Solr 备份到与 HDInsight 群集关联的 Azure Blob 存储**。作为一种很好的做法，你应该将索引数据从 Solr 群集节点备份到 Azure Blob 存储上。执行以下步骤来完成此操作：

	1. 在 RDP 会话中，打开 Internet Explorer，然后指向以下 URL：

			http://localhost:8983/solr/replication?command=backup

		你应该看到如下所示的响应：

			<?xml version="1.0" encoding="UTF-8"?>
			<response>
			  <lst name="responseHeader">
			    <int name="status">0</int>
			    <int name="QTime">9</int>
			  </lst>
			  <str name="status">OK</str>
			</response>

	2. 在远程会话中，导航到 {SOLR\_HOME}{Collection}\\data。对于通过示例脚本创建的群集，该目录应该是 **C:\\apps\\dist\\solr-4.7.2\\example\\solr\\collection1\\data**。在此位置，你应该会看到使用类似于 **snapshot.*timestamp*** 的名称创建的快照文件夹。
	
	3. 压缩快照文件夹，并将其上载到 Azure Blob 存储。从 Hadoop 命令行，通过使用以下命令导航到快照文件夹所在的位置：

			  hadoop fs -CopyFromLocal snapshot._timestamp_.zip /example/data

		此命令会将快照复制到与群集关联的默认存储帐户中容器下方的 /example/data/。

<a name="usingPS"></a> 
## 使用 Azure PowerShell 安装 Solr

在本部分中，我们使用 **<a href = "http://msdn.microsoft.com/zh-cn/library/dn858088.aspx" target="_blank">Add-AzureHDInsightScriptAction</a>** cmdlet 通过脚本操作来调用脚本，以自定义群集。在继续前，确保你已安装并配置 Azure PowerShell。有关配置工作站以运行 HDInsight Windows Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

执行以下步骤：

1. 打开 Azure PowerShell 窗口，并声明以下变量：

		# PROVIDE VALUES FOR THESE VARIABLES
		$subscriptionName = "<SubscriptionName>"		# Name of the Azure subscription
		$clusterName = "<HDInsightClusterName>"			# HDInsight cluster name
		$storageAccountName = "<StorageAccountName>"	# Azure Storage account that hosts the default container
		$storageAccountKey = "<StorageAccountKey>"      # Key for the Storage account
		$containerName = $clusterName
		$location = "<MicrosoftDataCenter>"				# Location of the HDInsight cluster. It must be in the same data center as the Storage account.
		$clusterNodes = <ClusterSizeInNumbers>			# Number of nodes in the HDInsight cluster
		$version = "<HDInsightClusterVersion>"          # For example, "3.1"
	
2. 指定配置值，例如群集中的节点，以及要使用的默认存储。

		# Specify the configuration options
		Select-AzureSubscription $subscriptionName
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes
		$config.DefaultStorageAccount.StorageAccountName="$storageAccountName.blob.core.chinacloudapi.cn"
		$config.DefaultStorageAccount.StorageAccountKey=$storageAccountKey
		$config.DefaultStorageAccount.StorageContainerName=$containerName
	
3. 使用 **Add-AzureHDInsightScriptAction** cmdlet 将脚本操作添加到群集配置中。稍后，在创建群集时，将执行脚本操作。

		# Add the script action to the cluster configuration
		$config = Add-AzureHDInsightScriptAction -Config $config -Name "Install Solr" -ClusterRoleCollection HeadNode,DataNode -Uri https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1

	**Add-AzureHDInsightScriptAction** cmdlet 采用以下参数：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
<tr>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">参数</th>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:550px; padding-left:5px; padding-right:5px;">定义</th></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Config</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">脚本操作信息添加到的配置对象。</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Name</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">脚本操作的名称。</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">ClusterRoleCollection</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">在其上运行自定义脚本的节点。有效值是 HeadNode（在头节点上安装）或 DataNode（在所有数据节点上安装）。你可以使用任一值或两个值。</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Uri</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">执行的脚本的 URI。</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Parameters</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">脚本所需的参数。本主题中使用的示例脚本不需要任何参数，因此，你在上述代码段中看不到此参数。
</td></tr>
</table>
	
4. 最后，开始设置安装有 Solr 的自定义群集。
	
		# Start provisioning a cluster with Solr installed
		New-AzureHDInsightCluster -Config $config -Name $clusterName -Location $location -Version $version 

出现提示时，请输入群集的凭据。创建群集可能需要几分钟时间。

<a name="usingSDK"></a> 
## 使用 .NET SDK 安装 Solr

HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 应用程序使用 HDInsight 的操作。本部分提供有关如何使用 SDK 中的脚本操作设置安装有 Solr 的群集的说明。必须执行以下过程：

- 安装 HDInsight .NET SDK
- 创建自签名证书
- 创建控制台应用程序
- 运行应用程序


**安装 HDInsight .NET SDK**

你可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装该 SDK 的最新发行版。下一过程中将显示说明。

**创建自签名证书**

创建自签名证书，将其安装到工作站上，然后将其上传到你的 Azure 订阅。有关说明，请参阅[创建自签名证书](/documentation/articles/hdinsight-administer-use-management-portal-v1/#cert)。


**创建 Visual Studio 应用程序**

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
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">CreateSolrCluster</td></tr>
</table>

4. 单击“确定”以创建该项目。

5. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。

6. 在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

	此命令将从当前 Visual Studio 项目添加 .NET 库以及对这些库的引用。

	
7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。

8. 将下列 using 语句添加到文件顶部：

		using System.Security.Cryptography.X509Certificates;
		using Microsoft.WindowsAzure.Management.HDInsight;
		using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;
		using Microsoft.WindowsAzure.Management.HDInsight.Framework.Logging;
	
9. 在 Main() 函数中，复制并粘贴以下代码，然后提供变量值：
		
        var clusterName = args[0];

        // Provide values for the variables
        string thumbprint = "<CertificateThumbprint>";  
        string subscriptionId = "<AzureSubscriptionID>";
        string location = "<MicrosoftDataCenterLocation>";
        string storageaccountname = "<AzureStorageAccountName>.blob.core.chinacloudapi.cn";
        string storageaccountkey = "<AzureStorageAccountKey>";
        string username = "<HDInsightUsername>";
        string password = "<HDInsightUserPassword>";
        int clustersize = <NumberOfNodesInTheCluster>;

        // Provide the certificate thumbprint to retrieve the certificate from the certificate store 
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.Thumbprint == thumbprint);

        // Create an HDInsight client object
        HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionId), cert);
        var client = HDInsightClient.Connect(creds);
		client.IgnoreSslErrors = true;
        
        // Provide the cluster information
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

10. 将以下代码追加到 Main() 函数，以使用 [ScriptAction](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.management.hdinsight.clusterprovisioning.data.scriptaction.aspx) 类调用自定义脚本来安装 Solr。

		// Add the script action to install Solr
        clusterInfo.ConfigActions.Add(new ScriptAction(
          "Install Solr", // Name of the config action
          new ClusterNodeType[] { ClusterNodeType.HeadNode, ClusterNodeType.DataNode }, // List of nodes to install Solr on
          new Uri("https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1"), // Location of the script to install Solr
		  null //Because the script used does not require any parameters
        ));

11. 最后，创建群集。

		client.CreateCluster(clusterInfo);

11. 将更改保存到应用程序并构建解决方案。

**运行应用程序**

打开 Windows PowerShell 或 Azure PowerShell 控制台，导航到保存 Visual Studio 项目的位置，再导航到项目中的 \\bin\\debug 目录，然后运行以下命令：

	.\CreateSolrCluster <cluster-name>

提供群集名称，并按 ENTER 预配安装有 Solr 的群集。


## 另请参阅

- [在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)：有关如何创建 HDInsight 群集的一般信息
- [使用脚本操作自定义 HDInsight 群集][hdinsight-cluster-customize]：有关如何使用脚本操作自定义 HDInsight 群集的一般信息
- [为 HDInsight 开发脚本操作脚本](/documentation/articles/hdinsight-hadoop-script-actions/)
- [在 HDInsight 群集上安装 R][hdinsight-install-r]：有关如何安装 R 的脚本操作示例
- [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-v1/)：有关如何安装 Giraph 的脚本操作示例




[powershell-install-configure]: /documentation/articles/powershell-install-configure/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-cluster-customize]: /documentation/articles/hdinsight-hadoop-customize-cluster-v1/

<!---HONumber=79-->