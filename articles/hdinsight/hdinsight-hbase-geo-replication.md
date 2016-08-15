<properties 
   pageTitle="在两个 Azure 数据中心之间配置 HBase 复制 | Azure" 
   description="了解如何在两个 Azure 虚拟网络之间配置 VPN 连接，如何在两个虚拟网络之间配置域名解析，以及如何配置 HBase 地域复制" 
   services="hdinsight" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/04/2016"
	wacn.date="05/24/2016"/>

# 在 HDInsight 中配置 HBase 地域复制

- [配置 VPN 连接](/documentation/articles/hdinsight-hbase-geo-replication-configure-VNets/)
- [配置 DNS](/documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/)
- [配置 HBase 复制](/documentation/articles/hdinsight-hbase-geo-replication/) 
 
了解如何跨两个数据中心配置 HBase 复制。群集复制的部分用例包括：

- 备份和灾难恢复
- 数据聚合
- 地理数据分布
- 联机数据引入并结合脱机数据分析

群集复制使用源推送方法。HBase 群集可以是一个源、目标，也可以同时充当这两个角色。复制是异步的，复制的目标是保持最终一致性。当源接收到对列系列的编辑并启用复制时，该编辑将传播到所有目标群集。当将数据从一个群集复制到另一个群集，将会跟踪源群集和所有已使用数据的群集，以防止复制循环。在本教程中，你将要配置源-目标复制。对于其他群集拓扑，请参阅 [Apache HBase 参考指南](http://hbase.apache.org/book.html#_cluster_replication)。

这是系列教程的第三个部分：

- [在两个虚拟网络之间配置 VPN 连接][hdinsight-hbase-replication-vnet]
- [为虚拟网络配置 DNS][hdinsight-hbase-replication-dns]
- 配置 HBase 地域复制（本教程）

下图演示了你[在配置两个虚拟网络之间的 VPN 连接][hdinsight-hbase-geo-replication-vnet]和[为虚拟网络配置 DNS][hdinsight-hbase-replication-dns]中创建的两个虚拟网络和网络连接：

![HDInsight HBase 复制虚拟网络示意图][img-vnet-diagram]

##<a id="prerequisites"></a>先决条件
在开始阅读本教程前，你必须具有：

- **一个 Azure 订阅**。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅 [购买选项][azure-purchase-options]、[会员优惠][azure-member-offers] 或 [免费试用][azure-trial]。

- **已安装并已配置 Azure PowerShell 的工作站**。

	若要执行 PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为 *RemoteSigned*。请参阅 [使用 Set-ExecutionPolicy cmdlet][2]。

	[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

- **已配置 VPN 连接和 DNS 的两个 Azure 虚拟网络**。有关说明，请参阅[在两个 Azure 虚拟网络之间配置 VPN 连接][hdinsight-hbase-replication-vnet]和[在两个 Azure 虚拟网络之间配置 DNS][hdinsight-hbase-replication-dns]。


	在运行 PowerShell 脚本之前，请确保你已使用以下 cmdlet 连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	如果你有多个 Azure 订阅，请使用以下 cmdlet 来设置当前订阅：

		Select-AzureSubscription <AzureSubscriptionName>



##在 HDInsight 中设置 HBase 群集

在[在两个 Azure 虚拟网络之间配置 VPN 连接][hdinsight-hbase-replication-vnet]中，你已分别在欧洲数据中心和美国数据中心各创建了一个虚拟网络。这两个虚拟网络将通过 VPN 连接。在本课中，你将要在其中的每个虚拟网络中设置 HBase 群集。在本教程的后面，你将使其中一个 HBase 群集复制另一个 HBase 群集。

Azure 经典管理门户不支持使用自定义配置选项设置 HDInsight 群集。例如，将 *hbase.replication* 设置为 *true*。如果设置群集后在配置文件中设置该值，你将在重新创建群集映像后丢失该设置。有关详细信息，请参阅[在 HDInsight 中预配 Hadoop 群集][hdinsight-provision]。使用自定义选项设置 HDInsight 群集的选项之一是使用 Azure PowerShell。


**在 Contoso-VNet-CN 中预配 HBase 群集**

1. 在工作站上打开 Windows PowerShell ISE。
2. 设置脚本开头位置的变量，然后运行该脚本。

		# create hbase cluster with replication enabled
		
		$azureSubscriptionName = "[AzureSubscriptionName]"
		
		$hbaseClusterName = "Contoso-HBase-EU" # This is the HBase cluster name to be used.
		$hbaseClusterSize = 1   # You must provision a cluster that contains at least 3 nodes for high availability of the HBase services.
		$hadoopUserLogin = "[HadoopUserName]"
		$hadoopUserpw = "[HadoopUserPassword]"
		
		$vNetName = "Contoso-VNet-CN"  # This name must match your Europe virtual network name.
		$subNetName = 'Subnet-1' # This name must match your Europe virtual network subnet name.  The default name is "Subnet-1".
		
		$storageAccountName = 'ContosoStoreEU'  # The script will create this storage account for you.  The storage account name doesn't support hyphens. 
		$storageAccountName = $storageAccountName.ToLower() #Storage account name must be in lower case.
		$blobContainerName = $hbaseClusterName.ToLower()  #Use the cluster name as the default container name.
		
		#connect to your Azure subscription
		Add-AzureAccount -Environment AzureChinaCloud 
		Select-AzureSubscription $azureSubscriptionName
		
		# Create a storage account used by the HBase cluster
		$location = Get-AzureVNetSite -VNetName $vNetName | %{$_.Location} # use the virtual network location
		New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location
		
		# Create a blob container used by the HBase cluster
		$storageAccountKey = Get-AzureStorageKey -StorageAccountName $storageAccountName | %{$_.Primary}
		$storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
		New-AzureStorageContainer -Name $blobContainerName -Context $storageContext
		
		# Create provision configuration object
		$hbaseConfigValues = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightHBaseConfiguration'
		    
		$hbaseConfigValues.Configuration = @{ "hbase.replication"="true" } #this modifies the hbase-site.xml file
		
		# retrieve vnet id based on vnetname
		$vNetID = Get-AzureVNetSite -VNetName $vNetName | %{$_.id}
		
		$config = New-AzureHDInsightClusterConfig `
		                -ClusterSizeInNodes $hbaseClusterSize `
		                -ClusterType HBase `
		                -VirtualNetworkId $vNetID `
		                -SubnetName $subNetName `
		            | Set-AzureHDInsightDefaultStorage `
		                -StorageAccountName $storageAccountName `
		                -StorageAccountKey $storageAccountKey `
		                -StorageContainerName $blobContainerName `
		            | Add-AzureHDInsightConfigValues `
		                -HBase $hbaseConfigValues
		
		# provision HDInsight cluster
		$hadoopUserPassword = ConvertTo-SecureString -String $hadoopUserpw -AsPlainText -Force     
		$credential = New-Object System.Management.Automation.PSCredential($hadoopUserLogin,$hadoopUserPassword)
		
		$config | New-AzureHDInsightCluster -Name $hbaseClusterName -Location $location -Credential $credential



**在 Contoso-VNet-CE 中预配 HBase 群集**

- 使用包含以下值的同一个脚本：

		$hbaseClusterName = "Contoso-HBase-US" # This is the HBase cluster name to be used.
		$vNetName = "Contoso-VNet-CE"  # This name must match your Europe virtual network name.
		$storageAccountName = 'ContosoStoreUS'	

	由于你已连接到 Azure 帐户，因此不再需要运行以下 cmdlet：

		Add-AzureAccount -Environment AzureChinaCloud 
		Select-AzureSubscription $azureSubscriptionName




# 配置 DNS 条件转发器

在[为虚拟网络配置 DNS][hdinsight-hbase-replication-dns] 中，你已经为两个网络配置了 DNS 服务器。HBase 群集具有不同的域后缀。因此，你需要配置其他 DNS 条件转发器。

若要配置条件转发器，你需要知道两个 HBase 群集的域后缀。

**查找两个 HBase 群集的域后缀**

1. 通过 RDP 访问 **Contoso-HBase-EU**。有关说明，请参阅[使用 Azure 经典管理门户在 HDInsight 中管理 Hadoop 群集][hdinsight-manage-portal]。它实际上是群集的 headnode0。
2. 打开 Windows PowerShell 控制台或命令提示符。
3. 运行 **ipconfig**，并记下**特定于连接的 DNS 后缀**。
4. 请不要关闭 RDP 会话。稍后你要使用它来测试域名解析。
5. 重复相同的步骤，以找出 **Contoso-HBase-US** 的**特定于连接的 DNS 后缀**。


**配置 DNS 转发器**
 
1.	通过 RDP 访问 **Contoso-DNS-CN**。 
2.	单击左下角的 Windows 键。
2.	单击“管理工具”。
3.	单击“DNS”。
4.	在左窗格中，展开“DSN”、“Contoso-DNS-CN”。
5.	右键单击“条件转发器”，然后单击“新建条件转发器”。 
5.	输入以下信息：
	- **DNS 域**：输入 Contoso-HBase-US 的 DNS 后缀。示例：Contoso-HBase-US.f5.internal.chinacloudapp.cn.
	- **主服务器的 IP 地址**：输入 10.2.0.4，这是 Contoso-DNS-CE 的 IP 地址。请验证该 IP。你的 DNS 服务器可以具有不同的 IP 地址。
6.	按 **ENTER**，然后单击“确定”。现在，你可以从 Contoso-DNS-CN 解析 Contoso-DNS-CE 的 IP 地址。
7.	重复上述步骤，以使用以下值将 DNS 条件转发器添加到 Contoso-DNS-CE 虚拟机上的 DNS 服务：
	- **DNS 域**：输入 Contoso-HBase-EU 的 DNS 后缀。 
	- **主服务器的 IP 地址**：输入 10.2.0.4，这是 Contoso-DNS-CN 的 IP 地址。

**测试域名解析**

1. 切换到 Contoso-HBase-EU RDP 窗口。
2. 打开命令提示符。
3. 运行 ping 命令：

		ping headnode0.[DNS suffix of Contoso-HBase-US]

	HBase 群集的辅助节点将打开 ICM 协议

4. 请不要关闭 RDP 会话。本教程后面的步骤中仍会用到它。
5. 重复相同的步骤，以从 Contoso-HBase-US ping Contoso-HBase-EU 的 headnode0。

>[AZURE.IMPORTANT]在继续后面的步骤之前，DNS 必须正常工作。

##在 HBase 表之间启用复制

现在，你可以创建示例 HBase 表，启用复制，然后使用一些数据对其进行测试。要使用的示例表包含两个列系列：“个人”和“办公”。

在本教程中，你要将欧洲 HBase 群集用作源群集，将美国HBase 群集用作目标群集。

在源和目标群集上创建具有相同名称和列系列的 HBase 表，使目标群集知道在何处存储它要接收的数据。有关使用 HBase shell 的详细信息，请参阅 [HDInsight 中的 Apache HBase 入门][hdinsight-hbase-get-started]。

**在 Contoso-HBase-EU 中创建 HBase 表**

1. 切换到 **Contoso-HBase-EU** RDP 窗口。
2. 在桌面上单击“Hadoop 命令行”。
2. 将文件夹更改为 HBase 主目录：

		cd %HBASE_HOME%\bin
3. 打开 HBase shell：

		hbase shell
4. 创建 HBase 表：

		create 'Contacts', 'Personal', 'Office'
5. 请不要关闭 RDP 会话和 Hadoop 命令行窗口。本教程后面的步骤中仍会用到它们。
	
**在 Contoso-HBase-US 中创建 HBase 表**

- 重复相同的步骤，在 Contoso-HBase-US 中创建相同的表。


**将 Contoso-HBase-US 添加为复制对等方**

1. 切换到 **Contoso-HBase-EU** RDP 窗口。
2. 在 HBase shell 窗口中，添加目标群集 (Contoso-HBase-US) 作为对等方，例如：

		add_peer '1', 'zookeeper0.contoso-hbase-us.d4.internal.chinacloudapp.cn,zookeeper1.contoso-hbase-us.d4.internal.chinacloudapp.cn,zookeeper2.contoso-hbase-us.d4.internal.chinacloudapp.cn:2181:/hbase'

	在示例中，域后缀是 *contoso-hbase us.d4.internal.chinacloudapp.cn*。你需要更新它，以匹配美国 HBase 群集的域后缀。确保主机名之间没有空格。

**配置要在源群集上复制的每个列系列**

1. 从 **Contso-HBase-EU** RDP 会话的 HBase shell 窗口中，配置要复制的每个列系列：

		disable 'Contacts'
		alter 'Contacts', {NAME => 'Personal', REPLICATION_SCOPE => '1'}
		alter 'Contacts', {NAME => 'Office', REPLICATION_SCOPE => '1'}
		enable 'Contacts'

**将数据批量上载到 HBase 表**

已使用以下 URL 将一个示例数据文件上载到公共 Azure Blob 容器：

		wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt

文件的内容：

		8396	Calvin Raji	230-555-0191	5415 San Gabriel Dr.
		16600	Karen Wu	646-555-0113	9265 La Paz
		4324	Karl Xie	508-555-0163	4912 La Vuelta
		16891	Jonathan Jackson	674-555-0110	40 Ellis St.
		3273	Miguel Miller	397-555-0155	6696 Anchor Drive
		3588	Osarumwense Agbonile	592-555-0152	1873 Lion Circle
		10272	Julia Lee	870-555-0110	3148 Rose Street
		4868	Jose Hayes	599-555-0171	793 Crawford Street
		4761	Caleb Alexander	670-555-0141	4775 Kentucky Dr.
		16443	Terry Chander	998-555-0171	771 Northridge Drive

可以将同一个数据文件上载到 HBase 群集中，并从那里导入数据。

1. 切换到 **Contoso-HBase-EU** RDP 窗口。
2. 在桌面上单击“Hadoop 命令行”。
3. 将文件夹更改为 HBase 主目录：

		cd %HBASE_HOME%\bin

4. 上载数据：

		hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns="HBASE_ROW_KEY,Personal:Name, Personal:HomePhone, Office:Address" -Dimporttsv.bulk.output=/tmpOutput Contacts wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt

		hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /tmpOutput Contacts


##验证数据复制是否正在进行

你可以通过使用以下 HBase shell 命令扫描两个群集中的表，来验证复制是否正在进行：

		Scan 'Contacts'


##后续步骤

在本教程中，你已学习如何跨两个数据中心配置 HBase 复制。若要了解有关 HDInsight 和 HBase 的详细信息，请参阅：

- [HDInsight 服务页](/home/features/hdinsight/)
- [HDInsight 文档](/documentation/services/hdinsight/)
- [开始在 HDInsight 中使用 Apache HBase][hdinsight-hbase-get-started]
- [HDInsight HBase 概述][hdinsight-hbase-overview]
- [在 Azure 虚拟网络上设置 HBase 群集][hdinsight-hbase-provision-vnet-v1]
- [使用 HDInsight (Hadoop) 中的 Storm 和 HBase 分析传感器数据][hdinsight-sensor-data]

[hdinsight-hbase-geo-replication-vnet]: /documentation/articles/hdinsight-hbase-geo-replication-configure-VNets/
<!-- [hdinsight-hbase-geo-replication-dns]: /documentation/articles/hdinsight-hbase-geo-replication-configure-VNet/ -->


[img-vnet-diagram]: ./media/hdinsight-hbase-geo-replication/HDInsight.HBase.Replication.Network.diagram.png


[hdinsight-hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-v1/
[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-hbase-replication-vnet]: /documentation/articles/hdinsight-hbase-geo-replication-configure-VNets/
[hdinsight-hbase-replication-dns]: /documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/
<!-- [hdinsight-hbase-twitter-sentiment]: /documentation/articles/hdinsight-hbase-analyze-twitter-sentiment/ -->
[hdinsight-sensor-data]: /documentation/articles/hdinsight-storm-sensor-data-analysis/
[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview/
[hdinsight-hbase-provision-vnet-v1]: /documentation/articles/hdinsight-hbase-provision-vnet-v1/
[hdinsight-hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-v1/

<!---HONumber=71-->