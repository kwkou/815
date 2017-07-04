<properties
    pageTitle="在 Azure PowerShell 中创建 SQL Server 虚拟机 (Resource Manager) | Azure"
    description="提供用于创建具有 SQL Server 虚拟机库映像的 Azure VM 的步骤和 PowerShell 脚本。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="rothja"
    manager="jhubbard"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="98d50dd8-48ad-444f-9031-5378d8270d7b"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="01/17/2017"
    wacn.date="03/28/2017"
    ms.author="jroth" />  


# 使用 Azure PowerShell 预配 SQL Server 虚拟机 \(Resource Manager\)
> [AZURE.SELECTOR]
- [门户](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)
- [PowerShell](/documentation/articles/virtual-machines-windows-ps-sql-create/)

## 概述
本教程演示如何使用 Azure PowerShell cmdlet 通过 **Azure Resource Manager** 部署模型创建单个 Azure 虚拟机。在本教程中，我们将从 SQL 库中的映像使用单个磁盘驱动器创建单个虚拟机。我们将为虚拟机要使用的存储、网络和计算资源创建新的提供程序。如果拥有上述任何资源的提供程序，则可以改用现有的提供程序。

如果需要本主题的经典版本，请参阅[使用 Azure PowerShell 预配 SQL Server 虚拟机（经典）](/documentation/articles/virtual-machines-windows-classic-ps-sql-create/)。

## 先决条件
在本教程中，你需要：

* 在开始之前，你需要有 Azure 帐户和订阅。如果没有，请注册[试用版](/pricing/1rmb-trial/)。
* [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)，最低版本 1.4.0 或以上（本教程使用 1.5.0 版编写）。
    * 若要检索你的版本，请键入 **Get-Module Azure -ListAvailable**。

## <a name="configure-your-subscription"></a> 配置订阅
打开 Windows PowerShell，并通过运行以下 cmdlet 访问 Azure 帐户。随后将出现一个用于输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

    Add-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括用于登录的订阅 ID。除非更改为其他订阅，否则本教程中的资源都将在该订阅中创建。如果你有多个订阅 ID，请运行以下 cmdlet 以返回所有订阅 ID 的列表：

    Get-AzureRmSubscription

若要更改为另一个订阅 ID，请结合所需的订阅 ID 运行以下 cmdlet。

    Select-AzureRmSubscription -SubscriptionId xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

## 定义映像变量
为了简化可用性和了解本教程中的完整脚本，我们首先要定义几个变量。根据需要更改参数值，但在修改提供的值时，请注意与名称长度和特殊字符相关的命名限制。

### 位置和资源组
使用两个变量来定义数据区域，以及将在其中为虚拟机创建其他资源的资源组。

根据需要修改并执行以下 cmdlet，以初始化这些变量。

    $Location = "ChinaEast"
    $ResourceGroupName = "sqlvm1"

### 存储属性
使用以下变量来定义存储帐户和虚拟机要使用的存储类型。

根据需要修改并执行以下 cmdlet，以初始化这些变量。请注意，在本示例中，我们将使用建议用于生产工作负荷的[高级存储](/documentation/articles/storage-premium-storage/)。有关本指导和其他建议的详细信息，请参阅 [Azure 虚拟机中 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)。

    $StorageName = $ResourceGroupName + "storage"
    $StorageSku = "Premium_LRS"

### 网络属性
使用以下变量来定义网络接口、TCP/IP 分配方法、虚拟网络名称、虚拟子网名称、虚拟网络的 IP 地址范围、子网的 IP 地址范围，以及虚拟机中的网络要使用的公共域名标签。

根据需要修改并执行以下 cmdlet，以初始化这些变量。

    $InterfaceName = $ResourceGroupName + "ServerInterface"
    $TCPIPAllocationMethod = "Dynamic"
    $VNetName = $ResourceGroupName + "VNet"
    $SubnetName = "Default"
    $VNetAddressPrefix = "10.0.0.0/16"
    $VNetSubnetAddressPrefix = "10.0.0.0/24"
    $DomainName = "sqlvm1"

### 虚拟机属性
使用以下变量来定义虚拟机名称、计算机名称、虚拟机大小和虚拟机的操作系统磁盘名称。

根据需要修改并执行以下 cmdlet，以初始化这些变量。

    $VMName = $ResourceGroupName + "VM"
    $ComputerName = $ResourceGroupName + "Server"
    $VMSize = "Standard_DS13"
    $OSDiskName = $VMName + "OSDisk"

### 映像属性
使用以下变量来定义要用于虚拟机的映像。在本示例中，将使用 SQL Server 2016 Enterprise 映像。

根据需要修改并执行以下 cmdlet，以初始化这些变量。

    $PublisherName = "MicrosoftSQLServer"
    $OfferName = "SQL2016-WS2016"
    $Sku = "Enterprise"
    $Version = "latest"

请注意，可以使用 Get-AzureRmVMImageOffer 命令来获取 SQL Server 映像产品的完整列表：

    Get-AzureRmVMImageOffer -Location 'China East' -Publisher 'MicrosoftSQLServer'

可以使用 Get-AzureRmVMImageSku 来查看可用于产品的 SKU。以下命令将显示可用于 **SQL2014SP1-WS2012R2** 产品的所有 SKU。

    Get-AzureRmVMImageSku -Location 'China East' -Publisher 'MicrosoftSQLServer' -Offer 'SQL2014SP1-WS2012R2' | Select Skus

## 创建资源组
若使用 Resource Manager 部署模型，创建的第一个对象就是资源组。我们将使用 [New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt759837.aspx) cmdlet 以前面初始化的变量所定义的资源组名称和位置来创建 Azure 资源组及其资源。

执行以下 cmdlet，以创建新的资源组。

    New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location

## 创建存储帐户
虚拟机需要使用存储资源来存储操作系统磁盘及 SQL Server 数据和日志文件。为简单起见，我们将为这两者创建单个磁盘。稍后，你可以使用 [Add-Azure Disk](https://msdn.microsoft.com/zh-cn/library/azure/dn495252.aspx) cmdlet 来附加其他磁盘，以便将 SQL Server 数据和日志文件放在专用磁盘上。我们将使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet，以前面初始化的变量所定义的存储帐户名称、存储 SKU 名称和位置在新资源组中创建标准存储帐户。

执行以下 cmdlet，以创建新的存储帐户。

    $StorageAccount = New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageName -SkuName $StorageSku -Kind "Storage" -Location $Location

## 创建网络资源
虚拟机需要使用多个网络资源才能建立网络连接。

* 每个虚拟机需要一个虚拟网络。
* 必须为每个虚拟网络至少定义一个子网。
* 必须使用公共或专用 IP 地址定义网络接口。

### 创建虚拟网络子网配置
我们将首先创建虚拟网络的子网配置。在本教程中，我们将使用 [New-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt619412.aspx) cmdlet 创建默认子网。我们将使用前面初始化的变量所定义的子网名称和地址前缀来创建虚拟网络子网配置。

> [AZURE.NOTE]
你可以使用此 cmdlet 来定义虚拟网络子网配置的其他属性，但这已超出本教程的范围。
>
>

执行以下 cmdlet，以创建虚拟子网配置。

    $SubnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix

### 创建虚拟网络
接下来，我们将使用 [New-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603657.aspx) cmdlet 来创建虚拟网络。我们将使用前面初始化的变量所定义的名称、位置和地址前缀，以及前一步骤中定义的子网配置，在新的资源组中创建虚拟网络。

执行以下 cmdlet，以创建虚拟网络。

    $VNet = New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName -Location $Location -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig

### 创建公共 IP 地址
我们现已定义虚拟网络，接下来需要配置 IP 地址才能连接到虚拟机。在本教程中，我们将使用动态 IP 地址来创建公共 IP 地址，以支持 Internet 连接。我们将使用 [New-AzureRmPublicIpAddress](https://msdn.microsoft.com/zh-cn/library/mt603620.aspx) cmdlet，以前面初始化的变量所定义的名称、位置、分配方法和 DNS 域名标签，在前面创建的资源组中创建公共 IP 地址。

> [AZURE.NOTE]
你可以使用此 cmdlet 来定义公共 IP 地址的其他属性，但这已超出本初步教程的范围。你也可以创建专用地址或具有静态地址的地址，但这也超出了本教程的范围。
>
>

执行以下 cmdlet，以创建公共 IP 地址。

    $PublicIp = New-AzureRmPublicIpAddress -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName

### 创建网络接口
我们现在已准备好创建虚拟机将要使用的网络接口。我们将使用 [New-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619370.aspx) cmdlet，以前面定义的名称、位置、子网和公共 IP 地址，在前面创建的资源组中创建网络接口。

执行以下 cmdlet，以创建网络接口。

    $Interface = New-AzureRmNetworkInterface -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id

## 配置 VM 对象
我们现已定义存储和网络资源，接下来可以定义虚拟机的计算资源。在本教程中，我们将指定虚拟机大小和各种操作系统属性，指定我们前面创建的网络接口，定义 Blob 存储，然后指定操作系统磁盘。

### 创建 VM 对象
首先指定虚拟机大小。在本教程中，我们将指定 DS13。我们将使用 [New-AzureRmVMConfig](https://msdn.microsoft.com/zh-cn/library/mt603727.aspx) cmdlet，以前面初始化的变量所定义的名称和大小来创建可配置的虚拟机对象。

执行以下 cmdlet，以创建虚拟机对象。

    $VirtualMachine = New-AzureRmVMConfig -VMName $VMName -VMSize $VMSize

### 创建一个凭据对象，以保留本地管理员凭据的名称和密码
我们需要先提供本地管理员帐户的凭据作为安全字符串，然后才可以设置虚拟机的操作系统属性。为了实现此目的，我们将使用 [Get-Credential](https://technet.microsoft.com/zh-cn/library/hh849815.aspx) cmdlet。

执行以下 cmdlet，并在 Windows PowerShell 凭据请求窗口中，键入要用于 Windows 虚拟机中本地管理员帐户的名称和密码。

    $Credential = Get-Credential -Message "Type the name and password of the local administrator account."

### 设置虚拟机的操作系统属性
现在，我们已准备好设置虚拟机的操作系统属性。我们将使用 [Set-AzureRmVMOperatingSystem](https://msdn.microsoft.com/zh-cn/library/mt603843.aspx) cmdlet 将操作系统的类型设置为 Windows，要求安装[虚拟机代理](/documentation/articles/virtual-machines-windows-classic-agents-and-extensions/)，并指定该 cmdlet 允许使用前面初始化的变量自动更新和设置虚拟机名称、计算机名称和凭据。

执行以下 cmdlet，以设置虚拟机的操作系统属性。

    $VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate

### 将网络接口添加到虚拟机
接下来，我们需要将前面创建的网络接口添加到虚拟机。我们将使用 [Add-AzureRmVMNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619351.aspx) cmdlet 通过前面定义的网络接口变量来添加网络接口。

执行以下 cmdlet，以设置虚拟机的网络接口。

    $VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id

### 为虚拟机要使用的磁盘设置 Blob 存储位置
接下来，我们将使用前面定义的变量，为虚拟机要使用的磁盘设置 Blob 存储位置。

执行以下 cmdlet，以设置 Blob 存储位置。

    $OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"

### 设置虚拟机的操作系统磁盘属性
接下来，我们将设置虚拟机的操作系统磁盘属性。我们将使用 [Set-AzureRmVMOSDisk](https://msdn.microsoft.com/zh-cn/library/mt603746.aspx) cmdlet 指定虚拟机的操作系统来自于某个映像，将缓存设置为只读（因为要在同一磁盘上安装 SQL Server），并使用前面定义的变量来定义虚拟机名称和操作系统磁盘。

执行以下 cmdlet，以设置虚拟机的操作系统磁盘属性。

    $VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage

### 指定虚拟机的平台映像
最后一个配置步骤是指定虚拟机的平台映像。在本教程中，我们使用的是最新的 SQL Server 2016 CTP 映像。我们将通过 [Set-AzureRmVMSourceImage](https://msdn.microsoft.com/zh-cn/library/mt619344.aspx) cmdlet 来使用前面定义的变量所定义的映像。

执行以下 cmdlet，以指定虚拟机的平台映像。

    $VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName $PublisherName -Offer $OfferName -Skus $Sku -Version $Version

## 创建 SQL VM
现在你已完成配置步骤，接下来可以创建虚拟机了。我们将通过 [New-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603754.aspx) cmdlet，使用定义的变量来创建虚拟机。

执行以下 cmdlet，以创建虚拟机。

    New-AzureRmVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine

虚拟机已创建。请注意，由于为虚拟机磁盘指定的存储帐户是高级存储帐户，因此创建了一个标准存储帐户，以用于启动诊断。

现在，用户可以在 Azure 门户中查看此虚拟机，以了解[其公共 IP 地址及完全限定的域名](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)。

## 示例脚本
以下脚本包含本教程的完整 PowerShell 脚本。该脚本假设用户已将 Azure 订阅设置为配合使用 **Add-AzureRmAccount -EnvironmentName AzureChinaCloud** 和 **Select-AzureRmSubscription** 命令。

    # Variables
    ## Global
    $Location = "ChinaEast"
    $ResourceGroupName = "sqlvm1"
    ## Storage
    $StorageName = $ResourceGroupName + "storage"
    $StorageSku = "Premium_LRS"

    ## Network
    $InterfaceName = $ResourceGroupName + "ServerInterface"
    $VNetName = $ResourceGroupName + "VNet"
    $SubnetName = "Default"
    $VNetAddressPrefix = "10.0.0.0/16"
    $VNetSubnetAddressPrefix = "10.0.0.0/24"
    $TCPIPAllocationMethod = "Dynamic"
    $DomainName = "sqlvm1"

    ##Compute
    $VMName = $ResourceGroupName + "VM"
    $ComputerName = $ResourceGroupName + "Server"
    $VMSize = "Standard_DS13"
    $OSDiskName = $VMName + "OSDisk"

    ##Image
    $PublisherName = "MicrosoftSQLServer"
    $OfferName = "SQL2016-WS2016"
    $Sku = "Enterprise"
    $Version = "latest"

    # Resource Group
    New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location

    # Storage
    $StorageAccount = New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageName -SkuName $StorageSku -Kind "Storage" -Location $Location

    # Network
    $SubnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $VNetSubnetAddressPrefix
    $VNet = New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName -Location $Location -AddressPrefix $VNetAddressPrefix -Subnet $SubnetConfig
    $PublicIp = New-AzureRmPublicIpAddress -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -AllocationMethod $TCPIPAllocationMethod -DomainNameLabel $DomainName
    $Interface = New-AzureRmNetworkInterface -Name $InterfaceName -ResourceGroupName $ResourceGroupName -Location $Location -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PublicIp.Id

    # Compute
    $VirtualMachine = New-AzureRmVMConfig -VMName $VMName -VMSize $VMSize
    $Credential = Get-Credential -Message "Type the name and password of the local administrator account."
    $VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate #-TimeZone = $TimeZone
    $VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $Interface.Id
    $OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $OSDiskName + ".vhd"
    $VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -Caching ReadOnly -CreateOption FromImage

    # Image
    $VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName $PublisherName -Offer $OfferName -Skus $Sku -Version $Version

    ## Create the VM in Azure
    New-AzureRmVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine

## 后续步骤
创建虚拟机后，你就可以使用 RDP 和设置连接来连接虚拟机了。有关详细信息，请参阅[连接到 Azure 上的 SQL Server 虚拟机 \(Resource Manager\)](/documentation/articles/virtual-machines-windows-sql-connect/)。

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->