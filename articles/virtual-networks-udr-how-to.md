<properties 
   pageTitle="如何在 Azure 中创建路由和启用 IP 转发"
   description="了解如何管理 UDR 和 IP 转发"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn" />
<tags 
   ms.service="virtual-network"
   ms.date="08/10/2015"
   wacn.date="09/18/2015" />

# 如何在 Azure 中创建路由和启用 IP 转发
你可以使用 Azure 中的虚拟设备来处理 Azure 虚拟网络中的流量。但是，你需要创建相应的路由，以便虚拟网络中的 VM 和云服务将数据包发送到虚拟设备而非数据包的所需目标。你还需要在虚拟设备 VM 上启用 IP 转发，使之能够接收和转发其目标地址不是实际虚拟设备 VM 的数据包。

## 如何管理路由
你可以使用 PowerShell 添加、删除和更改 Azure 中的路由。在创建路由之前，必须创建一个路由表来托管该路由。

### 如何创建路由表
若要创建名为 *FrontEndSubnetRouteTable* 的路由表，请运行以下 PowerShell 命令：

```powershell
New-AzureRouteTable -Name FrontEndSubnetRouteTable `
	-Location uscentral `
	-Label "Route table for frontend subnet"
```

以上命令的输出应如下所示：

	Error          :
	HttpStatusCode : OK
	Id             : 085ac8bf-26c3-9c4c-b3ae-ebe880108c70
	Status         : Succeeded
	StatusCode     : OK
	RequestId      : a8cc03ca42d39f27adeaa9c1986c14f7

### 如何将路由添加到路由表
若要在上面创建的路由表中添加一个将 *10.1.1.10* 设置为 *10.2.0.0/16* 子网的下一跃点的路由，请运行以下 PowerShell 命令：

```powershell
Get-AzureRouteTable FrontEndSubnetRouteTable `
	|Set-AzureRoute -RouteName FirewallRoute -AddressPrefix 10.2.0.0/16 `
	-NextHopType VirtualAppliance `
	-NextHopIpAddress 10.1.1.10
```

以上命令的输出应如下所示：

	Name     : FrontEndSubnetRouteTable
	Location : China North
	Label    : Route table for frontend subnet
	Routes   : 
	           Name                 Address Prefix    Next hop type        Next hop IP address
	           ----                 --------------    -------------        -------------------
	           firewallroute        10.2.0.0/16       VirtualAppliance     10.1.1.10    

### 如何将路由关联到子网
路由表必须与一个或多个子网相关联才能使用。若要将 *FrontEndSubnetRouteTable* 路由表关联到虚拟网络 *ProductionVnet* 中名为 *FrontEndSubnet* 的子网，请运行以下 PowerShell 命令：

```powershell
Set-AzureSubnetRouteTable -VirtualNetworkName ProductionVnet `
	-SubnetName FrontEndSubnet `
	-RouteTableName FrontEndSubnetRouteTable
```

### 如何查看在 VM 中应用的路由
你可以通过查询 Azure 来了解针对特定 VM 或角色实例应用的实际路由。所示路由包括 Azure 提供的默认路由，以及 VPN 网关所公布的路由。所示路由数目限制为 800。

若要查看与名为 *FWAppliance1* 的 VM 上的主 NIC 关联的路由，请运行以下 PowerShell 命令：

```powershell
Get-AzureVM -Name FWAppliance1 -ServiceName ProductionVMs `
	| Get-AzureEffectiveRouteTable
```

以上命令的输出应如下所示：

	Effective routes : 
	                   Name            Address Prefix    Next hop type    Next hop IP address Status   Source     
	                   ----            --------------    -------------    ------------------- ------   ------     
	                                   {10.0.0.0/8}      VNETLocal                            Active   Default    
	                                   {0.0.0.0/0}       Internet                             Active   Default    
	                                   {25.0.0.0/8}      Null                                 Active   Default    
	                                   {100.64.0.0/10}   Null                                 Active   Default    
	                                   {172.16.0.0/12}   Null                                 Active   Default    
	                                   {192.168.0.0/16}  Null                                 Active   Default    
	                   firewallroute   {10.2.0.0/16}     Null             10.1.1.10           Active   User      

若要查看与名为 *FWAppliance1* 的 VM 上名为 *backendnic* 的辅助 NIC 关联的路由，请运行以下 PowerShell 命令：

```powershell
Get-AzureVM -Name FWAppliance1 -ServiceName ProductionVMs `
	| Get-AzureEffectiveRouteTable -NetworkInterfaceName backendnic
```

若要查看与名为 *myRole* 的角色实例（属于名为 *ProductionVMs* 的云服务的一部分）上的主 NIC 关联的路由，请运行以下 PowerShell 命令：

```powershell
Get-AzureEffectiveRouteTable -ServiceName ProductionVMs `
	-RoleInstanceName myRole
```

## 如何管理 IP 转发
如前所述，你需要在任何 VM 或将要充当虚拟设备的角色实例上启用 IP 转发。

### 如何启用 IP 转发
若要在名为 *FWAppliance1* 的 VM 中启用 IP 转发，请运行以下 PowerShell 命令：

```powershell
Get-AzureVM -Name FWAppliance1 -ServiceName ProductionVMs `
	| Set-AzureIPForwarding -Enable
```

若要在名为 *FWAppliance* 的角色实例（位于名为 *DMZService* 的云服务中）中启用 IP 转发，请运行以下 PowerShell 命令：

```powershell
Set-AzureIPForwarding -ServiceName DMZService `
	-RoleName FWAppliance -Enable
```

### 如何禁用 IP 转发
若要在名为 *FWAppliance1* 的 VM 中禁用 IP 转发，请运行以下 PowerShell 命令：

```powershell
Get-AzureVM -Name FWAppliance1 -ServiceName ProductionVMs `
	| Set-AzureIPForwarding -Disable
```

若要在名为 *FWAppliance* 的角色实例（位于名为 *DMZService* 的云服务中）中禁用 IP 转发，请运行以下 PowerShell 命令：

```powershell
Set-AzureIPForwarding -ServiceName DMZService `
	-RoleName FWAppliance -Disable
```

### 如何查看 IP 转发的状态
若要在名为 *FWAppliance1* 的 VM 中查看 IP 转发的状态，请运行以下 PowerShell 命令：

```powershell
Get-AzureVM -Name FWAppliance1 -ServiceName ProductionVMs `
	| Get-AzureIPForwarding
``` 

<!---HONumber=70-->