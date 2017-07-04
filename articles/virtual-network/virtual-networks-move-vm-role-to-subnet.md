<properties
    pageTitle="将 VM（经典）或云服务角色实例移动到其他子网 - Azure PowerShell | Azure"
    description="了解如何使用 PowerShell 将 VM（经典）和云服务角色实例移动到其他子网。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="de4135c7-dc5b-4ffa-84cc-1b8364b7b427"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/22/2016"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 通过 PowerShell，将 VM（经典）或云服务角色实例移动到其他子网
可使用 PowerShell 在同一虚拟网络 (VNet) 的不同子网之间移动 VM（经典）。可通过编辑 CSCFG 文件（而非使用 PowerShell）移动角色实例。

> [AZURE.NOTE]
本文介绍如何仅通过经典部署模型移动部署后的 VM。
> 
> 

为何要将 VM 移到另一个子网？ 当旧子网太小并且由于该子网中存在正在运行的 VM 而无法扩展时，子网迁移很有用。在这种情况下，可以创建一个更大的新子网，将 VM 迁移到新子网，然后在迁移完成后可删除空的旧子网。

## 如何将 VM 移到另一个子网
若要移动 VM，请使用以下示例作为模板运行 Set-AzureSubnet PowerShell cmdlet。在以下示例中，要将 TestVM 从其当前的子网移到 Subnet-2。请务必编辑此示例以反映所处环境。请注意，每当在过程中运行 Update-AzureVM cmdlet 时，都会在更新过程中重新启动 VM。

    Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
    | Set-AzureSubnet -SubnetNames Subnet-2 `
    | Update-AzureVM

如果为 VM 指定了静态内部专用 IP，则必须清除该设置，然后才能将 VM 移到新的子网。在这种情况下，请使用以下代码：

    Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
    | Remove-AzureStaticVNetIP `
    | Update-AzureVM
    Get-AzureVM -ServiceName TestVMCloud -Name TestVM `
    | Set-AzureSubnet -SubnetNames Subnet-2 `
    | Update-AzureVM

## 将角色实例移到另一个子网
若要移动角色实例，请编辑 CSCFG 文件。在以下示例中，要将虚拟网络 *VNETName* 中的“Role0”从其当前的子网移到 *Subnet-2*。由于已部署角色实例，只需将子网名称更改为“Subnet-2”。请务必编辑此示例以反映所处环境。

    <NetworkConfiguration>
        <VirtualNetworkSite name="VNETName" />
        <AddressAssignments>
           <InstanceAddress roleName="Role0">
                <Subnets><Subnet name="Subnet-2" /></Subnets>
           </InstanceAddress>
        </AddressAssignments>
    </NetworkConfiguration>

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->