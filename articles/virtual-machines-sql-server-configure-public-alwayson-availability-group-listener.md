<properties 
	pageTitle="为 AlwaysOn 可用性组配置外部侦听器 | Azure"
	description="本教程指导你完成在 Azure 中创建一个可以使用关联云服务公共虚拟 IP 地址从外部访问的 AlwaysOn 可用性组侦听器。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"
	tags="azure-service-management" />
<tags 
	ms.service="virtual-machines-windows"
	ms.date="05/08/2016"
	wacn.date="06/29/2016" />

# 在 Azure 中配置 AlwaysOn 可用性组的外部侦听器

> [AZURE.SELECTOR]
- [内部侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)
- [外部侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-ext-listener/)

本主题说明如何为 AlwaysOn 可用性组配置一个可以通过 Internet 从外部访问的侦听器。这是通过将云服务的**公共虚拟 IP (VIP)** 地址与侦听器关联来实现的。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]

## 外部侦听器的准则和限制

你的可用性组可以仅包含本地副本或 Azure 副本，也可以跨越本地和 Azure 以实现混合配置。Azure 副本可以位于同一区域，也可以跨越使用多个虚拟网络 (VNet) 的多个区域。以下步骤假设你已[配置了一个可用性组](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)但是没有配置侦听器。

在使用云服务的公共 VIP 地址部署时，请注意对 Azure 中可用性组侦听器的以下限制：

- Windows Server 2008 R2、Windows Server 2012 和 Windows Server 2012 R2 支持可用性组侦听器。

- 客户端应用程序必须位于与包含你的可用性组 VM 的云服务不同的云服务中。Azure 不支持客户端和服务器位于同一个云服务中的直接服务器返回。

- 每个云服务只支持一个可用性组侦听器，因为该侦听器将配置为使用云服务 VIP 地址。尽管 Azure 现在支持在给定的云服务中创建多个 VIP 地址，但此限制仍然有效。

- 如果你要为混合环境创建侦听器，则本地网络必须连接到公共 Internet，还通过 Azure 虚拟网络连接到站点到站点 VPN。位于 Azure 子网中时，只能通过相应云服务的公共 IP 地址来访问该可用性组侦听器。

## 确定侦听器的可访问性

[AZURE.INCLUDE [ag-listener-accessibility](../includes/virtual-machines-ag-listener-determine-accessibility.md)]

本文重点介绍如何创建使用**外部负载平衡**的侦听器。如果你想要创建专用于虚拟网络的侦听器，请参阅本文的另一个版本，其中提供了设置[使用 ILB 的侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)的步骤

## 创建支持直接服务器返回的负载平衡 VM 终结点

外部负载平衡使用托管 VM 的云服务的公共虚拟 IP 地址。因此，在这种情况下，你不需要创建或配置负载平衡器。

[AZURE.INCLUDE [load-balanced-endpoints](../includes/virtual-machines-ag-listener-load-balanced-endpoints.md)]

1. 将以下 PowerShell 脚本复制到文本编辑器中，并根据你的环境设置变量值（这里为某些参数提供了默认值）。请注意，如果可用性组跨 Azure 区域，则你必须在每个数据中心内对云服务和节点运行该脚本一次。

		# Define variables
		$ServiceName = "<MyCloudService>" # the name of the cloud service that contains the availability group nodes
		$AGNodes = "<VM1>","<VM2>","<VM3>" # all availability group nodes containing replicas in the same cloud service, separated by commas
		
		# Configure a load balanced endpoint for each node in $AGNodes, with direct server return enabled
		ForEach ($node in $AGNodes)
		{
		    Get-AzureVM -ServiceName $ServiceName -Name $node | Add-AzureEndpoint -Name "ListenerEndpoint" -Protocol "TCP" -PublicPort 1433 -LocalPort 1433 -LBSetName "ListenerEndpointLB" -ProbePort 59999 -ProbeProtocol "TCP" -DirectServerReturn $true | Update-AzureVM
		}

1. 设置变量后，将脚本从文本编辑器复制到 Azure PowerShell 会话中运行。如果提示符仍然显示 >>，请再次按 Enter，以确保脚本开始运行。

## 如果需要，请验证是否已安装 KB2854082

[AZURE.INCLUDE [kb2854082](../includes/virtual-machines-ag-listener-kb2854082.md)]

## 在可用性组节点中打开防火墙端口

[AZURE.INCLUDE [防火墙](../includes/virtual-machines-ag-listener-open-firewall.md)]

## 创建可用性组侦听器

[AZURE.INCLUDE [防火墙](../includes/virtual-machines-ag-listener-create-listener.md)]

1. 对于外部负载平衡，你必须获取包含副本的云服务的公共虚拟 IP 地址。登录到 Azure 经典管理门户。导航到包含你的可用性组 VM 的云服务。打开“仪表板”视图。 

3. 记下“公共虚拟 IP (VIP)地址”下显示的地址。如果解决方案跨 VNet，请针对包含副本所在 VM 的每个云服务重复此步骤。

4. 在某个 VM 上，将以下 PowerShell 脚本复制到文本编辑器中，将变量设置为之前记下的值。

		# Define variables
		$ClusterNetworkName = "<ClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
		$IPResourceName = "<IPResourceName>" # the IP Address resource name 
		$CloudServiceIP = "<X.X.X.X>" # Public Virtual IP (VIP) address of your cloud service
		
		Import-Module FailoverClusters
		
		# If you are using Windows Server 2012 or higher, use the Get-Cluster Resource command. If you are using Windows Server 2008 R2, use the cluster res command. Both commands are commented out. Choose the one applicable to your environment and remove the # at the beginning of the line to convert the comment to an executable line of code. 
		
		# Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$CloudServiceIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"OverrideAddressMatch"=1;"EnableDhcp"=0}
		# cluster res $IPResourceName /priv enabledhcp=0 overrideaddressmatch=1 address=$CloudServiceIP probeport=59999  subnetmask=255.255.255.255


1. 设置变量之后，打开提升的 Windows PowerShell 窗口，然后从文本编辑器复制脚本，并将其粘贴到 Azure PowerShell 会话中运行。如果提示符仍然显示 >>，请再次按 Enter，以确保脚本开始运行。

1. 在每个 VM 上重复此过程。此脚本将使用云服务的 IP 地址来配置 IP 地址资源，同时设置探测端口等其他参数。在 IP 地址资源联机后，它可以响应我们在本教程前面部分创建的负载平衡终结点在探测端口上的轮询。

## 使侦听器联机

[AZURE.INCLUDE [Bring-Listener-Online](../includes/virtual-machines-ag-listener-bring-online.md)]

## 后续操作项

[AZURE.INCLUDE [Follow-up](../includes/virtual-machines-ag-listener-follow-up.md)]

## 测试可用性组侦听器（在同一 VNet 中）

[AZURE.INCLUDE [Test-Listener-Within-VNET](../includes/virtual-machines-ag-listener-test.md)]

## 测试可用性组侦听器（通过 Internet）

若要从虚拟网络外部访问侦听器，你必须使用外部/公共负载平衡（如本主题中所述）而不是 ILB，因为 ILB 只能在同一 VNet 中进行访问。在连接字符串中指定云服务名称。例如，如果你的云服务名为 *mycloudservice*，则 sqlcmd 语句将如下所示：

	sqlcmd -S "mycloudservice.cloudapp.net,<EndpointPort>" -d "<DatabaseName>" -U "<LoginId>" -P "<Password>"  -Q "select @@servername, db_name()" -l 15

与前面的示例不同，现在必须使用 SQL 身份验证，因为调用方无法通过 Internet 使用 Windows 身份验证。有关详细信息，请参阅 [Azure VM 中的 AlwaysOn 可用性组：客户端连接方案](http://blogs.msdn.com/b/sqlcat/archive/2014/02/03/alwayson-availability-group-in-windows-azure-vm-client-connectivity-scenarios.aspx)。使用 SQL 身份验证时，请确保在两个副本上创建相同的登录名。有关排查可用性组登录问题的详细信息，请参阅[如何映射登录或使用包含的 SQL 数据库用户连接到其他副本并映射到可用性数据库](http://blogs.msdn.com/b/alwaysonpro/archive/2014/02/19/how-to-map-logins-or-use-contained-sql-database-user-to-connect-to-other-replicas-and-map-to-availability-databases.aspx)。

如果 AlwaysOn 副本位于不同子网中，客户端必须在连接字符串中指定 **MultisubnetFailover=True**。这会导致尝试并行连接到不同子网中的副本。请注意，这种情况下包括跨区域 AlwaysOn 可用性组部署。

## 后续步骤

[AZURE.INCLUDE [Listener-Next-Steps](../includes/virtual-machines-ag-listener-next-steps.md)]

<!---HONumber=76-->