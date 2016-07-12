<properties
	pageTitle="使用虚拟网络扩展 HDInsight | Azure"  
	description="了解如何使用 Azure 虚拟网络将 HDInsight 连接到其他云资源或者你数据中心内的资源"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>


#使用 Azure 虚拟网络扩展 HDInsight 功能

Azure 虚拟网络可让你扩展 Hadoop 解决方案以合并本地资源（例如 SQL Server），或者在云中资源之间创建安全的专用网络。

[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell-and-cli.md)]


##<a id="whatis"></a>Azure 虚拟网络是什么？

[Azure 虚拟网络](/documentation/services/networking/)允许你创建包含需要用于解决方案的资源的安全永久性网络。通过虚拟网络，你可以：

* 在专用网络（仅限云）中将云资源连接在一起。

	![仅限云配置示意图](./media/hdinsight-extend-hadoop-virtual-network/cloud-only.png)

	使用虚拟网络链接 Azure 服务与 HDInsight 可实现以下方案：

	* 从 Azure 虚拟机中运行的 Azure 网站或服务**调用 HDInsight 服务或作业**。

	* 在 HDInsight 与 SQL 数据库或 SQL Server 或其他运行于虚拟机的数据存储解决方案之间**直接传输数据**。

	* **组合多个 HDInsight 服务器**以构成单个解决方案。例如，使用 HDInsight Storm 服务器来使用传入数据，然后将经过处理的数据存储到 HDInsight HBase 服务器。原始数据也可以存储到 HDInsight Hadoop 服务器供将来使用 MapReduce 进行分析。

* 使用虚拟专用网络 (VPN) 将云资源连接到本地数据中心网络（站点到站点，或点到站点）。

	利用站点到站点配置，你可以使用硬件 VPN 或路由和远程访问服务将多个资源从数据中心连接到 Azure 虚拟网络。

	![站点到站点配置示意图](./media/hdinsight-extend-hadoop-virtual-network/site-to-site.png)

	利用点到站点配置，你可以使用软件 VPN 将特定资源连接到 Azure 虚拟网络。

	![点到站点配置示意图](./media/hdinsight-extend-hadoop-virtual-network/point-to-site.png)

	使用虚拟网络链接云和数据中心可让类似方案在仅限云的配置上实现。但是，如果不想受限于使用云中的资源，你也可以使用数据中心内的资源。

	* 在 HDInsight 与数据中心之间**直接传输数据**。例如，使用 Sqoop 在 SQL Server 往返传输数据，或读取业务线 (LOB) 应用程序生成的数据。

	* 从 LOB 应用程序**调用 HDInsight 服务或作业**。例如，使用 HBase Java API 来存储和检索 HDInsight HBase 群集的数据。

有关虚拟网络特性、优势和功能的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。

> [AZURE.NOTE] 你必须创建 Azure 虚拟网络才能设置一个 HDInsight 群集。有关详细信息，请参阅[虚拟网络配置任务](/documentation/services/networking/)。

## 虚拟网络要求

> [AZURE.IMPORTANT] 在虚拟网络上创建 HDInsight 群集需要本部分中所述的特定虚拟网络配置。

###基于位置的虚拟网络

Azure HDInsight 仅支持基于位置的虚拟网络，目前无法处理基于地缘的虚拟网络。

###子网

强烈建议针对每个 HDInsight 群集创建一个子网。

###经典虚拟网络

基于 Windows 的群集需要 V1（经典）虚拟网络。如果没有正确的网络类型，创建群集时它将不能使用。

如果具有计划创建的群集不可用的虚拟网络上的资源，可以新建一个该群集可用的虚拟网络，并将其连接到不兼容的虚拟网络。然后可以在其要求的网络版本中创建群集，由于两者已联接，该群集将能够访问另一个网络中的资源。有关连接经典虚拟网络和新虚拟网络的详细信息，请参阅 [Connecting classic VNets to new VNets（将经典 VNet 连接到新的 VNet）](/documentation/articles/virtual-networks-arm-asm-s2s/)。

###受保护的虚拟网络

HDInsight 服务是一个托管的服务，需要在预配期间和运行时访问 Internet。这样，Azure 便可以监视群集的运行状况、启动群集资源故障转移、通过缩放操作更改群集中的节点数，以及执行其他管理任务。

如果需要将 HDInsight 安装到受保护的虚拟网络，必须允许以下 IP 地址通过端口 443 进行入站访问，使 Azure 能够管理 HDInsight 群集。

* 168\.61.49.99
* 23\.99.5.239
* 168\.61.48.131
* 138\.91.141.162

如果允许这些地址通过端口 443 进行入站访问，则你可以成功地将 HDInsight 安装到受保护的虚拟网络。

以下示例演示如何新建一个允许所需地址的网络安全组，并将该安全组应用于虚拟网络中的子网。这些步骤假定你已创建想要将 HDInsight 安装到其中的虚拟网络和子网。

__使用 Azure PowerShell__

    $vnetName = "Replace with your virtual network name"
    $subnetName = "Replace with the name of the subnet that HDInsight will be installed into"
    # Get the Virtual Network object
    $vnet = Get-AzureVNetSite `
        -VNetName $vnetName 
    # Get the region the Virtual network is in.
    $location = $vnet.Location
    # Create a new Network Security Group.
    # And add exemptions for the HDInsight health and management services.
    $nsg = New-AzureNetworkSecurityGroup `
        -Name "hdisecure" `
        -Location $location `
        | Set-AzureNetworkSecurityRule `
            -name "hdirule1" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.49.99" `
            -DestinationAddressPrefix "*" `
            -Action Allow `
            -Priority 300 `
            -Type Inbound `
        | Set-AzureNetworkSecurityRule `
            -Name "hdirule2" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "23.99.5.239" `
            -DestinationAddressPrefix "*" `
            -Action Allow `
            -Priority 301 `
            -Type Inbound `
        | Set-AzureNetworkSecurityRule `
            -Name "hdirule3" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.48.131" `
            -DestinationAddressPrefix "*" `
            -Action Allow `
            -Priority 302 `
            -Type Inbound `
        | Set-AzureNetworkSecurityRule `
            -Name "hdirule4" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "138.91.141.162" `
            -DestinationAddressPrefix "*" `
            -Action Allow `
            -Priority 303 `
            -Type Inbound
    # Apply the NSG to the subnet
    Set-AzureNetworkSecurityGroupAssociation `
        -VirtualNetworkName $vnetName `
        -SubnetName $subnetName `
        -Name $nsg.Name

__使用 Azure CLI__

1. 使用以下命令创建名为 `hdisecure` 的新网络安全组。将 __LOCATION__ 替换为 Azure 虚拟网络的位置（区域）。

        azure network nsg create hdisecure LOCATION

2. 使用以下命令将规则添加新的网络安全组，这些规则允许从 Azure HDInsight 运行状况和管理服务通过端口 443 发起的入站通信。

        azure network nsg rule create hdisecure hdirule1 -p "*" -o "*" -u "443" -f "168.61.49.99" -e "*" -c "Allow" -y 300 -r "Inbound"
        azure network nsg rule create hdisecure hdirule2 -p "*" -o "*" -u "443" -f "23.99.5.239" -e "*" -c "Allow" -y 301 -r "Inbound"
        azure network nsg rule create hdisecure hdirule3 -p "*" -o "*" -u "443" -f "168.61.48.131" -e "*" -c "Allow" -y 302 -r "Inbound"
        azure network nsg rule create hdisecure hdirule4 -p "*" -o "*" -u "443" -f "138.91.141.162" -e "*" -c "Allow" -y 303 -r "Inbound"

3. 创建规则后，使用以下命令将新网络安全组应用到子网。将 __VNETNAME__ 和 __SUBNETNAME__ 分别替换为 Azure 虚拟网络的名称以及在安装 HDInsight 时要使用的子网。

        azure network nsg subnet add hdisecure VNETNAME SUBNETNAME

    此命令完成之后，你可以将 HDInsight 成功安装到这些步骤中使用的子网上的受保护虚拟网络。

> [AZURE.IMPORTANT] 使用上述步骤只会实现对 Azure 云中 HDInsight 运行状况和管理服务的访问。这一操作让你能够成功地将 HDInsight 群集安装到子网，但默认阻止从虚拟网络外部对 HDInsight 群集进行访问。如果想要启用从虚拟网络外部进行访问，将需要添加其他网络安全组规则。
><p> 例如，若要允许来自 Internet 的 RDP 访问，需要添加类似于下面的规则：
><p> * Azure PowerShell - ```Set-AzureNetworkSecurityRule -Name "RDP" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "3389" -SourceAddressPrefix "*" -DestinationAddressPrefix "*" -Action Allow -Priority 304 -Type Inbound```
><p> * Azure CLI - ```azure network nsg rule create hdisecure RDP -p "*" -o "*" -u "3389" -f "*" -e "*" -c "Allow" -y 304 -r "Inbound"```

有关网络安全组的详细信息，请参阅 [Network Security Groups overview（网络安全组概述）](/documentation/articles/virtual-networks-nsg/)。有关在 Azure 虚拟网络中控制路由的详细信息，请参阅 [User Defined Routes and IP forwarding（用户定义的路由和 IP 转发）。](/documentation/articles/virtual-networks-udr-overview/)

##<a id="tasks"></a>任务和信息

本部分包含常见任务信息和搭配使用 HDInsight 与虚拟网络时可能需要的信息。

###验证网络连接

某些服务，例如 SQL Server，可能会限制传入的网络连接。这会使得 HDInsight 无法成功使用这些服务。

如果你在从 HDInsight 访问服务时遇到问题，请参阅相关服务文档，以确保启用网络访问功能。你也可以通过在相同虚拟网络上创建 Azure 虚拟机来验证网络访问功能，并使用客户端实用工具来验证虚拟机是否可以通过虚拟网络连接服务。

##<a id="nextsteps"></a>后续步骤

以下示例演示了如何对 Azure 虚拟网络使用 HDInsight：

* [使用 HDInsight 中的 Storm 和 HBase 分析传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/) - 演示如何在虚拟网络中配置 Storm 和 HBase 群集，以及如何从 Storm 将数据远程写入到 HBase。

* [在 HDInsight 中设置 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/) - 提供有关设置 Hadoop 群集的信息，包括有关使用 Azure 虚拟网络的信息。

* [将 Sqoop 与 HDinsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-use-sqoop/) - 提供有关使用 Sqoop 通过虚拟网络传输 SQL Server 数据的信息。

若要了解有关 Azure 虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。

<!---HONumber=Mooncake_0530_2016-->