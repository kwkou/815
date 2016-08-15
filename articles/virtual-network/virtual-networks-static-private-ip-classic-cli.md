<properties 
   pageTitle="如何在经典模式下使用 CLI 设置静态专用 IP | Azure"
   description="了解静态专用 IP (DIP) 以及如何在经典模式下使用 CLI 对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/15/2016"
	wacn.date="04/26/2016"/>

# 如何在 Azure CLI 中设置静态专用 IP 地址（经典）

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../includes/virtual-networks-static-private-ip-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[管理资源管理器部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-arm-cli/)。

下面的示例 Azure CLI 命令需要已创建简单的环境。如果你想要运行本文档中所显示的命令，首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-cli/) 中所述的测试环境。

## 如何在创建 VM 时指定静态专用 IP 地址
若要根据上述方案在名为 *TestService* 的新云服务中创建名为 *DNS01* 的新 VM，请按照以下步骤进行操作：

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
1. 运行 **azure service create** 命令以创建云服务。

		azure service create TestService --location uscentral

	预期输出：

		info:    Executing command service create
		info:    Creating cloud service
		data:    Cloud service name TestService
		info:    service create command OK
	
2. 运行 **azure create vm** 命令以创建 VM。请注意静态专用 IP 地址的值。在输出后显示的列表说明了所用的参数。

		azure vm create -l centralus -n DNS01 -w TestVNet -S "192.168.1.101" TestService bd507d3a70934695bc2128e3e5a255ba__RightImage-Windows-2012R2-x64-v14.2 adminuser AdminP@ssw0rd

	预期输出：

		info:    Executing command vm create
		warn:    --vm-size has not been specified. Defaulting to "Small".
		info:    Looking up image bd507d3a70934695bc2128e3e5a255ba__RightImage-Windows-2012R2-x64-v14.2
		info:    Looking up virtual network
		info:    Looking up cloud service
		warn:    --location option will be ignored
		info:    Getting cloud service properties
		info:    Looking up deployment
		info:    Retrieving storage accounts
		info:    Creating VM
		info:    OK
		info:    vm create command OK

	- **-l（或 --location）**。将在其中创建 VM 的 Azure 区域。对于我们的方案，为 *centralus*。
	- **-n（或 --vm-name）**。要创建的 VM 的名称。
	- **-w（或 --virtual-network-name）**。将在其中创建 VM 的 VNet 的名称。 
	- **-S（或 --static-ip）**。VM 的静态专用 IP 地址。
	- **TestService**。将在其中创建 VM 的云服务的名称。
	- **bd507d3a70934695bc2128e3e5a255ba\_\_RightImage-Windows-2012R2-x64-v14.2**。用于创建 VM 的映像。
	- **adminuser**。Windows VM 的本地管理员。
	- **AdminP@ssw0rd**。Windows VM 的本地管理员密码。

## 如何检索 VM 的静态专用 IP 地址信息
若要查看使用上述脚本创建的 VM 的静态专用 IP 地址信息，请运行以下 Azure CLI 命令并观察 *Network StaticIP* 的值：

	azure vm static-ip show DNS01

预期输出：

	info:    Executing command vm static-ip show
	info:    Getting virtual machines
	data:    Network StaticIP "192.168.1.101"
	info:    vm static-ip show command OK

## 如何从 VM 中删除静态专用 IP 地址
若要删除使用上述脚本添加到 VM 的静态专用 IP 地址，请运行以下 Azure CLI 命令：
	
	azure vm static-ip remove DNS01

预期输出：

	info:    Executing command vm static-ip remove
	info:    Getting virtual machines
	info:    Reading network configuration
	info:    Updating network configuration
	info:    vm static-ip remove command OK

## 如何将静态专用 IP 添加到现有 VM
若要向使用上述脚本创建的 VM 添加静态专用 IP 地址，请运行以下命令：

	azure vm static-ip set DNS01 192.168.1.101

预期输出：

	info:    Executing command vm static-ip set
	info:    Getting virtual machines
	info:    Looking up virtual network
	info:    Reading network configuration
	info:    Updating network configuration
	info:    vm static-ip set command OK

## 后续步骤

- 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
- 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
- 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=76-->