<!-- ARM: tested -->

## 如何使用 Azure CLI 创建 VNet

可以使用 Azure CLI 从运行 Windows、Linux 或 OSX 的任何计算机上的命令提示符管理 Azure 资源。若要使用 Azure CLI 创建 VNet，请执行以下步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 如有必要，请运行 **azure group create** 以创建新资源组，如下所示。请注意命令的输出。在输出后显示的列表说明了所用的参数。有关资源组的详细信息，请访问 [Azure 资源管理器概述](/documentation/articles/resource-group-overview/#resource-groups)。

		azure group create -n TestRG -l centralus

	下面是上述命令的预期输出：

		info:    Executing command group create
		+ Getting resource group TestRG
		+ Creating resource group TestRG
		info:    Created resource group TestRG
		data:    Id:                  /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG
		data:    Name:                TestRG
		data:    Location:            centralus
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK

	- **-n（或 --name）**。新资源组的名称。对于我们的方案，为 *TestRG*。
	- **-l（或 --location）**。将在其中创建新资源组的 Azure 区域。对于我们的方案，为 *centralus*。

4. 运行 **azure network vnet create** 命令以创建 VNet 和子网，如下所示。

		azure network vnet create -g TestRG -n TestVNet -a 192.168.0.0/16 -l centralus

	下面是上述命令的预期输出：

		info:    Executing command network vnet create
		+ Looking up virtual network "TestVNet"
		+ Creating virtual network "TestVNet"
		+ Loading virtual network state
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet2
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : centralus
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		info:    network vnet create command OK

	- **-g（或 --resource-group）**。将在其中创建 VNet 的资源组的名称。对于我们的方案，为 *TestRG*。
	- **-n（或 --name）**。要创建的 VNet 的名称。对于我们的方案，为 *TestVNet*
	- **-a（或 --address-prefixes）**。用于 VNet 地址空间的 CIDR 块的列表。对于我们的方案，为 *192.168.0.0/16*
	- **-l（或 --location）**。要在其中创建 VNet 的 Azure 区域 。对于我们的方案，为 *centralus*。

5. 运行 **azure network vnet subnet create** 命令以创建子网，如下所示。请注意命令的输出。在输出后显示的列表说明了所用的参数。

		azure network vnet subnet create -g TestRG -e TestVNet -n FrontEnd -a 192.168.1.0/24

	下面是上述命令的预期输出：

		info:    Executing command network vnet subnet create
		+ Looking up the subnet "FrontEnd"
		+ Creating subnet "FrontEnd"
		+ Looking up the subnet "FrontEnd"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:    Type                            : Microsoft.Network/virtualNetworks/subnets
		data:    ProvisioningState               : Succeeded
		data:    Name                            : FrontEnd
		data:    Address prefix                  : 192.168.1.0/24
		data:
		info:    network vnet subnet create command OK

	- **-e（或 --vnet-name）**。将在其中创建子网的 VNet 的名称。对于我们的方案，为 *TestVNet*。
	- **-n（或 --name）**。新子网的名称。对于我们的方案，为 *FrontEnd*。
	- **-a（或 --address-prefix）**。子网 CIDR 块。对于我们的方案，为 *192.168.1.0/24*。

6. 如有必要，重复执行上面的步骤 5 以创建其他子网。对于我们的方案，请运行以下命令以创建 *BackEnd* 子网。

		azure network vnet subnet create -g TestRG -e TestVNet -n BackEnd -a 192.168.2.0/24

4. 运行 **azure network vnet show** 命令以查看新 vnet 的属性，如下所示。

		azure network vnet show -g TestRG -n TestVNet

	下面是上述命令的预期输出：

		info:    Executing command network vnet show
		+ Looking up virtual network "TestVNet"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		data:    Name                            : TestVNet
		data:    Type                            : Microsoft.Network/virtualNetworks
		data:    Location                        : centralus
		data:    ProvisioningState               : Succeeded
		data:    Address prefixes:
		data:      192.168.0.0/16
		data:    Subnets:
		data:      Name                          : FrontEnd
		data:      Address prefix                : 192.168.1.0/24
		data:
		data:      Name                          : BackEnd
		data:      Address prefix                : 192.168.2.0/24
		data:
		info:    network vnet show command OK

<!---HONumber=76-->