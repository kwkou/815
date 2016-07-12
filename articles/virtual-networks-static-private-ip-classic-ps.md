<properties 
   pageTitle="如何在经典模式下使用 PowerShell 设置静态专用 IP | Azure"
   description="了解静态专用 IP (DIP) 以及如何在经典模式下使用 PowerShell 对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="03/28/2016"/>

# 如何在 PowerShell 中设置静态专用 IP 地址（经典）

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../includes/virtual-networks-static-private-ip-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[管理资源管理器部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-arm-ps/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../includes/virtual-networks-static-ip-scenario-include.md)]

下面的示例 PowerShell 命令需要已创建简单的环境。首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/) 中所述的测试环境。

## 如何验证特定 IP 地址是否可用：
若要验证 IP 地址 *192.168.1.101* 在名为 *TestVnet* 的 VNet 中是否可用，请运行以下 PowerShell 命令并验证 *IsAvailable* 的值：

	Test-AzureStaticVNetIP –VNetName TestVNet –IPAddress 192.168.1.101 

预期输出：

	IsAvailable          : True
	AvailableAddresses   : {}
	OperationDescription : Test-AzureStaticVNetIP
	OperationId          : fd3097e1-5f4b-9cac-8afa-bba1e3492609
	OperationStatus      : Succeeded

## 如何在创建 VM 时指定静态专用 IP 地址
下面的 PowerShell 脚本将创建名为 *TestService* 的全新云服务，然后从 Azure 中检索映像，在新的云服务中使用检索到的映像创建名为 *DNS01* 的 VM，对该 VM 进行设置，使之位于名为 *FrontEnd* 的子网中，最后再将 *192.168.1.7* 设置为该 VM 的静态专用 IP 地址：

	New-AzureService -ServiceName TestService -Location "China North"
	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name DNS01 -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| Set-AzureSubnet –SubnetNames FrontEnd `
	| Set-AzureStaticVNetIP -IPAddress 192.168.1.7 `
	| New-AzureVM -ServiceName "TestService" –VNetName TestVNet

预期输出：

	WARNING: No deployment found in service: 'TestService'.
	OperationDescription OperationId                          OperationStatus
	-------------------- -----------                          ---------------
	New-AzureService     fcf705f1-d902-011c-95c7-b690735e7412 Succeeded      
	New-AzureVM          3b99a86d-84f8-04e5-888e-b6fc3c73c4b9 Succeeded  

## 如何检索 VM 的静态专用 IP 地址信息
若要查看使用上述脚本创建的 VM 的静态专用 IP 地址信息，请运行以下 PowerShell 命令，然后观察 *IpAddress* 的值：

	Get-AzureVM -Name DNS01 -ServiceName TestService

预期输出：

	DeploymentName              : TestService
	Name                        : DNS01
	Label                       : 
	VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
	InstanceStatus              : Provisioning
	IpAddress                   : 192.168.1.7
	InstanceStateDetails        : Windows is preparing your computer for first use...
	PowerState                  : Started
	InstanceErrorCode           : 
	InstanceFaultDomain         : 0
	InstanceName                : DNS01
	InstanceUpgradeDomain       : 0
	InstanceSize                : Small
	HostName                    : rsR2-797
	AvailabilitySetName         : 
	DNSName                     : http://testservice000.chinacloudapp.cn/
	Status                      : Provisioning
	GuestAgentStatus            : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.GuestAgentStatus
	ResourceExtensionStatusList : {Microsoft.Compute.BGInfo}
	PublicIPAddress             : 
	PublicIPName                : 
	NetworkInterfaces           : {}
	ServiceName                 : TestService
	OperationDescription        : Get-AzureVM
	OperationId                 : 34c1560a62f0901ab75cde4fed8e8bd1
	OperationStatus             : OK

## 如何从 VM 中删除静态专用 IP 地址
若要删除使用上述脚本添加到 VM 的静态专用 IP 地址，请运行以下 PowerShell 命令：
	
	Get-AzureVM -ServiceName TestService -Name DNS01 `
	| Remove-AzureStaticVNetIP `
	| Update-AzureVM

预期输出：

	OperationDescription OperationId                          OperationStatus
	-------------------- -----------                          ---------------
	Update-AzureVM       052fa6f6-1483-0ede-a7bf-14f91f805483 Succeeded

## 如何将静态专用 IP 地址添加到现有 VM
若要向使用上述脚本创建的 VM 添加静态专用 IP 地址，请运行以下命令：

	Get-AzureVM -ServiceName TestService -Name DNS01 `
	| Set-AzureStaticVNetIP -IPAddress 192.168.1.7 `
	| Update-AzureVM

预期输出：

	OperationDescription OperationId                          OperationStatus
	-------------------- -----------                          ---------------
	Update-AzureVM       77d8cae2-87e6-0ead-9738-7c7dae9810cb Succeeded 

## 后续步骤

- 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
- 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
- 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=76-->