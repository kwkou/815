<properties pageTitle="使用 PowerShell 和 Azure 服务管理创建 Windows 虚拟机" description="使用 Azure PowerShell 快速创建新的 Windows 虚拟机。" services="virtual-machines" documentationCenter="" authors="JoeDavies-MSFT" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.date="06/02/2015" wacn.date="06/26/2015"/>

# 使用 PowerShell 和 Azure 服务管理创建 Windows 虚拟机

本主题介绍如何使用 Azure 服务管理和 PowerShell 快速创建基于 Windows 的 Azure 虚拟机。


- [使用 Azure 资源管理器和 PowerShell 创建 Windows 虚拟机](virtual-machines-create-windows-powershell-resource-manager)-->

## 创建 Windows 虚拟机

如果你已安装 Azure PowerShell，必须确保安装的是 Azure PowerShell 版本 0.8.0 或更高版本。可以使用此命令在 Azure PowerShell 命令提示符下查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

如果你尚未这样做，现在请按[如何安装和配置 Azure PowerShell](install-configure-powershell) 中的说明在本地计算机上安装 Azure PowerShell。然后，打开 Azure PowerShell 命令提示符。

首先，必须使用以下命令登录到 Azure。

	Add-AzureAccount

在 Windows Azure 登录对话框中指定 Azure 帐户的电子邮件地址及其密码。

接下来，如果你有多个 Azure 订阅，则需要设置你的 Azure 订阅。若要查看当前订阅的列表，请运行以下命令。

	Get-AzureSubscription | sort SubscriptionName | Select SubscriptionName

现在，请将引号内的所有内容（包括 < and > 字符）替换为正确的订阅名称，然后运行以下命令。

	$subscrName="<subscription name>"
	Select-AzureSubscription -SubscriptionName $subscrName –Current

接下来，需要一个存储帐户。可以使用以下命令显示存储帐户的列表。

	Get-AzureStorageAccount | sort Label | Select Label

如果你还没有存储帐户，请新建一个。必须选择只包含小写字母和数字的唯一名称。你可以使用此命令测试存储帐户名称的唯一性。

	Test-AzureName -Storage <Proposed storage account name>

如果此命令返回“False”，则你建议的名称是唯一的。创建存储帐户时，需要指定 Azure 数据中心的位置。若要获取 Azure 数据中心的列表，请运行以下命令。

	Get-AzureLocation | sort Name | Select Name

现在，请使用以下命令创建并设置存储帐户。填写存储帐户的名称，并替换引号内的所有内容（包括 < and > 字符）。

	$stAccount="<chosen storage account name>"
	$locName="<Azure location>"
	New-AzureStorageAccount -StorageAccountName $stAccount -Location $locName
	Set-AzureStorageAccount -StorageAccountName $stAccount
	Set-AzureSubscription -SubscriptionName $subscrName -CurrentStorageAccountName $stAccount

接下来，需要一个云服务。如果目前没有云服务，则必须创建一个。必须选择只包含字母、数字和连字符的唯一名称。该字段中的第一个和最后一个字符必须是字母或数字。

例如，你可将其命名为 TestCS-*UniqueSequence*，其中 *UniqueSequence* 是你组织的缩写。例如，如果你的组织名为 Tailspin Toys，则可以将云服务命名为 TestCS-Tailspin。

你可以使用以下 Azure PowerShell 命令测试名称的唯一性。

	Test-AzureName -Service <Proposed cloud service name>

如果此命令返回“False”，则你建议的名称是唯一的。使用以下命令创建云服务。

	$csName="<cloud service name>"
	$locName="<Azure location>"
	New-AzureService -Service $csName -Location $locName

接下来，请将以下 PowerShell 命令集复制到文本编辑器（例如记事本）。

	$vmName="<machine name>"
	$csName="<cloud service name>"
	$locName="<Azure location>"
	$vm=New-AzureVMConfig -Name $vmName -InstanceSize Medium -ImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201503.01-en.us-127GB.vhd"
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password
	New-AzureVM –ServiceName $csName –Location $locName -VMs $vm

在文本编辑器中，填写虚拟机的名称、云服务名称和位置。

最后，将该命令集复制到剪贴板，然后右键单击打开的 Azure PowerShell 命令提示符。这将发出包含一系列 PowerShell 命令的命令集，提示你输入本地管理员帐户的名称和密码并创建 Azure 虚拟机。下面是运行该命令集时的画面示例。

	PS C:\> $vmName="PSTest"
	PS C:\> $csName=" TestCS-Tailspin"
	PS C:\> $locName="China North"
	PS C:\> $vm=New-AzureVMConfig -Name $vmName -InstanceSize Medium -ImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201503.01-en.us-127GB.vhd"
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
	

## 其他资源

<!--[使用 Azure 资源管理器和 PowerShell 创建 Windows 虚拟机](virtual-machines-create-windows-powershell-resource-manager)

[使用资源管理器模板和 PowerShell 创建 Windows 虚拟机](virtual-machines-create-windows-powershell-resource-manager-template-simple)-->

[虚拟机文档](/home/features/virtual-machines/)

[如何安装和配置 Azure PowerShell](install-configure-powershell)

[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](virtual-machines-ps-create-preconfigure-windows-vms)

<!---HONumber=61-->