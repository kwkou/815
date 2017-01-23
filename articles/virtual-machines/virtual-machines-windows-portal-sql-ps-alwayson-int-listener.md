<properties
    pageTitle="配置 Always On 可用性组侦听器 - Azure"
    description="使用具有一个或多个 IP 地址的内部负载均衡器在 Azure Resource Manager 模型中配置可用性组侦听器。"
    services="virtual-machines"
    documentationcenter="na"
    author="MikeRayMSFT"
    manager="jhubbard"
    editor="monicar" />
<tags
    ms.assetid="14b39cde-311c-4ddf-98f3-8694e01a7d3b"
    ms.service="virtual-machines"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="11/28/2016"
    wacn.date="01/20/2017"
    ms.author="mikeray" />  


# 配置一个或多个 Always On 可用性组侦听器 - Resource Manager
本主题说明如何执行以下操作：

* 使用 PowerShell cmdlet 为 SQL Server 可用性组创建内部负载均衡器。
* 将其他 IP 地址添加到负载均衡器，以支持多个可用性组。

可用性组侦听器是客户端为了访问数据库而连接的虚拟网络名称。在 Azure 虚拟机上，负载均衡器持有侦听器的 IP 地址。负载均衡器将流量路由到侦听探测端口的 SQL Server 实例。在大多数情况下，可用性组使用内部负载均衡器。Azure 内部负载均衡器可以托管一个或多个 IP 地址。每个 IP 地址使用特定的探测端口。本文档说明如何使用 PowerShell 创建新的负载均衡器，或将 IP 地址添加到 SQL Server 可用性组的现有负载均衡器。

将多个 IP 地址分配到内部负载均衡器是 Azure 的一项新增功能，只能在 Resource Manager 模型中使用。若要完成此任务，需要在 Resource Manager 模型中的 Azure 虚拟机上部署 SQL Server 可用性组。这两个 SQL Server 虚拟机必须属于同一个可用性集。

本主题要求事先配置可用性组。

相关主题包括：

* [在 Azure VM (GUI) 中配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)
* [使用 Azure Resource Manager 和 PowerShell 配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-vm-powershell.md)]

## 配置 Windows 防火墙
配置 Windows 防火墙以允许 SQL Server 访问。需要配置防火墙，允许通过 TCP 连接到 SQL Server 实例使用的端口，以及连接到侦听器探测使用的端口。有关详细的说明，请参阅 [Configure a Windows Firewall for Database Engine Access](http://msdn.microsoft.com/zh-cn/library/ms175043.aspx#Anchor_1)（配置 Windows 防火墙以允许数据库引擎访问）。为 SQL Server 端口和探测端口创建入站规则。

## 示例脚本：使用 PowerShell 创建内部负载均衡器

以下 PowerShell 脚本创建内部负载均衡器、配置负载均衡规则，并设置负载均衡器的 IP 地址。若要运行该脚本，请打开 Windows PowerShell ISE，然后将脚本粘贴到“脚本”窗格中。使用 `Login-AzureRMAccount` 登录到 PowerShell。如果有多个 Azure 订阅，请使用 `Select-AzureRmSubscription ` 设置订阅。

    # Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    # Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

    $ResourceGroupName = "<Resource Group Name>" # Resource group name
    $VNetName = "<Virtual Network Name>"         # Virtual network name
    $SubnetName = "<Subnet Name>"                # Subnet name
    $ILBName = "<Load Balancer Name>"            # ILB name
    $Location = "<Azure Region>"                 # Azure location
    $VMNames = "<VM1>","<VM2>"                   # Virtual machine names

    $ILBIP = "<n.n.n.n>"                         # IP address
    [int]$ListenerPort = "<nnnn>"                # AG listener port
    [int]$ProbePort = "<nnnn>"                   # Probe port

    $LBProbeName ="ILBPROBE_$ListenerPort"       # The Load balancer Probe Object Name              
    $LBConfigRuleName = "ILBCR_$ListenerPort"    # The Load Balancer Rule Object Name

    $FrontEndConfigurationName = "FE_SQLAGILB_1" # Object name for the Front End configuration 
    $BackEndConfigurationName ="BE_SQLAGILB_1"   # Object name for the Back End configuration

    $VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 

    $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName 

    $FEConfig = New-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.id

    $BEConfig = New-AzureRMLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName 

    $SQLHealthProbe = New-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -Protocol tcp -Port $ProbePort -IntervalInSeconds 15 -ProbeCount 2

    $ILBRule = New-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP 

    $ILB= New-AzureRmLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe 

    $bepool = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB 

    foreach($VMName in $VMNames)
        {
            $VM = Get-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VMName 
            $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)
            $NIC = Get-AzureRmNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName
            $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools = $BEPool
            Set-AzureRmNetworkInterface -NetworkInterface $NIC
            start-AzureRmVM -ResourceGroupName $ResourceGroupName -Name $VM.Name 
        }

## 示例脚本：使用 PowerShell 将 IP 地址添加到现有负载均衡器
若要使用多个可用性组，请使用 PowerShell 将附加的 IP 地址添加到现有负载均衡器。每个 IP 地址都需要有自身的负载均衡规则、探测端口和前端端口。

前端端口是应用程序用来连接到 SQL Server 实例的端口。不同可用性组的 IP 地址可以使用相同的前端端口。

> [AZURE.NOTE]
对于 SQL Server 可用性组，每个 IP 地址需要一个特定的探测端口。例如，如果负载均衡器上有一个 IP 地址使用探测端口 59999，该负载均衡器上的其他任何 IP 地址就不能使用探测端口 59999。

* 有关负载均衡器限制的信息，请参阅[网络限制 - Azure Resource Manager](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits) 下面的**每个负载均衡器的专用前端 IP**。
* 有关可用性组限制的信息，请参阅 [Restrictions (Availability Groups)](http://msdn.microsoft.com/zh-cn/library/ff878487.aspx#RestrictionsAG)（限制（可用性组））。

以下脚本将新的 IP 地址添加到现有负载均衡器。请更新环境的变量。ILB 使用侦听器端口作为负载均衡前端端口。此端口可以是 SQL Server 正在侦听的端口。对于 SQL Server 的默认实例，此端口为 1433。可用性组的负载均衡规则需要浮动 IP（直接服务器返回），因此后端端口与前端端口相同。

    # Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    # Select-AzureRmSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

    $ResourceGroupName = "<ResourceGroup>"          # Resource group name
    $VNetName = "<VirtualNetwork>"                  # Virtual network name
    $SubnetName = "<Subnet>"                        # Subnet name
    $ILBName = "<ILBName>"                          # ILB name                      

    $ILBIP = "<n.n.n.n>"                            # IP address
    [int]$ListenerPort = "<nnnn>"                   # AG listener port
    [int]$ProbePort = "<nnnnn>"                     # Probe port 

    $ILB = Get-AzureRmLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName 

    $count = $ILB.FrontendIpConfigurations.Count+1
    $FrontEndConfigurationName ="FE_SQLAGILB_$count"  

    $LBProbeName = "ILBPROBE_$count"
    $LBConfigrulename = "ILBCR_$count"

    $VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 
    $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

    $ILB | Add-AzureRmLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id 

    $ILB | Add-AzureRmLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 15  | Set-AzureRmLoadBalancer 

    $ILB = Get-AzureRmLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

    $FEConfig = get-AzureRMLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB

    $SQLHealthProbe  = Get-AzureRmLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

    $BEConfig = Get-AzureRmLoadBalancerBackendAddressPoolConfig -Name $ILB.BackendAddressPools[0].Name -LoadBalancer $ILB 

    $ILB | Add-AzureRmLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort  $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP | Set-AzureRmLoadBalancer   

## 将群集配置为使用负载均衡器 IP 地址
下一步是在群集上配置侦听器，然后将侦听器联机。若要实现此目的，请执行以下步骤：

1. 在故障转移群集上创建可用性组侦听器
2. 使侦听器联机

## 在故障转移群集上创建可用性组侦听器

可用性组侦听器是 SQL Server 可用性组侦听的 IP 地址和网络名称。若要创建可用性组侦听器，请执行以下步骤：

1. [获取群集网络资源的名称](#getnet)。

1. [添加客户端接入点](#addcap)。

1. [配置可用性组的 IP 资源](#congroup)。

1. [使可用性组资源依赖于侦听器资源名称](#listname)。

1. [在 PowerShell 中设置群集参数](#setparam)。

以下部分提供其中每个步骤的详细说明。

### <a name="getnet">获取群集网络资源的名称</a> 

1. 使用 RDP 连接到托管主副本的 Azure 虚拟机。

1. 打开故障转移群集管理器。

1. 选择“网络”节点，并记下群集网络名称。稍后在 PowerShell 脚本的 `$ClusterNetworkName` 变量中将要使用此名称。

### <a name="addcap">添加客户端接入点</a>

1. 展开群集名称，然后单击“角色”。

1. 在“角色”窗格中，右键单击可用性组名称，然后选择“添加资源”>“客户端访问点”。

1. 在“名称”框中，创建此新侦听器的名称。

    新侦听器的名称是应用程序用来连接 SQL Server 可用性组中数据库的网络名称。
   
    若要完成创建侦听器，请单击“下一步”两次，然后单击“完成”。此时不要使侦听器或资源联机。
   
### <a name="congroup">配置可用性组的 IP 资源</a>

1. 单击“资源”选项卡，然后展开刚创建的客户端访问点。右键单击 IP 资源，然后单击“属性”。记下 IP 地址的名称。稍后在 PowerShell 脚本的 `$IPResourceName` 变量中将要使用此名称。

1. 单击“IP 地址”下面的“静态 IP 地址”，并将静态 IP 地址设置为在 Azure 门户预览上设置负载均衡器 IP 地址时所用的相同地址。

1. 为此地址禁用 NetBIOS，然后单击“确定”。如果你的解决方案跨越多个 Azure Vnet，请为每个 IP 资源重复此步骤。

### <a name="listname">使可用性组资源依赖于侦听器资源</a>

1. 在故障转移群集管理器中单击“角色”，然后单击你的可用性组。

1. 在“资源”选项卡上，右键单击可用性资源组，然后单击“属性”。

1. 选择“依赖项”选项卡。设置与侦听器资源名称之间的依赖关系。如果有多个资源列出，请验证 IP 地址具有 OR 而不是 AND 依赖项。单击**“确定”**。

1. 右键单击侦听器名称，然后单击“联机”。

### <a name="setparam">在 PowerShell 中设置群集参数</a>

设置群集参数。为此，请更新以下 PowerShell 脚本。使用环境值设置变量。在某个群集节点上运行 PowerShell 脚本。

       $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
       $IPResourceName = "<IPResourceName>" # the IP Address resource name
       $ILBIP = "<n.n.n.n>" # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal preview.
       [int]$ProbePort = <nnnnn>

       Import-Module FailoverClusters

       Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}

> [AZURE.NOTE]
如果 SQL Server 位于不同的区域，则需要运行 PowerShell 脚本两次。第一次请使用第一个区域中的 `$ILBIP` 和 `$ProbePort`。第二次请使用第二个区域中的 `$ILBIP` 和 `$ProbePort`。群集网络名称和群集 IP 资源名称相同。

## 在 SQL Server Management Studio 中设置侦听器端口

1. 启动 SQL Server Management Studio 并连接到主副本。

1. 导航到“AlwaysOn 高可用性”|“可用性组”|“可用性组侦听器”。

1. 你现在应看到在故障转移群集管理器中创建的侦听器名称。右键单击侦听器名称，然后单击“属性”。

1. 在“端口”框中，通过使用先前使用过的 $EndpointPort 为可用性组侦听器指定端口号（默认值为 1433），然后单击“确定”。

现在，Resource Manager 模式下运行的 Azure 虚拟机中便有了一个 SQL Server 可用性组。

## 测试与侦听器的连接

若要测试连接，请执行以下操作：

1. 通过 RDP 连接到同一虚拟网络中不拥有副本的 SQL Server。这可以是群集中的其他 SQL Server。

1. 使用 **sqlcmd** 实用工具测试连接。例如，以下脚本通过侦听器与 Windows 身份验证来与主副本建立 **sqlcmd** 连接：

        sqlmd -S <listenerName> -E

如果侦听器使用的端口不是默认端口 (1433)，请在连接字符串中指定该端口。例如，以下 sqlcmd 命令连接到位于端口 1435 的侦听器：

        sqlcmd -S <listenerName>,1435 -E

SQLCMD 连接将自动连接到托管主副本的 SQL Server 实例。

> [AZURE.NOTE]
确保指定的端口已在两个 SQL Server 的防火墙上打开。这两个服务器需要所用 TCP 端口的入站规则。有关详细信息，请参阅 [Add or Edit Firewall Rule](http://technet.microsoft.com/zh-cn/library/cc753558.aspx)（添加或编辑防火墙规则）。
> 
> 

## 指导原则和限制
请注意有关 Azure 中使用内部负载均衡器的可用性组侦听器的以下指导原则：

* 使用内部负载均衡器只能从同一个虚拟网络中访问侦听器。

## 更多信息
有关详细信息，请参阅 [Configure Always On availability group in Azure VM manually](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)（在 Azure VM 中手动配置 Always On 可用性组）。

### PowerShell cmdlet
使用以下 PowerShell cmdlet 为 Azure 虚拟机创建内部负载均衡器。

* `New-AzureRmLoadBalancer` 创建负载均衡器。有关详细信息，请参阅 [New-AzureRmLoadBalancer](http://msdn.microsoft.com/zh-cn/library/mt619450.aspx)。
* `New-AzureRMLoadBalancerFrontendIpConfig` 创建负载均衡器的前端 IP 配置。有关详细信息，请参阅 [New-AzureRMLoadBalancerFrontendIpConfig](http://msdn.microsoft.com/zh-cn/library/mt603510.aspx)。
* `New-AzureRmLoadBalancerRuleConfig` 创建负载均衡器的规则配置。有关详细信息，请参阅 [New-AzureRmLoadBalancerRuleConfig](http://msdn.microsoft.com/zh-cn/library/mt619391.aspx)。
* `New-AzureRMLoadBalancerBackendAddressPoolConfig` 创建负载均衡器的后端地址池配置。有关详细信息，请参阅 [New-AzureRmLoadBalancerBackendAddressPoolConfig](http://msdn.microsoft.com/zh-cn/library/mt603791.aspx)。
* `New-AzureRmLoadBalancerProbeConfig` 创建负载均衡器的探测配置。有关详细信息，请参阅 [New-AzureRmLoadBalancerProbeConfig](http://msdn.microsoft.com/zh-cn/library/mt603847.aspx)。

如果需要从 Azure 资源组中删除负载均衡器，请使用 `Remove-AzureRmLoadBalancer`。有关详细信息，请参阅 [Remove-AzureRmLoadBalancer](http://msdn.microsoft.com/zh-cn/library/mt603862.aspx)。

<!---HONumber=Mooncake_0116_2017-->