<!-- ARM: tested -->

## 下载并了解 ARM 模板

可以从 github 下载用于创建 VNet 和两个子网的现有 ARM 模板，进行任何所需的更改，然后重用该模板。为此，请执行以下步骤。

1. 导航到[示例模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets)。
2. 单击 **azuredeploy.json**，然后单击 **RAW**。
3. 将该文件保存到你计算机上的本地文件夹。
4. 如果你熟悉 ARM 模板，则跳到步骤 7。
5. 打开刚保存的文件，并查看 **parameters** 下第 5 行中的内容。ARM 模板参数提供了在部署过程中可以填充的值的占位符。

	| 参数 | 说明 |
	|---|---|
	| **位置** | 要在其中创建 VNet 的 Azure 区域 |
	| **vnetName** | 新 VNet 的名称 |
	| **addressPrefix** | VNet 的地址空间，采用 CIDR 格式 |
	| **subnet1Name** | 第一个 VNet 的名称 |
	| **subnet1Prefix** | 第一个子网的 CIDR 块 |
	| **subnet2Name** | 第二个 VNet 的名称 |
	| **subnet2Prefix** | 第二个子网的 CIDR 块 |

	>[AZURE.IMPORTANT] 在 github 中维护的 ARM 模板可能随着时间的推移发生变化。请确保在使用该模板之前对其进行检查。
	
6. 查看 **resources** 下的内容，并注意以下项：

	- **type**。模板创建的资源的类型。在此实例中为 **Microsoft.Network/virtualNetworks**，表示 VNet。
	- **name**。资源的名称。请注意使用 **[parameters('vnetName')]**，这意味着在部署过程中将由用户或参数文件作为输入提供该名称。
	- **properties**。资源的属性列表。此模板在 VNet 创建期间使用地址空间和子网属性。

7. 导航回[示例模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets)。
8. 单击 **azuredeploy-paremeters.json**，然后单击 **RAW**。
9. 将该文件保存到你计算机上的本地文件夹。
10. 打开刚保存的文件并编辑参数的值。使用以下值来部署我们的方案中所述的 VNet。

		{
		  "location": {
		    "value": "China North"
		  },
		  "vnetName": {
		      "value": "TestVNet"
		  },
		  "addressPrefix": {
		      "value": "192.168.0.0/16"
		  },
		  "subnet1Name": {
		      "value": "FrontEnd"
		  },
		  "subnet1Prefix": {
		    "value": "192.168.1.0/24"
		  },
		  "subnet2Name": {
		      "value": "BackEnd"
		  },
		  "subnet2Prefix": {
		      "value": "192.168.2.0/24"
		  }
		}

11. 保存文件。

<!---HONumber=Mooncake_0418_2016-->