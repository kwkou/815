<properties
    pageTitle="使用虚拟网络扩展 HDInsight | Microsoft Azure"
    description="了解如何使用 Azure 虚拟网络将 HDInsight 连接到其他云资源或数据中心内的资源"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="37b9b600-d7f8-4cb1-a04a-0b3a827c6dcc"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/13/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 使用 Azure 虚拟网络扩展 HDInsight 功能
使用 Azure 虚拟网络可扩展 Hadoop 解决方案以合并本地资源（如 SQL Server）、组合多种 HDInsight 群集类型或者在云中资源之间创建安全专用网络。

## 先决条件

* Azure CLI 2.0（预览版）：有关详细信息，请参阅[安装和配置 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)。

    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

* Azure PowerShell：有关详细信息，请参阅[安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

> [AZURE.NOTE]
本文档中的步骤需要最新版本的 Azure CLI 和 Azure PowerShell。如果你使用的是较旧版本，则命令可能会有所不同。为获得最佳结果，请使用前面的链接安装最新版本。

## <a id="whatis"></a>Azure 虚拟网络是什么？

[Azure 虚拟网络](/documentation/services/networking/)可创建包含解决方案所需资源的安全持久网络。通过虚拟网络可以：

* 在专用网络（仅限云）中将云资源连接在一起。
  
    ![仅限云配置示意图](./media/hdinsight-extend-hadoop-virtual-network/cloud-only.png)
  
    使用虚拟网络链接 Azure 服务与 HDInsight 可实现以下方案：
  
    * 从 Azure 虚拟机中运行的 Azure 网站或服务**调用 HDInsight 服务或作业**。
    * 在 HDInsight 和 Azure SQL 数据库，SQL Server 或在虚拟机上运行的其他数据存储解决方案之间**直接传输数据**。
    * **组合多个 HDInsight 服务器**以构成单个解决方案。HDInsight 群集有各种类型，分别与针对其优化群集的工作负荷或技术相对应。没有任何方法支持创建组合多种类型的群集，如一个群集同时具有 Storm 和 HBase 类型。使用虚拟网络允许多个群集直接相互通信。

* 使用虚拟专用网络 (VPN) 将云资源连接到本地数据中心网络（站点到站点，或点到站点）。
  
    利用站点到站点配置，可使用硬件 VPN 或路由和远程访问服务将多个资源从数据中心连接到 Azure 虚拟网络。
  
    ![站点到站点配置示意图](./media/hdinsight-extend-hadoop-virtual-network/site-to-site.png)
  
    利用点到站点配置，可使用软件 VPN 将特定资源连接到 Azure 虚拟网络。
  
    ![点到站点配置示意图](./media/hdinsight-extend-hadoop-virtual-network/point-to-site.png)
  
    使用虚拟网络链接云和数据中心，可使类似方案实现仅限云的配置。但是，如果不希望仅限于使用云中的资源，还可以使用数据中心内的资源。
  
    * 在 HDInsight 与数据中心之间**直接传输数据**。例如，使用 Sqoop 在 SQL Server 往返传输数据，或读取业务线 (LOB) 应用程序生成的数据。
    * 从 LOB 应用程序**调用 HDInsight 服务或作业**。例如，使用 HBase Java API 存储和检索 HDInsight HBase 群集的数据。

有关虚拟网络特性、优势和功能的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。

> [AZURE.NOTE]
在预配 HDInsight 群集前，必须创建 Azure 虚拟网络。有关详细信息，请参阅[虚拟网络配置任务](/documentation/services/networking/)。

## 虚拟网络要求

> [AZURE.IMPORTANT]
在虚拟网络上创建 HDInsight 群集，需要本节中所述的特定虚拟网络配置。

### 基于位置的虚拟网络

Azure HDInsight 仅支持基于位置的虚拟网络，目前无法处理基于地缘组的虚拟网络。

### 经典或 V2 虚拟网络

基于 Windows 的群集需要经典虚拟网络，而基于 Linux 的群集需要 Azure Resource Manager 虚拟网络。如果没有正确的网络类型，则在创建群集时无法使用网络。

如果计划创建的群集无法使用虚拟网络上的资源，可以创建群集使用的虚拟网络，并将其连接到不兼容的虚拟网络。然后，可以在所需的网络版本中创建群集，并且由于两个网络已联接，因此群集可以访问其他网络中的资源。有关如何连接经典虚拟网络和新虚拟网络的详细信息，请参阅[将经典 VNet 连接到新的 VNet](/documentation/articles/vpn-gateway-connect-different-deployment-models-portal/)。

### 自定义 DNS

创建虚拟网络时，Azure 将为网络中安装的 Azure 服务（如 HDInsight）提供默认名称解析。但是，对于跨网络域名解析等情况，可能需要使用自己的域名系统 (DNS)。例如，在位于两个联接的虚拟网络中的服务之间通信时。与 Azure 虚拟网络配合使用时，HDInsight 同时支持默认 Azure 名称解析和自定义 DNS。

有关如何配合 Azure 虚拟网络使用 DNS 服务器的详细信息，请参阅 [VM 和角色实例的名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/#name-resolution-using-your-own-dns-server)文档中“使用 DNS 服务器的名称解析”一节。

### 受保护的虚拟网络

HDInsight 服务是托管服务，需要在预配期间和运行时访问 Internet。这样，Azure 可以监视群集的运行状况、启动群集资源故障转移、通过缩放操作更改群集中的节点数，以及执行其他管理任务。

如果需要将 HDInsight 安装到受保护的虚拟网络，必须允许以下 IP 地址通过端口 443 进行入站访问，从而允许 Azure 管理 HDInsight 群集。

<!-- need to be verified -->


* 168\.61.49.99
* 23\.99.5.239
* 168\.61.48.131
* 138\.91.141.162

允许这些地址通过端口 443 进行入站访问，可以成功将 HDInsight 安装到受保护的虚拟网络。

> [AZURE.IMPORTANT]
HDInsight 不支持限制出站流量，仅限制入站流量。在定义包含 HDInsight 的子网的网络安全组规则时，仅使用入站规则。

下例演示如何新建允许所需地址的网络安全组，并将该安全组应用到虚拟网络中的子网。

以下步骤假设已创建要安装 HDInsight 的虚拟网络和子网。

> [AZURE.IMPORTANT]
请注意这些示例中使用的 `priority` 值；系统根据优先级按顺序针对网络流量测试规则。一旦某个规则与测试条件匹配并进行了应用，将不再测试其他规则。
><p> 
> 如果用户的自定义规则（例如 **deny all** 规则）大范围阻止了入站流量，则可能需要调整这些示例或自定义规则中的优先级值，使得示例中的规则出现在阻止访问的规则之前。否则会先测试 **deny all** 规则，示例中的规则就得不到应用。还需注意不要阻止 Azure 虚拟网络的默认规则。例如，不应让创建的 **deny all** 规则应用在默认的 **ALLOW VNET INBOUND** 规则（优先级为 65000）前面。
><p> 
> 如需详细了解规则应用方式以及默认的入站和出站规则，请参阅[什么是网络安全组？](/documentation/articles/virtual-networks-nsg/)。

**使用 Azure PowerShell**

    $vnetName = "Replace with your virtual network name"
    $resourceGroupName = "Replace with the resource group the virtual network is in"
    $subnetName = "Replace with the name of the subnet that HDInsight will be installed into"
    # Get the Virtual Network object
    $vnet = Get-AzureRmVirtualNetwork `
        -Name $vnetName `
        -ResourceGroupName $resourceGroupName
    # Get the region the Virtual network is in.
    $location = $vnet.Location
    # Get the subnet object
    $subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
    # Create a new Network Security Group.
    # And add exemptions for the HDInsight health and management services.
    $nsg = New-AzureRmNetworkSecurityGroup `
        -Name "hdisecure" `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -name "hdirule1" `
            -Description "HDI health and management address 168.61.49.99" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.49.99" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 300 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule2" `
            -Description "HDI health and management 23.99.5.239" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "23.99.5.239" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 301 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule3" `
            -Description "HDI health and management 168.61.48.131" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.48.131" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 302 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule4" `
            -Description "HDI health and management 138.91.141.162" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "138.91.141.162" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 303 `
            -Direction Inbound
    # Set the changes to the security group
    Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
    # Apply the NSG to the subnet
    Set-AzureRmVirtualNetworkSubnetConfig `
        -VirtualNetwork $vnet `
        -Name $subnetName `
        -AddressPrefix $subnet.AddressPrefix `
        -NetworkSecurityGroup $nsg

**使用 Azure CLI**

1. 使用以下命令创建名为 `hdisecure` 的新网络安全组。将 **RESOURCEGROUPNAME** 和 **LOCATION** 分别替换为包含 Azure 虚拟网络的资源组以及在其中创建组的位置（区域）。
   
        az network nsg create -g RESOURCEGROUPNAME -n hdisecure -l LOCATION

    创建组后，你将收到有关新组的信息。

2. 使用以下命令将规则添加新网络安全组，以允许从 Azure HDInsight 运行状况和管理服务通过端口 443 发起的入站通信。将 **RESOURCEGROUPNAME** 替换为包含 Azure 虚拟网络的资源组的名称。
    
        az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule1 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.49.99" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 300 --direction "Inbound"
        az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "23.99.5.239" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 301 --direction "Inbound"
        az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule3 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.48.131" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 302 --direction "Inbound"
        az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule4 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "138.91.141.162" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 303 --direction "Inbound"

3. 创建规则后，使用以下命令检索此网络安全组的唯一标识符：

        az network nsg show -g RESOURCEGROUPNAME -n hdisecure --query 'id'

    此命令将返回类似于以下文本的值：

        "/subscriptions/55b1016c-0f27-43d2-b908-b8c373d6d52e/resourceGroups/mygroup/providers/Microsoft.Network/networkSecurityGroups/hdisecure"

4. 使用以下命令将网络安全组应用于子网。将 __GUID__ 和 __RESOURCEGROUPNAME__ 值替换为从上一步骤返回的值。将 __VNETNAME__ 和 __SUBNETNAME__ 替换为创建 HDInsight 群集时要使用的虚拟网络名称和子网名称。
   
        az network vnet subnet update -g RESOURCEGROUPNAME --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
   
    此命令执行完成后，可以将 HDInsight 成功安装到这些步骤中使用的子网上的受保护虚拟网络。

> [AZURE.IMPORTANT]
使用上述步骤只可访问 Azure 云上的 HDInsight 运行状况和管理服务。此操作可成功将 HDInsight 群集安装到子网，但默认阻止从虚拟网络外部访问 HDInsight 群集。如果要启用从虚拟网络外部进行访问，需要添加其他网络安全组规则。
> 
> 例如，若要允许来自 Internet 的 SSH 访问，需要添加类似于下面的规则：
> 
> * Azure PowerShell - ```Add-AzureRmNetworkSecurityRuleConfig -Name "SSSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 304 -Direction Inbound```
> * Azure CLI - ```az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule4 --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 304 --direction "Inbound"```

有关网络安全组的详细信息，请参阅[网络安全组概述](/documentation/articles/virtual-networks-nsg/)。有关在 Azure 虚拟网络中控制路由的详细信息，请参阅[用户定义的路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview/)。

## <a id="tasks"></a>任务和信息

本节包含常见任务的相关信息，以及搭配使用 HDInsight 与虚拟网络时可能需要的信息。

### 确定 FQDN

系统将为 HDInsight 群集分配虚拟网络接口的特定完全限定域名 (FQDN)。在从虚拟网络上的其他资源连接到群集时，应该使用这个地址。若要确定 FQDN，请使用以下 URL 来查询 Ambari 管理服务:

    https://<clustername>.azurehdinsight.cn/ambari/api/v1/clusters/<clustername>.azurehdinsight.cn/services/<servicename>/components/<componentname>

> [AZURE.NOTE]
有关如何将 Ambari 与 HDInsight 配合使用的详细信息，请参阅[使用 Ambari API 监视 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-monitor-use-ambari-api/)。

必须指定群集名称和群集上运行的服务和组件，例如 YARN 资源管理器。

> [AZURE.NOTE]
返回的数据是一个 JavaScript 对象表示法 (JSON) 文档，其中包含有关组件的大量信息。若只要提取 FQDN，你应该使用 JSON 分析器来检索 `host_components[0].HostRoles.host_name` 值。

例如，若要从 HDInsight Hadoop 群集返回 FQDN，可以使用以下其中一种方法检索 YARN 资源管理器的数据：

* [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)
  
        $ClusterDnsName = <clustername>
        $Username = <cluster admin username>
        $Password = <cluster admin password>
        $DnsSuffix = ".azurehdinsight.cn"
        $ClusterFQDN = $ClusterDnsName + $DnsSuffix
  
        $webclient = new-object System.Net.WebClient
        $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)
  
        $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/yarn/        components/resourcemanager"
        $Response = $webclient.DownloadString($Url)
        $JsonObject = $Response | ConvertFrom-Json
        $FQDN = $JsonObject.host_components[0].HostRoles.host_name
        Write-host $FQDN

* [cURL](http://curl.haxx.se/) 和 [jq](http://stedolan.github.io/jq/)
  
        curl -G -u <username>:<password> https://<clustername>.azurehdinsight.cn/ambari/api/v1/clusters/<clustername>.azurehdinsight.cn/services/yarn/components/resourcemanager | jq .host_components[0].HostRoles.host_name

### 连接到 HBase

若要使用 Java API 远程连接到 HBase，必须确定 HBase 群集的 Zookeeper 仲裁地址，并在应用程序中加以指定。

若要获取 Zookeeper 仲裁地址，请使用以下其中一种方法来查询 Ambari 管理服务：

* [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)
  
        $ClusterDnsName = <clustername>
        $Username = <cluster admin username>
        $Password = <cluster admin password>
        $DnsSuffix = ".azurehdinsight.cn"
        $ClusterFQDN = $ClusterDnsName + $DnsSuffix
  
        $webclient = new-object System.Net.WebClient
        $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)
  
        $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum"
        $Response = $webclient.DownloadString($Url)
        $JsonObject = $Response | ConvertFrom-Json
        Write-host $JsonObject.items[0].properties.'hbase.zookeeper.quorum'

* [cURL](http://curl.haxx.se/) 和 [jq](http://stedolan.github.io/jq/)
  
        curl -G -u <username>:<password> "https://<clustername>.azurehdinsight.cn/ambari/api/v1/clusters/<clustername>.azurehdinsight.cn/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum" | jq .items[0].properties[]

> [AZURE.NOTE]
有关如何将 Ambari 与 HDInsight 配合使用的详细信息，请参阅[使用 Ambari API 监视 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-monitor-use-ambari-api/)。

获取仲裁信息后，请将其用于客户端应用程序中。

例如，对于使用 HBase API 的 Java 应用程序，你可以将 **hbase-site.xml** 文件添加到项目，并在此文件中指定仲裁信息，如下所示：

    <configuration>
      <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
      </property>
      <property>
        <name>hbase.zookeeper.quorum</name>
        <value>zookeeper0.address,zookeeper1.address,zookeeper2.address</value>
      </property>
      <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
      </property>
    </configuration>

### 验证网络连接

某些服务（如 SQL Server）可能会限制传入网络连接。这将防止 HDInsight 成功使用这些服务。

如果在从 HDInsight 访问服务时遇到问题，请参阅相关服务文档，以确保启用网络访问功能。还可以通过在相同虚拟网络上创建 Azure 虚拟机验证网络访问功能，并使用客户端实用工具验证虚拟机是否可通过虚拟网络连接服务。

## <a id="nextsteps"></a>后续步骤

下例演示如何对 Azure 虚拟网络使用 HDInsight：

* [使用 HDInsight 中的 Storm 和 HBase 分析传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/) - 演示如何在虚拟网络中配置 Storm 和 HBase 群集，以及如何从 Storm 将数据远程写入 HBase。
* [在 HDInsight 中设置 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/) - 提供有关预配 Hadoop 群集的信息，包括有关使用 Azure 虚拟网络的信息。
* [将 Sqoop 与 HDinsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-use-sqoop-mac-linux/) - 提供有关使用 Sqoop 通过虚拟网络传输 SQL Server 数据的信息。

要了解有关 Azure 虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。

<!---HONumber=Mooncake_0120_2017-->