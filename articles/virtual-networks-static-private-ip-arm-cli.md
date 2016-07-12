<!-- ARM: tested -->

<properties 
   pageTitle="如何在 ARM 模式下使用 CLI 设置静态专用 IP | Azure"
   description="了解静态 IP (DIP) 以及如何在 ARM 模式下使用 CLI 对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/15/2016"
	wacn.date="06/06/2016"/>

# 如何在 Azure CLI 中设置静态专用 IP 地址

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../includes/virtual-networks-static-private-ip-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[管理经典部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-classic-cli/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../includes/virtual-networks-static-ip-scenario-include.md)]

下面的示例 Azure CLI 命令需要已创建简单的环境。如果你想要运行本文档中所显示的命令，首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-arm-cli/)中所述的测试环境。

## 如何在创建 VM 时指定静态专用 IP 地址
若要在名为 *TestVNet* 的 VNet 的 *FrontEnd* 子网中使用静态专用 IP *192.168.1.101* 创建名为 *DNS01* 的 VM，请按照以下步骤进行操作：

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。

2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	预期输出：

		info:    New mode is arm

3. 运行 **azure network public-ip create** 以为该 VM 创建公共 IP。在输出后显示的列表说明了所用的参数。

		azure network public-ip create -g TestRG -n TestPIP -l centralus
	
	预期输出：

		info:    Executing command network public-ip create
		+ Looking up the public ip "TestPIP"
		+ Creating public ip address "TestPIP"
		+ Looking up the public ip "TestPIP"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP
		data:    Name                            : TestPIP
		data:    Type                            : Microsoft.Network/publicIPAddresses
		data:    Location                        : centralus
		data:    Provisioning state              : Succeeded
		data:    Allocation method               : Dynamic
		data:    Idle timeout                    : 4
		info:    network public-ip create command OK

	- **-g（或 --resource-group）**。将在其中创建公共 IP 的资源组的名称。
	- **-n（或 --name）**。公共 IP 的名称。
	- **-l（或 --location）**。将在其中创建公共 IP 的 Azure 区域。对于我们的方案，为 *centralus*。

3. 运行 **azure network nic create** 命令以创建具有静态专用 IP 的 NIC。在输出后显示的列表说明了所用的参数。

		azure network nic create -g TestRG -n TestNIC -l centralus -a 192.168.1.101 -m TestVNet -k FrontEnd

	预期输出：

		+ Looking up the network interface "TestNIC"
		+ Looking up the subnet "FrontEnd"
		+ Creating network interface "TestNIC"
		+ Looking up the network interface "TestNIC"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
		data:    Name                            : TestNIC
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : centralus
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.1.101
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:
		info:    network nic create command OK

	- **-a（或 --private-ip-address）**。NIC 的静态专用 IP 地址。
	- **-m（或 --subnet-vnet-name）**。将在其中创建 NIC 的 VNet 的名称。
	- **-k（或 --subnet-name）**。将在其中创建 NIC 的子网的名称。

4. 运行 **azure vm create** 命令以使用上面创建的公共 IP 和 NIC 创建 VM。在输出后显示的列表说明了所用的参数。

		azure vm create -g TestRG -n DNS01 -l centralus -y Windows -f TestNIC -i TestPIP -F TestVNet -j FrontEnd -o vnetstorage -q bd507d3a70934695bc2128e3e5a255ba__RightImage-Windows-2012R2-x64-v14.2 -u adminuser -p AdminP@ssw0rd

	预期输出：

		info:    Executing command vm create
		+ Looking up the VM "DNS01"
		info:    Using the VM Size "Standard_A1"
		warn:    The image "bd507d3a70934695bc2128e3e5a255ba__RightImage-Windows-2012R2-x64-v14.2" will be used for VM
		info:    The [OS, Data] Disk or image configuration requires storage account
		+ Looking up the storage account vnetstorage
		+ Looking up the NIC "TestNIC"
		info:    Found an existing NIC "TestNIC"
		info:    Found an IP configuration with virtual network subnet id "/subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd" in the NIC "TestNIC"
		info:    Found public ip parameters, trying to setup PublicIP profile
		+ Looking up the public ip "TestPIP"
		info:    Found an existing PublicIP "TestPIP"
		info:    Configuring identified NIC IP configuration with PublicIP "TestPIP"
		+ Updating NIC "TestNIC"
		+ Looking up the NIC "TestNIC"
		+ Creating VM "DNS01"
		info:    vm create command OK

	- **-y（或 --os-type）**。VM 的操作系统类型，可以为 *Windows* 或 *Linux*。
	- **-f（或 --nic-name）**。VM 将使用的 NIC 的名称。
	- **-i（或 --public-ip-name）**。VM 将使用的公共 IP 的名称。
	- **-F（或 --vnet-name）**。将在其中创建 VM 的 VNet 的名称。
	- **-j（或 --vnet-subnet-name）**。将在其中创建 VM 的子网的名称。

## 如何检索 VM 的静态专用 IP 地址信息

若要查看使用上面的脚本创建的 VM 的静态专用 IP 地址信息，请运行以下 Azure CLI 命令并观察 *Private IP alloc-method* 和 *Private IP address* 的值：

	azure vm show -g TestRG -n DNS01

预期输出：

	info:    Executing command vm show
	+ Looking up the VM "DNS01"
	+ Looking up the NIC "TestNIC"
	+ Looking up the public ip "TestPIP
	data:    Id                              :/subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01
	data:    ProvisioningState               :Succeeded
	data:    Name                            :DNS01
	data:    Location                        :centralus
	data:    Type                            :Microsoft.Compute/virtualMachines
	data:
	data:    Hardware Profile:
	data:      Size                          :Standard_A1
	data:
	data:    Storage Profile:
	data:      Source image:
	data:        Id                          :/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/services/images/bd507d3a70934695bc2128e3e5a255ba__RightImage-Windows-2012R2-x64-v14.2
	data:
	data:      OS Disk:
	data:        OSType                      :Windows
	data:        Name                        :cli08d7bd987a0112a8-os-1441774961355
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://vnetstorage2.blob.core.chinacloudapi.cn/vhds/cli08d7bd987a0112a8-os-1441774961355vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :DNS01
	data:      User Name                     :adminuser
	data:      Windows Configuration:
	data:        Provision VM Agent          :true
	data:        Enable automatic updates    :true
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Id                        :/subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-90-1A-A8
	data:          Provisioning State        :Succeeded
	data:          Name                      :TestNIC
	data:          Location                  :centralus
	data:            Private IP alloc-method :Static
	data:            Private IP address      :192.168.1.101
	data:            Public IP address       :40.122.213.159
	info:    vm show command OK

## 如何从 VM 中删除静态专用 IP 地址
无法在用于资源管理器的 Azure CLI 中删除 NIC 的静态专用 IP 地址。你必须创建使用动态 IP 的新 NIC，从 VM 中删除以前的 NIC，然后将新 NIC 添加到 VM 中。若要更改上述命令使用的 VM 的 NIC，请执行以下步骤。
	
1. 运行 **azure network nic create** 命令以使用动态 IP 分配创建新 NIC。请注意，这次不需要指定 IP 地址。

		azure network nic create -g TestRG -n TestNIC2 -l centralus -m TestVNet -k FrontEnd

	预期输出：

		info:    Executing command network nic create
		+ Looking up the network interface "TestNIC2"
		+ Looking up the subnet "FrontEnd"
		+ Creating network interface "TestNIC2"
		+ Looking up the network interface "TestNIC2"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2
		data:    Name                            : TestNIC2
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : centralus
		data:    Provisioning state              : Succeeded
		data:    Enable IP forwarding            : false
		data:    IP configurations:
		data:      Name                          : NIC-config
		data:      Provisioning state            : Succeeded
		data:      Private IP address            : 192.168.1.6
		data:      Private IP Allocation Method  : Dynamic
		data:      Subnet                        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
		data:
		info:    network nic create command OK

2. 运行 **azure vm set** 命令以更改 VM 使用的 NIC。

		azure vm set -g TestRG -n DNS01 -N TestNIC2

	预期输出：

		info:    Executing command vm set
		+ Looking up the VM "DNS01"
		+ Looking up the NIC "TestNIC2"
		+ Updating VM "DNS01"
		info:    vm set command OK

3. 如果需要，请运行 **azure network nic delete** 命令以删除旧 NIC。

		azure network nic delete -g TestRG -n TestNIC --quiet

	预期输出：

		info:    Executing command network nic delete
		+ Looking up the network interface "TestNIC"
		+ Deleting network interface "TestNIC"
		info:    network nic delete command OK

## 如何将静态专用 IP 地址添加到现有 VM
若要将静态专用 IP 地址添加到使用上面的脚本创建的 VM 所使用的 NIC，请运行以下命令：

	azure network nic set -g TestRG -n TestNIC2 -a 192.168.1.101

预期输出：

	info:    Executing command network nic set
	+ Looking up the network interface "TestNIC2"
	+ Updating network interface "TestNIC2"
	+ Looking up the network interface "TestNIC2"
	data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2
	data:    Name                            : TestNIC2
	data:    Type                            : Microsoft.Network/networkInterfaces
	data:    Location                        : centralus
	data:    Provisioning state              : Succeeded
	data:    MAC address                     : 00-0D-3A-90-29-25
	data:    Enable IP forwarding            : false
	data:    Virtual machine                 : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01
	data:    IP configurations:
	data:      Name                          : NIC-config
	data:      Provisioning state            : Succeeded
	data:      Private IP address            : 192.168.1.101
	data:      Private IP Allocation Method  : Static
	data:      Subnet                        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd
	data:
	info:    network nic set command OK

## 后续步骤

- 了解[保留公共 IP 地址](/documentation/articles/virtual-networks-reserved-public-ip/)。
- 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
- 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=76-->