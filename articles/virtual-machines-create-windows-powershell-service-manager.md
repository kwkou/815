<properties
	pageTitle="使用 Azure PowerShell 在服务管理中创建和管理 Windows 虚拟机。"
	description="使用 Azure PowerShell 在服务管理中快速创建新的基于 Windows 的虚拟机并执行管理功能。"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags 
	ms.service="virtual-machines"	
	ms.date="07/09/2015"
	wacn.date="09/15/2015"/>

# 通过使用 Azure PowerShell 在服务管理中创建和管理基于 Windows 的虚拟机

本文介绍如何使用 Azure PowerShell 在服务管理中创建和管理基于 Windows 的虚拟机。

[AZURE.INCLUDE [service-management-pointer-to-resource-manager](../includes/service-management-pointer-to-resource-manager.md)]

- [使用 Azure 资源管理器模板与 PowerShell 来部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)

## 安装 Azure PowerShell

如果你已安装 Azure PowerShell，必须确保安装的是 Azure PowerShell 版本 0.8.0 或更高版本。可以在 Azure PowerShell 命令提示符下使用此命令查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

如果你尚未这样做，现在请按[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell) 中的说明在本地计算机上安装 Azure PowerShell。然后，打开 Azure PowerShell 命令提示符。

首先，必须使用此命令登录到 Azure：

	Add-AzureAccount -Environment AzureChinaCloud

在 Windows Azure 登录对话框中指定 Azure 帐户的电子邮件地址及其密码。

接下来，如果你有多个 Azure 订阅，则需要设置你的 Azure 订阅。若要查看当前订阅的列表，请运行以下命令：

	Get-AzureSubscription | sort SubscriptionName | Select SubscriptionName

现在，请将引号内的所有内容（包括 < and > 字符）替换为正确的订阅名称，然后运行以下命令：

	$subscrName="<subscription name>"
	Select-AzureSubscription -SubscriptionName $subscrName –Current

## 创建虚拟机

首先，需要一个存储帐户。可以使用以下命令显示当前存储帐户列表：

	Get-AzureStorageAccount | sort Label | Select Label

如果你还没有存储帐户，请新建一个。必须选择只包含小写字母和数字的唯一名称。你可以使用此命令测试存储帐户名称的唯一性：

	Test-AzureName -Storage <Proposed Storage account name>

如果此命令返回“False”，则你建议的名称是唯一的。

创建存储帐户时，需要指定 Azure 数据中心的位置。若要获取 Azure 数据中心的列表，请运行以下命令：

	Get-AzureLocation | sort Name | Select Name

现在，请使用以下命令创建并设置存储帐户。填写存储帐户的名称，并替换引号内的所有内容（包括 < and > 字符）。

	$stAccount="<chosen Storage account name>"
	$locName="<Azure location>"
	New-AzureStorageAccount -StorageAccountName $stAccount -Location $locName
	Set-AzureStorageAccount -StorageAccountName $stAccount
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscrName -CurrentStorageAccountName $stAccount

接下来，需要一个云服务。如果目前没有云服务，则必须创建一个。必须选择只包含字母、数字和连字符的唯一名称。该字段中的第一个和最后一个字符必须是字母或数字。

例如，你可将其命名为 TestCS-*UniqueSequence*，其中 *UniqueSequence* 是你组织的缩写。例如，如果你的组织名为 Tailspin Toys，则可以将云服务命名为 TestCS-Tailspin。

你可以使用此 Azure PowerShell 命令测试名称的唯一性：

	Test-AzureName -Service <Proposed cloud service name>

如果此命令返回“False”，则你建议的名称是唯一的。使用以下命令创建云服务。

	$csName="<cloud service name>"
	$locName="<Azure location>"
	New-AzureService -Service $csName -Location $locName

接下来，请将此 PowerShell 命令集复制到文本编辑器（例如记事本）：

	$vmName="<machine name>"
	$csName="<cloud service name>"
	$locName="<Azure location>"
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm=New-AzureVMConfig -Name $vmName -InstanceSize Medium -ImageName $image
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password
	New-AzureVM –ServiceName $csName –Location $locName -VMs $vm

在文本编辑器中，填写虚拟机的名称、云服务名称和位置。

最后，将该命令集复制到剪贴板，然后右键单击打开的 Azure PowerShell 命令提示符。这将发出包含一系列 Azure PowerShell 命令的命令集，提示你输入本地管理员帐户的名称和密码并创建 Azure 虚拟机。下面是运行该命令集时的画面示例：

	PS C:\> $vmName="PSTest"
	PS C:\> $csName=" TestCS-Tailspin"
	PS C:\> $locName="China North"
	PS C:\> $image=Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	VERBOSE: 3:01:17 PM - Begin Operation: Get-AzureVMImage
	VERBOSE: 3:01:22 PM - Completed Operation: Get-AzureVMImage
	VERBOSE: 3:01:22 PM - Begin Operation: Get-AzureVMImage
	VERBOSE: 3:01:23 PM - Completed Operation: Get-AzureVMImage
	PS C:\> $vm=New-AzureVMConfig -Name $vmName -InstanceSize Medium -ImageName $image
	PS C:\> $cred=Get-Credential -Message "Type the name and password of the local administrator account."
	PS C:\> $vm | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.
	GetNetworkCredential().Password

	
	AvailabilitySetName               :
	ConfigurationSets                 : PSTest,Microsoft.WindowsAzure.Commands.ServiceManagement.Model.NetworkConfigurationSet}
	DataVirtualHardDisks              : {}
	Label                             : PSTest
	OSVirtualHardDisk                 : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.OSVirtualHardDisk
	RoleName                          : PSTest
	RoleSize                          : Medium
	RoleType                          : PersistentVMRole
	WinRMCertificate                  :
	X509Certificates                  : {}
	NoExportPrivateKey                : False
	NoRDPEndpoint                     : False
	NoSSHEndpoint                     : False
	DefaultWinRmCertificateThumbprint :
	ProvisionGuestAgent               : True
	ResourceExtensionReferences       : {BGInfo}
	DataVirtualHardDisksToBeDeleted   :
	VMImageInput                      :
	
	PS C:\> New-AzureVM -ServiceName $csName -Location $locName -VMs $vm
	VERBOSE: 3:01:46 PM - Begin Operation: New-AzureVM - Create Deployment with VM PSTest
	VERBOSE: 3:02:49 PM - Completed Operation: New-AzureVM - Create Deployment with VM PSTest
	
	OperationDescription                    OperationId                            OperationStatus
	--------------------                    -----------                            --------------
	New-AzureVM                             8072cbd1-4abe-9278-9de2-8826b56e9221   Succeeded

## 显示有关虚拟机的信息
这是你将经常使用的基本任务。使用它可获取有关 VM 的信息、对 VM 执行任务或获取输出以存储在变量中。

若要获取有关 VM 的信息，请运行此命令，并替换引号内的所有内容（包括 < and > 字符）：

     Get-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

若要将输出存储在 $vm 变量中，请运行：

    $vm = Get-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

## 登录到基于 Windows 的虚拟机

运行以下命令：

	$svcName="<cloud service name>"
	$vmName="<virtual machine name>"
	$localPath="<drive and folder location to store the downloaded RDP file, example: c:\temp >"
	$localFile=$localPath + "" + $vmname + ".rdp"
	Get-AzureRemoteDesktopFile -ServiceName $svcName -Name $vmName -LocalPath $localFile -Launch

>[AZURE.NOTE]可以从显示的 **Get-AzureVM** 命令中获取虚拟机和云服务名称。

## 停止 VM

运行以下命令：

    Stop-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

>[AZURE.IMPORTANT]如果此 VM 是云服务中的最后一个 VM，则使用 **StayProvisioned** 参数可以保留该云服务的虚拟 IP (VIP)。如果你使用此参数，则仍要支付 VM 的费用。

## 启动 VM

运行以下命令：

    Start-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

## 附加数据磁盘
此任务需要几个步骤。首先，使用 **Add-AzureDataDisk** cmdlet 将磁盘添加到 $vm 对象。然后，使用 Update-AzureVM cmdlet 更新 VM 的配置。

你还需要确定是要附加新的磁盘还是附加包含数据的磁盘。对于新磁盘，此命令将创建 .vhd 文件，然后将它附加在同一个命令中。

若要附加新磁盘，请运行以下命令：

    Add-AzureDataDisk -CreateNew -DiskSizeInGB <disk size> -DiskLabel "<label name>" -LUN <LUN number> -VM $vm | Update-AzureVM

若要附加现有数据磁盘，请运行以下命令：

    Add-AzureDataDisk -Import -DiskName "<existing disk name>" -LUN <LUN number> | Update-AzureVM

若要从 blob 存储中现有的 .vhd 文件附加数据磁盘，请运行以下命令：

    $diskLoc="https://mystorage.blob.core.chinacloudapi.cn/mycontainer/" + "<existing disk name>" + ".vhd"
	Add-AzureDataDisk -ImportFrom -MediaLocation  $diskLoc -DiskLabel "<label name>" -LUN <LUN number> | Update-AzureVM

## 其他资源

[使用资源管理器和 Azure PowerShell 创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-resource-manager)

[使用资源管理器模板和 Azure PowerShell 创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-resource-manager-template-simple)

[虚拟机文档](/documentation/services/virtual-machines/)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)

<!---HONumber=69-->