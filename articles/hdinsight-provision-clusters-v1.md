<properties 
   pageTitle="在 HDInsight 中预配自定义 Hadoop 群集 | Azure" 
   description="了解如何通过使用 Azure 经典管理门户、Azure PowerShell、命令行或 .NET SDK 预配自定义的 Azure HDInsight 群集。" 
   services="hdinsight" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="04/28/2016"
	wacn.date="06/06/2016"/>

#在 HDInsight 中设置 Hadoop 群集

了解如何规划 HDInsight 群集的预配。

> [AZURE.IMPORTANT] 目前，Azure 中国区的 HDInsight 只能通过 Azure 服务管理器 (ASM) 进行管理。适用于 HDInsight 的 Azure Resource Manager (ARM) 模型尚不可用。

**先决条件：**

在开始按照本文中的说明操作之前，你必须具有以下内容：

- Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。


## 基本配置选项


- **群集名称**

	群集名称用于标识群集。群集名称必须遵循以下准则：
	
	- 字段必须是包含 3 到 63 个字符的字符串
	- 字段只能包含字母、数字和连字符。

- **订阅名称**

	一个 HDInsight 群集与一个 Azure 订阅绑定。
 
- **操作系统**

	可在以下两个操作系统之一上预配 HDInsight 群集：
	- **Windows 上的 HDInsight (Windows Server 2012 R2 Datacenter)**：


- **HDInsight 版本**

	用于确定要用于此群集的 HDInsight 版本。有关详细信息，请参阅 [HDInsight 中的 Hadoop 群集版本和组件](/documentation/articles/hdinsight-component-versioning-v1/)。

- **群集类型**和**群集大小（也称为数据节点）**

	HDInsight 可让客户部署各种不同的群集类型，用于不同的数据分析工作负荷。目前提供的群集类型包括：
	
	- Hadoop 群集：用于查询和分析工作负荷
	- HBase 群集：用于 NoSQL 工作负荷
	- Storm 群集：用于实时事件处理工作负荷

	![HDInsight 群集](./media/hdinsight-provision-clusters-v1/hdinsight.clusters.png)
 
	> [AZURE.NOTE] Azure HDInsight 群集也称为 HDInsight 中的 Hadoop 群集或者 HDInsight 群集。有时，该术语可与 Hadoop 群集换用。它们都代表托管在 Azure 环境中的 Hadoop 群集。

	在给定的群集类型中，各节点有不同的角色，使客户能够针对特定角色，根据适合其工作负荷的详细信息来调整节点的大小。例如，如果执行的分析作业类型会消耗大量内存，Hadoop 群集可以使用大量内存来预配辅助节点。

	![HDInsight Hadoop 群集角色](./media/hdinsight-provision-clusters-v1/HDInsight.Hadoop.roles.png)

	HDInsight 的 Hadoop 群集是使用两个角色部署的：

	- 头节点（2 个节点）
	- 数据节点（至少 1 个节点）

	![HDInsight Hadoop 群集角色](./media/hdinsight-provision-clusters-v1/HDInsight.HBase.roles.png)

	HDInsight 的 HBase 群集是使用三个角色部署的：
	- 头服务器（2 个节点）
	- 区域服务器（至少 1 个节点）
	- 主控/Zookeeper 节点（3 个节点）
	
	![HDInsight Hadoop 群集角色](./media/hdinsight-provision-clusters-v1/HDInsight.Storm.roles.png)

	HDInsight 的 Storm 群集是使用三个角色部署的：
	- Nimbus 节点（2 个节点）
	- Supervisor 服务器（至少 1 个节点）
	- Zookeeper 节点（3 个节点）
	

	客户需根据群集的生存期，支付这些节点的使用费。创建群集之后便开始计费，删除群集时便停止计费（无法取消分配或保留群集）。群集大小会影响群集价格。为了方便学习，建议使用 1 个数据节点。有关 HDInsight 定价的详细信息，请参阅 [HDInsight 定价](/home/features/hdinsight/pricing/)。


	>[AZURE.NOTE] 群集大小限制因 Azure 订阅而异。若要提高限制的大小，请联系计费支持人员。
	
- **区域/虚拟网络（也称为位置）**

	![Azure 区域](./media/hdinsight-provision-clusters-v1/Azure.regions.png)

	有关受支持区域的列表，请单击 [HDInsight 定价](/home/features/hdinsight/pricing/)中的“区域”下拉列表。

- **节点大小**

	![hdinsight VM 节点大小](./media/hdinsight-provision-clusters-v1/hdinsight.node.sizes.png)

	选择节点的 VM 大小。有关详细信息，请参阅[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)

	根据所选的 VM，你的成本可能会有所不同。HDInsight 对群集节点使用所有标准层 VM。有关 VM 大小如何影响价格的信息，请参阅 <a href="/home/features/hdinsight/pricing/" target="_blank">HDInsight 价格</a>。


- **HDInsight 用户**

	HDInsight 群集允许你在预配期间配置两个用户帐户：

	- HTTP 用户。默认用户名是在 Azure 经典管理门户上使用基本配置创建的 admin。
	- RDP 用户（Windows 群集）：用于通过 RDP 连接到群集。在创建帐户时，必须将过期日期设置为从当天算起的 90 天。 
  
 

- **Azure 存储帐户**


	原始 HDFS 使用群集上的多个本地磁盘。HDInsight 使用 Azure Blob 存储来存储数据。Azure Blob 存储是一种稳健、通用的存储解决方案，它与 HDInsight 无缝集成。通过 Hadoop 分布式的文件系统 (HDFS) 界面，可以针对 Blob 存储中的结构化或非结构化数据直接运行 HDInsight 中的整套组件。通过将数据存储在 Blob 存储中，你可以安全删除用于计算的 HDInsight 群集而不会丢失用户数据。
	
	在配置期间，你必须指定 Azure 存储帐户，并在该 Azure 存储帐户中指定 Azure Blob 存储容器。某些预配过程要求事先创建 Azure 存储帐户和 Blob 存储容器。群集使用该 Blob 存储容器作为默认存储位置。你也可以选择指定群集可访问的其他 Azure 存储帐户（链接的存储）。此外，群集还可以访问任何配置有完全公共读取权限或仅限对 blob 的公共读取权限的 Blob 容器。有关限制访问的详细信息，请参阅[管理对 Azure 存储资源的访问](/documentation/articles/storage-manage-access-to-resources/)。

	![HDInsight 存储](./media/hdinsight-provision-clusters-v1/HDInsight.storage.png)
	
	>[AZURE.NOTE] Blob 存储容器提供一组 Blob 集，如图所示：
	
	![Azure Blob 存储](./media/hdinsight-provision-clusters-v1/Azure.blob.storage.jpg)
	  
	
	>[AZURE.WARNING] 不要对多个群集共享一个 Blob 存储容器。此操作不受支持。
	
	有关使用辅助 Blob 存储的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。

- **Hive/Oozie 元存储**

	元存储包含 Hive 和 Oozie 元数据，例如 Hive 表、分区、架构和列。使用元存储有助于保留你的 Hive 和 Oozie 元数据，这样，在设置新群集时，你不需要重新创建 Hive 表或 Oozie 作业。默认情况下，Hive 使用嵌入的 Azure SQL 数据库存储此信息。在删除群集时，嵌入的数据库无法保留元数据。例如，你使用 Hive 元存储预配了一个群集。你创建了一些 Hive 表。在删除群集并通过相同的 Hive 元存储重建群集之后，便可以看到在原始群集中创建的 Hive 表。

## 高级配置选项

>[AZURE.NOTE] 本部分目前仅适用于基于 Windows 的 HDInsight 群集。

### 使用 HDInsight 群集自定义功能来自定义群集

有时，你可能需要配置配置文件。以下是其中的一些配置文件。

- core-site.xml
- hdfs-site.xml
- mapred-site.xml
- yarn-site.xml
- hive-site.xml
- oozie-site.xml

群集无法保留重新制作映像所造成的更改。有关详细信息，请参阅[重新启动角色实例进行 OS 升级](http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx)。若要在群集生存期保留更改，你可以在预配过程中使用 HDInsight 群集自定义。

下面是用于自定义 Hive 配置的 Azure PowerShell 脚本示例：

	# hive-site.xml configuration 
	$hiveConfigValues = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightHiveConfiguration'
	$hiveConfigValues.Configuration = @{ "hive.metastore.client.socket.timeout"="90" } #default 60
	
	$config = New-AzureHDInsightClusterConfig `
	            -ClusterSizeInNodes $clusterSizeInNodes `
	            -ClusterType $clusterType `
	          | Set-AzureHDInsightDefaultStorage `
	            -StorageAccountName $defaultStorageAccount `
	            -StorageAccountKey $defaultStorageAccountKey `
	            -StorageContainerName $defaultBlobContainer `
	          | Add-AzureHDInsightConfigValues `
	            -Hive $hiveConfigValues 
	
	New-AzureHDInsightCluster -Name $clusterName -Location $location -Credential $credential -OSType Windows -Config $config

下面是有关自定义其他配置文件的更多示例：

	# hdfs-site.xml configuration
	$HdfsConfigValues = @{ "dfs.blocksize"="64m" } #default is 128MB in HDI 3.0 and 256MB in HDI 2.1
	
	# core-site.xml configuration
	$CoreConfigValues = @{ "ipc.client.connect.max.retries"="60" } #default 50
	
	# mapred-site.xml configuration
	$MapRedConfigValues = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightMapReduceConfiguration'
	$MapRedConfigValues.Configuration = @{ "mapreduce.task.timeout"="1200000" } #default 600000
	
	# oozie-site.xml configuration
	$OozieConfigValues = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightOozieConfiguration'
	$OozieConfigValues.Configuration = @{ "oozie.service.coord.normal.default.timeout"="150" }  # default 120
	
有关详细信息，请参阅 Azim Uddin 标题为[自定义 HDInsight 群集预配](http://blogs.msdn.com/b/bigdatasupport/archive/2014/04/15/customizing-hdinsight-cluster-provisioning-via-powershell-and-net-sdk.aspx)的博客。




### 使用脚本操作自定义群集

你可以在设置期间通过使用脚本安装其他组件或自定义群集配置。此类脚本可通过**脚本操作**调用，脚本操作是一种配置选项，可通过 Azure 经典管理门户、HDInsight Windows PowerShell cmdlet 或 HDInsight .NET SDK 使用。有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-v1/)。


### 使用 Azure 虚拟网络

[Azure 虚拟网络](/documentation/services/networking/)允许你创建包含需要用于解决方案的资源的安全永久性网络。通过虚拟网络，你可以：

* 在专用网络（仅限云）中将云资源连接在一起。

	![仅限云配置示意图](./media/hdinsight-provision-clusters-v1/hdinsight-vnet-cloud-only.png)

* 通过使用虚拟专用网络 (VPN) 将云资源连接到本地数据中心网络（站点到站点或点到站点）。

	利用站点到站点配置，你可以通过使用硬件 VPN 或路由和远程访问服务将多个资源从数据中心连接到 Azure 虚拟网络。

	![站点到站点配置示意图](./media/hdinsight-provision-clusters-v1/hdinsight-vnet-site-to-site.png)

	利用点到站点配置，你可以通过使用软件 VPN 将特定资源连接到 Azure 虚拟网络。

	![点到站点配置示意图](./media/hdinsight-provision-clusters-v1/hdinsight-vnet-point-to-site.png)

有关将 HDInsight 与虚拟网络配合使用的信息（包括虚拟网络的特定配置要求），请参阅 [Extend HDInsight capbilities by using an Azure Virtual Network（使用 Azure 虚拟网络扩展 HDInsight 功能）](/documentation/articles/hdinsight-extend-hadoop-virtual-network/)。


##<a name="cluster-creation-methods"></a> 预配工具

- Azure 经典管理门户
- Azure PowerShell
- .NET SDK
- CLI

###<a name="portal"></a> 使用 Azure 经典管理门户

你可以参考 [基本配置选项] 和 [高级配置选项] 以获取有关字段的说明。

**通过使用“自定义创建”选项创建 HDInsight 群集**

1. 登录到 [Azure 经典管理门户][azure-management-portal]。
2. 单击页面底部的“+ 新建'，然后依次单击“数据服务”、“HDINSIGHT”和“自定义创建”。
3. 在“群集详细信息”页上，键入或选择以下值：

	- 群集名称
	- 订阅名称
	- 群集类型
	- 操作系统
	- HDInsight 版本

4. 在“配置群集”页上，输入或选择以下值：

	- 数据节点
	- 区域/虚拟网络
	- 头节点大小
	- 数据节点大小

5. 在“配置群集用户”页上提供以下值：

	- HTTP 用户名
	- HTTP 密码/确认密码
	- 为群集启用远程桌面
	- 输入 Hive/Oozie 元存储

6. 如果在上一屏幕中选择了输入 Hive/Oozie 元存储，请在“配置 Hive/Oozie 元存储”页上提供以下值：

	- Hive 元存储数据库
	- 数据库用户
	- 数据库用户密码
	- Oozie 元存储数据库
	- 数据库用户
	- 数据库用户密码

7. 在“存储帐户”页上提供以下值：

	- 存储帐户
	- 帐户名
	- 默认容器
	- 其他存储帐户

8. 在“脚本操作”页上，单击“添加脚本操作”以提供自定义脚本的详细信息，在创建群集时，你需要运行该脚本来自定义群集。有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-v1/)。

	- 名称：指定脚本操作的名称
	- 脚本 URI：指定调用以自定义群集的脚本的统一资源标识符 (URI)。
	- 节点类型：指定在其上运行自定义脚本的节点。你可以选择“所有节点”、“仅限头节点”或“仅限数据节点”。<b></b><b></b><b></b>
	- 参数：根据脚本的需要，指定参数。</td></tr>

	你可以添加多个脚本操作，以在群集上安装多个组件。添加脚本后，单击复选标记以开始设置群集。

### 使用 Azure PowerShell
Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。本部分提供有关如何通过使用 Azure PowerShell 设置 HDInsight 群集的说明。有关配置工作站以运行 HDInsight Windows Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。有关将 Azure PowerShell 与 HDInsight 配合使用的详细信息，请参阅[使用 PowerShell 管理 HDInsight](/documentation/articles/hdinsight-administer-use-powershell/)。有关 HDInsight Windows PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn858087.aspx)。

> [AZURE.NOTE] 虽然本部分中的脚本可用于在 Azure 虚拟网络上配置 HDInsight 群集，但它们不能用于创建 Azure 虚拟网络。有关创建 Azure 虚拟网络的信息，请参阅[虚拟网络配置任务](/documentation/articles/virtual-networks-create-vnet-arm-ps/)。

通过使用 Azure PowerShell 设置 HDInsight 群集需要执行以下过程：

- 创建 Azure 存储帐户
- 创建 Azure Blob 容器
- 创建 HDInsight 群集

你可以使用 Windows PowerShell 控制台或 Windows PowerShell 集成脚本环境 (ISE) 来运行脚本。

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。存储帐户必须与 HDInsight 群集位于同一数据中心。

**连接到 Azure 帐户**

	Add-AzureAccount -Environment AzureChinaCloud

系统将提示你输入 Azure 帐户凭据。

**创建 Azure 存储帐户**

	$storageAccountName = "<StorageAcccountName>"	# Provide a Storage account name
	$location = "<MicrosoftDataCenter>"				# For example, "China North"

	# Create an Azure Storage account
	New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location

如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下 Windows PowerShell 命令来检索该信息：

	# List Storage accounts for the current subscription
	Get-AzureStorageAccount

	# List the keys for a Storage account
	Get-AzureStorageKey "<StorageAccountName>"

**创建 Azure Blob 存储容器**

	$storageAccountName = "<StorageAccountName>"	# Provide the Storage account name
	$containerName="<ContainerName>"				# Provide a container name

	# Create a storage context object
	$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
	$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName
	                                       -StorageAccountKey $storageAccountKey  

	# Create a Blob storage container
	New-AzureStorageContainer -Name $containerName -Context $destContext

准备好存储帐户和 Blob 容器后，你就可以创建群集了。

**设置 HDInsight 群集**

> [AZURE.NOTE] Azure PowerShell cmdlet 是唯一推荐用于更改 HDInsight 群集中的配置变量的方法。如果对群集进行修补，可能会覆盖通过远程桌面连接到群集时对 Hadoop 配置文件所做的更改。如果对群集进行修补，将保留通过 Azure PowerShell 设置的配置值。

- 在 Azure PowerShell 控制台窗口中运行以下命令：

		$subscriptionName = "<AzureSubscriptionName>"	  # The Azure subscription used for the HDInsight cluster to be created
	
		$storageAccountName = "<AzureStorageAccountName>" # HDInsight cluster requires an existing Azure Storage account to be used as the default file system
	
		$clusterName = "<HDInsightClusterName>"			  # The name for the HDInsight cluster to be created
		$clusterNodes = <ClusterSizeInNodes>              # The number of nodes in the HDInsight cluster
		$hadoopUserName = "<HadoopUserName>"              # User name for the Hadoop user. You will use this account to connect to the cluster and run jobs.
		$hadoopUserPassword = "<HadoopUserPassword>"
	
		$secPassword = ConvertTo-SecureString $hadoopUserPassword -AsPlainText -Force
		$credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$secPassword)
	
		# Get the storage primary key based on the account name
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$containerName = $clusterName				# Azure Blob container that is used as the default file system for the HDInsight cluster
	
		# The location of the HDInsight cluster. It must be in the same data center as the Storage account.
		$location = Get-AzureStorageAccount -StorageAccountName $storageAccountName | %{$_.Location}
	
		# Create a new HDInsight cluster
		New-AzureHDInsightCluster -Name $clusterName -Credential $credential -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes -ClusterType Hadoop

	>[AZURE.NOTE] $hadoopUserName 和 $hadoopUserPassword 命令用于群集创建的 Hadoop 用户帐户。此帐户将用于连接到群集并运行作业。如果你在 Azure 经典管理门户中使用“快速创建”选项来设置群集，则默认 Hadoop 用户名为“admin”。不要将此帐户与远程桌面协议 (RDP) 用户帐户相混淆。RDP 用户帐户不能与 Hadoop 用户帐户相同。有关详细信息，请参阅 [使用 Azure 经典管理门户管理 HDInsight 中的 Hadoop 群集][hdinsight-admin-portal]。

	设置群集可能需要几分钟时间。

	![HDI.CLI.Provision][image-hdi-ps-provision]

**通过使用自定义配置选项设置 HDInsight 群集**

设置群集时，你可以使用其他配置选项，例如，连接到多个 Azure Blob 存储，使用虚拟网络，或者对 Hive 和 Oozie 元存储使用 Azure SQL 数据库。这样，你便可以将数据和元数据的生存期与群集的生存期分开。

> [AZURE.NOTE] Windows PowerShell cmdlet 是唯一推荐用于更改 HDInsight 群集中的配置变量的方法。如果对群集进行修补，可能会覆盖通过远程桌面连接到群集时对 Hadoop 配置文件所做的更改。如果对群集进行修补，将保留通过 Azure PowerShell 设置的配置值。

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

		# Get the Storage account keys
		Select-AzureSubscription $subscriptionName
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$storageAccountKey_Add1 = Get-AzureStorageKey $storageAccountName_Add1 | %{ $_.Primary }

		$oozieCreds = Get-Credential -Message "Oozie metastore"
		$hiveCreds = Get-Credential -Message "Hive metastore"

		# Create a Blob storage container
		$dest1Context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName_Default -StorageAccountKey $storageAccountKey_Default  
		New-AzureStorageContainer -Name $containerName_Default -Context $dest1Context

		# Create a new HDInsight cluster
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
		    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
		    Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Add1.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Add1 |
		    Add-AzureHDInsightMetastore -SqlAzureServerName "$hiveSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $hiveSQLDatabaseName -Credential $hiveCreds -MetastoreType HiveMetastore |
		    Add-AzureHDInsightMetastore -SqlAzureServerName "$oozieSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $oozieSQLDatabaseName -Credential $oozieCreds -MetastoreType OozieMetastore |
		        New-AzureHDInsightCluster -Name $clusterName -Location $location -VirtualNetworkId $vnetID -SubnetName $subNetName

	>[AZURE.NOTE] 用于元存储的 Azure SQL 数据库必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击“配置”，单击“Azure 服务”对应的“是”，然后单击“保存”。

**列出 HDInsight 群集**

- 在 Azure PowerShell 控制台窗口中运行以下命令，以列出 HDInsight 群集并验证是否已成功设置群集：

		Get-AzureHDInsightCluster -Name <ClusterName>


### 使用 Azure CLI

> [AZURE.NOTE] 自 2014 年 8 月 29 日起，Azure CLI 无法用于将群集与 Azure 虚拟网络相关联。

用于预配 HDInsight 群集的另一选项是 Azure CLI。Azure CLI 是以 Node.js 实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。

有关如何使用 Azure CLI 的一般指导，请参阅 [Azure CLI](/documentation/articles/xplat-cli-install/)。

以下说明将指导你在 Linux 和 Windows 上安装 Azure CLI，然后使用命令行来预配群集。

- [设置适用于 Linux 的 Azure CLI](#clilin)
- [设置适用于 Windows 的 Azure CLI](#cliwin)
- [使用 Azure CLI 预配 HDInsight 群集](#cliprovision)

#### <a id="clilin"></a>安装适用于 Linux 的 Azure CLI

执行以下过程，将你的 Linux 计算机设置为使用 Azure 命令行界面 (Azure CLI)：

- 使用 Node.js 包管理器 (NPM) 安装 Azure CLI
- 连接到你的 Azure 订阅

**使用 NPM 安装 Azure CLI**

1.	在 Linux 计算机上打开终端窗口，然后运行以下命令：

		sudo npm install -g https://github.com/azure/azure-xplat-cli/archive/hdinsight-February-18-2015.tar.gz

2.	运行以下命令以验证安装：

		azure hdinsight -h

	可以在不同级别使用 *-h* 开关以显示帮助信息。例如：

		azure -h
		azure hdinsight -h
		azure hdinsight cluster -h
		azure hdinsight cluster create -h

**连接到 Azure 订阅**

在使用 Azure CLI 之前，你必须配置工作站和 Azure 之间的连接。Azure CLI 使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 的发布设置文件中获取此信息。稍后可以导入发布设置文件作为永久性本地配置设置，Azure CLI 会将此设置用于后续操作。你只需导入你的发布设置一次。

> [AZURE.NOTE] 发布设置文件包含敏感信息。Azure 建议你删除该文件或采取其他措施来加密包含该文件的用户文件夹。在 Windows 上，修改文件夹属性或使用 BitLocker 驱动程序加密。


1.	打开终端窗口。
2.	运行以下命令以登录到你的 Azure 订阅：

		azure account download

	![HDI.Linux.CLIAccountDownloadImport](./media/hdinsight-provision-clusters-v1/HDI.Linux.CLIAccountDownloadImport.png)

	该命令将启动要从中下载发布设置文件的网页。如果网页未打开，请单击终端窗口中的链接以打开浏览器页并登录到该经典管理门户。

3.	将发布设置文件下载到计算机。
5.	从命令提示符窗口，运行以下命令以导入发布设置文件：

		azure account import <path/to/the/file>


#### <a id="cliwin"></a>安装适用于 Windows 的 Azure CLI

执行以下过程，将你的 Windows 计算机设置为使用 Azure CLI：

- 安装 Azure CLI（使用 NPM 或 Windows 安装程序）
- 下载并导入 Azure 帐户发布设置


Azure CLI 可通过 NPM 或 Windows 安装程序来安装。Azure 建议你只使用这两个选项中的一个来进行安装。

**使用 NPM 安装 Azure CLI**

1.	浏览到 **www.nodejs.org**。
2.	单击“安装”，然后使用默认设置按照说明操作。
3.	从工作站打开“命令提示符”（或“Azure 命令提示符”，或“VS2012 的开发人员命令提示符”）。
4.	在命令提示符窗口中运行以下命令：

		npm install -g https://github.com/azure/azure-xplat-cli/archive/hdinsight-February-18-2015.tar.gz

	> [AZURE.NOTE] 如果收到“未找到 NPM 命令”的错误消息，请验证以下路径位于 PATH 环境变量中：<i>C:\\Program Files (x86)\\nodejs;C:\\Users[用户名]\\AppData\\Roaming\\npm</i> 或 <i>C:\\Program Files\\nodejs;C:\\Users[用户名]\\AppData\\Roaming\\npm</i>

5.	运行以下命令以验证安装：

		azure hdinsight -h

	可以在不同级别使用 *-h* 开关以显示帮助信息。例如：

		azure -h
		azure hdinsight -h
		azure hdinsight cluster -h
		azure hdinsight cluster create -h

**使用 Windows 安装程序安装 Azure CLI**

1.	浏览到 **/downloads/**。
2.	向下滚动到“命令行工具”部分，单击“Azure 命令行界面”，然后根据 Web 平台安装程序向导的提示操作。

**下载和导入发布设置**

在使用 Azure CLI 之前，你必须配置工作站和 Azure 之间的连接。Azure CLI 使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 的发布设置文件中获取此信息。稍后可以导入发布设置文件作为永久性本地配置设置，Azure CLI 会将此设置用于后续操作。你只需导入你的发布设置一次。

> [AZURE.NOTE] 发布设置文件包含敏感信息。Azure 建议你删除该文件或采取其他措施来加密包含该文件的用户文件夹。在 Windows 上，修改文件夹属性或使用 BitLocker。


1.	打开**命令提示符**。
2.	运行以下命令来下载发布设置文件：

		azure account download

	![HDI.CLIAccountDownloadImport][image-cli-account-download-import]

	该命令将启动要从中下载发布设置文件的网页。

3.	出现保存文件的提示时，请单击“保存”并提供文件的保存位置。
5.	从命令提示符窗口，运行以下命令以导入发布设置文件：

		azure account import <path/to/the/file>

#### <a id="cliprovision"></a>使用 Azure CLI 预配 HDInsight 群集

使用 Azure CLI 预配 HDInsight 群集的过程如下：

- 创建 Azure 存储帐户
- 设置群集

**创建 Azure 存储帐户**

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。存储帐户必须位于同一数据中心内。

- 在命令提示符窗口中运行以下命令，以创建 Azure 存储帐户：

		azure storage account create [options] <StorageAccountName>

	出现指定位置的提示时，请选择 HDInsight 群集可以设置到的位置。该存储位置必须与 HDInsight 群集所在的位置相同。目前，只有**中国北部**和**中国东部**区域可以托管 HDInsight 群集。

有关使用 Azure 经典管理门户创建 Azure 存储帐户的信息，请参阅 [Create, manage, or delete a storage account（创建、管理或删除存储帐户）](/documentation/articles/storage-create-storage-account/)。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下命令来检索该信息：

	-- Lists Storage accounts
	azure storage account list

	-- Shows information for a Storage account
	azure storage account show <StorageAccountName>

	-- Lists the keys for a Storage account
	azure storage account keys list <StorageAccountName>

有关使用 Azure 经典管理门户获取信息的详细信息，请参阅 [Create, manage, or delete a storage account（创建、管理或删除存储帐户）](/documentation/articles/storage-create-storage-account/)中的 How to: View, copy and regenerate storage access keys（如何：查看、复制和重新生成存储访问密钥）部分。

HDInsight 群集还需要在存储帐户中提供一个容器。如果你提供的存储帐户尚不包含容器，*azure hdinsight cluster create* 命令将提示你输入容器名称，然后会创建该容器。但是，如果你想要预先创建容器，则可以使用以下命令：

	azure storage container create --account-name <StorageAccountName> --account-key <StorageAccountKey> [ContainerName]

准备好存储帐户和 Blob 容器后，你就可以创建群集了。

**设置 HDInsight 群集**

- 从命令提示符窗口，运行以下命令：

		azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey <storageAccountKey> --storageContainer <StorageContainerName> --dataNodeCount <NumberOfNodes> --location <DataCenterLocation> --userName <HDInsightClusterUsername> --password <HDInsightClusterPassword> --osType windows

	![HDI.CLIClusterCreation][image-cli-clustercreation]


**通过使用配置文件设置 HDInsight 群集**

通常，你要设置 HDInsight 群集，运行作业，然后删除该群集以降低成本。在 Azure CLI 上，你可以选择将配置保存到文件，以便在每次预配群集时重用这些配置。

- 在命令提示符窗口中运行以下命令：


		#Create the config file
		azure hdinsight cluster config create <file>

		#Add commands to create a basic cluster
		azure hdinsight cluster config set <file> --clusterName <ClusterName> --dataNodeCount <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --userName "<Username>" --password "<UserPassword>" --osType windows

		#If required, include commands to use additional Blob storage with the cluster
		azure hdinsight cluster config storage add <file> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn"
		       --storageAccountKey "<StorageAccountKey>"

		#If required, include commands to use a SQL database as a Hive metastore
		azure hdinsight cluster config metastore set <file> --type "hive" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
		       --database "<HiveDatabaseName>" --user "<Username>" --metastorePassword "<UserPassword>"

		#If required, include commands to use a SQL database as an Oozie metastore
		azure hdinsight cluster config metastore set <file> --type "oozie" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
		       --database "<OozieDatabaseName>" --user "<SQLUsername>" --metastorePassword "<SQLPassword>"

		#Run this command to create a cluster by using the config file
		azure hdinsight cluster create --config <file>

	>[AZURE.NOTE] 用于元存储的 Azure SQL 数据库必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击“配置”，单击“Azure 服务”对应的“是”，然后单击“保存”。


	![HDI.CLIClusterCreationConfig][image-cli-clustercreation-config]


**列出和显示群集详细信息**

- 使用以下命令来列出和显示群集详细信息：

		azure hdinsight cluster list
		azure hdinsight cluster show <ClusterName>

	![HDI.CLIListCluster][image-cli-clusterlisting]


**删除群集**

- 使用以下命令来删除群集：

		azure hdinsight cluster delete <ClusterName>



###<a name="sdk"></a> 使用 HDInsight .NET SDK
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 应用程序使用 HDInsight 的操作。

通过使用 SDK 设置 HDInsight 群集时必须执行以下过程：

- 安装 HDInsight .NET SDK
- 创建自签名证书
- 创建控制台应用程序
- 运行应用程序


**安装 HDInsight .NET SDK**

你可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装该 SDK 的最新发行版。下一过程中将显示说明。

**创建自签名证书**

创建自签名证书，将其安装到工作站上，然后将其上传到你的 Azure 订阅。有关说明，请参阅[创建自签名证书](/documentation/articles/hdinsight-administer-use-management-portal-v1/#cert)。


**创建 Visual Studio 控制台应用程序**

1. 打开 Visual Studio 2013 或 2015。

2. 在“文件”菜单中，单击“新建”，然后单击“项目”。

3. 在“新建项目”中，键入或选择以下值：

	|属性|值|
	|--------|-----|
	|模板|模板/Visual C#/Windows/控制台应用程序|
	|Name|CreateHDICluster|

4. 单击“确定”以创建该项目。

5. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。

6. 在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

	这些命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。
8. 将此代码替换为以下代码：

		using System;
		using Microsoft.WindowsAzure.Management.HDInsight;
		using System.Security.Cryptography.X509Certificates;

		namespace CreateHDICluster
		{
		    internal class Program
		    {
		        private static IHDInsightClient _hdinsightClient;
		
		        private static String SubscriptionId = "<Your Azure Subscription ID>";
				private static Uri baseUri = new Uri("https://management.core.chinacloudapi.cn");
				private static X509Certificate2 cert = new X509Certificate2("c:/path/to/cert.cer");

		        private const string NewClusterName = "<HDINSIGHT CLUSTER NAME>";
		        private const int NewClusterNumNodes = <NUMBER OF NODES>;
		        private const string NewClusterLocation = "<LOCATION>";  // Must match the Azure Storage account location
		        private const ClusterType NewClusterType = ClusterType.Hadoop;
		        private const OSType NewClusterOSType = OSType.Windows;
		        private const string NewClusterVersion = "3.2";

		        private const string NewClusterUsername = "admin";
		        private const string NewClusterPassword = "<HTTP USER PASSWORD>";
		        private const string ExistingStorageName = "<STORAGE ACCOUNT NAME>.blob.core.chinacloudapi.cn";
		        private const string ExistingStorageKey = "<STORAGE ACCOUNT KEY>";
		        private const string ExistingContainer = "<DEFAULT CONTAINER NAME>"; 
		
		
		        static void Main(string[] args)
				{
					System.Console.WriteLine("Creating a cluster.  The process takes 10 to 20 minutes ...");

					_hdinsightClient = HDInsightClient.Connect(new HDInsightCertificateCredential(new Guid(SubscriptionId), cert, baseUri));

					ClusterCreateParametersV2 parameters = new ClusterCreateParametersV2 {
						ClusterSizeInNodes = NewClusterNumNodes,
						UserName = NewClusterUsername,
						Password = NewClusterPassword,
						Location = NewClusterLocation,
						DefaultStorageAccountName = ExistingStorageName,
						DefaultStorageAccountKey = ExistingStorageKey,
						DefaultStorageContainer = ExistingContainer,
						ClusterType = NewClusterType,
						OSType = NewClusterOSType
					};

					var cluster = _hdinsightClient.CreateCluster(parameters);

                    System.Console.WriteLine("The cluster has been created. Press ENTER to continue ...");
                    System.Console.ReadLine();
                    
				}
			}
		}
		
10. 替换类成员值。

**运行应用程序**

该应用程序在 Visual Studio 中打开时，按 **F5** 运行该应用程序。控制台窗口应打开并显示应用程序的状态。设置一个 HDInsight 群集可能需要几分钟时间。


##<a id="nextsteps"></a>后续步骤
在本文中，你已经学习了几种设置 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/) - 了解如何开始使用你的 HDInsight 群集
* [将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/) - 了解如何在 HDInsight 和 SQL 数据库或 SQL Server 之间复制数据
* [使用 PowerShell 管理 HDInsight](/documentation/articles/hdinsight-administer-use-powershell/) - 了解如何通过 Azure PowerShell 使用 HDInsight
* [以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/) - 了解如何以编程方式将作业提交到 HDInsight
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation] - 探索 HDInsight SDK


[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx
[azure-management-portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0530_2016-->