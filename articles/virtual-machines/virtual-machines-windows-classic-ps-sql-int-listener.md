<properties
    pageTitle="在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器 | Azure"
    description="本教程使用通过经典部署模型创建的资源，并使用内部负载均衡器 (ILB) 在 Azure 中创建 AlwaysOn 可用性组侦听器。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="MikeRayMSFT"
    manager="jhubbard"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="291288a0-740b-4cfa-af62-053218beba77"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="03/01/2017"
    wacn.date="04/27/2017"
    ms.author="MikeRayMSFT" />  


# 在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器
> [AZURE.SELECTOR]
- [内部侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)
- [外部侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-ext-listener/)

## 概述
本主题说明如何使用**内部负载均衡器 \(ILB\)** 为 AlwaysOn 可用性组配置侦听器。

> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

若要为 Resource Manager 模型中的 Always On 可用性组配置 ILB 侦听器，请参阅[在 Azure 中为 Always On 可用性组配置内部负载均衡器](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-int-listener/)。

你的可用性组可以仅包含本地副本或 Azure 副本，也可以跨越本地和 Azure 以实现混合配置。Azure 副本可以位于同一区域，也可以跨越使用多个虚拟网络 \(VNet\) 的多个区域。以下步骤假设你已[配置了一个可用性组](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)但是没有配置侦听器。

## 内部侦听器的准则和限制
请注意有关 Azure 中使用 ILB 的可用性组侦听器的以下准则：

* Windows Server 2008 R2、Windows Server 2012 和 Windows Server 2012 R2 支持可用性组侦听器。
* 每个云服务只支持一个内部可用性组侦听器，因为该侦听器将配置给 ILB，而每个云服务只有一个 ILB。但是，可以创建多个外部侦听器。有关详细信息，请参阅 [Configure an external listener for Always On Availability Groups in Azure](/documentation/articles/virtual-machines-windows-classic-ps-sql-ext-listener/)（在 Azure 中配置 AlwaysOn 可用性组的外部侦听器）。

## 确定侦听器的可访问性
[AZURE.INCLUDE [ag-listener-accessibility](../../includes/virtual-machines-ag-listener-determine-accessibility.md)]

本文重点介绍如何创建使用**内部负载均衡器 \(ILB\)** 的侦听器。如果你需要一个公共/外部侦听器，请参阅本文的另一个版本，其中提供了有关设置[外部侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-ext-listener/)的步骤

## 创建支持直接服务器返回的负载均衡 VM 终结点
对于 ILB，必须先创建内部负载均衡器。以下脚本将执行此操作。

你必须为每个托管 Azure 副本的 VM 创建一个负载均衡的终结点。如果你在多个区域中拥有副本，该区域的每个副本必须位于同一个 VNet 的同一个云服务中。跨越多个 Azure 区域创建可用性组副本需要配置多个 Vnet。有关配置跨 VNet 连接的详细信息，请参阅[配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-howto-vnet-vnet-portal-classic/)。

1. 在 Azure 门户预览中，导航到托管副本的每个 VM 并查看详细信息。
2. 单击每个 VM 的“终结点”选项卡。
3. 验证你想要使用的侦听器终结点“名称”和“公用端口”是否已被使用。在下面的示例中，名称为“MyEndpoint”，端口为“1433”。
4. 在你本地的客户端上，下载并安装[最新的 PowerShell 模块](/downloads/)。
5. 启动 Azure PowerShell。将打开新的 PowerShell 会话，其中加载了 Azure 管理模块。
6. 运行 Get-AzurePublishSettingsFile。此 cmdlet 将你定向到浏览器，以将发布设置文件下载到本地目录。系统可能会提示输入 Azure 订阅的登录凭据。
7. 运行 Import-azurepublishsettingsfile 命令以及你下载发布设置文件的路径：
   
        Import-AzurePublishSettingsFile -PublishSettingsFile <PublishSettingsFilePath>
   
    导入发布设置文件后，便可以在 PowerShell 会话中管理 Azure 订阅。
    
1. 对于 **ILB**，应该分配一个静态 IP 地址。首先，通过运行以下命令检查当前 VNet 配置：
   
        (Get-AzureVNetConfig).XMLConfiguration
2. 先记下包含副本所在虚拟机的子网的 **Subnet** 名称。脚本中的 **$SubnetName** 参数将要使用此值。
3. 然后，记下包含副本所在虚拟机的子网的 **VirtualNetworkSite** 名称和起始 **AddressPrefix**。再通过将这两个值传递给 **Test-AzureStaticVNetIP** 命令并检查 **AvailableAddresses** 来查找可用的 IP 地址。例如，如果 VNet 名为 *MyVNet*，并包含起始自 *172.16.0.128* 的子网地址范围，则以下命令将列出可用地址：
   
        (Test-AzureStaticVNetIP -VNetName "MyVNet"-IPAddress 172.16.0.128).AvailableAddresses
4. 选择一个可用地址，并在以下脚本的 **$ILBStaticIP** 参数中使用它。
5. 将以下 PowerShell 脚本复制到文本编辑器中，并根据你的环境设置变量值（注意，某些参数已提供默认值）。请注意，使用地缘组的现有部署不能添加 ILB。有关 ILB 要求的详细信息，请参阅[内部负载均衡器](/documentation/articles/load-balancer-internal-overview/)。此外，如果可用性组跨 Azure 区域，则你必须在每个数据中心内对其中的云服务和节点运行该脚本一次。
   
        # Define variables
        $ServiceName = "<MyCloudService>" # the name of the cloud service that contains the availability group nodes
        $AGNodes = "<VM1>","<VM2>","<VM3>" # all availability group nodes containing replicas in the same cloud service, separated by commas
        $SubnetName = "<MySubnetName>" # subnet name that the replicas use in the VNet
        $ILBStaticIP = "<MyILBStaticIPAddress>" # static IP address for the ILB in the subnet
        $ILBName = "AGListenerLB" # customize the ILB name or use this default value
   
        # Create the ILB
        Add-AzureInternalLoadBalancer -InternalLoadBalancerName $ILBName -SubnetName $SubnetName -ServiceName $ServiceName -StaticVNetIPAddress $ILBStaticIP
   
        # Configure a load balanced endpoint for each node in $AGNodes using ILB
        ForEach ($node in $AGNodes)
        {
            Get-AzureVM -ServiceName $ServiceName -Name $node | Add-AzureEndpoint -Name "ListenerEndpoint" -LBSetName "ListenerEndpointLB" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ILBName -DirectServerReturn $true | Update-AzureVM
        }
6. 设置变量后，将脚本从文本编辑器复制到 Azure PowerShell 会话中运行。如果提示符仍然显示 \>\>，请再次按 Enter，以确保脚本开始运行。注意：

> [AZURE.NOTE]
Azure 经典管理门户目前不支持内部负载均衡器，因此在 Azure 经典管理门户中看不到 ILB 或终结点。但是，如果负载均衡器在某个内部 IP 地址上运行，则 **Get-AzureEndpoint** 将返回该地址。否则，将返回 null。
> 
> 

## 如果需要，请验证是否已安装 KB2854082
[AZURE.INCLUDE [kb2854082](../../includes/virtual-machines-ag-listener-kb2854082.md)]

## 在可用性组节点中打开防火墙端口
[AZURE.INCLUDE [防火墙](../../includes/virtual-machines-ag-listener-open-firewall.md)]

## 创建可用性组侦听器

通过两个步骤创建可用性组侦听器。首先，创建客户端接入点群集资源并配置依赖关系。其次，使用 PowerShell 配置群集资源。

### 创建客户端接入点并配置群集依赖关系
[AZURE.INCLUDE [防火墙](../../includes/virtual-machines-ag-listener-create-listener.md)]

### 在 PowerShell 中配置群集资源
1. 对于 ILB，必须使用前面创建的内部负载均衡器 \(ILB\) 的 IP 地址。在 PowerShell 中使用以下脚本获取此 IP 地址。
   
        # Define variables
        $ServiceName="<MyServiceName>" # the name of the cloud service that contains the AG nodes
        (Get-AzureInternalLoadBalancer -ServiceName $ServiceName).IPAddress
2. 在某个 VM 上，将操作系统的 PowerShell 脚本复制到文本编辑器中，将变量设置为之前记下的值。
   
    对于 Windows Server 2012 或更高版本，请使用以下脚本：
   
        # Define variables
        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP Address resource name
        $ILBIP = "<X.X.X.X>" # the IP Address of the Internal Load Balancer (ILB)
   
        Import-Module FailoverClusters
   
        Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   
    对于 Windows Server 2008 R2，请使用以下脚本：
   
        # Define variables
        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP Address resource name
        $ILBIP = "<X.X.X.X>" # the IP Address of the Internal Load Balancer (ILB)
   
        Import-Module FailoverClusters
   
        cluster res $IPResourceName /priv enabledhcp=0 address=$ILBIP probeport=59999  subnetmask=255.255.255.255
3. 设置变量之后，打开提升的 Windows PowerShell 窗口，然后从文本编辑器复制脚本，并将其粘贴到 Azure PowerShell 会话中运行。如果提示符仍然显示 \>\>，请再次按 Enter，以确保脚本开始运行。
4. 在每个 VM 上重复此过程。此脚本将使用云服务的 IP 地址来配置 IP 地址资源，同时设置探测端口等其他参数。在 IP 地址资源联机后，它可以响应我们在本教程前面部分创建的负载均衡终结点在探测端口上的轮询。

## 使侦听器联机
[AZURE.INCLUDE [Bring-Listener-Online](../../includes/virtual-machines-ag-listener-bring-online.md)]

## 后续操作项
[AZURE.INCLUDE [Follow-up](../../includes/virtual-machines-ag-listener-follow-up.md)]

## 测试可用性组侦听器（在同一 VNet 中）
[AZURE.INCLUDE [Test-Listener-Within-VNET](../../includes/virtual-machines-ag-listener-test.md)]

## 后续步骤
[AZURE.INCLUDE [Listener-Next-Steps](../../includes/virtual-machines-ag-listener-next-steps.md)]

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->