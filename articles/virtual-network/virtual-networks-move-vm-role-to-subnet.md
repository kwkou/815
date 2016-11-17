<properties 
   pageTitle="如何将 VM 或角色实例移到其他子网"
   description="了解如何将 VM 和角色实例移到其他子网"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="03/22/2016"
	wacn.date="05/24/2016"/>

# 如何将 VM 或角色实例移到其他子网

可以使用 PowerShell 将 VM 在同一虚拟网络 (VNet) 中的子网之间移动。可以通过编辑 CSCFG（而不是使用 PowerShell）移动角色实例。

为何要将 VM 移到另一个子网？ 当旧子网太小并且由于该子网中存在正在运行的 VM 而无法扩展时，子网迁移很有用。在这种情况下，你可以创建一个更大的新子网，将 VM 迁移到新子网，然后在迁移完成后，可以删除空的旧子网。

## 如何将 VM 移到另一个子网

若要移动 VM，请使用下面的示例作为模板运行 Set-AzureSubnet PowerShell cmdlet。在下面的示例中，我们将 TestVM 从其当前的子网移到 Subnet-2。请务必编辑此示例以反映你的环境。请注意，每当你在过程中运行 Update-AzureVM cmdlet 时，都会在更新过程中重新启动 VM。

	Get-AzureVM –ServiceName TestVMCloud –Name TestVM `
	| Set-AzureSubnet –SubnetNames Subnet-2 `
	| Update-AzureVM

如果你为 VM 指定了静态内部私有 IP，则必须清除该设置，然后才能将 VM 移到新子网。在这种情况下，请使用以下代码：

	Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
	| Remove-AzureStaticVNetIP `
	| Update-AzureVM
	Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
	| Set-AzureSubnet -SubnetNames Subnet-2 `
	| Update-AzureVM

## 将角色实例移到另一个子网

若要移动角色实例，请编辑 CSCFG 文件。在以下示例中，我们要将虚拟网络 *VNETName* 中的“Role0”从其当前的子网移到 *Subnet-2*。由于已部署角色实例，你只需将子网名称更改为“Subnet-2”。请务必编辑此示例以反映你的环境。

	<NetworkConfiguration>
	    <VirtualNetworkSite name="VNETName" />
	    <AddressAssignments>
	       <InstanceAddress roleName="Role0">
	            <Subnets><Subnet name="Subnet-2" /></Subnets>
	       </InstanceAddress>
	    </AddressAssignments>
	</NetworkConfiguration> 

<!---HONumber=74-->