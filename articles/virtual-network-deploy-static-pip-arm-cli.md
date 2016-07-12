<!-- ARM: tested -->

<properties 
   pageTitle="在 Resource Manager 中使用 Azure CLI 通过静态公共 IP 部署 VM | Azure"
   description="了解如何在 Resource Manager 中使用 Azure CLI 通过静态公共 IP 部署 VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/15/2016"
	wacn.date="06/06/2016"/>

# 使用 Azure CLI 通过静态公共 IP 部署 VM

[AZURE.INCLUDE [virtual-network-deploy-static-pip-arm-selectors-include.md](../includes/virtual-network-deploy-static-pip-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

[AZURE.INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../includes/virtual-network-deploy-static-pip-scenario-include.md)]

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../includes/azure-cli-prerequisites-include.md)]

## 步骤 1 - 启动脚本

你可以在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/virtual-network-deploy-static-pip-arm-cli.sh)下载所用的完整 bash 脚本。请按照以下步骤更改要在你的环境中使用的脚本。

1. 根据需要用于部署的值更改以下变量的值。以下值映射到本文档中使用的方案。

		# Set variables for the new resource group
		rgName="IaaSStory"
		location="chinanorth"
		
		# Set variables for VNet
		vnetName="TestVNet"
		vnetPrefix="192.168.0.0/16"
		subnetName="FrontEnd"
		subnetPrefix="192.168.1.0/24"
		
		# Set variables for storage
		stdStorageAccountName="iaasstorystorage"
		
		# Set variables for VM
		vmSize="Standard_A1"
		diskSize=127
		publisher="Canonical"
		offer="UbuntuServer"
		sku="14.04.2-LTS"
		version="latest"
		vmName="WEB1"
		osDiskName="osdisk"
		nicName="NICWEB1"
		privateIPAddress="192.168.1.101"
		username='adminuser'
		password='adminP@ssw0rd'
		pipName="PIPWEB1"
		dnsName="iaasstoryws1"

## 步骤 2 - 为你的 VM 创建必要的资源

在创建 VM 之前，你需要可供 VM 使用的资源组、VNet、公共 IP 和 NIC。

1. 创建新的资源组。

		azure group create $rgName $location
		
2. 创建 VNet 和子网。
		
		azure network vnet create --resource-group $rgName \
		    --name $vnetName \
		    --address-prefixes $vnetPrefix \
		    --location $location
		azure network vnet subnet create --resource-group $rgName \
		    --vnet-name $vnetName \
		    --name $subnetName \
		    --address-prefix $subnetPrefix

3. 创建公共 IP 资源。

		azure network public-ip create --resource-group $rgName \
		    --name $pipName \
		    --location $location \
		    --allocation-method Static \
		    --domain-name-label $dnsName 

4. 使用公共 IP 为上面创建的子网中的 VM 创建网络接口 (NIC)。请注意，第一组命令用于检索上面创建的子网的 **Id**。

		subnetId="$(azure network vnet subnet show --resource-group $rgName \
		                --vnet-name $vnetName \
		                --name $subnetName|grep Id)"

		subnetId=${subnetId#*/}
		
		azure network nic create --name $nicName \
		    --resource-group $rgName \
		    --location $location \
		    --private-ip-address $privateIPAddress \
		    --subnet-id $subnetId \
		    --public-ip-name $pipName

>[AZURE.TIP] 上面的第一个命令使用 [grep](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_04_02.html) 和[字符串操作](http://tldp.org/LDP/abs/html/string-manipulation.html)（更具体地说，是子字符串删除）。

5. 创建存储帐户，以托管 VM OS 驱动器。

		azure storage account create $stdStorageAccountName \
		    --resource-group $rgName \
		    --location $location --type LRS 

## 步骤 3 - 创建 VM 

现在，所有必需的资源均已就绪，你可以创建新的 VM 了。

1. 创建 VM。

		azure vm create --resource-group $rgName \
		    --name $vmName \
		    --location $location \
		    --vm-size $vmSize \
		    --subnet-id $subnetId \
		    --nic-names $nicName \
		    --os-type linux \
		    --image-urn $publisher:$offer:$sku:$version \
		    --storage-account-name $stdStorageAccountName \
		    --storage-account-container-name vhds \
		    --os-disk-vhd $osDiskName.vhd \
		    --admin-username $username \
		    --admin-password $password

2. 保存脚本文件。

## 步骤 4 - 运行脚本

进行必要的更改并理解上面显示的脚本以后，可运行该脚本。

1. 在 bash 控制台中运行上面的脚本。

		sh myscript.sh

2. 几分钟后，将会显示以下输出。

		info:    Executing command group create
		info:    Getting resource group IaaSStory
		info:    Creating resource group IaaSStory
		info:    Created resource group IaaSStory
		data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory
		data:    Name:                IaaSStory
		data:    Location:            chinanorth
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK
		info:    Executing command network vnet create
		info:    Looking up virtual network "TestVNet"
		info:    Creating virtual network "TestVNet"
		info:    Loading virtual network state
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/TestVNet
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : chinanorth
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		info:    network vnet create command OK
		info:    Executing command network vnet subnet create
		info:    Looking up the subnet "FrontEnd"
		info:    Creating subnet "FrontEnd"
		info:    Looking up the subnet "FrontEnd"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:    Type                            : Microsoft.Network/virtualNetworks/subnets
		data:    ProvisioningState               : Succeeded
		data:    Name                            : FrontEnd
		data:    Address prefix                  : 192.168.1.0/24
		data:
		info:    network vnet subnet create command OK
		info:    Executing command network public-ip create
		info:    Looking up the public ip "PIPWEB1"
		info:    Creating public ip address "PIPWEB1"
		info:    Looking up the public ip "PIPWEB1"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/publicIPAddresses/PIPWEB1
		data:    Name                            : PIPWEB1
		data:    Type                            : Microsoft.Network/publicIPAddresses
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Allocation method               : Static
		data:    Idle timeout                    : 4
		data:    IP Address                      : 40.78.63.253
		data:    Domain name label               : iaasstoryws1
		data:    FQDN                            : iaasstoryws1.chinanorth.chinacloudapp.cn
		info:    network public-ip create command OK
		info:    Executing command network nic create
		info:    Looking up the network interface "NICWEB1"
		info:    Looking up the public ip "PIPWEB1"
		info:    Creating network interface "NICWEB1"
		info:    Looking up the network interface "NICWEB1"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/networkInterfaces/NICWEB1
		data:    Name                            : NICWEB1
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Public IP address             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/publicIPAddresses/PIPWEB1
		data:      Private IP address            : 192.168.1.101
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/IaaSStory2/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:
		info:    network nic create command OK
		info:    Executing command storage account create
		info:    Creating storage account
		info:    storage account create command OK
		info:    Executing command vm create
		info:    Looking up the VM "WEB1"
		info:    Using the VM Size "Standard_A1"
		info:    The [OS, Data] Disk or image configuration requires storage account
		info:    Looking up the storage account iaasstorystorage
		info:    Looking up the NIC "NICWEB1"
		info:    Creating VM "WEB1"
		info:    vm create command OK

<!---HONumber=Mooncake_0418_2016-->