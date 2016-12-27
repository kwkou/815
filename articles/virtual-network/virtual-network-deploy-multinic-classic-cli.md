<properties
    pageTitle="使用 Azure CLI 创建具有多个 NIC 的 VM（经典）| Azure"
    description="了解如何使用 Azure CLI 通过经典部署模型创建具有多个 NIC 的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-service-management" />  

<tags
    ms.assetid="b436e41e-866c-439f-a7c7-7b4b041725ef"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/02/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 Azure CLI 创建具有多个 NIC 的 VM（经典）
[AZURE.INCLUDE [virtual-network-deploy-multinic-classic-selectors-include.md](../../includes/virtual-network-deploy-multinic-classic-selectors-include.md)]

你可以在 Azure 中创建虚拟机 (VM)，然后将多个网络接口 (NIC) 附加到每个 VM。有多个 NIC 时，可跨各个 NIC 分隔不同的流量类型。例如，一个 NIC 可能会与 Internet 通信，而另一个 NIC 则只与未连接到 Internet 的内部资源通信。跨多个 NIC 分隔网络流量是许多网络虚拟设备（例如应用程序交付和 WAN 优化解决方案）所需的功能。

> [AZURE.IMPORTANT]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型的情况。Azure 建议大多数新部署使用 Resource Manager 模型。了解如何使用 [Resource Manager 部署模型](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)执行这些步骤。

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

以下步骤将名为 *IaaSStory* 的资源组用于 Web 服务器，并将名为 *IaaSStory-BackEnd* 的资源组用于数据库服务器。

## <a name="Prerequisites"></a> 先决条件
需要先创建具有此方案需要的所有资源的 *IaaSStory* 资源组，然后才能创建数据库服务器。若要创建这些资源，请完成以下步骤。若要创建虚拟网络，请完成[创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-classic-cli/)一文中的步骤。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## 部署后端 VM
后端 VM 取决于以下资源的创建：

* **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保部署到的 Azure 位置支持高级存储。
* **NIC**。每个 VM 都将具有两个 NIC，一个用于数据库访问，另一个用于管理。
* **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。

### 步骤 1 - 启动脚本
可在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/virtual-network-deploy-multinic-classic-cli.sh)下载所用的完整 bash 脚本。完成以下步骤，更改脚本，以便用于具体环境：

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

3. 创建 VM。请注意 `--nic-config` 参数的用法，其中包含所有 NIC 的列表（包含名称、子网和 IP 地址）。

        azure vm create $backendCSName $image $username $password \
            --connect $backendCSName \
            --vm-name $vmNamePrefix$suffixNumber \
            --vm-size $vmSize \
            --availability-set $avSetName \
            --blob-url $prmStorageAccountName.blob.core.chinacloudapi.cn/vhds/$osDiskName$suffixNumber.vhd \
            --virtual-network-name $vnetName \
            --subnet-names $backendSubnetName \
            --nic-config $nic1Name:$backendSubnetName:$ipAddress1::,$nic2Name:$backendSubnetName:$ipAddress2::

4. 为每个 VM 创建两个数据磁盘。

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

<!---HONumber=Mooncake_1219_2016-->