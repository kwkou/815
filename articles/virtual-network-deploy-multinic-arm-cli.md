<!-- ARM: tested -->

<properties 
   pageTitle="在资源管理器中使用 Azure CLI 部署多 NIC VM | Azure"
   description="了解如何在资源管理器中使用 Azure CLI 部署多 NIC VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="06/06/2016"/>

#使用 Azure CLI 部署多 NIC VM

[AZURE.INCLUDE [virtual-network-deploy-multinic-arm-selectors-include.md](../includes/virtual-network-deploy-multinic-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/virtual-network-deploy-multinic-classic-cli/)。

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../includes/virtual-network-deploy-multinic-scenario-include.md)]

由于目前你不能使用具有单个 NIC 的 VM 以及具有同一个资源组中的多个 NIC 的 VM，因此你将在一个资源组中实现后端服务器，而在其他资源组中实现所有其他组件。以下步骤使用名为 *IaaSStory* 的资源组作为主资源组，并在名为 *IaaSStory-BackEnd* 的资源组中实现后端服务器。

##<a name="Prerequisites"></a> 先决条件

在部署后端服务器之前，你需要先使用此方案的所有必需资源部署主资源组。若要部署这些资源，请按照以下步骤操作。

1. 导航到[模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/IaaS-Story/11-MultiNIC)。
2. 在模板页中“父资源组”的右侧，单击“部署到 Azure”。
3. 如果需要，更改参数值，然后按照 Azure 门户预览中的步骤部署资源组。

> [AZURE.IMPORTANT]请确保你的存储帐户名称是唯一的。不能在 Azure 中有重复的存储帐户名称。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../includes/azure-cli-prerequisites-include.md)]

## 部署后端 VM

后端 VM 取决于下面列出的资源的创建。

- **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保你部署的 Azure 位置支持高级存储。
- **NIC**。每个 VM 都将具有两个 NIC，一个用于数据库访问，另一个用于管理。
- **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。 

### 步骤 1 - 启动脚本

你可以在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/arm/virtual-network-deploy-multinic-arm-cli.sh)下载所用的完整 bash 脚本。请按照以下步骤更改要在你的环境中使用的脚本。

1. 基于你在上面[先决条件](#Prerequisites)中部署的现有资源组更改以下变量的值。

		existingRGName="IaaSStory"
		location="chinanorth"
		vnetName="WTestVNet"
		backendSubnetName="BackEnd"
		remoteAccessNSGName="NSG-RemoteAccess"
		
2. 基于要用于后端部署的值更改以下变量的值。

		backendRGName="IaaSStory-Backend"
		prmStorageAccountName="wtestvnetstorageprm"
		avSetName="ASDB"
		vmSize="Standard_DS3"
		diskSize=127
		publisher="Canonical"
		offer="UbuntuServer"
		sku="14.04.2-LTS"
		version="latest"
		vmNamePrefix="DB"
		osDiskName="osdiskdb"
		dataDiskName="datadisk"
		nicNamePrefix="NICDB"
		ipAddressPrefix="192.168.2."
		username='adminuser'
		password='adminP@ssw0rd'
		numberOfVMs=2

3. 检索要在其中创建 VM 的 `BackEnd` 子网的 ID。你需要执行此操作，因为要关联到此子网的 NIC 位于不同的资源组中。

		subnetId="$(azure network vnet subnet show --resource-group $existingRGName \
		                --vnet-name $vnetName \
		                --name $backendSubnetName|grep Id)"
		subnetId=${subnetId#*/}

>[AZURE.TIP]上面的第一个命令使用 [grep](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_04_02.html) 和[字符串操作](http://tldp.org/LDP/abs/html/string-manipulation.html)（更具体地说，是子字符串删除）。

4. 检索 `NSG-RemoteAccess` NSG 的 ID。你需要执行此操作，因为要关联到此 NSG 的 NIC 位于不同的资源组中。

		nsgId="$(azure network nsg show --resource-group $existingRGName \
		                --name $remoteAccessNSGName|grep Id)"
		nsgId=${nsgId#*/}

### 步骤 2 - 为你的 VM 创建必要的资源

1. 为所有后端资源创建新的资源组。请注意使用 `$backendRGName` 变量表示资源组名称，使用 `$location` 表示 Azure 区域。

		azure group create $backendRGName $location

2. 为要由你的 VM 使用的 OS 和数据磁盘创建高级存储帐户。

		azure storage account create $prmStorageAccountName \
		    --resource-group $backendRGName \
		    --location $location \
			--type PLRS 

3. 为 VM 创建可用性集。

		azure availset create --resource-group $backendRGName \
		    --location $location \
		    --name $avSetName

### 步骤 3 - 创建 NIC 和后端 VM

1. 基于 `numberOfVMs` 变量启动一个循环来创建多个 VM。

		for ((suffixNumber=1;suffixNumber<=numberOfVMs;suffixNumber++));
		do

2. 对于每个 VM，创建一个 NIC 用于数据库访问。

		    nic1Name=$nicNamePrefix$suffixNumber-DA
		    x=$((suffixNumber+3))
		    ipAddress1=$ipAddressPrefix$x
		    azure network nic create --name $nic1Name \
		        --resource-group $backendRGName \
		        --location $location \
		        --private-ip-address $ipAddress1 \
		        --subnet-id $subnetId

3. 对于每个 VM，创建一个 NIC 用于远程访问。请注意 `--network-security-group` 参数，它用于将 NIC 关联到 NSG。

		    nic2Name=$nicNamePrefix$suffixNumber-RA
		    x=$((suffixNumber+53))
		    ipAddress2=$ipAddressPrefix$x
		    azure network nic create --name $nic2Name \
		        --resource-group $backendRGName \
		        --location $location \
		        --private-ip-address $ipAddress2 \
		        --subnet-id $subnetId $vnetName \
		        --network-security-group-id $nsgId

4. 创建 VM。

		    azure vm create --resource-group $backendRGName \
		        --name $vmNamePrefix$suffixNumber \
		        --location $location \
		        --vm-size $vmSize \
		        --subnet-id $subnetId \
		        --availset-name $avSetName \
		        --nic-names $nic1Name,$nic2Name \
		        --os-type linux \
		        --image-urn $publisher:$offer:$sku:$version \
		        --storage-account-name $prmStorageAccountName \
		        --storage-account-container-name vhds \
		        --os-disk-vhd $osDiskName$suffixNumber.vhd \
		        --admin-username $username \
		        --admin-password $password

5. 对于每个 VM，创建两个数据磁盘，并以 `done` 命令结束循环。

		    azure vm disk attach-new --resource-group $backendRGName \
		        --vm-name $vmNamePrefix$suffixNumber \        
		        --storage-account-name $prmStorageAccountName \
		        --storage-account-container-name vhds \
		        --vhd-name $dataDiskName$suffixNumber-1.vhd \
		        --size-in-gb $diskSize \
		        --lun 0
		
		    azure vm disk attach-new --resource-group $backendRGName \
		        --vm-name $vmNamePrefix$suffixNumber \        
		        --storage-account-name $prmStorageAccountName \
		        --storage-account-container-name vhds \
		        --vhd-name $dataDiskName$suffixNumber-2.vhd \
		        --size-in-gb $diskSize \
		        --lun 1
		done


### 步骤 4 - 运行脚本

现在，你已根据需要下载并更改了脚本，请运行该脚本以创建具有多个 NIC 的后端数据库 VM。

1. 保存该脚本并从 **Bash** 终端运行它。你将看到最初的输出，如下所示。

		info:    Executing command group create
		info:    Getting resource group IaaSStory-Backend
		info:    Creating resource group IaaSStory-Backend
		info:    Created resource group IaaSStory-Backend
		data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend
		data:    Name:                IaaSStory-Backend
		data:    Location:            chinanorth
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK
		info:    Executing command storage account create
		info:    Creating storage account
		info:    storage account create command OK
		info:    Executing command availset create
		info:    Looking up the availability set "ASDB"
		info:    Creating availability set "ASDB"
		info:    availset create command OK
		info:    Executing command network nic create
		info:    Looking up the network interface "NICDB1-DA"
		info:    Creating network interface "NICDB1-DA"
		info:    Looking up the network interface "NICDB1-DA"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/Microsoft.Network/networkInterfaces/NICDB1-DA
		data:    Name                            : NICDB1-DA
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.2.4
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/BackEnd
		data:
		info:    network nic create command OK
		info:    Executing command network nic create
		info:    Looking up the network interface "NICDB1-RA"
		info:    Creating network interface "NICDB1-RA"
		info:    Looking up the network interface "NICDB1-RA"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/Microsoft.Network/networkInterfaces/NICDB1-RA
		data:    Name                            : NICDB1-RA
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    Network security group          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/networkSecurityGroups/NSG-RemoteAccess
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.2.54
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/BackEnd
		data:
		info:    network nic create command OK
		info:    Executing command vm create
		info:    Looking up the VM "DB1"
		info:    Using the VM Size "Standard_DS3"
		info:    The [OS, Data] Disk or image configuration requires storage account
		info:    Looking up the storage account wtestvnetstorageprm
		info:    Looking up the availability set "ASDB"
		info:    Found an Availability set "ASDB"
		info:    Looking up the NIC "NICDB1-DA"
		info:    Looking up the NIC "NICDB1-RA"
		info:    Creating VM "DB1"

2. 几分钟后，执行将结束，你将看到输出的其余部分，如下所示。

		info:    vm create command OK
		info:    Executing command vm disk attach-new
		info:    Looking up the VM "DB1"
		info:    Looking up the storage account wtestvnetstorageprm
		info:    New data disk location: https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/datadisk1-1.vhd
		info:    Updating VM "DB1"
		info:    vm disk attach-new command OK
		info:    Executing command vm disk attach-new
		info:    Looking up the VM "DB1"
		info:    Looking up the storage account wtestvnetstorageprm
		info:    New data disk location: https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/datadisk1-2.vhd
		info:    Updating VM "DB1"
		info:    vm disk attach-new command OK
		info:    Executing command network nic create
		info:    Looking up the network interface "NICDB2-DA"
		info:    Creating network interface "NICDB2-DA"
		info:    Looking up the network interface "NICDB2-DA"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/Microsoft.Network/networkInterfaces/NICDB2-DA
		data:    Name                            : NICDB2-DA
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.2.5
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/BackEnd
		data:
		info:    network nic create command OK
		info:    Executing command network nic create
		info:    Looking up the network interface "NICDB2-RA"
		info:    Creating network interface "NICDB2-RA"
		info:    Looking up the network interface "NICDB2-RA"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/Microsoft.Network/networkInterfaces/NICDB2-RA
		data:    Name                            : NICDB2-RA
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    Network security group          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/networkSecurityGroups/NSG-RemoteAccess
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.2.55
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/BackEnd
		data:
		info:    network nic create command OK
		info:    Executing command vm create
		info:    Looking up the VM "DB2"
		info:    Using the VM Size "Standard_DS3"
		info:    The [OS, Data] Disk or image configuration requires storage account
		info:    Looking up the storage account wtestvnetstorageprm
		info:    Looking up the availability set "ASDB"
		info:    Found an Availability set "ASDB"
		info:    Looking up the NIC "NICDB2-DA"
		info:    Looking up the NIC "NICDB2-RA"
		info:    Creating VM "DB2"
		info:    vm create command OK
		info:    Executing command vm disk attach-new
		info:    Looking up the VM "DB2"
		info:    Looking up the storage account wtestvnetstorageprm
		info:    New data disk location: https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/datadisk2-1.vhd
		info:    Updating VM "DB2"
		info:    vm disk attach-new command OK
		info:    Executing command vm disk attach-new
		info:    Looking up the VM "DB2"
		info:    Looking up the storage account wtestvnetstorageprm
		info:    New data disk location: https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/datadisk2-2.vhd
		info:    Updating VM "DB2"
		info:    vm disk attach-new command OK

<!---HONumber=Mooncake_1221_2015-->