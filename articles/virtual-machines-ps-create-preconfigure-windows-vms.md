<properties
	pageTitle="使用 PowerShell 创建 Windows VM | Azure"
	description="使用 Azure PowerShell 和经典部署模型创建 Windows 虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/05/2016"
	wacn.date="06/29/2016"/>

# 使用 Powershell 和经典部署模型创建 Windows 虚拟机 

> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/virtual-machines-windows-classic-tutorial/)
- [PowerShell - Windows](/documentation/articles/virtual-machines-windows-classic-create-powershell/)

<br>


> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-windows-ps-create/)。


这些步骤演示了如何使用构建基块方法自定义一组 Azure PowerShell 命令以创建和预配置基于 Windows 的 Azure 虚拟机。可以使用此过程快速创建用于新的基于 Windows 的虚拟机的命令集并扩展现有部署，或者创建多个命令集以快速构建出自定义开发/测试或 IT 专业环境。

这些步骤采用填空方法来创建 Azure PowerShell 命令集。如果你不熟悉 PowerShell 或只想知道为成功的配置指定什么值，则此方法很有用。高级 PowerShell 用户可以使用命令并将变量（以“$”开头的行）替换为他们自己的值。

如果你尚未这样做，现在请按[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中的说明在本地计算机上安装 Azure PowerShell。然后，打开 Windows PowerShell 命令提示符。

## 步骤 1：添加帐户

1. 在 PowerShell 命令提示下，键入 “Add-AzureAccount” 并单击“Enter”。 
2. 键入与你的 Azure 订阅相关联的电子邮件地址并单击“继续”。 
3. 键入你的帐户的密码。 
4. 单击“登录”。

## 步骤 2：设置订阅和存储帐户

通过在 Windows PowerShell 命令提示符下运行以下命令设置你的 Azure 订阅和存储帐户。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<subscription name>"
	$staccount="<storage account name>"
	Select-AzureSubscription -SubscriptionName $subscr -Current
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscr -CurrentStorageAccountName $staccount

你可以从 **Get-AzureSubscription** 命令输出的 SubscriptionName 属性获取相应的订阅名称。运行 **Select-AzureSubscription** 命令后，你可以从 **Get-AzureStorageAccount** 命令输出的 Label 属性获取相应的存储帐户名称。

## 步骤 3：确定 ImageFamily

接下来，你需要确定与要创建的 Azure 虚拟机对应的特定映像的 ImageFamily 或 Label 值。你可以使用此命令获取可用 ImageFamily 值的列表。

	Get-AzureVMImage | select ImageFamily -Unique

下面是基于 Windows 的计算机的 ImageFamily 值的一些示例：

- Windows Server 2012 R2 Datacenter
- Windows Server 2008 R2 SP1
- Windows Server 2016 Technical Preview 4
- Windows Server 2012 上的 SQL Server 2102 SP1 Enterprise

如果你找到要查找的映像，请打开所选文本编辑器的一个新实例或 PowerShell 集成脚本环境 (ISE)。将以下内容复制到新的文本文件或 PowerShell ISE，并替换 ImageFamily 值。

	$family="<ImageFamily value>"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

在某些情况下，映像名称在 Label 属性中而不是 ImageFamily 值。如果你使用 ImageFamily 属性找不到要查找的映像，请使用此命令按映像的 Label 属性列出映像。

	Get-AzureVMImage | select Label -Unique

如果使用此命令找到正确的映像，请打开所选文本编辑器的一个新实例或 PowerShell ISE。将以下内容复制到新的文本文件或 PowerShell ISE，并替换 Label 值。

	$label="<Label value>"
	$image = Get-AzureVMImage | where { $_.Label -eq $label } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

## 步骤 4：生成命令集

通过将下面相应的一组程序块复制到新文本文件或 ISE 中，然后填写变量值并删除 < and > 字符来生成命令集的其余部分。请参阅本文末尾的两个[示例](#examples)，了解最终结果。

通过选择这两个命令块之一启动命令集（必需）。

选项 1：指定虚拟机名称和大小。

	$vmname="<machine name>"
	$vmsize="<Specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7>"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

选项 2：指定名称、大小和可用性集名称。

	$vmname="<machine name>"
	$vmsize="<Specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7>"
	$availset="<set name>"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image -AvailabilitySetName $availset

有关 D、DS 或 G 系列虚拟机的 InstanceSize 值，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/cloud-services-sizes-specs/)。

（可选）为独立 Windows 计算机指定本地管理员帐户和密码。

	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.Username -Password $cred.GetNetworkCredential().Password

 选择一个强密码。若要检查其强度，请参阅[密码检查器：使用强密码](https://www.microsoft.com/security/pc-security/password-checker.aspx)。

（可选）若要将 Windows 计算机添加到现有的 Active Directory 域，请指定本地管理员帐户和密码、域以及域帐户的名称和密码。

	$cred1=Get-Credential -Message "Type the name and password of the local administrator account."
	$cred2=Get-Credential -Message "Now type the name (not including the domain) and password of an account that has permission to add the machine to the domain."
	$domaindns="<FQDN of the domain that the machine is joining>"
	$domacctdomain="<domain of the account that has permission to add the machine to the domain>"
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $domacctdomain -DomainUserName $cred2.Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domaindns

有关基于 Windows 的虚拟机的其他预配置选项，请参阅 [Add-AzureProvisioningConfig](https://msdn.microsoft.com/zh-cn/library/azure/dn495299.aspx) 中 **Windows** 和 **WindowsDomain** 参数集的语法。

（可选）为虚拟机分配一个特定 IP 地址（称为静态 DIP）。

	$vm1 | Set-AzureStaticVNetIP -IPAddress <IP address>

可以使用以下命令来验证特定 IP 地址是否可用：

	Test-AzureStaticVNetIP -VNetName <VNet name> -IPAddress <IP address>

（可选）将虚拟机分配到 Azure 虚拟网络中的特定子网。

	$vm1 | Set-AzureSubnet -SubnetNames "<name of the subnet>"

（可选）将单个数据磁盘添加到虚拟机。

	$disksize=<size of the disk in GB>
	$disklabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$hcaching="<Specify one: ReadOnly, ReadWrite, None>"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

对于 Active Directory 域控制器，将 $hcaching 设为“None”。

（可选）将虚拟机添加到用于外部流量的现有负载平衡集。

	$protocol="<Specify one: tcp, udp>"
	$localport=<port number of the internal port>
	$pubport=<port number of the external port>
	$endpointname="<name of the endpoint>"
	$lbsetname="<name of the existing load-balanced set>"
	$probeprotocol="<Specify one: tcp, http>"
	$probeport=<TCP or HTTP port number of probe traffic>
	$probepath="<URL path for probe traffic>"
	$vm1 | Add-AzureEndpoint -Name $endpointname -Protocol $protocol -LocalPort $localport -PublicPort $pubport -LBSetName $lbsetname -ProbeProtocol $probeprotocol -ProbePort $probeport -ProbePath $probepath

最后，请选择这些必需的命令块之一来创建虚拟机。

选项 1：在现有云服务中创建虚拟机。

	New-AzureVM -ServiceName "<short name of the cloud service>" -VMs $vm1

云服务的短名称是在 Azure 经典管理门户的云服务列表中。

选项 2：在现有的云服务和虚拟网络中创建虚拟机。

	$svcname="<short name of the cloud service>"
	$vnetname="<name of the virtual network>"
	New-AzureVM -ServiceName $svcname -VMs $vm1 -VNetName $vnetname

## 步骤 5：运行命令集

复查你在步骤 4 中在文本编辑器或 PowerShell ISE 中生成的包含多个命令块的 Azure PowerShell 命令集。确保你指定了所需的所有变量，并且这些变量具有正确的值。另请确保你已删除所有 < and > 字符。

如果你使用的是文本编辑器，则将命令集复制到剪贴板，然后右键单击打开的 Windows PowerShell 命令提示符。这将发出作为一系列 PowerShell 命令的命令集，并创建 Azure 虚拟机。或者，在 PowerShell ISE 中运行命令集。

如果你要再次创建此虚拟机或类似的虚拟机，则可以：

- 将此命令集保存为 PowerShell 脚本文件 (*.ps1)。
- 在 Azure 经典管理门户的“自动化”部分中将此命令集保存为 Azure 自动化 Runbook。

## <a id="examples"></a>示例

以下是使用上述步骤构建 Azure PowerShell 命令集以创建基于 Windows 的 Azure 虚拟机的两个示例。

### 示例 1

我需要 PowerShell 命令集来为 Active Directory 域控制器创建满足以下条件的初始虚拟机：

- 使用 Windows Server 2012 R2 Datacenter 映像。
- 具有名称 AZDC1。
- 是独立计算机。
- 具有 20 GB 的附加数据磁盘。
- 具有静态 IP 地址 192.168.244.4。
- 位于 AZDatacenter 虚拟网络的 BackEnd 子网中。
- 位于 Azure-TailspinToys 云服务中。

下面是用于创建此虚拟机的相应 Azure PowerShell 命令集，为了便于阅读，每个程序块之间留有空行。

	$family="Windows Server 2012 R2 Datacenter"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vmname="AZDC1"
	$vmsize="Medium"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.Username -Password $cred.GetNetworkCredential().Password

	$vm1 | Set-AzureSubnet -SubnetNames "BackEnd"

	$vm1 | Set-AzureStaticVNetIP -IPAddress 192.168.244.4

	$disksize=20
	$disklabel="DCData"
	$lun=0
	$hcaching="None"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

	$svcname="Azure-TailspinToys"
	$vnetname="AZDatacenter"
	New-AzureVM -ServiceName $svcname -VMs $vm1 -VNetName $vnetname

### 示例 2

我需要 PowerShell 命令集来为业务线服务器创建虚拟机：

- 使用 Windows Server 2012 R2 Datacenter 映像。
- 具有名称 LOB1。
- 是 corp.contoso.com 域的成员。
- 具有 200 GB 的附加数据磁盘。
- 位于 AZDatacenter 虚拟网络的 FrontEnd 子网中。
- 位于 Azure-TailspinToys 云服务中。

下面是用于创建此虚拟机的相应 Azure PowerShell 命令集。

	$family="Windows Server 2012 R2 Datacenter"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vmname="LOB1"
	$vmsize="Large"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

	$cred1=Get-Credential -Message "Type the name and password of the local administrator account."
	$cred2=Get-Credential -Message "Now type the name (not including the domain) and password of an account that has permission to add the machine to the domain."
	$domaindns="corp.contoso.com"
	$domacctdomain="CORP"
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $domacctdomain -DomainUserName $cred2.Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domaindns

	$vm1 | Set-AzureSubnet -SubnetNames "FrontEnd"

	$disksize=200
	$disklabel="LOBData"
	$lun=0
	$hcaching="ReadWrite"
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $disksize -DiskLabel $disklabel -LUN $lun -HostCaching $hcaching

	$svcname="Azure-TailspinToys"
	$vnetname="AZDatacenter"
	New-AzureVM -ServiceName $svcname -VMs $vm1 -VNetName $vnetname


## 后续步骤

如果需要大于 127 GB 的 OS 驱动器，你可以[展开 OS 驱动器](/documentation/articles/virtual-machines-windows-expand-os-disk/)。





<!---HONumber=Mooncake_0509_2016-->