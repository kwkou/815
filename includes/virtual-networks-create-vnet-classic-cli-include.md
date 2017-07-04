## 如何使用 Azure CLI 创建经典 VNet

可以使用 Azure CLI 从运行 Windows、Linux 或 OSX 的任何计算机上的命令提示符管理 Azure 资源。若要使用 Azure CLI 创建 VNet，请执行以下步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure network vnet create** 命令以创建 VNet 和子网，如下所示。在输出后显示的列表说明了所用的参数。

			azure network vnet create --vnet TestVNet -e 192.168.0.0 -i 16 -n FrontEnd -p 192.168.1.0 -r 24 -l "China North"
	
	预期输出：

			info:    Executing command network vnet create
			+ Looking up network configuration
			+ Looking up locations
			+ Setting network configuration
			info:    network vnet create command OK

	- **--vnet**。要创建的 VNet 的名称。对于我们的方案，为 *TestVNet*
	- **-e（或 --address-space）**。VNet 地址空间。对于我们的方案，为 *192.168.0.0*
	- **-i（或 -cidr）**。采用 CIDR 格式的网络掩码。对于我们的方案，为 *16*。
	- **-n（或 --subnet-name）**。第一个子网的名称。对于我们的方案，为 *FrontEnd*。
	- **-p（或 --subnet-start-ip）**。子网或子网地址空间的起始 IP 地址。对于我们的方案，为 *192.168.1.0*。
	- **-r（或 --subnet-cidr）**。子网的网络掩码（采用 CIDR 格式）。对于我们的方案，为 *24*。
	- **-l（或 --location）**。要在其中创建 VNet 的 Azure 区域 。对于我们的方案，为 *China North*。

3. 运行 **azure network vnet subnet create** 命令以创建子网，如下所示。在输出后显示的列表说明了所用的参数。

			azure network vnet subnet create -t TestVNet -n BackEnd -a 192.168.2.0/24
	
	下面是上述命令的预期输出：

			info:    Executing command network vnet subnet create
			+ Looking up network configuration
			+ Creating subnet "BackEnd"
			+ Setting network configuration
			+ Looking up the subnet "BackEnd"
			+ Looking up network configuration
			data:    Name                            : BackEnd
			data:    Address prefix                  : 192.168.2.0/24
			info:    network vnet subnet create command OK

	- **-t（或 --vnet-name）**。将在其中创建子网的 VNet 的名称。对于我们的方案，为 *TestVNet*。
	- **-n（或 --name）**。新子网的名称。对于我们的方案，为 *BackEnd*。
	- **-a（或 --address-prefix）**。子网 CIDR 块。对于我们的方案，为 *192.168.2.0/24*。

4. 运行 **azure network vnet show** 命令以查看新 vnet 的属性，如下所示。

			azure network vnet show

	下面是上述命令的预期输出：

			info:    Executing command network vnet show
			Virtual network name: TestVNet
			+ Looking up the virtual network sites
			data:    Name                            : TestVNet
			data:    Location                        : China North
			data:    State                           : Created
			data:    Address space                   : 192.168.0.0/16
			data:    Subnets:
			data:      Name                          : FrontEnd
			data:      Address prefix                : 192.168.1.0/24
			data:
			data:      Name                          : BackEnd
			data:      Address prefix                : 192.168.2.0/24
			data:
			info:    network vnet show command OK

<!---HONumber=76-->