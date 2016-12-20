<properties 
   pageTitle="在经典部署模型中使用 PowerShell 部署多 NIC VM | Azure"
   description="了解如何在经典部署模型中使用 PowerShell 部署多 NIC VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-service-management"
/>
<tags  
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/02/2016"
   wacn.date="12/16/2016"
   ms.author="jdial" />

#使用 PowerShell 部署多 NIC VM（经典）

[AZURE.INCLUDE [virtual-network-deploy-multinic-classic-selectors-include.md](../../includes/virtual-network-deploy-multinic-classic-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。学习如何[使用资源管理器模型执行这些步骤](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)。

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

由于目前同一云服务中不能同时存在具有单个 NIC 的 VM 和具多个 NIC 的 VM，因此在一个云服务中实现后端服务器，在另一云服务中实现方案中的所有其他组件。以下步骤在名为 *IaaSStory* 的云服务中实现主要资源，并在名为 *IaaSStory-BackEnd* 的云服务中实现后端服务器。

## <a name="Prerequisites"></a> 先决条件

需要先为主要云服务部署此方案的所有必需资源，才能部署后端服务器。至少需要创建包含后端子网的虚拟网络。若要了解如何部署虚拟网络，请访问[使用 PowerShell 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/)。

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## 部署后端 VM

后端 VM 取决于下列资源的创建。

- **后端子网**。各数据库服务器将分别属于各个子网，以便隔离流量。以下脚本期望此子网存在于名为 *WTestVnet* 的虚拟网络中。
- **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保部署到的 Azure 位置支持高级存储。
- **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。 

### 步骤 1 - 启动脚本

可以在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/virtual-network-deploy-multinic-classic-ps.ps1)下载所用的完整 PowerShell 脚本。请按以下步骤更改要在环境中使用的脚本。

1. 基于在上面[先决条件](#Prerequisites)中部署的现有资源组更改以下变量的值。

		$location              = "China North"
		$vnetName              = "WTestVNet"
		$backendSubnetName     = "BackEnd"

2. 基于要用于后端部署的值更改以下变量的值。

		$backendCSName         = "IaaSStory-Backend"
		$prmStorageAccountName = "iaasstoryprmstorage"
		$avSetName             = "ASDB"
		$vmSize                = "Standard_DS3"
		$diskSize              = 127
		$vmNamePrefix          = "DB"
		$dataDiskSuffix        = "datadisk"
		$ipAddressPrefix       = "192.168.2."
		$numberOfVMs           = 2

### 步骤 2 - 为 VM 创建必要资源

需要为所有 VM 的数据磁盘创建新的云服务和存储帐户。还需要为 VM 指定映像和本地管理员帐户。若要创建这些资源，请执行以下步骤。

1. 创建新的云服务。

		New-AzureService -ServiceName $backendCSName -Location $location

2. 创建新的高级存储帐户。

		New-AzureStorageAccount -StorageAccountName $prmStorageAccountName `
		    -Location $location `
		    -Type Premium_LRS

3. 将上面创建的存储帐户设置为订阅的当前存储帐户。

		$subscription = Get-AzureSubscription `
		    | where {$_.IsCurrent -eq $true}  
		Set-AzureSubscription -SubscriptionName $subscription.SubscriptionName `
		    -CurrentStorageAccountName $prmStorageAccountName

4. 为 VM 选择映像。

		$image = Get-AzureVMImage `
		    | where{$_.ImageFamily -eq "SQL Server 2014 RTM Web on Windows Server 2012 R2"} `
		    | sort PublishedDate -Descending `
		    | select -ExpandProperty ImageName -First 1

5. 设置本地管理员帐户凭据。

		$cred = Get-Credential -Message "Enter username and password for local admin account"

### 步骤 3 - 创建 VM

需要使用循环创建所需数量的 VM，并在循环中创建所需的 NIC 和 VM。若要创建 NIC 和 VM，请执行以下步骤。

1. 启动 `for` 循环以基于 `$numberOfVMs` 变量的值重复执行命令以根据需要创建任意多次一个 VM 和两个 NIC。

		for ($suffixNumber = 1; $suffixNumber -le $numberOfVMs; $suffixNumber++){

2. 创建 `VMConfig` 对象，为 VM 指定映像、大小和可用性集。

		    $vmName = $vmNamePrefix + $suffixNumber
		    $vmConfig = New-AzureVMConfig -Name $vmName `
		                    -ImageName $image `
		                    -InstanceSize $vmSize `
		                    -AvailabilitySetName $avSetName  

3. 将 VM 预配为 Windows VM。

		    Add-AzureProvisioningConfig -VM $vmConfig -Windows `
		        -AdminUsername $cred.UserName `
		        -Password $cred.Password

4. 设置默认 NIC，并为其分配静态 IP 地址。

		    Set-AzureSubnet -SubnetNames $backendSubnetName -VM $vmConfig
		    Set-AzureStaticVNetIP -IPAddress ($ipAddressPrefix+$suffixNumber+3) -VM $vmConfig

5. 为每个 VM 添加第二个 NIC。

		    Add-AzureNetworkInterfaceConfig -Name ("RemoteAccessNIC"+$suffixNumber) `
		        -SubnetName $backendSubnetName `
		        -StaticVNetIPAddress ($ipAddressPrefix+(53+$suffixNumber)) `
		        -VM $vmConfig 

6. 为每个 VM 创建两个数据磁盘。

		    $dataDisk1Name = $vmName + "-" + $dataDiskSuffix + "-1"    
		    Add-AzureDataDisk -CreateNew -VM $vmConfig `
		        -DiskSizeInGB $diskSize `
		        -DiskLabel $dataDisk1Name `
		        -LUN 0       
		
		    $dataDisk2Name = $vmName + "-" + $dataDiskSuffix + "-2"   
		    Add-AzureDataDisk -CreateNew -VM $vmConfig `
		        -DiskSizeInGB $diskSize `
		        -DiskLabel $dataDisk2Name `
		        -LUN 1

7. 创建每个 VM，然后结束循环。

		    New-AzureVM -VM $vmConfig `
		        -ServiceName $backendCSName `
		        -Location $location `
		        -VNetName $vnetName
		}

### 步骤 4 - 运行脚本

既已根据需要下载并更改脚本，请运行该脚本以创建具有多个 NIC 的后端数据库 VM。

1. 保存脚本并从 **PowerShell** 命令提示符或 **PowerShell ISE** 运行它。将看到最初的输出，如下所示。

		OperationDescription    OperationId                          OperationStatus
		--------------------    -----------                          ---------------
		New-AzureService        xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded      
		New-AzureStorageAccount xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded      
		                                                                            
		WARNING: No deployment found in service: 'IaaSStory-Backend'.

2. 在凭据提示中填写所需信息，然后单击“确定”。此时将显示以下输出。

		New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded
		New-AzureVM             xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Succeeded 

<!---HONumber=Mooncake_Quality_Review_1202_2016-->