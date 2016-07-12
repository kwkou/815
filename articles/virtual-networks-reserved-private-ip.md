<properties 
   pageTitle="如何设置静态内部专用 IP"
   description="了解静态内部 IP (DIP) 以及如何对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="03/22/2016"
	wacn.date="05/24/2016"/>

# 如何设置静态内部专用 IP
大多数情况下，你不需要指定虚拟机的静态内部 IP 地址。虚拟网络中的 VM 将自动从你所指定的范围接收内部 IP 地址。但在某些情况下，需要为特定 VM 指定静态 IP 地址。例如，在你的 VM 需要运行 DNS 或将要成为域控制器的情况下。

>[AZURE.NOTE]静态内部 IP 地址会始终与 VM 关联在一起，即使经历“停止/取消预配”状态变化。

## 如何验证特定 IP 地址是否可用：
若要验证 IP 地址 *10.0.0.7* 在名为 *TestVnet* 的 VNet 中是否可用，请运行以下 PowerShell 命令并验证 *IsAvailable* 的值：

	Test-AzureStaticVNetIP –VNetName TestVNet –IPAddress 10.0.0.7 

	IsAvailable          : True
	AvailableAddresses   : {}
	OperationDescription : Test-AzureStaticVNetIP
	OperationId          : fd3097e1-5f4b-9cac-8afa-bba1e3492609
	OperationStatus      : Succeeded

>[AZURE.NOTE]如果你想要在安全的环境中测试上述命令，请遵循[创建虚拟网络](/documentation/articles/virtual-networks-create-vnet/)中的准则创建名为 *TestVnet* 的 VNet，并确保其使用 *10.0.0.0/8* 地址空间。

## 如何在创建 VM 时指定静态内部 IP
下面的 PowerShell 脚本将创建名为 *TestService* 的全新云服务，然后从 Azure 中检索映像，接着在新的云服务中使用检索的映像创建名为 *TestVM* 的 VM，对该 VM 进行设置，使之位于名为 *Subnet-1* 的子网中，最后再将 *10.0.0.7* 设置为 VM 的静态内部 IP：

	New-AzureService -ServiceName TestService -Location "China North"
	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| Set-AzureSubnet –SubnetNames Subnet-1 `
	| Set-AzureStaticVNetIP -IPAddress 10.0.0.7 `
	| New-AzureVM -ServiceName "TestService" –VNetName TestVnet

## 如何检索 VM 的静态内部 IP 信息
若要查看使用上述脚本创建的 VM 的静态内部 IP 信息，请运行以下 PowerShell 命令，然后观察 *IpAddress* 的值：

	Get-AzureVM -Name TestVM -ServiceName TestService

	DeploymentName              : TestService
	Name                        : TestVM
	Label                       : 
	VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
	InstanceStatus              : Provisioning
	IpAddress                   : 10.0.0.7
	InstanceStateDetails        : Windows is preparing your computer for first use...
	PowerState                  : Started
	InstanceErrorCode           : 
	InstanceFaultDomain         : 0
	InstanceName                : TestVM
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

## 如何从 VM 中删除静态内部 IP
若要删除在上述脚本中添加到 VM 中的静态内部 IP，请运行以下 PowerShell 命令：
	
	Get-AzureVM -ServiceName TestService -Name TestVM `
	| Remove-AzureStaticVNetIP `
	| Update-AzureVM

## 如何向现有 VM 添加静态内部 IP
若要向使用以上脚本创建的 VM 添加静态内部 IP，请运行以下命令：

	Get-AzureVM -ServiceName TestService000 -Name TestVM `
	| Set-AzureStaticVNetIP -IPAddress 10.10.0.7 `
	| Update-AzureVM

## 后续步骤

[保留 IP](/documentation/articles/virtual-networks-reserved-public-ip/)

[实例级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/)

[保留 IP REST API](https://msdn.microsoft.com/zh-CN/library/azure/dn722420.aspx)

<!---HONumber=70-->