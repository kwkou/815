<properties 
	pageTitle="使用 Powershell 设置 VNET 中的静态 IP" 
	description="本页介绍如何使用 PowerShell 来使用静态 IP。" 
	services="virtual network" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-network-aog" ms.date="" wacn.date="06/08/2016"/>

#使用 Powershell 设置 VNET 中的静态 IP 

###本文包含以下内容

- [对已有虚机设置静态 Internal IP](#exist)
- [取消对对已有虚机设置的静态 Internal IP](#cancle)
- [创建静态 Internal IP的虚机](#create)
- [使用中的注意点](#note)
 
**请注意**：以下操作需要下载最新版本的 Azure PowerShell（版本 0.8.5 及更高版本），请按照[此处](/documentation/articles/powershell-install-configure/)的说明进行安装。


## <a id="exist"></a>对已有虚机设置静态 Internal IP	

	PS C:\> $vm2=Get-AzureVM -servicename 'dnstest01' -name 'dnstest1'
	PS C:\> $vmchange=Set-AzureStaticVNetIP -vm $vm2 -IPAddress 10.0.1.4
	PS C:\> $vmchange |Update-AzureVM

或者：

	PS C:\> $VM2=Get-AzureVM -ServiceName 'dnstest01' -name 'test12' 
	PS C:\> Set-AzureStaticVNetIP -vm $vm2 -IPAddress 10.0.1.9 | Update-AzureVM
 
## <a id="cancle"></a>取消对已有虚机设置的静态 Internal IP 

	PS C:\> $VM2=Get-AzureVM -ServiceName 'dnstest01' -name 'test12'
	PS C:\> Remove-AzureStaticVNetIP $vm2 |Update-AzureVM
 
## <a id="create"></a>创建静态 Internal IP 的虚机

先设置默认存储账户：

	PS C:\> Set-AzureSubscription -SubscriptionName cranetest02 -CurrentStorageAccountName portalvhdszls6kbzqlcpdn

获取 Azure 平台提供的 VM 镜像：

	PS C:\> Get-AzureVMImage | Select ImageName


选择镜像并创建虚拟机：

*选择 Linux 虚拟机镜像并创建 Linux 虚机：

	PS C:\> $imagename='f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-65-20140121'	
	PS C:\> $vm1=New-AzureVMConfig -Name 'test12' -ImageName $imagename -InstanceSize Small |Add-AzureProvisioningConfig -Linux -LinuxUser 'crane' -Password '*****'; Set-AzureSubnet -VM $vm1 -SubnetNames 'testtest1'; Set-AzureStaticVNetIP -IPAddress 10.0.1.10 -VM $vm1; New-AzureVM -ServiceName 'test11' -vm $vm1 -VNetName 'test001'

*选择 Windows 虚拟机镜像并创建 Windows 虚机：

	PS C:\> $imagename='0c5c79005aae478e8883bf950a861ce0__Windows-Server-2012-Essentials-20131018-zhcn'
	PS C:\> $vm1=New-AzureVMConfig -Name 'test12' -ImageName $imagename -InstanceSize Small |Add-AzureProvisioningConfig -Windows -AdminUsername 'crane' -Password 'xxxxxxxx'; Set-AzureSubnet -VM $vm1 -SubnetNames 'testtest1'; Set-AzureStaticVNetIP -IPAddress 10.0.1.10 -VM $vm1; New-AzureVM -ServiceName 'test11' -vm $vm1 -VNetName 'test001'
 


## <a id="note"></a>使用中的注意点

1. Windows Azure 平台默认是关闭固定内网 IP 功能的。如果您需要开启固定虚拟机内网 IP 的功能，需要将新建的虚拟机建立在虚拟机网络内，通过 Windows Azure Powershell 命令语句设置静态 IP，参见相关链接： [如何设置静态内部专用 IP](/documentation/articles/virtual-networks-reserved-private-ip/)  
2. 请您最好对您在虚拟机网络下的所有虚拟机均设置该功能。尽量不要混合使用该功能：比如对有些虚拟机设置了固定 DIP，而有些则没有设置固定 DIP 功能。
3. 当您在管理 Portal 中对虚拟机进行停机后，由于该虚拟机的资源被释放，该虚拟机的 DIP 地址可能会重新分配给您在该虚拟网络中新建的虚拟机。
   - 如果您希望对虚拟机停机后，依然保留该虚拟机的 DIP 地址，请您在 Azure PowerShell 中使用`Stop-AzureVM -StayProvisioned -ServiceName xxxxxxx -Name xxxxxxx` 命令（注释：ServiceName 指虚拟机所在的云服务的名称；name 指该虚拟机的名称）对该虚拟机停机。如下图：
 		![](./media/aog-virtual-network-how-to-use-internal-ip/stop-vm-stay.jpg)
		停机后该虚拟机的状态在管理 Portal 上将显示为如下状态：
		![](./media/aog-virtual-network-how-to-use-internal-ip/stop-vm-stay-status.gif)
 
		**请注意**：由于资源不被释放，用这种方式停机将会对虚拟机继续收费。
   - 如果您希望释放该虚拟机的 DIP 资源，您可以在 Azure PowerShell 中使用`Stop-AzureVM -ServiceName xxxxxxx -Name xxxxxxx` 命令（注释：ServiceName 指虚拟机所在的云服务的名称；name 指该虚拟机的名称）对该虚拟机停机；
或者在管理 Portal,“虚拟机—仪表板”页面底部的“关闭”按钮，如下图所示：
 		![](./media/aog-virtual-network-how-to-use-internal-ip/stop-vm-shut.jpg)
		关闭后，该虚拟机在管理 Portal 上将显示为如下状态：
		![](./media/aog-virtual-network-how-to-use-internal-ip/stop-vm-shut-status.jpg)
		该虚拟机的资源将被释放。
