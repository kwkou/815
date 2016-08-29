<properties
	pageTitle="创建虚拟机规模集 | Azure"
	description="使用 PowerShell 创建虚拟机规模集"
	services="virtual-machine-scale-sets"
    documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machine-scale-sets"
	ms.date="06/10/2016"
	wacn.date="08/29/2016"/>

# 使用 Azure PowerShell 创建 Windows 虚拟机规模集

这些步骤采用填空方法来创建 Azure 虚拟机规模集。有关规模集的详细信息，请参阅[虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)。

执行本文中的步骤大约需要 30 分钟时间。

## 步骤 1：安装 Azure PowerShell

有关如何安装最新版 Azure PowerShell、选择要使用的订阅和登录到 Azure 帐户的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 步骤 2：创建资源

创建新的虚拟机规模集所需的资源。

### 资源组

虚拟机规模集必须包含在资源组中。

1.  获取可用位置和受支持服务的列表：

        Get-AzureLocation | Sort Name | Select Name, AvailableServices

    您应看到与下面类似的内容

        Name                AvailableServices
        ----                -----------------
        Australia East      {Compute, Storage, PersistentVMRole, HighMemory}
        Australia Southeast {Compute, Storage, PersistentVMRole, HighMemory}
        Brazil South        {Compute, Storage, PersistentVMRole, HighMemory}
        Central India       {Compute, Storage, PersistentVMRole, HighMemory}
        China North          {Compute, Storage, PersistentVMRole, HighMemory}
        China East           {Compute, Storage, PersistentVMRole, HighMemory}
        China East             {Compute, Storage, PersistentVMRole, HighMemory}
        China East 2           {Compute, Storage, PersistentVMRole, HighMemory}
        China East          {Compute, Storage, PersistentVMRole, HighMemory}
        China East          {Compute, Storage, PersistentVMRole, HighMemory}
        China North    {Compute, Storage, PersistentVMRole, HighMemory}
        China North        {Compute, Storage, PersistentVMRole, HighMemory}
        China East    {Compute, Storage, PersistentVMRole, HighMemory}
        South India         {Compute, Storage, PersistentVMRole, HighMemory}
        China North      {Compute, Storage, PersistentVMRole, HighMemory}
        West Europe         {Compute, Storage, PersistentVMRole, HighMemory}
        West India          {Compute, Storage, PersistentVMRole, HighMemory}
        China North             {Compute, Storage, PersistentVMRole, HighMemory}

2. 选择最适合您的位置，将 **$locName** 的值替换为该位置名称，然后创建变量：

        $locName = "location name from the list, such as China North"

3. 将 **$rgName** 的值替换为新资源组要使用的名称，然后创建变量：

        $rgName = "resource group name"
        
4. 创建资源组：
    
        New-AzureRmResourceGroup -Name $rgName -Location $locName

    您应看到与下面类似的内容：

        ResourceGroupName : myrg1
        Location          : chinaeast
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/########-####-####-####-############/resourceGroups/myrg1

### 存储帐户

虚拟机使用存储帐户来存储操作系统磁盘和用于缩放的诊断数据。如有可能，最佳做法是在规模集中为每个虚拟机创建存储帐户。如果不能，则为每个存储帐户规划不超过 20 个 VM。本文中的示例演示了为规模集中的 3 个虚拟机创建 3 个存储帐户。

1. 将 **saName** 的值替换为存储帐户要使用的名称，然后创建变量：

        $saName = "storage account name"
        
2. 测试所选名称是否是唯一的：
    
        Test-AzureName -Storage $saName

    如果答案为 **False**，则所选名称是唯一的。

3. 将 **$saType** 的值替换为存储帐户的类型，然后创建变量：

        $saType = "storage account type"
        
    可能的值包括：Standard\_LRS、Standard\_GRS、Standard\_RAGRS 或 Premium\_LRS。
        
4. 创建帐户：
    
        New-AzureRmStorageAccount -Name $saName -ResourceGroupName $rgName -Type $saType -Location $locName

    您应看到与下面类似的内容：

        ResourceGroupName   : myrg1
        StorageAccountName  : myst1
        Id                  : /subscriptions/########-####-####-####-############/resourceGroups/myrg1/providers/Microsoft
	                    	.Storage/storageAccounts/myst1
        Location            : chinaeast
        AccountType         : StandardLRS
        CreationTime        : 3/15/2016 4:51:52 PM
        CustomDomain        :
        LastGeoFailoverTime :
        PrimaryEndpoints    : Microsoft.Azure.Management.Storage.Models.Endpoints
        PrimaryLocation     : chinaeast
        ProvisioningState   : Succeeded
        SecondaryEndpoints  :
        SecondaryLocation   :
        StatusOfPrimary     : Available
        StatusOfSecondary   :
        Tags                : {}
        Context             : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext

5. 重复步骤 1 至 4 以创建 3 个存储帐户，如 myst1、myst2 和 myst3。

### 虚拟网络

规模集中的虚拟机必须具有虚拟网络。

1. 将 **$subName** 的值替换为虚拟网络中的子网要使用的名称，然后创建变量：

        $subName = "subnet name"
        
2. 创建子网配置：
    
        $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subName -AddressPrefix 10.0.0.0/24
        
    地址前缀在虚拟网络中可能会有所不同。

3. 将 **$netName** 的值替换为虚拟网络要使用的名称，然后创建变量：

        $netName = "virtual network name"
        
4. 创建虚拟网络：
    
        $vnet = New-AzureRmVirtualNetwork -Name $netName -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $subnet

### 公共 IP 地址

在创建网络接口之前，首先需要创建一个公共 IP 地址。

1. 将 **$domName** 的值替换为要与公共 IP 地址一起使用的域名标签，然后创建变量：

        $domName = "domain name label"
        
    该标签仅可包含字母、数字和连字符，且最后一个字符必须是字母或数字。
    
2. 测试该名称是否是唯一的：
    
        Test-AzureRmDnsAvailability -DomainQualifiedName $domName -Location $locName

    如果答案为 **True**，则所选名称是唯一的。

3. 将 **$pipName** 的值替换为公共 IP 地址要使用的名称，然后创建变量。

        $pipName = "public ip address name"
        
4. 创建公共 IP 地址：
    
        $pip = New-AzureRmPublicIpAddress -Name $pipName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic -DomainNameLabel $domName

### 网络接口

现在已拥有了公共 IP 地址，便可以开始创建网络接口。

1. 将 **$nicName** 的值替换为网络接口要使用的名称，然后创建变量：

        $nicName = "network interface name"
        
2. 创建网络接口：
    
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

### 规模集的配置

您已拥有了规模集配置需要的所有资源，因此，让我们一起来创建它。

1. 将 **$ipName** 的值替换为 IP 配置要使用的名称，然后创建变量：

        $ipName = "IP configuration name"
        
2. 创建 IP 配置：

        $ipConfig = New-AzureRmVmssIpConfig -Name $ipName -LoadBalancerBackendAddressPoolsId $null -SubnetId $vnet.Subnets[0].Id

2. 将 **$vmssConfig** 的值替换为规模集配置要使用的名称，然后创建变量：

        $vmssConfig = "Scale set configuration name"
        
3. 创建规模集的配置：

        $vmss = New-AzureRmVmssConfig -Location $locName -SkuCapacity 3 -SkuName "Standard_A0" -UpgradePolicyMode "manual"
        
    此示例演示了使用 3 个虚拟机创建规模集。有关规模集容量的详细信息，请参阅[虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)。此步骤还包括在规模集中设置虚拟机的大小（也称为 SkuName）。在[虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)中查找满足您需求的虚拟机大小。
    
4. 将网络接口配置添加到规模集配置中：
        
        Add-AzureRmVmssNetworkInterfaceConfiguration -VirtualMachineScaleSet $vmss -Name $vmssConfig -Primary $true -IPConfiguration $ipConfig
        
    您应看到与下面类似的内容：

        Sku                   : Microsoft.Azure.Management.Compute.Models.Sku
        UpgradePolicy         : Microsoft.Azure.Management.Compute.Models.UpgradePolicy
        VirtualMachineProfile : Microsoft.Azure.Management.Compute.Models.VirtualMachineScaleSetVMProfile
        ProvisioningState     :
        OverProvision         :
        Id                    :
        Name                  :
        Type                  :
        Location              : China North
        Tags                  :

#### 操作系统配置文件

1. 将 **$computerName** 的值替换为要使用的计算机名称前缀，然后创建变量：

        $computerName = "computer name prefix"
        
2. 替换虚拟机上管理员帐户的 **$adminName** 名称的值，然后创建变量：

        $adminName = "administrator account name"
        
3. 将 **$adminPassword** 的值替换为帐户密码，然后创建变量：

        $adminPassword = "password for administrator accounts"
        
4. 创建操作系统配置文件：

        Set-AzureRmVmssOsProfile -VirtualMachineScaleSet $vmss -ComputerNamePrefix $computerName -AdminUsername $adminName -AdminPassword $adminPassword

#### 存储配置文件

1. 将 **$storageProfile** 的值替换为存储配置文件要使用的名称，然后创建变量：

        $storageProfile = "storage profile name"
        
2. 创建定义要使用的映像的变量：
      
        $imagePublisher = "MicrosoftWindowsServer"
        $imageOffer = "WindowsServer"
        $imageSku = "2012-R2-Datacenter"
        
    在[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)中查找有关其他要使用的映像的信息。
        
3. 将 **$vhdContainers** 的值替换为包含虚拟硬盘的存储路径的列表，如“https://mystorage.blob.core.chinacloudapi.cn/vhds”，然后创建变量：
       
        $vhdContainers = @("https://myst1.blob.core.chinacloudapi.cn/vhds","https://myst2.blob.core.chinacloudapi.cn/vhds","https://myst3.blob.core.chinacloudapi.cn/vhds")
        
4. 创建存储帐户配置文件：

        Set-AzureRmVmssStorageProfile -VirtualMachineScaleSet $vmss -ImageReferencePublisher $imagePublisher -ImageReferenceOffer $imageOffer -ImageReferenceSku $imageSku -ImageReferenceVersion "latest" -Name $storageProfile -VhdContainer $vhdContainers -OsDiskCreateOption "FromImage" -OsDiskCaching "None"  

### 虚拟机规模集

最后，可以创建规模集。

1. 将 **$vmssName** 的值替换为虚拟机规模集的名称，然后创建变量：

        $vmssName = "scale set name"
        
2. 创建规模集：

        New-AzureRmVmss -ResourceGroupName $rgName -Name $vmssName -VirtualMachineScaleSet $vmss

    您应看到与下面类似的内容，这表示部署已成功：

        Sku                   : Microsoft.Azure.Management.Compute.Models.Sku
        UpgradePolicy         : Microsoft.Azure.Management.Compute.Models.UpgradePolicy
        VirtualMachineProfile : Microsoft.Azure.Management.Compute.Models.VirtualMachineScaleSetVMProfile
        ProvisioningState     : Updating
        OverProvision         :
        Id                    : /subscriptions/########-####-####-####-############/resourceGroups/myrg1/providers/Microso
                               ft.Compute/virtualMachineScaleSets/myvmss1
        Name                  : myvmss1
        Type                  : Microsoft.Compute/virtualMachineScaleSets
        Location              : chinaeast
        Tags                  :

## 步骤 3：浏览资源

使用这些资源来浏览刚才创建的虚拟机规模集：

- Azure 门户预览 - 使用门户可以获取有限数量的信息。
- [Azure 资源浏览器](https://resources.azure.com/) - 要浏览缩放集的当前状态，这是最好的工具。
- Azure PowerShell - 使用此命令可获取信息：

        Get-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name"
        
        Or 
        
        Get-AzureRmVmssVM -ResourceGroupName "resource group name" -VMScaleSetName "scale set name"
        

## 后续步骤

- 使用[在虚拟机规模集中管理虚拟机](/documentation/articles/virtual-machine-scale-sets-windows-manage/)中的信息管理刚刚创建的规模集
- 请考虑使用[自动缩放和虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-autoscale-overview/)中的信息设置规模集的自动缩放
- 若要了解有关垂直缩放的详细信息，请参阅[虚拟机规模集的垂直缩放](/documentation/articles/virtual-machine-scale-sets-vertical-scale-reprovision/)

<!---HONumber=Mooncake_0822_2016-->