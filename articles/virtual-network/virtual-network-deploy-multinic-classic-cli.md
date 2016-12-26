<properties 
   pageTitle="在经典部署模型中使用 Azure CLI 部署多 NIC VM | Azure"
   description="了解如何在经典部署模型中使用 Azure CLI 部署多 NIC VM"
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
   wacn.date="12/26/2016"
   ms.author="jdial" />

#使用 Azure CLI 部署多 NIC VM（经典）

[AZURE.INCLUDE [virtual-network-deploy-multinic-classic-selectors-include.md](../../includes/virtual-network-deploy-multinic-classic-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。学习如何[使用资源管理器模型执行这些步骤](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)。

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

由于目前不能将具有单个 NIC 的 VM 和具有多个 NIC 的 VM 用于同一云服务，因此要在一个云服务中实现后端服务器，在另一云服务中实现方案中的所有其他组件。以下步骤在名为 *IaaSStory* 的云服务中实现主要资源，并在名为 *IaaSStory-BackEnd* 的云服务中实现后端服务器。

## <a name="Prerequisites"></a> 先决条件

部署后端服务器之前，需要先使用此方案的所有必需资源部署主云服务。至少需要为后端创建包含子网的虚拟网络。若要了解如何部署虚拟网络，请访问[使用 Azure CLI 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-classic-cli/)。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## 部署后端 VM

后端 VM 取决于下列资源的创建。

- **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保部署到的 Azure 位置支持高级存储。
- **NIC**。每个 VM 都将具有两个 NIC，一个用于数据库访问，另一个用于管理。
- **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。 

### 步骤 1 - 启动脚本

可在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/virtual-network-deploy-multinic-classic-cli.sh)下载所用的完整 bash 脚本。按照以下步骤更改脚本，以便用于具体环境。

1. 根据在上述[先决条件](#Prerequisites)中部署的现有资源组，更改以下变量的值。

		location="chinaeast"
		vnetName="WTestVNet"
		backendSubnetName="BackEnd"

2. 根据要用于后端部署的值，更改以下变量的值。

		backendCSName="IaaSStory-Backend"
		prmStorageAccountName="iaasstoryprmstorage"
		image="0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1"
		avSetName="ASDB"
		vmSize="Standard_DS3"
		diskSize=127
		vmNamePrefix="DB"
		osDiskName="osdiskdb"
		dataDiskPrefix="db"
		dataDiskName="datadisk"
		ipAddressPrefix="192.168.2."
		username='adminuser'
		password='adminP@ssw0rd'
		numberOfVMs=2

### 步骤 2 - 为 VM 创建必要的资源

1. 为所有后端 VM 创建新的云服务。请注意，使用 `$backendCSName` 变量表示资源组名称，使用 `$location` 表示 Azure 区域。

		azure service create --serviceName $backendCSName \
		    --location $location

2. 为 VM 将使用的 OS 和数据磁盘创建高级存储帐户。

		azure storage account create $prmStorageAccountName \
		    --location $location \
		    --type PLRS 

### 步骤 3 - 创建具有多个 NIC 的 VM

1. 启动循环，根据 `numberOfVMs` 变量创建多个 VM。

		for ((suffixNumber=1;suffixNumber<=numberOfVMs;suffixNumber++));
		do

2. 对于每个 VM，分别指定两个 NIC 的名称和 IP 地址。

		    nic1Name=$vmNamePrefix$suffixNumber-DA
		    x=$((suffixNumber+3))
		    ipAddress1=$ipAddressPrefix$x
		
		    nic2Name=$vmNamePrefix$suffixNumber-RA
		    x=$((suffixNumber+53))
		    ipAddress2=$ipAddressPrefix$x

4. 创建 VM。请注意 `--nic-config` 参数的用法，其中包含所有 NIC 的列表（包含名称、子网和 IP 地址）。

		    azure vm create $backendCSName $image $username $password \
		        --connect $backendCSName \
		        --vm-name $vmNamePrefix$suffixNumber \
		        --vm-size $vmSize \
		        --availability-set $avSetName \
		        --blob-url $prmStorageAccountName.blob.core.chinacloudapi.cn/vhds/$osDiskName$suffixNumber.vhd \
		        --virtual-network-name $vnetName \
		        --subnet-names $backendSubnetName \
		        --nic-config $nic1Name:$backendSubnetName:$ipAddress1::,$nic2Name:$backendSubnetName:$ipAddress2::

5. 为每个 VM 创建两个数据磁盘。

		    azure vm disk attach-new $vmNamePrefix$suffixNumber \
		        $diskSize \
		        vhds/$dataDiskPrefix$suffixNumber$dataDiskName-1.vhd
		
		    azure vm disk attach-new $vmNamePrefix$suffixNumber \
		        $diskSize \
		        vhds/$dataDiskPrefix$suffixNumber$dataDiskName-2.vhd
		done

### 步骤 4 - 运行脚本

既已根据需要下载并更改脚本，可运行该脚本以创建具有多个 NIC 的后端数据库 VM。

1. 保存该脚本并从 **Bash** 终端运行它。最初的输出将如下所示。

		info:    Executing command service create
		info:    Creating cloud service
		data:    Cloud service name IaaSStory-Backend
		info:    service create command OK
		info:    Executing command storage account create
		info:    Creating storage account
		info:    storage account create command OK
		info:    Executing command vm create
		info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
		info:    Looking up virtual network
		info:    Looking up cloud service
		info:    Getting cloud service properties
		info:    Looking up deployment
		info:    Creating VM

2. 几分钟后，执行将结束，其余输出如下所示。

		info:    OK
		info:    vm create command OK
		info:    Executing command vm disk attach-new
		info:    Getting virtual machines
		info:    Adding Data-Disk
		info:    vm disk attach-new command OK
		info:    Executing command vm disk attach-new
		info:    Getting virtual machines
		info:    Adding Data-Disk
		info:    vm disk attach-new command OK
		info:    Executing command vm create
		info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
		info:    Looking up virtual network
		info:    Looking up cloud service
		info:    Getting cloud service properties
		info:    Looking up deployment
		info:    Creating VM
		info:    OK
		info:    vm create command OK
		info:    Executing command vm disk attach-new
		info:    Getting virtual machines
		info:    Adding Data-Disk
		info:    vm disk attach-new command OK
		info:    Executing command vm disk attach-new
		info:    Getting virtual machines
		info:    Adding Data-Disk
		info:    vm disk attach-new command OK

<!---HONumber=Mooncake_Quality_Review_1215_2016-->