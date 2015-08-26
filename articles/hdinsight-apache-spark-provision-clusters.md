<properties 
   pageTitle="在 HDInsight 中设置 Apache Spark 群集 | Azure" 
   description="了解如何通过使用 Azure 门户、Azure PowerShell、命令行或 HDInsight .NET SDK 设置 Azure HDInsight 的 Spark 群集" 
   services="hdinsight" 
   documentationCenter="" 
   authors="nitinme" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.date="07/10/2015"
   wacn.date=""/>

# 使用自定义选项在 HDInsight 中设置 Apache Spark 群集

在大多数情况下，你可以使用 [HDInsight 上的 Apache Spark 入门](/documentation/articles/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql)中所述的快速创建方法设置 Spark 群集。但在某些情况下，你可能想要设置自定义群集。例如，你可能想要连接到外部元数据存储，让 Hive 元数据的持久性超出群集生存期，或是者你可能想要对群集使用其他存储。

针对此类方案和其他方案，本文说明了如何使用 Azure 门户、Azure PowerShell 或 HDInsight .NET SDK，在 HDInsight 上设置自定义 Spark 群集。


**先决条件：**

在开始按照本文中的说明操作之前，你必须有一个 Azure 订阅。

##<a id="configuration"></a>有哪些不同的配置选项？

###其他存储

在配置期间，你必须指定 Azure Blob 存储帐户和默认容器。该容器被集群用作默认存储位置。你可以选择指定还要与群集关联的其他 Azure 存储帐户。

>[AZURE.NOTE]不要对多个群集共享一个 Blob 存储容器。此操作不受支持。

有关使用辅助 Blob 存储的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-use-blob-storage)。

###元存储

Spark 可让你基于原始数据定义架构和 Hive 表。可以将这些架构和表元数据存储在外部元存储中。使用元存储有助于保留你的 Hive 元数据，这样，在设置新群集时，你不需要重新创建 Hive 表。默认情况下，Hive 使用嵌入的数据库存储该信息。在删除群集时，嵌入的数据库无法保留元数据。

### 群集自定义

你可以在设置期间通过使用脚本安装其他组件或自定义群集配置。此类脚本可通过**脚本操作**调用，脚本操作是一种配置选项，可通过 Azure 门户、HDInsight Windows PowerShell cmdlet 或 HDInsight .NET SDK 使用。有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集][hdinsight-customize-cluster]。


###虚拟网络

[Azure 虚拟网络](http://www.windowsazure.cn/home/features/networking/)允许你创建包含需要用于解决方案的资源的安全永久性网络。通过虚拟网络，你可以：

* 在专用网络（仅限云）中将云资源连接在一起。

	![仅限云配置示意图](./media/hdinsight-apache-spark-provision-clusters/hdinsight-vnet-cloud-only.png)

* 通过使用虚拟专用网络 (VPN) 将云资源连接到本地数据中心网络（站点到站点或点到站点）。

	利用站点到站点配置，你可以通过使用硬件 VPN 或路由和远程访问服务将多个资源从数据中心连接到 Azure 虚拟网络。

	![站点到站点配置示意图](./media/hdinsight-apache-spark-provision-clusters/hdinsight-vnet-site-to-site.png)

	利用点到站点配置，你可以通过使用软件 VPN 将特定资源连接到 Azure 虚拟网络。

	![点到站点配置示意图](./media/hdinsight-apache-spark-provision-clusters/hdinsight-vnet-point-to-site.png)

有关虚拟网络特性、优势和功能的详细信息，请参阅 [Azure 虚拟网络概述](http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)。

> [AZURE.NOTE]你必须先创建 Azure 虚拟网络，然后才能设置群集。有关详细信息，请参阅[虚拟网络配置任务](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx)。
>
> Azure HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。
>
> 强烈建议为一个群集指定一个子网。

##<a id="portal"></a>使用 Azure 门户

HDInsight 上的 Spark 群集使用 Azure Blob 存储容器作为默认文件系统。创建 HDInsight 群集前，需要具有位于同一数据中心的 Azure 存储帐户。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage)。有关创建 Azure 存储帐户的详细信息，请参阅[如何创建存储帐户][azure-create-storageaccount]。

**通过使用“自定义创建”选项创建 HDInsight 群集**

1. 登录到 [Azure 门户][azure-management-portal]。
2. 单击页面底部的“+ 新建'，然后依次单击“数据服务”、“HDINSIGHT”和“自定义创建”。
3. 在“群集详细信息”页上，键入或选择以下值：

	![在 HDInsight 群集上设置 Spark 的详细信息](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page1.png "在 HDInsight 群集上设置 Spark 的详细信息")

    <table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>群集名称</td>
			<td><p>命名群集。</p>
				<ul>
				<li>域名系统 (DNS) 名称必须以字母数字字符开头和结尾，并且可以包含短划线。</li>
				<li>字段必须是介于 3 到 63 个字符之间的字符串。</li>
				</ul></td></tr>
		<tr><td>群集类型</td>
			<td>对于群集类型，请选择 <strong>Spark</strong>。</td></tr>
		<tr><td>操作系统</td>
			<td>选择 <b>Windows Server 2012 R2 Data Center</b> 来设置 Windows 上运行的 Spark 群集。</td></tr>
		<tr><td>HDInsight 版本</td>
			<td>选择版本。对于 Spark，唯一可用的版本选项是使用 Spark 1.3.1 的 <b>HDInsight 3.2</b>。</td></tr>
		</table>

	输入或选择表中所示的值，然后单击右箭头。

4. 在“配置群集”页上，输入或选择以下值：

	![提供群集的节点数和节点类型](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page2.png "提供群集的节点数和节点类型")

	<table border="1">
	<tr><th>Name</th><th>值</th></tr>
	<tr><td>数据节点</td><td>要部署的数据节点的数目。出于测试目的，创建一个单节点群集。<br />群集大小限制因 Azure 订阅而异。若要提高限制的大小，请联系 Azure 计费支持。</td></tr>
	<tr><td>区域/虚拟网络</td><td><p>选择与上一个过程中创建的存储帐户相同的区域。HDInsight 要求存储帐户位于同一区域中。稍后，在配置中，你只能选择位于你在此处指定的区域中的存储帐户。</p>。<br/>如果你创建了 Azure 虚拟网络，则可以选择 HDInsight 群集要配置使用的网络。</p><p>有关创建 Azure 虚拟网络的详细信息。</p></td></tr>
	<tr><td>头节点大小</td><td><p>为头节点选择虚拟机 (VM) 大小。</p></td></tr>
	<tr><td>数据节点大小</td><td><p>为数据节点选择 VM 大小。</p></td></tr>
	</table>

	>[AZURE.NOTE]根据所选的 VM，你的成本可能会有所不同。HDInsight 对群集节点使用所有标准层 VM。有关 VM 大小如何影响价格的信息，请参阅 <a href="http://azure.microsoft.com/pricing/details/hdinsight/" target="_blank">HDInsight 价格</a>。	


5. 在“配置群集用户”页上提供以下值：

    ![提供管理用户和 RDP 用户详细信息](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page3.png "提供管理用户和 RDP 用户详细信息")

    <table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>HTTP 用户名</td>
			<td>指定 HDInsight 群集用户名。</td></tr>
		<tr><td>HTTP 密码/确认密码</td>
			<td>指定 HDInsight 群集用户密码。</td></tr>
		<tr><td>为群集启用远程桌面</td>
			<td>在设置后，选中此复选框，以为可远程连接到群集节点的远程桌面用户指定用户名、密码和到期日期。稍后，你还可以在设置了群集后启用远程桌面。有关说明，请参阅<a href="hdinsight-administer-use-management-portal/#rdp" target="_blank">使用 RDP 连接到 HDInsight 群集</a>。</td></tr>
		<tr><td>输入 Hive/Oozie 元存储</td>
			<td>选中此复选框可指定要用作 Hive/Oozie 元存储的群集所在数据中心上的 SQL 数据库。如果你选中此复选框，则必须在向导的后续页中指定有关 Azure SQL 数据库的详细信息。如果你希望即使在删除群集后也会保留有关 Hive/Oozie 作业的元数据，则此选项将十分有用。有关如何创建元存储的说明，请参阅<a href="http://www.windowsazure.cn/documentation/articles/sql-database-get-started/" target="_blank">创建你的第一个 Azure SQL Database</a>。</td></tr>
		</td></tr>
	</table>

	>[AZURE.NOTE]用于元存储的 Azure SQL Database 必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击“配置”，单击“Microsoft Azure 服务”对应的“是”，然后单击“保存”。

    单击右箭头。

6. 在“配置 Hive/Oozie 元存储”页上提供以下值：

    ![提供 Hive/Oozie 元存储数据库详细信息](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page4.png "提供 Hive/Oozie 元存储数据库详细信息")

	指定要用作 Hive/Oozie 元存储的 Azure SQL 数据库。你可以为 Hive 和 Oozie 元存储指定相同的数据库。此 SQL 数据库必须与 HDInsight 群集位于同一数据中心。该列表框只列出你在“群集详细信息”页中指定的同一数据中心的 SQL 数据库<strong></strong>。另请指定用于连接到你选择的 Azure SQL 数据库的用户名和密码。

    >[AZURE.NOTE]用于元存储的 Azure SQL Database 必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击“配置”，单击“Azure 服务”对应的“是”，然后单击“保存”。

    单击右箭头。

7. 在“存储帐户”页上提供以下值：

    ![输入存储帐户详细信息](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page5.png "输入存储帐户详细信息")

	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>存储帐户</td>
			<td>为 HDInsight 群集指定将用作默认文件系统的 Azure 存储帐户。可以选择以下三个选项之一：
			<ul>
				<li><strong>使用现有存储</strong></li>
				<li><strong>创建新存储</strong></li>
				<li><strong>使用其他订阅中的存储</strong></li>
			</ul>
			</td></tr>
		<tr><td>帐户名</td>
			<td><ul>
				<li>如果选择了使用现有存储，请为“帐户名”选择现有的存储帐户<strong></strong>。下拉列表仅列出你选择设置群集的相同数据中心内的存储帐户。</li>
				<li>如果选择了“创建新存储”或“使用其他订阅中的存储”选项，则必须提供存储帐户名<strong></strong><strong></strong>。</li>
			</ul></td></tr>
		<tr><td>帐户密钥</td>
			<td>如果选择了“使用其他订阅中的存储”选项，请指定该存储帐户的帐户密钥<strong></strong>。</td></tr>
		<tr><td>默认容器</td>
			<td><p>指定存储帐户上用作 HDInsight 群集默认文件系统的默认容器。如果为“存储帐户”字段选择了“使用现有存储”，并且该帐户中不存在现有容器，则默认情况下，将创建与群集同名的容器<strong></strong><strong></strong>。如果已存在与群集同名的容器，则将在容器名称后追加一个序列号。例如，mycontainer1、mycontainer2，等等。但是，如果现有存储帐户的容器名称与你指定的群集名称不同，则你也可以使用该容器。</p>
            <p>如果选择了创建新存储或使用其他 Azure 订阅中的存储，则必须指定默认容器名称。</p>
        </td></tr>
		<tr><td>其他存储帐户</td>
			<td>HDInsight 支持多个存储帐户。一个群集可以使用的其他存储帐户数没有限制。但是，如果你通过使用 Azure 门户创建群集，则由于 UI 限制，你最多只能创建七个存储帐户。指定的每个其他存储帐户将在向导中添加一个额外的“存储帐户”页，以便你在此指定帐户信息。例如，在上面的屏幕截图中，选择了 1 个附加的存储帐户，因此第 5 页添加到了对话框。</td></tr>
	</table>

	单击右箭头。

8. 如果你选择了为群集配置其他存储，请在“存储帐户”页上，输入其他存储帐户的帐户信息：

	![输入其他存储帐户详细信息](./media/hdinsight-apache-spark-provision-clusters/HDI.CustomProvision.Page6.png "输入其他存储帐户详细信息")

    同样，你可以选择从现有存储创建新存储，或者使用其他 Azure 订阅中的存储。提供值的过程类似于前面的步骤。

    > [AZURE.NOTE]一旦为 HDInsight 群集选择了 Azure 存储帐户，就不能再删除该帐户，也不能将它更改为另一帐户。

8. 在“脚本操作”页上，单击“添加脚本操作”，并在创建群集时提供有关要运行以自定义群集的自定义脚本的详细信息。有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集][hdinsight-customize-cluster]。

	如果需要使用脚本操作，则必须提供以下输入。
	
	<table border='1'>
		<tr><th>属性</th><th>值</th></tr>
		<tr><td>Name</td>
			<td>指定脚本操作的名称。</td></tr>
		<tr><td>脚本 URI</td>
			<td>指定调用以自定义群集的脚本的统一资源标识符 (URI)。</td></tr>
		<tr><td>节点类型</td>
			<td>指定在其上运行自定义脚本的节点。你可以选择“所有节点”、“仅限头节点”或“仅限从节点”<b></b><b></b><b></b>。
		<tr><td>Parameters</td>
			<td>根据脚本的需要，指定参数。</td></tr>
	</table>

	你可以添加多个脚本操作，以在群集上安装多个组件。添加脚本后，单击复选标记以开始设置群集。



##<a id="powershell"></a>使用 Azure PowerShell

本部分提供有关如何通过使用 Azure PowerShell 在 HDInsight 中设置 Apache Spark 群集的说明。有关配置工作站以运行 HDInsight Windows Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。有关将 Azure PowerShell 与 HDInsight 配合使用的详细信息，请参阅[使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]。有关 HDInsight Windows PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][hdinsight-powershell-reference]。

> [AZURE.NOTE]虽然本部分中的脚本可用于在 Azure 虚拟网络上配置 HDInsight 群集，但它们不能用于创建 Azure 虚拟网络。有关创建 Azure 虚拟网络的信息，请参阅[虚拟网络配置任务](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx)。

通过使用 Azure PowerShell 设置 HDInsight 群集需要执行以下过程：

- 创建 Azure 存储帐户
- 创建 Azure Blob 容器
- 创建 HDInsight 群集

你可以使用 Windows PowerShell 控制台或 Windows PowerShell 集成脚本环境 (ISE) 来运行脚本。
 
HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。存储帐户必须与 HDInsight 群集位于同一数据中心。

**连接到 Azure 帐户**

		Add-AzureAccount 

系统将提示你输入 Azure 帐户凭据。

**创建 Azure 存储帐户**

		$storageAccountName = "<StorageAcccountName>"	# Provide a Storage account name
		$location = "<MicrosoftDataCenter>"				# For example, "West US"

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
		$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName
		                                       -StorageAccountKey $storageAccountKey  

		# Create a Blob storage container
		New-AzureStorageContainer -Name $containerName -Context $destContext

准备好存储帐户和 Blob 容器后，你就可以创建群集了。

**设置 HDInsight 群集**

- 在 Azure PowerShell 控制台窗口中运行以下命令：

		$subscriptionName = "<AzureSubscriptionName>"	  # The Azure subscription used for the HDInsight cluster to be created

		$storageAccountName = "<AzureStorageAccountName>" # HDInsight cluster requires an existing Azure Storage account to be used as the default file system

		$clusterName = "<HDInsightClusterName>"			  # The name for the HDInsight cluster to be created
		$clusterNodes = <ClusterSizeInNodes>              # The number of nodes in the HDInsight cluster
        $credentials = Get-Credential		              # User name and password for the Hadoop account. You will use this account to connect to the cluster and run jobs.
           
        # Get the storage primary key based on the account name
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$containerName = $clusterName				# Azure Blob container that is used as the default file system for the HDInsight cluster

        # The location of the HDInsight cluster. It must be in the same data center as the Storage account.
        $location = Get-AzureStorageAccount -StorageAccountName $storageAccountName | %{$_.Location} 

		# Create a new HDInsight cluster
		New-AzureHDInsightCluster -Name $clusterName -Credential $credentials -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName -ClusterSizeInNodes $clusterNodes -Version "3.2" -ClusterType Spark

	>[AZURE.NOTE]指定的凭据将用于创建群集的 Hadoop 用户帐户。此帐户将用于连接到群集并运行作业。如果你在 Azure 门户中使用“快速创建”选项来设置群集，则默认 Hadoop 用户名为“admin”。不要将此帐户与远程桌面协议 (RDP) 用户帐户相混淆。RDP 用户帐户不能与 Hadoop 用户帐户相同。有关详细信息，请参阅[在 HDInsight 中使用 Azure 管理门户管理 Hadoop 群集][hdinsight-admin-portal]。

	设置群集可能需要几分钟时间。

	![HDI.PS.Provision](./media/hdinsight-apache-spark-provision-clusters/hdi-spark-ps-provisioning.png "在 PowerShell 中设置使用基本配置的 Spark 群集")


**通过使用自定义配置选项设置 HDInsight 群集**

设置群集时，你可以使用其他配置选项，例如，连接到多个 Azure Blob 存储容器，或者对 Hive 和 Oozie 元存储使用 Azure SQL 数据库。这样，你便可以将数据和元数据的生存期与群集的生存期分开。

> [AZURE.NOTE]Windows PowerShell cmdlet 是唯一推荐用于更改 HDInsight 群集中的配置变量的方法。如果对群集进行修补，可能会覆盖通过远程桌面连接到群集时对 Hadoop 配置文件所做的更改。如果对群集进行修补，将保留通过 Azure PowerShell 设置的配置值。

- 从 Windows PowerShell 窗口运行以下命令：

		$subscriptionName = "<Azure subscription>"
		$clusterName = "<cluster name>"
		$clusterNodes = <cluster size in nodes>
		                
		# Get credentials for Hadoop user and Hive/Oozie metastores
		$clusterCredentials = Get-Credential
		$oozieCreds = Get-Credential -Message "Oozie metastore"
		$hiveCreds = Get-Credential -Message "Hive metastore"
		
		# Get default storage account, storage container, and add-on storage account
		$storageAccountName_Default = "<Default storage account name>"
		$containerName_Default = $clusterName
		$storageAccountName_Add1 = "<Add-on storage account name>"
		
		# Get information about Hive and Oozie metastores
		$hiveSQLDatabaseServerName = "<SQLDatabaseServerNameForHiveMetastore>"
		$hiveSQLDatabaseName = "<SQLDatabaseNameForHiveMetastore>"
		$oozieSQLDatabaseServerName = "<SQLDatabaseServerNameForOozieMetastore>""
		$oozieSQLDatabaseName = "<SQLDatabaseNameForOozieMetastore>"

		# Get the virtual network ID and subnet name
		$vnetID = "<AzureVirtualNetworkID>"
		$subNetName = "<AzureVirtualNetworkSubNetName>"

		# Retrieve storage account keys and storage account location
		Select-AzureSubscription $subscriptionName
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$storageAccountKey_Add1 = Get-AzureStorageKey $storageAccountName_Add1 | %{ $_.Primary }
		$location = Get-AzureStorageAccount -StorageAccountName $storageAccountName_Default | %{$_.Location}
		
		# Specify custom configuration and cluster type
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes -ClusterType Spark -VirtualNetworkId $vnetID -SubnetName $subNetName | Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default | Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Add1.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Add1 | Add-AzureHDInsightMetastore -SqlAzureServerName "$hiveSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $hiveSQLDatabaseName -Credential $hiveCreds -MetastoreType HiveMetastore | Add-AzureHDInsightMetastore -SqlAzureServerName "$oozieSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $oozieSQLDatabaseName -Credential $oozieCreds -MetastoreType OozieMetastore
		
		# Provision the cluster with custom configuration                
		New-AzureHDInsightCluster -Name $clusterName -Config $config -Credential $clusterCredentials -Location $location -Version "3.2"

	>[AZURE.NOTE]用于元存储的 Azure SQL Database 必须允许连接到其他 Azure 服务，包括 Azure HDInsight。在 Azure SQL 数据库仪表板的右侧单击服务器名称。这是运行 SQL 数据库实例的服务器。进入服务器视图后，请单击“配置”，单击“Microsoft Azure 服务”对应的“是”，然后单击“保存”。

	设置群集可能需要几分钟时间。

**列出 HDInsight 群集**

- 在 Azure PowerShell 控制台窗口中运行以下命令，以列出 HDInsight 群集并验证是否已成功设置群集：

		Get-AzureHDInsightCluster -Name <ClusterName>

##<a id="sdk"></a>使用 HDInsight .NET SDK
HDInsight .NET SDK 提供 .NET 客户端库，可简化从 .NET 应用程序使用 HDInsight 的操作。

通过使用 SDK 设置 HDInsight 群集时必须执行以下过程：

- 安装 HDInsight .NET SDK
- 创建自签名证书
- 创建控制台应用程序
- 运行应用程序


**安装 HDInsight .NET SDK**

你可以从 [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started) 安装该 SDK 的最新发行版。下一过程中将显示说明。

**创建自签名证书**

创建自签名证书，将其安装到工作站上，然后将其上传到你的 Azure 订阅。有关说明，请参阅[创建自签名证书](http://go.microsoft.com/fwlink/?LinkId=511138)。


**创建 Visual Studio 控制台应用程序**

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
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">CreateHDICluster</td></tr>
</table>

4. 单击“确定”以创建该项目。

5. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。

6. 在控制台中运行下列命令以安装程序包：

		Install-Package Microsoft.WindowsAzure.Management.HDInsight

	此命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7. 在“解决方案资源管理器”中，双击 **Program.cs** 将其打开。

8. 将下列 using 语句添加到文件顶部：

		using System.Security.Cryptography.X509Certificates;
		using Microsoft.WindowsAzure.Management.HDInsight;
		using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;
		using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning.Data;


9. 在 main() 函数中，复制并粘贴以下代码：

        string thumbprint = "<CertificateFriendlyName>";  
        string subscriptionid = "<AzureSubscriptionID>";
        string clustername = "<HDInsightClusterName>";
        string location = "<MicrosoftDataCenter>";
        string storageaccountname = "<AzureStorageAccountName>.blob.core.chinacloudapi.cn";
        string storageaccountkey = "<AzureStorageAccountKey>";
        string containername = "<HDInsightDefaultContainerName>";
        string username = "<HDInsightUsername>";
        string password = "<HDInsightUserPassword>";
        int clustersize = <NumberOfNodesInTheCluster>;

        // Get the certificate object from certificate store by using the friendly name to identify it
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.Thumbprint == thumbprint);

        // Create the Storage account if it doesn't exist

        // Create the container if it doesn't exist.

		// Create an HDInsightClient object
        HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionid), cert);
        var client = HDInsightClient.Connect(creds);

		// Supply the cluster information
        ClusterCreateParametersV2 clusterInfo = new ClusterCreateParametersV2()
		{
		    Name = clustername,
		    Location = location,
		    DefaultStorageAccountName = storageaccountname,
		    DefaultStorageAccountKey = storageaccountkey,
		    DefaultStorageContainer = containername,
		    UserName = username,
		    Password = password,
		    ClusterSizeInNodes = clustersize,
		    Version = "3.2",
		    OSType = OSType.Windows,
			ClusterType = ClusterType.Spark		    
		};


		// Create the cluster
        Console.WriteLine("Creating the HDInsight cluster ...");

        ClusterDetails cluster = client.CreateCluster(clusterInfo);

        Console.WriteLine("Created cluster: {0}.", cluster.ConnectionUrl);
        Console.WriteLine("Press ENTER to continue ...");
        Console.ReadKey();

10. 替换 main() 函数开头的变量。

**运行应用程序**

该应用程序在 Visual Studio 中打开时，按 **F5** 运行该应用程序。控制台窗口应打开并显示应用程序的状态。设置一个 HDInsight 群集可能需要几分钟时间。


##<a id="nextsteps"></a>后续步骤

* 请参阅[使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools)了解如何将使用 HDInsight 中的 Apache Spark 与 BI 工具（如 Power BI 和 Tableau）配合使用。 
* 请参阅[使用 HDInsight 中的 Spark 生成机器学习应用程序](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning)了解如何使用 HDInsight 上的 Apache Spark 生成机器学习应用程序。
* 请参阅[使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming)了解如何使用 HDInsight 上的 Apache Spark 生成机器流式处理应用程序。
* 请参阅[管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager)了解如何使用资源管理器来管理分配给 Spark 群集的资源。


[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx
[hdinsight-hbase-custom-provision]: http://www.windowsazure.cn/documentation/articles/hdinsight-hbase-get-started

[hdinsight-customize-cluster]: /documentation/articles/hdinsight-hadoop-customize-cluster
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started

[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal
[hadoop-hdinsight-intro]: /documentation/articles/hdinsight-hadoop-introduction
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically
[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx
[hdinsight-storm-get-started]: /documentation/articles/hdinsight-storm-getting-started

[azure-management-portal]: https://manage.windowsazure.cn/

[azure-command-line-tools]: /documentation/articles/xplat-cli

[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account

[apache-hadoop]: http://go.microsoft.com/fwlink/?LinkId=510084
[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[hdi-remote]: http://www.windowsazure.cn/documentation/articles/hdinsight-administer-use-management-portal/#rdp


[powershell-install-configure]: /documentation/articles/install-configure-powershell

<!---HONumber=66-->