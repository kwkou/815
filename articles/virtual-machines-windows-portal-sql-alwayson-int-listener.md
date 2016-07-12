<!-- Ibiza portal: tested -->

<properties
   pageTitle="为 Azure 虚拟机中的 SQL Server AlwaysOn 可用性组创建侦听器"
   description="有关为 Azure 虚拟机中的 SQL Server AlwaysOn 可用性组创建侦听器的分步说明"
   services="virtual-machines"
   documentationCenter="na"
   authors="MikeRayMSFT"
   manager="jhubbard"
   editor="monicar"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/17/2016"
	wacn.date="06/20/2016"/>

# 在 Azure 中为 AlwaysOn 可用性组配置内部负载平衡器

本主题说明如何在资源管理器模型中运行的 Azure 虚拟机上创建 SQL Server AlwaysOn 可用性组的内部负载平衡器。当 SQL Server 实例位于 Azure 虚拟机时，AlwaysOn 可用性组需要负载平衡器。负载平衡器存储可用性组侦听器的 IP 地址。如果可用性组跨多个区域，则每个区域都需要一个负载平衡器。

若要完成此任务，需要在 Resource Manager 模型中的 Azure 虚拟机上部署 SQL Server AlwaysOn 可用性组。这两个 SQL Server 虚拟机必须属于同一个可用性集。

如果需要，你可以[手动配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)。

本主题要求事先配置可用性组。

相关主题包括：

 - [在 Azure VM (GUI) 中配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)   
 
 - [使用 Azure Resource Manager 和 PowerShell 配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)

## 步骤

执行本文档中的每个步骤可以在 Azure 门户预览中创建和配置负载平衡器。完成这些步骤后，你将配置群集，以将负载平衡器中的 IP 地址用于 AlwaysOn 可用性组侦听器。

## 在 Azure 门户预览中创建并配置负载平衡器

在任务的这一部分，你将在 Azure 门户预览中执行以下步骤：

1. 创建负载平衡器并配置 IP 地址

1. 配置后端池

1. 创建探测

1. 设置负载平衡规则

>[AZURE.NOTE] 如果 SQL Server 位于不同的资源组和区域，则需要执行上述所有步骤两次：在每个资源组中各执行一次。

## 1\.创建负载平衡器并配置 IP 地址

第一步是创建负载平衡器。在 Azure 门户预览中，打开包含 SQL Server 虚拟机的资源组。在资源组中，单击“添加”。

- 搜索“负载平衡器”。从搜索结果中选择由 **Microsoft** 发布的**负载平衡器**。

- 在“负载平衡器”边栏选项卡上，单击“创建”。

- 在“创建负载平衡器”中配置负载平衡器，如下所示：

| 设置 | 值 |
| ----- | ----- |
| **名称** | 表示负载平衡器的文本名称。例如 **sqlLB**。 |
| **架构** | **内部** |
| **虚拟网络** | 选择 SQL Server 所在的虚拟网络。 |
| **子网** | 选择 SQL Server 所在的子网。 |
| **订阅** | 如果你有多个订阅，可能会出现此字段。选择要与此资源关联的订阅。通常是与可用性组的所有资源相同的订阅。 |
| **资源组** | 选择 SQL Server 所在的资源组。 | 
| **位置** | 选择 SQL Server 所在的 Azure 位置。 |

- 单击“创建”。 

Azure 将创建上面配置的负载平衡器。该负载平衡器属于特定的网络、子网、资源组和位置。在 Azure 完成操作后，请在 Azure 中验证负载平衡器设置。

现在，请配置负载平衡器 IP 地址。

- 在负载平衡器的“设置”边栏选项卡上，单击“IP 地址”。“IP 地址”边栏选项卡显示这是与 SQL Server 位于相同虚拟网络上的专用负载平衡器。 

- 指定以下设置：

| 设置 | 值 |
| ----- | ----- |
| **子网** | 选择 SQL Server 所在的子网。 |
| **分配** | **静态** |
| **IP 地址** | 键入子网中未使用的虚拟 IP 地址。 |

- 保存设置。

现在，负载平衡器有了一个 IP 地址。请记下此 IP 地址。当你在群集上创建侦听器时，将要用到此 IP 地址。本文稍后的 PowerShell 脚本会将此地址用于 `$ILBIP` 变量。



## 2\.配置后端池

下一步是创建后端地址池。Azure 将后端地址池称作*后端池*。在本例中，后端池是可用性组中两个 SQL Server 的地址。

- 在资源组中，单击你创建的负载平衡器。 

- 在“设置”中，单击“后端池”。

- 在“后端地址池”中，单击“添加”以创建后端地址池。

- 在“添加后端池”中的“名称”下，键入后端池的名称。

- 在“虚拟机”下，单击“+ 添加虚拟机”。

- 在“选择虚拟机”下，单击“选择可用性集”，然后指定 SQL Server 虚拟机所属的可用性集。

- 选择可用性集后，请单击“选择虚拟机”。单击在可用性组中托管 SQL Server 实例的两个虚拟机。单击“选择”。

- 单击“确定”，以关闭“选择虚拟机”和“添加后端池”边栏选项卡。

Azure 将更新后端地址池的设置。现在，可用性集具有包含两个 SQL 服务器的池。

## 3\.创建探测

下一步是创建探测。探测定义 Azure 如何确认哪一个 SQL Server 当前拥有可用性组侦听器。Azure 根据创建探测时定义的端口上的 IP 地址来探测服务。

- 在负载平衡器的“设置”边栏选项卡上，单击“探测”。 

- 在“探测”边栏选项卡上，单击“添加”。

- 在“添加探测”边栏选项卡上配置探测。使用以下值配置探测：

| 设置 | 值 |
| ----- | ----- |
| **名称** | 表示探测的文本名称。例如 **SQLAlwaysOnEndPointProbe**。 |
| **协议** | **TCP** |
| **端口** | 可以使用任何可用的端口。例如 *59999*。 |
| **时间间隔** | *5* | 
| **不正常阈值** | *2* | 

- 单击**“确定”**。 

>[AZURE.NOTE] 确保指定的端口已在两个 SQL Server 的防火墙上打开。这两个服务器需要所用 TCP 端口的入站规则。有关详细信息，请参阅 [Add or Edit Firewall Rule（添加或编辑防火墙规则）](http://technet.microsoft.com/zh-cn/library/cc753558.aspx)。

Azure 将创建探测。Azure 使用探测来测试哪个 SQL Server 具有可用性组的侦听器。

## 4\.设置负载平衡规则

设置负载平衡规则。负载平衡规则设置负载平衡器将流量路由到 SQL Server 的方式。对此负载平衡器，需要启用直接服务器返回，因为在两个 SQL Server 中，每次只有一个拥有可用性组侦听器资源。

- 在负载平衡器的“设置”边栏选项卡上，单击“负载平衡规则”。 

- 在“负载平衡规则”边栏选项卡上，单击“添加”。

- 使用“添加负载平衡规则”边栏选项卡配置负载平衡规则。使用以下设置：

| 设置 | 值 |
| ----- | ----- |
| **名称** | 表示负载平衡规则的文本名称。例如 **SQLAlwaysOnEndPointListener**。 |
| **协议** | **TCP** |
| **端口** | *1433* |
| **后端端口** | *1433*。请注意，将禁用此项，因为此规则使用“浮动 IP (直接服务器返回)”。 |
| **探测** | 使用为此负载平衡器创建的探测的名称。 |
| **会话持久性** | **无** | 
| **空闲超时(分钟)** | *4* | 
| **浮动 IP (直接服务器返回)** | **已启用** | 

 >[AZURE.NOTE] 可能需要在边栏选项卡中向下滚动才能看到所有设置。

- 单击**“确定”**。 

- Azure 将配置负载平衡规则。负载平衡器现已配置为将流量路由到托管可用性组侦听器的 SQL Server。

此时，资源组有一个连接到这两个 SQL Server 计算机的负载平衡器。负载平衡器还包含 SQL Server AlwaysOn 可用性组侦听器的 IP 地址，以便任一计算机可以响应针对可用性组的请求。

>[AZURE.NOTE] 如果 SQL Server 位于两个不同的区域，请在另一个区域重复上述步骤。每个区域都需要一个负载平衡器。

## 将群集配置为使用负载平衡器 IP 地址 

下一步是在群集上配置侦听器，然后将侦听器联机。若要实现此目的，请执行以下步骤：

1. 在故障转移群集上创建可用性组侦听器 

1. 使侦听器联机

## 1\.在故障转移群集上创建可用性组侦听器

在此步骤中，你在故障转移群集管理器和 SQL Server Management Studio (SSMS) 中手动创建可用性组侦听器。

- 使用 RDP 连接到托管主副本的 Azure 虚拟机。 

- 打开故障转移群集管理器。

- 选择“网络”节点，并记下群集网络名称。PowerShell 脚本中的 `$ClusterNetworkName` 变量将使用此名称。

- 展开群集名称，然后单击“角色”。

- 在“角色”窗格中，右键单击可用性组名称，然后选择“添加资源”>“客户端访问点”。

- 在“名称”框中，为此新的侦听器创建一个名称，单击“下一步”两次，然后单击“完成”。不要在此时使侦听器或资源联机。

 >[AZURE.NOTE] 新侦听器的名称是应用程序用来连接 SQL Server 可用性组中数据库的网络名称。

- 单击“资源”选项卡，然后展开刚创建的客户端访问点。右键单击 IP 资源，然后单击“属性”。记下 IP 地址的名称。你将在 PowerShell 脚本的 `$IPResourceName` 变量中使用此名称。

- 单击“IP 地址”下面的“静态 IP 地址”，并将静态 IP 地址设置为在 Azure 门户预览上设置负载平衡器 IP 地址时所用的相同地址。为此地址启用 NetBIOS，然后单击“确定”。如果你的解决方案跨越多个 Azure Vnet，请为每个 IP 资源重复此步骤。

- 在当前托管主副本的群集节点上，打开已提升权限的 PowerShell ISE，然后将以下命令粘贴到新脚本中。

        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP Address resource name
        $ILBIP = "<X.X.X.X>" # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal.
    
        Import-Module FailoverClusters
    
        Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}

- 更新变量并运行 PowerShell 脚本，以配置新侦听器的 IP 地址和端口。

 >[AZURE.NOTE] 如果 SQL Server 位于不同的区域，则需要运行 PowerShell 脚本两次。第一次使用第一个资源组中的群集网络名称、群集 IP 资源名称和负载平衡器 IP 地址。第二次使用第二个资源组中的群集网络名称、群集 IP 资源名称和负载平衡器 IP 地址。

现在，群集包含可用性组侦听器资源。

##<a name="2-bring-the-listener-online"></a> 2\.使侦听器联机

配置可用性组侦听器资源后，可将侦听器联机，以便应用程序使用侦听器连接到可用性组中的数据库。

- 导航回故障转移群集管理器。展开“角色”，然后突出显示可用性组。在“资源”选项卡上，右键单击侦听器名称，然后单击“属性”。

- 选择“依赖项”选项卡。如果有多个资源列出，请验证 IP 地址具有 OR 而不是 AND 依赖项。单击**“确定”**。

- 右键单击侦听器名称，然后单击“联机”。


- 侦听器处于联机状态后，在“资源”选项卡上，右键单击可用性组，然后单击“属性”。

- 在侦听器名称资源（而不是 IP 地址资源名称）上创建依赖项。单击**“确定”**。


- 启动 SQL Server Management Studio 并连接到主副本。


- 导航到“AlwaysOn 高可用性”|“可用性组”|“可用性组侦听器”。


- 你现在应看到在故障转移群集管理器中创建的侦听器名称。右键单击侦听器名称，然后单击“属性”。


- 在“端口”框中，通过使用先前使用过的 $EndpointPort 为可用性组侦听器指定端口号（默认值为 1433），然后单击“确定”。

现在，你在 Resource Manager 模式下运行的 Azure 虚拟机中有了一个 SQL Server AlwaysOn 可用性组。

## 测试与侦听器的连接

若要测试连接，请执行以下操作：

1. 通过 RDP 连接到同一虚拟网络中不拥有副本的 SQL Server。这可以是群集中的其他 SQL Server。

1. 使用 **sqlcmd** 实用工具来测试连接。例如，以下脚本通过侦听器与 Windows 身份验证来与主副本建立 **sqlcmd** 连接：

        sqlmd -S <listenerName> -E

SQLCMD 连接将自动连接到托管主副本的 SQL Server 实例。

## 指导原则和限制

请注意有关 Azure 中使用内部负载平衡器的可用性组侦听器的以下准则：

- 每个云服务只支持一个内部可用性组侦听器，因为该侦听器将配置给负载平衡器，并且只有一个内部负载平衡器。但是，可以创建多个外部侦听器。 

- 使用内部负载平衡器只能从同一个虚拟网络中访问侦听器。
 

<!---HONumber=Mooncake_0613_2016-->