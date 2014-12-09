<properties urlDisplayName="HDInsight Administration" pageTitle="在 HDInsight 中设置 Hadoop 群集 | Azure" metaKeywords="hdinsight, 管理, hdinsight 管理 azure" description="Learn how to provision clusters for Azure HDInsight using the management portal, PowerShell, or the command line." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="Provision Hadoop clusters in HDInsight" authors="jgao" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="09/25/2014" ms.author="jgao" />

# 使用自定义选项在 HDInsight 中设置 Hadoop 群集

在本文中，你将了解如何通过下述不同方法，使用自定义选项在 Azure HDInsight 上设置 Hadoop 群集 - 使用 Azure 管理门户、PowerShell、命令行工具或 HDInsight .NET SDK。本文将会介绍 Hadoop 群集的设置。有关如何设置 HBase 群集的说明，请参阅[在 HDInsight 中设置 HBase 群集][hdinsight-hbase-custom-provision]。请参阅 <a href="http://go.microsoft.com/fwlink/?LinkId=510237">Hadoop 与 HBase 之间的差别</a>，以了解为何要选择其中的某一种群集。

## 什么是 HDInsight 群集？

你是否曾经疑惑过，为何我们每次谈论 Hadoop 或大数据时都会提到群集？这是因为，Hadoop 能够针对分散在不同群集节点上的大型数据实现分布式处理。群集采用主/从体系结构，其中包括一个主节点（又称头节点或命名节点）和若干个从节点（又称数据节点）。有关详细信息，请参阅 [Apache Hadoop][apache-hadoop]。

![HDInsight 群集][img-hdi-cluster]

HDInsight 群集会抽象化 Hadoop 实现详细信息，因此你不必担心如何与不同的群集节点通信。当你设置 HDInsight 群集时，便设置了包含 Hadoop 和相关应用程序的 Azure 计算资源。有关详细信息，请参阅 [HDInsight 中的 Hadoop 简介][hadoop-hdinsight-intro]。要改动的数据存储在 Azure Blob 存储中，它在 HDInsight 的上下文中又称为 *Azure 存储 - Blob*（或 WASB）。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

本文提供有关群集的不同设置方法的说明。如果你正在寻求通过一种快捷的方法设置群集，请参阅 [Azure HDInsight 入门][hdinsight-get-started]。 

**先决条件：**

在开始阅读本文前，你必须具有：

- Azure 订阅。Azure 是基于订阅的平台。HDInsight PowerShell cmdlet 通过订阅执行任务。有关获取订阅的详细信息，请参阅[购买选项][azure-purchase-options]或[免费试用][azure-free-trial]。

<!-- [Member Offers][azure-member-offers],--> 



## 本文内容

* [配置选项](#configuration)
* [使用 Azure 管理门户](#portal)
* [使用 Azure PowerShell](#powershell)
* [使用跨平台命令行](#cli)
* [使用 HDInsight .NET SDK](#sdk)
* [后续步骤](#nextsteps)

## <a id="configuration"></a>配置选项

### 其他存储

在配置期间，你必须指定一个 Azure Blob 存储帐户和一个默认容器。该容器被集群用作默认存储位置。或者，你也可以指定也会与集群相关联的其他 Blob。

有关使用 Hive 的更多信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-use-blob-storage/)。

### 元存储

元存储包含有关 Hive 表、分区、架构、列等的信息。该信息被 Hive 用于定位数据在 HDFS 上的存储位置（或用于 HDInsight 的 WASB）。默认情况下，Hive 使用嵌入的数据库存储该信息。

设置 HDInsight 群集时，你可以指定一个 SQL Database 来包含 Hive 元存储。由此，当你删除一个群集时将保留元数据信息，因为它存储到外部的 SQL Database 中。

### 虚拟网络

[使用 Azure 虚拟网络](http://www.windowsazure.cn/manage/services/networking/)，你可以创建一个安全的永久性网络，包含你的解决方案所需的资源。通过虚拟网络，你可以：

* 在专用网络（仅包含云）中将云资源连接在一起

	![diagram of cloud-only configuration](./media/hdinsight-provision-clusters/cloud-only.png)

* 使用虚拟专用网络 (VPN) 将云资源连接到本地数据中心网络（站点到站点或点到站点）

	利用站点到站点配置，你可以使用硬件 VPN 或路由和远程访问服务将多个资源从数据中心连接到 Azure 虚拟网络

	![diagram of site-to-site configuration](.\media\hdinsight-provision-clusters\site-to-site.png)

	利用点到站点配置，你可以使用软件 VPN 将特定资源连接到 Azure 虚拟网络

	![diagram of point-to-site configuration](.\media\hdinsight-provision-clusters\point-to-site.png)

有关虚拟网络特性、优势和功能的更多信息，请参阅 [Azure 虚拟网络概述](http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)。

> [WACOM.NOTE] 你必须创建 Azure 虚拟网络才能设置一个 HDInsight 群集。有关详细信息，请参阅[虚拟网络配置任务](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx)。
> 
> Azure HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。


> [WACOM.NOTE] 强烈建议为一个群集指定一个子网。 

## <a id="portal"></a> 使用 Azure 管理门户

HDInsight 群集使用 Azure Blob 存储容器作为默认文件系统。创建 HDInsight 群集前，要先具有位于同一数据中心的 Azure 存储帐户。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。有关创建 Azure 存储帐户的详细信息，请参阅[如何创建存储帐户][azure-create-storageaccount]。


> [WACOM.NOTE] 目前，只有**亚洲东部**、**亚洲东南部**、**欧洲北部**、**欧洲西部**、**美国东部**、**美国西部**、**美国中北部**、**中国东部**、**中国北部**和**美国中南部**区域可以托管 HDInsight 群集。

**使用"自定义创建"选项创建 HDInsight 群集**

1. 登录到 [Azure 管理门户][azure-management-portal]。
2. 依次单击页面底部的 **+ 新建**、**数据服务**、**HDINSIGHT**和**自定义创建**。
3. 从**群集详细信息**页上，键入或选择以下值：

	![HDI.CustomCreateCluster][image-hdi-customcreatecluster]

    <table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>群集名称</td>
			<td><p>命名群集。 </p>
				<ul>
				<li>DNS 名称必须以字母数字开头和结尾，且可包含短划线。</li>
				<li>字段必须是介于 3 到 63 个字符之间的字符串。</li>
				</ul></td></tr>
		<tr><td>群集类型</td>
			<td>对于群集类型，请选择 <strong>Hadoop</strong>。</td></tr>
		<tr><td>HDInsight 版本</td>
			<td>选择版本。对于 Hadoop，默认值为 HDInsight 版本 3.1，该版本使用 Hadoop 2.4。</td></tr>
		</table>

	输入或选择表中所示的值，然后单击右箭头。

4. 在**配置群集**页面上，输入或选择以下值：

	<table border="1">
	<tr><th>名称</th><th>值</th></tr>
	<tr><td>数据节点</td><td>要部署的数据节点的数目。出于测试目的，请创建单节点群集。<br />群集大小限制因 Azure 订阅而异。若要提高限制的大小，请联系 Azure 计费支持。</td></tr>
	<tr><td>区域/虚拟网络</td><td><p>选择与上一个过程中创建的存储帐户相同的区域。HDInsight 要求存储帐户位于同一区域。在以后的配置中，你只能选择你在此处指定的区域中的存储帐户。</p><p>可以选择的区域为： <strong>China East</strong>, <strong>China North</strong>, <strong>East Asia</strong>, <strong>Southeast Asia</strong>, <strong>North Europe</strong>, <strong>West Europe</strong>, <strong>East US</strong>, <strong>West US</strong>, <strong>North Central US</strong>, <strong>South Central US</strong><br/>If you have created an Azure Virtual Network, you can select the network that the HDInsight cluster will be configured to use.</p><p>For more information on creating an Azure Virtual Network, see [Virtual Network configuration tasks](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx).</p></td></tr>
	</table>


5. 在**配置群集用户**页上提供以下值：

    ![HDI.CustomCreateCluster.ClusterUser][image-hdi-customcreatecluster-clusteruser]
	
    <table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>用户名</td>
			<td>指定 HDInsight 群集用户名。</td></tr>
		<tr><td>密码/确认密码</td>
			<td>指定 HDInsight 群集用户密码。</td></tr>
		<tr><td>输入配置单元/Oozie 元存储</td>
			<td>选中此复选框可将同一数据中心上的 SQL 数据库指定为群集，以用作 Hive/Oozie 元存储。如果你希望即使在删除群集后也保留有关 Hive/Oozie 作业的元数据，此选项将十分有用。</td></tr>
		<tr><td>元存储数据库</td>
			<td>指定将用作 Hive/OOzie 的元存储的 Azure SQL 数据库。此 SQL 数据库必须与 HDInsight 群集位于同一数据中心。列表框仅列出在<strong>群集详细信息</strong>页上指定的数据中心的 SQL 数据库。</td></tr>
		<tr><td>数据库用户</td>
			<td>指定将用于连接到数据库的 SQL 数据库用户。</td></tr>
		<tr><td>数据库用户密码</td>
			<td>指定 SQL 数据库用户密码。</td></tr>
	</table>

	>[WACOM.NOTE] 用于元存储的 Azure SQL Database 必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击**配置**、单击 **Windows Azure 服务**对应的**是**，然后单击**保存**。   

    单击右箭头。


6. 在**存储帐户**页上提供以下值：

    ![HDI.CustomCreateCluster.StorageAccount][image-hdi-customcreatecluster-storageaccount]

	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>存储帐户</td>
			<td>为 HDInsight 群集指定将用作默认文件系统的 Azure 存储帐户。可以选择以下三个选项之一：
			<ul>
				<li>使用现有存储</li>
				<li>创建新存储</li>
				<li>使用其他订阅中的存储</li>
			</ul>
			</td></tr>
		<tr><td>帐户名</td>
			<td><ul>
				<li>如果选择使用现有存储，请为<strong>帐户名</strong>选择现有的存储帐户。下拉列表中只列出了你选择在其中设置群集的同一数据中心内的存储帐户。</li>
				<li>如果选择<strong>创建新存储</strong>或<strong>使用其他订阅中的存储</strong>选项，则必须提供存储帐户名。</li>
			</ul></td></tr>
		<tr><td>帐户密钥</td>
			<td>如果选择<strong>使用其他订阅中的存储</strong>选项，请指定该存储帐户的帐户密钥。</td></tr>	
		<tr><td>默认容器</td>
			<td><p>指定存储帐户上用作 HDInsight 群集默认文件系统的默认容器。如果为<strong>存储帐户</strong>字段选择了<strong>使用现有存储</strong>并且该帐户不存在现有容器，则在默认情况下会创建与群集同名的容器。如果已存在与群集同名的容器，将在容器名称后附加一个序列号。例如，mycontainer1、mycontainer2，等等。但是，如果现有存储帐户的容器名称与你指定的群集名称不同，则你也可以使用该容器。</p>
            <p>如果选择创建新存储或使用其他 Azure 订阅中的存储，则必须指定默认容器名称</p>
        </td></tr>
		<tr><td>其他存储帐户</td>
			<td>HDInsight 支持多个存储帐户。一个群集可以使用的其他存储帐户数没有限制。但是，如果你使用管理门户创建群集，由于 UI 限制，你最多只能创建七个存储帐户。指定的每个其他存储帐户将在向导中添加一个额外的"存储帐户"页，以便你在此指定帐户信息。例如，在上面的屏幕截图中，选择了 1 个附加的存储帐户，因此第 5 页添加到了对话框。</td></tr>		
	</table>

	单击右箭头。

7. 在**存储帐户**页上，输入其他存储帐户的帐户信息：

	![HDI.CustomCreateCluster.AddOnStorage][image-hdi-customcreatecluster-addonstorage]

    同样，你可以使用相应的选项从现有帐户中做出选择、创建新存储，或者使用其他 Azure 订阅中的存储。提供值的过程类似于前面的步骤。

    单击复选标记以开始设置群集。完成设置后，状态列将显示**正在运行**。

	> [WACOM.NOTE] 一旦为 HDInsight 群集选择了 Azure 存储帐户，就不能再删除该帐户，也不能将它更改为另一帐户。

## <a id="powershell"></a> 使用 Azure PowerShell
Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。本部分提供有关如何设置 HDInsight 群集的说明。有关配置工作站运行 HDInsight Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。有关将 PowerShell 用于 HDInsight 的更多信息，请参见[使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]。有关 HDInsight PowerShell cmdlet 的列表，请参见 [HDInsight cmdlet 参考][hdinsight-powershell-reference]。

> [WACOM.NOTE] 虽然本部分中的脚本可用于配置 Azure 虚拟网络上的 HDInsight 群集，但它们不会创建 Azure 虚拟网络。有关创建 Azure 虚拟网络的信息，请参阅[虚拟网络配置任务](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx)。

使用 PowerShell 设置 HDInsight 群集的步骤如下：

- 创建 Azure 存储帐户
- 创建 Azure Blob 容器
- 创建 HDInsight 群集

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。存储帐户必须与 HDInsight 群集位于同一数据中心。


**创建 Azure 存储帐户**

- 在 Azure PowerShell 控制台窗口中运行以下命令：

		$storageAccountName = "<StorageAcccountName>"	# Provide a storage account name 
		$location = "<MicrosoftDataCenter>"				# For example, "China East"
		
		# Create an Azure storage account
		New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location
	
	如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下 PowerShell 命令来检索该信息：
	
		# List storage accounts for the current subscription
		Get-AzureStorageAccount

		# List the keys for a storage account
		Get-AzureStorageKey "<StorageAccountName>"
	
**创建 Azure Blob 存储容器**

- 在 Azure PowerShell 控制台窗口中运行以下命令：

		$storageAccountName = "<StorageAccountName>"	# Provide the storage account name 
		$storageAccountKey = "<StorageAccountKey>"		# Provide either primary or secondary key
		$containerName="<ContainerName>"				# Provide a container name

		# Create a storage context object
		$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName 
		                                       -StorageAccountKey $storageAccountKey  
		 
		# Create a Blob storage container
		New-AzureStorageContainer -Name $containerName -Context $destContext

准备好存储帐户和 blob 容器后，您就可以创建群集了。 

**设置 HDInsight 群集**

> [WACOM.NOTE] PowerShell cmdlet 是用于更改 HDInsight 群集中的配置变量的唯一推荐方法。如果对群集进行修补，可能会覆盖通过远程桌面连接到群集时对 Hadoop 配置文件所做的更改。如果对群集进行修补，将保留通过 PowerShell 设置的配置值。 

- 在 Azure PowerShell 控制台窗口中运行以下命令：		

		$subscriptionName = "<SubscriptionName>"		# Name of the Azure subscription.
		$storageAccountName = "<StorageAccountName>"	# Azure storage account that hosts the default container. 
		$containerName = "<ContainerName>"				# Azure Blob container that is used as the default file system for the HDInsight cluster.

		$clusterName = "<HDInsightClusterName>"			# The name you will name your HDInsight cluster.
		$location = "<MicrosoftDataCenter>"				# The location of the HDInsight cluster. It must in the same data center as the storage account.
		$clusterNodes = <ClusterSizeInNodes>			# The number of nodes in the HDInsight cluster.

		# Get the storage primary key based on the account name
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }

		# Create a new HDInsight cluster
		New-AzureHDInsightCluster -Name $clusterName -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes

	出现提示时，请输入群集的凭据。设置群集可能需要几分钟时间。

	![HDI.CLI.Provision][image-hdi-ps-provision]

 

**使用自定义配置选项设置 HDInsight 群集**

设置群集时，你可以使用其他配置选项，例如，连接到多个 Azure Blob 存储，或者为 Hive 和 Oozie 元存储使用虚拟网络或 Azure SQL 数据库。这样，你便可以将数据和元数据的生存期与群集的生存期分开。

> [WACOM.NOTE] PowerShell cmdlet 是用于更改 HDInsight 群集中的配置变量的唯一推荐方法。如果对群集进行修补，可能会覆盖通过远程桌面连接到群集时对 Hadoop 配置文件所做的更改。如果对群集进行修补，将保留通过 PowerShell 设置的配置值。 

- 从 Windows PowerShell 窗口运行以下命令：

		$subscriptionName = "<SubscriptionName>"
		$clusterName = "<ClusterName>"
		$location = "<MicrosoftDataCenter>"
		$clusterNodes = <ClusterSizeInNodes>
		
		$storageAccountName_Default = "<DefaultFileSystemStorageAccountName>"
		$containerName_Default = "<DefaultFileSystemContainerName>"
		
		$storageAccountName_Add1 = "<AdditionalStorageAccountName>"
		
		$hiveSQLDatabaseServerName = "<SQLDatabaseServerNameForHiveMetastore>"
		$hiveSQLDatabaseName = "<SQLDatabaseDatabaseNameForHiveMetastore>"
		$oozieSQLDatabaseServerName = "<SQLDatabaseServerNameForOozieMetastore>"
		$oozieSQLDatabaseName = "<SQLDatabaseDatabaseNameForOozieMetastore>"
		
		# Get the virtual network ID and subnet name
		$vnetID = "<AzureVirtualNetworkID>"
		$subNetName = "<AzureVirtualNetworkSubNetName>" 

		# Get the storage account keys
		Select-AzureSubscription $subscriptionName
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$storageAccountKey_Add1 = Get-AzureStorageKey $storageAccountName_Add1 | %{ $_.Primary }
		
		$oozieCreds = Get-Credential -Message "Oozie metastore"
		$hiveCreds = Get-Credential -Message "Hive metastore"
		
		# Create a Blob storage container
		$dest1Context = New-AzureStorageContext -StorageAccountName $storageAccountName_Default -StorageAccountKey $storageAccountKey_Default  
		New-AzureStorageContainer -Name $containerName_Default -Context $dest1Context

		# Create a new HDInsight cluster
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
		    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
		    Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Add1.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Add1 |
		    Add-AzureHDInsightMetastore -SqlAzureServerName "$hiveSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $hiveSQLDatabaseName -Credential $hiveCreds -MetastoreType HiveMetastore |
		    Add-AzureHDInsightMetastore -SqlAzureServerName "$oozieSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $oozieSQLDatabaseName -Credential $oozieCreds -MetastoreType OozieMetastore |
		        New-AzureHDInsightCluster -Name $clusterName -Location $location -VirtualNetworkId $vnetID -SubnetName $subNetName

	>[WACOM.NOTE] 用于元存储的 Azure SQL 数据库必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击**配置**、单击 **Windows Azure 服务**对应的**是**，然后单击**保存**。

**列出 HDInsight 群集**

- 在 Azure PowerShell 控制台窗口中运行以下命令，以列出 HDInsight 群集并验证是否已成功设置群集：

		Get-AzureHDInsightCluster -Name <ClusterName>


##<a id="cli"></a> 使用跨平台命令行

> [WACOM.NOTE] 自 2014 年 8 月 29 日起，跨平台命令行界面无法用于将群集与 Azure 虚拟网络相关联。

设置 HDInsight 群集的另一方法是使用跨平台命令行界面。该命令行工具是在 Node.js 中实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。该命令行工具是开源的。在 GitHub 中管理源代码（网址为 <a href= "https://github.com/Azure/azure-sdk-tools-xplat">https://github.com/Azure/azure-sdk-tools-xplat</a>）。有关如何使用命令行界面的一般指南，请参阅[如何使用针对 Mac 和 Linux 的 Azure 命令行工具][azure-command-line-tools]。有关完整的参考文档，请参阅[针对 Mac 和 Linux 的 Azure 命令行工具][azure-command-line-tool]。本文只涉及从 Windows 使用命令行界面。

使用跨平台命令行设置 HDInsight 群集的步骤如下：

- 安装跨平台命令行
- 下载并导入 Azure 帐户发布设置
- 创建 Azure 存储帐户
- 设置群集

可以使用 *Node.js 包管理器 (NPM)* 或 Windows 安装程序安装该命令行界面。Microsoft 建议你只使用这两个选项中的一个来进行安装。

**使用 NPM 安装该命令行界面**

1.	浏览到 **www.nodejs.org**。
2.	单击**安装**，然后使用默认设置按说明操作。
3.	从工作站打开**命令提示符**（或 *Azure 命令提示符*，或 *VS2012 的开发人员命令提示符*）。
4.	在命令提示符窗口中运行以下命令。

		npm install -g azure-cli

	> [WACOM.NOTE] 如果收到"未找到 NPM 命令"的错误消息，请验证以下路径位于 PATH 环境变量中：<i>C:\Program Files (x86)\nodejs;C:\Users\[username]\AppData\Roaming\npm</i> 或 <i>C:\Program Files\nodejs;C:\Users\[username]\AppData\Roaming\npm</i>
	
5.	运行以下命令以验证安装：

		azure hdinsight -h

	可以在不同级别使用 *-h* 开关以显示帮助信息。例如：
		
		azure -h
		azure hdinsight -h
		azure hdinsight cluster -h
		azure hdinsight cluster create -h

**使用 Windows 安装程序安装该命令行界面**

1.	浏览到 **http://www.windowsazure.cn/downloads/#cmd-line-tools**。
2.	Scroll down to the **Command line tools** section, and then click **Cross-platform Command Line Interface** and follow the Web Platform Installer wizard.

**下载和导入发布设置**

在使用命令行界面前，你必须配置工作站和 Azure 之间的连接。命令行界面使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 的发布设置文件中获取此信息。稍后可以导入发布设置文件作为永久性本地配置设置，命令行界面会将此设置用于后续操作。您只需导入您的发布设置一次。

> [WACOM.NOTE]发布设置文件包含敏感信息。Microsoft 建议你删除该文件或采取其他措施来加密包含该文件的用户文件夹。在 Windows 上，修改文件夹属性或使用 BitLocker。


1.	打开**命令提示符**。
2.	运行以下命令来下载发布设置文件。

		azure account download
 
	![HDI.CLIAccountDownloadImport][image-cli-account-download-import]

	该命令将启动要从中下载发布设置文件的网页。 

3.	出现保存文件的提示时，请单击**保存**并提供文件的保存位置。
5.	从命令提示符窗口，运行以下命令以导入发布设置文件：

		azure account import <file>

	在上一屏幕快照中，发布设置文件已保存到工作站上的 C:\HDInsight 文件夹。

**创建 Azure 存储帐户**

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。存储帐户位于同一数据中心。

- 在命令提示符窗口中运行以下命令，以创建 Azure 存储帐户：

		azure storage account create [options] <StorageAccountName>

	出现指定位置的提示时，请选择 HDINsight 群集可以设置到的位置。该存储位置必须与 HDInsight 群集所在的位置相同。目前，只有**中国东部**、**中国北部**、**亚洲东部**、**亚洲东南部**、**欧洲北部**、**欧洲西部**、**美国东部**、**美国西部**、**美国中北部**和**美国中南部**区域可以托管 HDInsight 群集。  

有关使用 Azure 管理门户创建 Azure 存储帐户的信息，请参阅[如何创建存储帐户][azure-create-storageaccount]。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下命令来检索该信息：

	-- lists storage accounts
	azure storage account list

	-- Shows information for a storage account
	azure storage account show <StorageAccountName>

	-- Lists the keys for a storage account
	azure storage account keys list <StorageAccountName>

For details on getting the information using the management portal, see the *How to: View, copy and regenerate storage access keys* section of [How to Manage Storage Accounts][azure-manage-storageaccount].

HDInsight 群集还需要在存储帐户中提供一个容器。如果你提供的存储帐户尚不包含容器，*azure hdinsight cluster create* 将提示你输入容器名称，然后会创建该容器。但是，如果你想要预先创建容器，可以使用以下命令：

	azure storage container create --account-name <StorageAccountName> --account-key <StorageAccountKey> [ContainerName]
		
准备好存储帐户和 blob 容器后，你就可以创建群集了。

**设置 HDInsight 群集**

- 从命令提示符窗口，运行以下命令：

		azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey <storageAccountKey> --storageContainer <SorageContainerName> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

	![HDI.CLIClusterCreation][image-cli-clustercreation]


**使用配置文件设置 HDInsight 群集**

通常，先设置 HDInsight 群集、运行作业，然后删除该群集以降低成本。在命令行界面上，您可以选择将配置保存到文件，以便在每次设置群集时重用这些配置。

- 在命令提示符窗口中运行以下命令：
 
		
		#Create the config file
		azure hdinsight cluster config create <file> 
		 
		#Add commands to create a basic cluster
		azure hdinsight cluster config set <file> --clusterName <ClusterName> --nodes <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --username "<Username>" --clusterPassword "<UserPassword>"
		 
		#If requred, include commands to use additional blob storage with the cluster
		azure hdinsight cluster config storage add <file> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn"
		       --storageAccountKey "<StorageAccountKey>"
		 
		#If required, include commands to use a SQL database as a Hive metastore
		azure hdinsight cluster config metastore set <file> --type "hive" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
		       --database "<HiveDatabaseName>" --user "<Username>" --metastorePassword "<UserPassword>"
		
		#If required, include commands to use a SQL database as an Oozie metastore 
		azure hdinsight cluster config metastore set <file> --type "oozie" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
		       --database "<OozieDatabaseName>" --user "<SQLUsername>" --metastorePassword "<SQLPassword>"
		 
		#Run this command to create a cluster using the config file		
		azure hdinsight cluster create --config <file>
		 
	>[WACOM.NOTE] 用于元存储的 Azure SQL Database 必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击**配置**、单击 **Windows Azure 服务**对应的**是**，然后单击**保存**。


	![HDI.CLIClusterCreationConfig][image-cli-clustercreation-config]


**列出和显示群集详细信息**

- 使用以下命令来列出和显示群集详细信息：

		azure hdinsight cluster list
		azure hdinsight cluster show <ClusterName>
	
	![HDI.CLIListCluster][image-cli-clusterlisting]


**删除群集**

- 使用以下命令来删除群集：

		azure hdinsight cluster delete <ClusterName>



##<a id="sdk"></a>使用 HDInsight .NET SDK
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 应用程序中使用 HDInsight 的操作。 

使用 SDK 设置 HDInsight 群集时必须执行以下过程：

- 安装 HDInsight .NET SDK
- 创建自签名证书
- 创建控制台应用程序
- 运行应用程序


**安装 HDInsight .NET SDK**

可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装该 SDK 的最新发行版。这些说明将显示在下一步中。

**创建自签名证书**

创建自签名证书，将其安装到工作站上，然后将其上传到你的 Azure 订阅。有关说明，请参阅[创建自签名证书](http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-administer-use-management-portal/#cert)。 


**创建 Visual Studio 控制台应用程序**

1. 打开 Visual Studio 2013。

2. 在"文件"菜单上单击**新建**，然后单击**项目**。

3. 从"新建项目"中，键入或选择以下值：

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
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">CreateHDICluster</td></tr>
	</table>

4. 单击**确定**创建项目。

5. 从**工具**菜单，单击 **NuGet 包管理器**，然后单击**程序包管理器控制台**。

6. 在控制台中运行下列命令以安装程序包。

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

	此命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7. 从解决方案资源管理器中，双击 **Program.cs** 以将其打开。

8. 将下列 using 语句添加到文件顶部：

		using System.Security.Cryptography.X509Certificates;
		using Microsoft.WindowsAzure.Management.HDInsight;
		using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;

	
9. 在 Main() 函数中，复制并粘贴以下代码：
		
        string certfriendlyname = "<CertificateFriendlyName>";     // Friendly name for the certificate your created earlier  
        string subscriptionid = "<AzureSubscriptionID>";
        string clustername = "<HDInsightClusterName>";
        string location = "<MicrosoftDataCenter>";
        string storageaccountname = "<AzureStorageAccountName>.blob.core.chinacloudapi.cn";
        string storageaccountkey = "<AzureStorageAccountKey>";
        string containername = "<HDInsightDefaultContainerName>";
        string username = "<HDInsightUsername>";
        string password = "<HDInsightUserPassword>";
        int clustersize = <NumberOfNodesInTheCluster>;

        // Get the certificate object from certificate store using the friendly name to identify it
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certfriendlyname);

        // Create the storage account if it doesn't exist.

        // Create the container if it doesn't exist.

		// Create an HDInsightClient object
        HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionid), cert);
        var client = HDInsightClient.Connect(creds);

		// Supply th cluster information
        ClusterCreateParameters clusterInfo = new ClusterCreateParameters()
        {
            Name = clustername,
            Location = location,
            DefaultStorageAccountName = storageaccountname,
            DefaultStorageAccountKey = storageaccountkey,
            DefaultStorageContainer = containername,
            UserName = username,
            Password = password,
            ClusterSizeInNodes = clustersize
        };

		// Create the cluster
        Console.WriteLine("Creating the HDInsight cluster ...");

        ClusterDetails cluster = client.CreateCluster(clusterInfo);

        Console.WriteLine("Created cluster: {0}.", cluster.ConnectionUrl);
        Console.WriteLine("Press ENTER to continue.");
        Console.ReadKey();

10. 替换 main() 函数开头的变量。 

**运行应用程序**

在 Visual Studio 中打开应用程序时，按 **F5** 以运行应用程序。控制台窗口应打开并显示应用程序的状态。创建 HDInsight 群集可能需要几分钟时间。



## <a id="nextsteps"></a> 后续步骤
在本文中，您了解了几种设置 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门][hdinsight-get-started]
* [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx
[hdinsight-hbase-custom-provision]: http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-hbase-get-started/

[hdinsight-get-started]: ../hdinsight-get-started/
[hdinsight-storage]: ../hdinsight-use-blob-storage/
[hdinsight-admin-powershell]: ../hdinsight-administer-use-powershell/
[hadoop-hdinsight-intro]: ../hdinsight-hadoop-introduction/
[hdinsight-submit-jobs]: ../hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-powershell-reference]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn479228.aspx

[azure-create-storageaccount]: ../storage-create-storage-account/ 
[azure-management-portal]: https://manage.windowsazure.cn/

[azure-command-line-tools]: ../xplat-cli/
[azure-command-line-tool]: ../command-line-tools/
[azure-manage-storageaccount]: ../storage-manage-storage-account/

[apache-hadoop]: http://go.microsoft.com/fwlink/?LinkId=510084
[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/

<!--
[azure-member-offers]: http://azure.microsoft.com/zh-cn/pricing/member-offers/
-->

[azure-free-trial]: http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial-full/
[hdi-remote]: http://www.windowsazure.cn/zh-cn/documentation/articles/hdinsight-administer-use-management-portal/#rdp


[Powershell-install-configure]: ../install-configure-powershell/

[image-hdi-customcreatecluster]: ./media/hdinsight-get-started/HDI.CustomCreateCluster.png
[image-hdi-customcreatecluster-clusteruser]: ./media/hdinsight-get-started/HDI.CustomCreateCluster.ClusterUser.png
[image-hdi-customcreatecluster-storageaccount]: ./media/hdinsight-get-started/HDI.CustomCreateCluster.StorageAccount.png
[image-hdi-customcreatecluster-addonstorage]: ./media/hdinsight-get-started/HDI.CustomCreateCluster.AddOnStorage.png

[image-customprovision-page1]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page1.png 
[image-customprovision-page2]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page2.png 
[image-customprovision-page3]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page3.png 
[image-customprovision-page4]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page4.png 

[image-cli-account-download-import]: ./media/hdinsight-provision-clusters/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-provision-clusters/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-provision-clusters/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-provision-clusters/HDI.CLIListClusters.png "List and show clusters"

[image-hdi-ps-provision]: ./media/hdinsight-provision-clusters/HDI.ps.provision.png

[img-hdi-cluster]: ./media/hdinsight-provision-clusters/HDI.Cluster.png


