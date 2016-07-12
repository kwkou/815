<properties 
   pageTitle="保留 IP | Azure"
   description="了解保留 IP 以及如何对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="02/10/2016"
	wacn.date="03/17/2016"/>

# 保留 IP 概述
Azure 中的 IP 地址分为两类：动态 IP 地址和保留 IP 地址。由 Azure 管理的公共 IP 地址默认为动态 IP 地址。这意味着，用于给定云服务的 IP 地址 (VIP) 或用于直接访问 VM 或角色示例的 IP 地址 (ILPIP) 可能会在关闭资源或释放资源的情况下不时进行更改。

若要防止 IP 地址更改，可将其设置为保留 IP 地址。保留 IP 只能用作 VIP，可确保云服务的 IP 地址即使在关闭资源或释放资源的情况下也是相同的。此外，你还可以将用作 VIP 的现有动态 IP 转换为保留 IP 地址。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-network-ip-addresses-overview-arm/)。

请确保你了解 [IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-classic/)在 Azure 中的工作原理。

## 何时需要保留 IP？
- **你想要确保订阅中的 IP 为保留 IP**。如果你想要保留一个 IP 地址，使得该 IP 地址在任何情况下都不会从你的订阅中释放，则应使用保留的公共 IP。  
- **你想要 IP 始终与云服务相关联，即使 VM 处于停止或释放状态下**。如果你想要通过即使在云服务中的 VM 处于停止或释放状态下也不会更改的 IP 地址来访问服务。
- **你想要确保 Azure 的出站流量使用可预测的 IP 地址**。你可以将本地防火墙配置为仅允许来自特定 IP 地址的流量。通过对 IP 进行保留，你就可以了解源 IP 地址，不必因 IP 更改而更新防火墙规则。

## 常见问题
1. 可以将保留 IP 用于所有 Azure 服务吗？  
  - 保留 IP 只能用于通过 VIP 公开的 VM 和云服务实例角色。
1. 我可以有多少个保留 IP？  
  - 目前，所有 Azure 订阅都有权使用 20 个保留 IP。不过，你可以请求更多的保留 IP。请参阅[订阅和服务限制](/documentation/articles/azure-subscription-service-limits/)页以获取更多的信息。
1. 保留 IP 是否收费？ 
  - 请参阅[保留 IP 地址定价详细信息](/pricing/details/reserved-ip-addresses)以获取定价信息。
1. 如何保留某个 IP 地址？ 
  - 你可以使用 PowerShell 或 [Azure 管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx) 来请求特定区域的保留 IP。Azure 会保留该区域的 IP 地址并将其关联到你的订阅。然后，你就可以使用该区域的保留 IP。你不能使用经典管理门户来保留 IP 地址。
1. 我可以将保留 IP 用于基于地缘组的 VNet 吗？ 
  - 仅区域 VNet 支持保留 IP。与地缘组关联的 VNet 不支持保留 IP。有关如何将 VNet 与区域或地缘组关联的详细信息，请参阅[关于区域 VNet 和地缘组](/documentation/articles/virtual-networks-migrate-to-regional-vnet/)。 

## 如何管理保留 VIP

在使用保留 IP 之前，必须先将其添加到订阅。若要从*中国北部*位置中提供的公共 IP 地址池创建保留 IP，请运行以下 PowerShell 命令：

	New-AzureReservedIP -ReservedIPName MyReservedIP -Location "China North"

但请注意，你不能指定要保留的具体 IP。若要查看你的订阅中哪些 IP 地址为保留 IP 地址，请运行以下 PowerShell 命令，然后注意观察 *ReservedIPName* 和 *Address* 的值：

	Get-AzureReservedIP

	ReservedIPName       : MyReservedIP
	Address              : 23.101.114.211
	Id                   : d73be9dd-db12-4b5e-98c8-bc62e7c42041
	Label                : 
	Location             : China North
	State                : Created
	InUse                : False
	ServiceName          : 
	DeploymentName       : 
	OperationDescription : Get-AzureReservedIP
	OperationId          : 55e4f245-82e4-9c66-9bd8-273e815ce30a
	OperationStatus      : Succeeded

某个 IP 成为保留 IP 以后，它就会始终与你的订阅相关联，直至你将它删除。若要删除如上所示的保留 IP，请运行以下 PowerShell 命令：

	Remove-AzureReservedIP -ReservedIPName "MyReservedIP"

## 如何将保留 IP 关联到已存在的云服务

你可以添加 “-ServiceName” 参数，把保留 IP 关联到已存在的云服务。把保留 IP 关联到名为 “TestService”，在“中国北部”的云服务，请运行以下 PowerShell 命令：

	New-AzureReservedIP -ReservedIPName MyReservedIP -Location "China North" -ServiceName TestService


## 如何将保留 IP 关联到新的云服务
下面的脚本将创建新的保留 IP，然后将其关联到新的名为 *TestService* 的云服务。

	New-AzureReservedIP -ReservedIPName MyReservedIP -Location "China North"
	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| New-AzureVM -ServiceName TestService -ReservedIPName MyReservedIP -Location "China North"

>[AZURE.NOTE] 创建用于云服务的保留 IP 时，仍需使用 *VIP:&lt;端口号>* 来引用 VM，以便进行入站通信。使用保留 IP 并不意味着你可以直接连接到 VM。保留 IP 将分配给 VM 所部署到的云服务。如果你想要直接通过 IP 连接到 VM，则必须配置实例级公共 IP。实例级公共 IP 是一类可直接分配给 VM 的公共 IP（称为 ILPIP）。它不能保留。有关详细信息，请参阅[实例级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/)。

## 如何从正在运行的部署中删除保留 IP
若要删除已添加到以上脚本中创建的新服务中的保留 IP，请运行以下 PowerShell 命令：

	Remove-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService

>[AZURE.NOTE] 从正在运行的部署中删除保留 IP 并不会从你的订阅中删除保留 IP。它只是释放该 IP，以供订阅中的其他资源使用。

## 如何将保留 IP 关联到正在运行的部署
下面的脚本将创建名为 *TestService2* 的新的云服务，以及名为 *TestVM2* 的新的 VM，然后将名为 *MyReservedIP* 的现有保留 IP 关联到云服务。

	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM2 -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| New-AzureVM -ServiceName TestService2 -Location "China North"
	Set-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService2

## 如何使用服务配置文件将保留 IP 关联到云服务
你也可以使用服务配置 (CSCFG) 文件将保留 IP 关联到云服务。下面的示例 xml 显示了如何将云服务配置为使用名为 *MyReservedIP* 的保留 VIP：
	
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceConfiguration serviceName="ReservedIPSample" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2014-01.2.3">
	  <Role name="WebRole1">
	    <Instances count="1" />
	    <ConfigurationSettings>
	      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
	    </ConfigurationSettings>
	  </Role>
	  <NetworkConfiguration>
	    <AddressAssignments>
	      <ReservedIPs>
	       <ReservedIP name="MyReservedIP"/>
	      </ReservedIPs>
	    </AddressAssignments>
	  </NetworkConfiguration>
	</ServiceConfiguration>

## 后续步骤

- 了解 [IP 寻址](/documentation/articles/virtual-network-ip-addresses-overview-classic/)在经典部署模型中的工作原理。

- 了解[保留专用 IP 地址](/documentation/articles/virtual-networks-reserved-private-ip/)。

- 了解[实例级公共 IP (ILPIP) 地址](/documentation/articles/virtual-networks-instance-level-public-ip/)。

<!---HONumber=Mooncake_0307_2016-->