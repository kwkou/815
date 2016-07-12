<!-- ARM: tested -->

## 使用 Azure CLI 部署 ARM 模板

若要使用 Azure CLI 部署下载的 ARM 模板，请执行以下步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 如有必要，请运行 **azure group create** 以创建新资源组，如下所示。请注意命令的输出。在输出后显示的列表说明了所用的参数。有关资源组的详细信息，请访问 [Azure 资源管理器概述](documentation/articles/resource-group-overview)。

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

4. 运行 **azure group deployment create** cmdlet 以使用你在前面下载并修改的模板和参数文件部署新 VNet。在输出后显示的列表说明了所用的参数。

		azure group deployment create -g TestRG -n TestVNetDeployment -f C:\ARM\azuredeploy.json -e C:\ARM\azuredeploy-parameters.json

	下面是上述命令的预期输出：

		info:    Executing command group deployment create
		+ Initializing template configurations and parameters
		+ Creating a deployment
		info:    Created template deployment "TestVNetDeployment"
		+ Registering providers
		info:    Registering provider microsoft.network
		+ Waiting for deployment to complete
		data:    DeploymentName     : TestVNetDeployment
		data:    ResourceGroupName  : TestRG
		data:    ProvisioningState  : Succeeded
		data:    Timestamp          : 2015-08-14T21:56:11.152759Z
		data:    Mode               : Incremental
		data:    Name           Type    Value
		data:    -------------  ------  --------------
		data:    location       String  Central US
		data:    vnetName       String  TestVNet
		data:    addressPrefix  String  192.168.0.0/16
		data:    subnet1Prefix  String  192.168.1.0/24
		data:    subnet1Name    String  FrontEnd
		data:    subnet2Prefix  String  192.168.2.0/24
		data:    subnet2Name    String  BackEnd
		info:    group deployment create command OK

	- **-g（或 --resource-group）**。将在其中创建新 VNet 的资源组的名称。
	- **-f（或 --template-file）**。ARM 模板文件的路径。
	- **-e（或 --parameters-file）**。ARM 参数文件的路径。

5. 运行 **azure network vnet show** 命令以查看新 vnet 的属性，如下所示。

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