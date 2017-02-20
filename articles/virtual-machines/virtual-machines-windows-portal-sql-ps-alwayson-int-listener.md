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
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="12/28/2016"
    wacn.date="02/20/2017"
    ms.author="mikeray" />  


# 配置一个或多个 Always On 可用性组侦听器 - Resource Manager
本主题说明如何执行以下操作：

* 使用 PowerShell cmdlet 为 SQL Server 可用性组创建内部负载均衡器。
* 将其他 IP 地址添加到负载均衡器，以支持多个可用性组。

可用性组侦听器是客户端为了访问数据库而连接的虚拟网络名称。在 Azure 虚拟机上，负载均衡器持有侦听器的 IP 地址。负载均衡器将流量路由到侦听探测端口的 SQL Server 实例。通常情况下，可用性组使用内部负载均衡器。Azure 内部负载均衡器可以托管一个或多个 IP 地址。每个 IP 地址使用特定的探测端口。本文档说明如何使用 PowerShell 创建负载均衡器，或将 IP 地址添加到 SQL Server 可用性组的现有负载均衡器。

将多个 IP 地址分配到内部负载均衡器是 Azure 的一项新增功能，只能在 Resource Manager 模型中使用。若要完成此任务，需要在 Resource Manager 模型中的 Azure 虚拟机上部署 SQL Server 可用性组。这两个 SQL Server 虚拟机必须属于同一个可用性集。如果需要，可以[手动配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)。

本主题要求事先配置可用性组。

相关主题包括：

* [在 Azure VM \(GUI\) 中配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)
* [使用 Azure Resource Manager 和 PowerShell 配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-vm-powershell.md)]

## 配置 Windows 防火墙
配置 Windows 防火墙以允许 SQL Server 访问。防火墙规则允许到端口（由 SQL Server 实例使用）的 TCP 连接，并且允许侦听器探测。有关详细的说明，请参阅 [Configure a Windows Firewall for Database Engine Access](http://msdn.microsoft.com/zh-cn/library/ms175043.aspx#Anchor_1)（配置 Windows 防火墙以允许数据库引擎访问）。为 SQL Server 端口和探测端口创建入站规则。

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

    $FrontEndConfigurationName = "FE_SQLAGILB_1" # Object name for the front-end configuration 
    $BackEndConfigurationName ="BE_SQLAGILB_1"   # Object name for the back-end configuration

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

## <a name="Add-IP"></a> 示例脚本：使用 PowerShell 将 IP 地址添加到现有负载均衡器
若要使用多个可用性组，请将附加的 IP 地址添加到负载均衡器。每个 IP 地址都需要有自身的负载均衡规则、探测端口和前端端口。

前端端口是应用程序用来连接到 SQL Server 实例的端口。不同可用性组的 IP 地址可以使用相同的前端端口。

> [AZURE.NOTE]
对于 SQL Server 可用性组，每个 IP 地址需要一个特定的探测端口。例如，如果负载均衡器上有一个 IP 地址使用探测端口 59999，该负载均衡器上的其他任何 IP 地址就不能使用探测端口 59999。

* 有关负载均衡器限制的信息，请参阅[网络限制 - Azure Resource Manager](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits) 下面的**每个负载均衡器的专用前端 IP**。
* 有关可用性组限制的信息，请参阅 [Restrictions \(Availability Groups\)](http://msdn.microsoft.com/zh-cn/library/ff878487.aspx#RestrictionsAG)（限制（可用性组））。

以下脚本将新的 IP 地址添加到现有负载均衡器。ILB 使用侦听器端口作为负载均衡前端端口。此端口可以是 SQL Server 正在侦听的端口。对于 SQL Server 的默认实例，此端口为 1433。可用性组的负载均衡规则需要浮动 IP（直接服务器返回），因此后端端口与前端端口相同。请更新环境的变量。

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

## 配置侦听器

[AZURE.INCLUDE [ag-listener-configure](../../includes/virtual-machines-ag-listener-configure.md)]

## 在 SQL Server Management Studio 中设置侦听器端口

1. 启动 SQL Server Management Studio 并连接到主副本。

1. 导航到“AlwaysOn 高可用性”\|“可用性组”\|“可用性组侦听器”。

1. 你现在应看到在故障转移群集管理器中创建的侦听器名称。右键单击侦听器名称，然后单击“属性”。

1. 在“端口”框中，通过使用先前使用过的 $EndpointPort 为可用性组侦听器指定端口号（默认值为 1433），然后单击“确定”。

## 测试与侦听器的连接

若要测试连接，请执行以下操作：

1. 通过 RDP 连接到同一虚拟网络中不拥有副本的 SQL Server。这可以是群集中的其他 SQL Server。

1. 使用 **sqlcmd** 实用工具测试连接。例如，以下脚本通过侦听器与 Windows 身份验证来与主副本建立 **sqlcmd** 连接：

        sqlmd -S <listenerName> -E

如果侦听器使用的端口不是默认端口 \(1433\)，请在连接字符串中指定该端口。例如，以下 sqlcmd 命令连接到位于端口 1435 的侦听器：

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
有关详细信息，请参阅[在 Azure VM 中手动配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)。

## PowerShell cmdlet
使用以下 PowerShell cmdlet 为 Azure 虚拟机创建内部负载均衡器。

* [New-AzureRmLoadBalancer](http://msdn.microsoft.com/zh-cn/library/mt619450.aspx) 创建负载均衡器。
* [New-AzureRMLoadBalancerFrontendIpConfig](http://msdn.microsoft.com/zh-cn/library/mt603510.aspx) 创建负载均衡器的前端 IP 配置。
* [New-AzureRmLoadBalancerRuleConfig](http://msdn.microsoft.com/zh-cn/library/mt619391.aspx) 创建负载均衡器的规则配置。
* [New-AzureRmLoadBalancerBackendAddressPoolConfig](http://msdn.microsoft.com/zh-cn/library/mt603791.aspx) 创建负载均衡器的后端地址池配置。
* [New-AzureRmLoadBalancerProbeConfig](http://msdn.microsoft.com/zh-cn/library/mt603847.aspx) 创建负载均衡器的探测配置。
* [Remove-AzureRmLoadBalancer](http://msdn.microsoft.com/zh-cn/library/mt603862.aspx) 从 Azure 资源组中删除负载均衡器。

<!---HONumber=Mooncake_0213_2017-->