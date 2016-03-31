<properties
	pageTitle="使用 Azure PowerShell 创建 Linux VM | Azure"
	description="了解如何使用 Azure PowerShell 创建和预配置 Linux VM。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="11/11/2015"
	wacn.date="12/31/2015"/>


# 使用 Azure PowerShell 创建和预配置 Linux 虚拟机

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-machines-ps-create-preconfigure-linux-vms)


[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]
 
下列步骤演示如何通过使用填空方法创建 Azure PowerShell 命令集来创建 Linux 虚拟机。如果你不熟悉 Azure PowerShell 或只想知道为成功的配置指定什么值，则此方法很有用。

你将通过将几组命令块复制到文本文件或 PowerShell ISE 中，然后填写变量值并删除 < and > 字符来生成命令集。请参阅本文末尾的两个[示例](#examples)，了解最终结果。

有关基于 Windows 的虚拟机的配套主题，请参阅[使用 Azure PowerShell 创建基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)。

## 安装 Azure PowerShell

如果你尚未安装，请[安装并配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。然后，打开 Azure PowerShell 命令提示符。

## 设置订阅和存储帐户

通过在 Azure PowerShell 命令提示符下运行以下命令设置你的 Azure 订阅和存储帐户。

你可以从 **Get-AzureSubscription** 命令输出的 **SubscriptionName** 属性获取相应的订阅名称。

发出 Select-AzureSubscription 命令后，你可以从 **Get-AzureStorageAccount** 命令输出的 **Label** 属性获取相应的存储帐户名称。

将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<subscription name>"
	$staccount="<storage account name>"
	Select-AzureSubscription -SubscriptionName $subscr –Current
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscr -CurrentStorageAccountName $staccount


## 找到你要使用的映像

接下来，你需要确定要使用的映像的 ImageFamily 值。你可以使用以下命令获取可用 ImageFamily 值的列表。

	Get-AzureVMImage | select ImageFamily -Unique

下面是基于 Linux 的计算机的 ImageFamily 值的一些示例：

- Ubuntu Server 12.10
- CoreOS Alpha
- SUSE Linux Enterprise Server 12

打开所选文本编辑器的一个新实例或 PowerShell 集成脚本环境 (ISE) 的一个实例。将以下内容复制到新的文本文件或 PowerShell ISE，并替换 ImageFamily 值。

	$family="<ImageFamily value>"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

## 指定名称、大小和（可选）可用性集

通过选择这两个命令块之一启动命令集（必需）。

**选项 1**：指定虚拟机名称和大小。

	$vmname="<machine name>"
	$vmsize="<Specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7>"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

**选项 2**：指定名称、大小和可用性集名称。

	$vmname="<machine name>"
	$vmsize="<Specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7>"
	$availset="<set name>"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image -AvailabilitySetName $availset

有关 D、DS 或 G 系列虚拟机的 InstanceSize 值，请参阅 [Azure 的虚拟机和云服务大小](https://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)。


## 设置用户访问安全选项

**选项 1**：指定初始 Linux 用户名和密码（必需）。选择一个强密码。若要检查其强度，请参阅[密码检查器：使用强密码](https://www.microsoft.com/security/pc-security/password-checker.aspx)。

	$cred=Get-Credential -Message "Type the name and password of the initial Linux account."
	$vm1 | Add-AzureProvisioningConfig -Linux -LinuxUser $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

**选项 2**：指定一组已在订阅中部署的 SSH 密钥对。

	$vm1 | Add-AzureProvisioningConfig -Linux -SSHKeyPairs "<SSH key pairs>"

有关详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-use-ssh-key)。

**选项 3**：指定已在订阅中部署的 SSH 公钥的列表。

	$vm1 | Add-AzureProvisioningConfig -Linux - SSHPublicKeys "<SSH public keys>"

有关基于 Linux 的虚拟机的其他预配置选项，请参阅 [Add-AzureProvisioningConfig](https://msdn.microsoft.com/zh-cn/library/azure/dn495299.aspx) 中 **Linux** 参数集的语法。


## 可选：分配一个静态 DIP

（可选）为虚拟机分配一个特定 IP 地址（称为静态 DIP）。

	$vm1 | Set-AzureStaticVNetIP -IPAddress <IP address>

可以使用以下命令来验证特定 IP 地址是否可用。

	Test-AzureStaticVNetIP –VNetName <VNet name> –IPAddress <IP address>

## 可选：将虚拟机分配到特定子网 

将虚拟机分配到 Azure 虚拟网络中的特定子网。

	$vm1 | Set-AzureSubnet -SubnetNames "<name of the subnet>"

	
## 可选：添加数据磁盘
	
将以下代码添加到命令集以将数据磁盘添加到虚拟机。

	$disksize=<size of the disk in GB>
	$disklabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$hcaching="<Specify one: ReadOnly, ReadWrite, None>"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

## 可选：将虚拟机添加到现有负载平衡集 

将以下代码添加到命令集以将虚拟机添加到用于外部流量的现有负载平衡集。

	$prot="<Specify one: tcp, udp>"
	$localport=<port number of the internal port>
	$pubport=<port number of the external port>
	$endpointname="<name of the endpoint>"
	$lbsetname="<name of the existing load-balanced set>"
	$probeprotocol="<Specify one: tcp, http>"
	$probeport=<TCP or HTTP port number of probe traffic>
	$probepath="<URL path for probe traffic>"
	$vm1 | Add-AzureEndpoint -Name $endpointname -Protocol $prot -LocalPort $localport -PublicPort $pubport -LBSetName $lbsetname -ProbeProtocol $probeprotocol -ProbePort $probeport -ProbePath $probepath

## 决定如何启动虚拟机创建过程 

通过选择以下命令块之一将命令块添加到命令集以启动虚拟机创建过程。

**选项 1**：在现有云服务中创建虚拟机。

	New-AzureVM –ServiceName "<short name of the cloud service>" -VMs $vm1

云服务的短名称是在 Azure 门户的 Azure 云服务列表中或 Azure 门户的资源组列表中显示的名称。

**选项 2**：在现有的云服务和虚拟网络中创建虚拟机。

	$svcname="<short name of the cloud service>"
	$vnetname="<name of the virtual network>"
	New-AzureVM –ServiceName $svcname -VMs $vm1 -VNetName $vnetname

## 运行命令集

复查你在文本编辑器或 PowerShell ISE 中生成的 Azure PowerShell 命令集并确保你指定了所有变量，并且它们具有正确的值。另请确保你已删除所有 < and > 字符。

将该命令集复制到剪贴板，然后右键单击打开的 Azure PowerShell 命令提示符。这将发出作为一系列 PowerShell 命令的命令集，并创建 Azure 虚拟机。

创建虚拟机后，请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。

如果要重复使用此命令集，你可以：

- 将此命令集保存为 PowerShell 脚本文件 (*.ps1)
- 在 Azure 门户的“自动化”部分中将此命令集保存为 Azure Automation Runbook

## <a id="examples"></a>示例

以下是使用上述步骤生成 Azure PowerShell 命令集以在 Azure 中创建基于 Linux 的虚拟机的两个示例。

### 示例 1

我需要 PowerShell 命令集来为 MySQL 服务器创建满足以下条件的初始 Linux 虚拟机：

- 使用 Ubuntu Server 12.10 映像。
- 具有名称 AZMYSQL1。
- 具有 500 GB 的附加数据磁盘。
- 具有静态 IP 地址 192.168.244.4。
- 位于 AZDatacenter 虚拟网络的 BackEnd 子网中。
- 位于 Azure-TailspinToys 云服务中。

下面是用于创建此虚拟机的相应 Azure PowerShell 命令集，为了便于阅读，每个程序块之间留有空行。

	$family="Ubuntu Server 12.10"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

	$vmname="AZMYSQL1"
	$vmsize="Large"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

	$cred=Get-Credential -Message "Type the name and password of the initial Linux account."
	$vm1 | Add-AzureProvisioningConfig -Linux -LinuxUser $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

	$vm1 | Set-AzureSubnet -SubnetNames "BackEnd"

	$vm1 | Set-AzureStaticVNetIP -IPAddress 192.168.244.4

	$disksize=500
	$disklabel="MySQLData"
	$lun=0
	$hcaching="None"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

	$svcname="Azure-TailspinToys"
	$vnetname="AZDatacenter"
	New-AzureVM –ServiceName $svcname -VMs $vm1 -VNetName $vnetname

### 示例 2

我需要 PowerShell 命令集来为 Apache 服务器创建满足以下条件的 Linux 虚拟机：

- 使用 SUSE Linux Enterprise Server 12 映像。
- 具有名称 LOB1。
- 具有 50 GB 的附加数据磁盘。
- 是用于标准 Web 流量的 LOBServers 负载平衡器集的成员。
- 位于 AZDatacenter 虚拟网络的 FrontEnd 子网中。
- 位于 Azure-TailspinToys 云服务中。

下面是用于创建此虚拟机的相应 Azure PowerShell 命令集。

	$family="SUSE Linux Enterprise Server 12"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

	$vmname="LOB1"
	$vmsize="Medium"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

	$cred=Get-Credential -Message "Type the name and password of the initial Linux account."
	$vm1 | Add-AzureProvisioningConfig -Linux -LinuxUser $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

	$vm1 | Set-AzureSubnet -SubnetNames "FrontEnd"

	$disksize=50
	$disklabel="LOBApp"
	$lun=0
	$hcaching="ReadWrite"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

	$prot="tcp"
	$localport=80
	$pubport=80
	$endpointname="LOB1"
	$lbsetname="LOBServers"
	$probeprotocol="tcp"
	$probeport=80
	$probepath="/"
	$vm1 | Add-AzureEndpoint -Name $endpointname -Protocol $prot -LocalPort $localport -PublicPort $pubport -LBSetName $lbsetname -ProbeProtocol $probeprotocol -ProbePort $probeport -ProbePath $probepath

	$svcname="Azure-TailspinToys"
	$vnetname="AZDatacenter"
	New-AzureVM –ServiceName $svcname -VMs $vm1 -VNetName $vnetname

## 其他资源

[虚拟机文档](/documentation/services/virtual-machines/)

[Azure 虚拟机常见问题](/documentation/articles/virtual-machines-questions)

[Azure 虚拟机概述](/documentation/articles/virtual-machines-about/)

[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)

[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)

[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)

<!---HONumber=Mooncake_1221_2015-->