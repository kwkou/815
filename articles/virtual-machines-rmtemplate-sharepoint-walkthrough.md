<properties
	pageTitle="三服务器 SharePoint 场资源管理器模板"
	description="逐步完成三服务器 SharePoint 场的 Azure 资源管理器模板结构。"
	services="virtual-machines"
	documentationCenter=""
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/28/2015"
	wacn.date="09/18/2015"/>

# 三服务器 SharePoint 场资源管理器模板

本主题将指导你逐步完成三服务器 SharePoint 场的 azuredeploy.json 模板文件结构。你可以在浏览器中从[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json)查看该模板的内容 。

或者，若要查看 azuredeploy.json 文件的本地副本，请将本地文件夹指定为该文件的位置，然后创建该文件（例如，C:\\Azure\\Templates\\SharePointFarm）。填写文件夹名称，并在 Azure PowerShell 命令提示符下运行这些命令。

	$folderName="<folder name, such as C:\Azure\Templates\SharePointFarm>"
	$webclient = New-Object System.Net.WebClient
	$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json"
	$filePath = $folderName + "\azuredeploy.json"
	$webclient.DownloadFile($url,$filePath)

在所选的文本编辑器或工具中打开 azuredeploy.json 模板。下面介绍该模板文件的结构以及每个节的用途。

## “parameters”节

“parameters”节指定用于将数据输入到模板的参数。执行模板时，必须提供数据。最多可以定义 50 个参数。下面是 Azure 位置参数的示例：

	"deploymentLocation": {
		"type": "string",
		"allowedValues": [
			"China North",
			"China East",
		],
		"metadata": {
			"Description": "The region to deploy the resources into"
		},
		"defaultValue":"West Europe"
	},

## “variables”节

“variables”节指定该模板使用的变量及其值。变量值可以显式设置或从参数值得到。与参数不同，在执行模板时不用提供这些值。最多可以定义 100 个变量。下面是一些示例：

	"LBFE": "LBFE",
	"LBBE": "LBBE",
	"RDPNAT": "RDP",
	"spWebNAT": "spWeb",
	"adSubnetName": "adSubnet",
	"sqlSubnetName": "sqlSubnet",
	"spSubnetName": "spSubnet",
	"adNicName": "adNic",
	"sqlNicName": "sqlNic",
	"spNicName": "spNic",

## “resources”节

**“resources”**节指定在资源组中部署 SharePoint 场的资源所需的信息。最多可以定义 250 个资源，并且只能定义深度为 5 层的资源依赖关系。

此节包含以下子节：

### Microsoft.Storage/storageAccounts  

此节为场的所有 VHD 和磁盘资源创建新的存储帐户。下面是存储帐户的 JSON 代码：

	{
	  "type": "Microsoft.Storage/storageAccounts",
	  "name": "[parameters('newStorageAccountName')]",
	  "apiVersion": "2015-05-01-preview",
	  "location": "[parameters('deploymentLocation')]",
	  "properties": {
		"accountType": "[parameters('storageAccountType')]"
	  }
	},

### Microsoft.Network/publicIPAddresses

这些节创建一组公共 IP 地址，通过这些 IP 地址可在 Internet 上访问每个虚拟机。下面是一个示例：

	{
		"apiVersion": "2015-05-01-preview",
		"type": "Microsoft.Network/publicIPAddresses",
		"name": "[variables('adpublicIPAddressName')]",
		"location": "[parameters('deploymentLocation')]",
		"properties": {
			"publicIPAllocationMethod": "[parameters('publicIPAddressType')]",
			"dnsSettings": {
				"domainNameLabel": "[parameters('adDNSPrefix')]"
			}
		}
	},

### Microsoft.Compute/availabilitySets

这些节创建三个可用性集（每个部署层对应一个可用性集）：

- Active Directory 域控制器
- SQL Server 群集
- SharePoint Server

下面是一个示例：

	{
		"type": "Microsoft.Compute/availabilitySets",
		"name": "[variables('spAvailabilitySetName')]",
		"apiVersion": "2015-05-01-preview",
		"location": "[parameters('deploymentLocation')]"
	},

### Microsoft.Network/virtualNetworks

这些节创建具有三个子网（其中放置虚拟机，每个部署层对应一个子网）的仅限云虚拟网络。下面是 JSON 代码：

	{
		"name": "[parameters('virtualNetworkName')]",
		"type": "Microsoft.Network/virtualNetworks",
		"location": "[parameters('deploymentLocation')]",
		"apiVersion": "2015-05-01-preview",
		"properties": {
			"addressSpace": {
			"addressPrefixes": [
				"[parameters('virtualNetworkAddressRange')]"
			]
			},
			"subnets": "[variables('subnets')]"
		}
	},


### Microsoft.Network/loadBalancers

这些节为每个虚拟机创建负载平衡器实例，以便为来自 Internet 的入站流量提供网络地址转换 (NAT) 和流量筛选。对于每个负载平衡器，设置将配置前端、后端和入站 NAT 规则。例如，每个虚拟机都有远程桌面通信规则，而 SharePoint Server 则有允许接收来自 Internet 的入站 Web 流量（TCP 端口 80）的规则。下面是 SharePoint Server 的示例：


	{
		"apiVersion": "2015-05-01-preview",
		"name": "[variables('spLBName')]",
		"type": "Microsoft.Network/loadBalancers",
		"location": "[parameters('deploymentLocation')]",
		"dependsOn": [
			"[resourceId('Microsoft.Network/publicIPAddresses',variables('sppublicIPAddressName'))]"
		],
		"properties": {
			"frontendIPConfigurations": [
				{
					"name": "[variables('LBFE')]",
					"properties": {
						"publicIPAddress": {
							"id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('sppublicIPAddressName'))]"
						}
					}
				}
			],
			"backendAddressPools": [
				{
					"name": "[variables('LBBE')]"
				}
			],
			"inboundNatRules": [
				{
					"name": "[variables('RDPNAT')]",
					"properties": {
						"frontendIPConfiguration": {
							"id": "[variables('splbFEConfigID')]"
						},
						"protocol": "tcp",
						"frontendPort": "[parameters('RDPPort')]",
						"backendPort": 3389,
						"enableFloatingIP": false
					}
				},
				{
					"name": "[variables('spWebNAT')]",
					"properties": {
						"frontendIPConfiguration": {
							"id": "[variables('splbFEConfigID')]"
						},
						"protocol": "tcp",
						"frontendPort": 80,
						"backendPort": 80,
						"enableFloatingIP": false
					}
				}
			]
		}
	},

### Microsoft.Network/networkInterfaces

这些节为每个虚拟机创建一个网络接口，并为域控制器配置静态 IP 地址。下面是域控制器的网络接口示例：

	{
		"name": "[variables('adNicName')]",
		"type": "Microsoft.Network/networkInterfaces",
		"location": "[parameters('deploymentLocation')]",
		"dependsOn": [
			"[parameters('virtualNetworkName')]",
			"[concat('Microsoft.Network/loadBalancers/',variables('adlbName'))]"
		],
		"apiVersion": "2015-05-01-preview",
		"properties": {
			"ipConfigurations": [
				{
					"name": "ipconfig1",
					"properties": {
						"privateIPAllocationMethod": "Static",
						"privateIPAddress": "[parameters('adNicIPAddress')]",
						"subnet": {
							"id": "[variables('adSubnetRef')]"
						},
						"loadBalancerBackendAddressPools": [
							{
								"id": "[variables('adBEAddressPoolID')]"
							}
						],
						"loadBalancerInboundNatRules": [
							{
								"id": "[variables('adRDPNATRuleID')]"
							}
						]
					}
				}
			]
		}
	},


### Microsoft.Compute/virtualMachines

这些节在部署中创建并配置三个虚拟机。

第一个节创建并配置域控制器，它：

- 指定存储帐户、可用性集、网络接口和负载平衡器实例。
- 添加额外的磁盘。
- 运行 PowerShell 脚本，以配置域控制器。

下面是 JSON 代码：

		{
			"apiVersion": "2015-05-01-preview",
			"type": "Microsoft.Compute/virtualMachines",
			"name": "[parameters('adVMName')]",
			"location": "[parameters('deploymentLocation')]",
			"dependsOn": [
				"[resourceId('Microsoft.Storage/storageAccounts',parameters('newStorageAccountName'))]",
				"[resourceId('Microsoft.Network/networkInterfaces',variables('adNicName'))]",
				"[resourceId('Microsoft.Compute/availabilitySets', variables('adAvailabilitySetName'))]",
				"[resourceId('Microsoft.Network/loadBalancers',variables('adlbName'))]"
			],
			"properties": {
				"hardwareProfile": {
					"vmSize": "[parameters('adVMSize')]"
				},
				"availabilitySet": {
					"id": "[resourceId('Microsoft.Compute/availabilitySets', variables('adAvailabilitySetName'))]"
				},
				"osProfile": {
					"computername": "[parameters('adVMName')]",
					"adminUsername": "[parameters('adminUsername')]",
					"adminPassword": "[parameters('adminPassword')]"
				},
				"storageProfile": {
					"imageReference": {
						"publisher": "[parameters('adImagePublisher')]",
						"offer": "[parameters('adImageOffer')]",
						"sku": "[parameters('adImageSKU')]",
						"version": "latest"
					},
					"osDisk": {
						"name": "osdisk",
						"vhd": {
							"uri": "[concat('http://',parameters('newStorageAccountName'),'.blob.core.chinacloudapi.cn/',parameters('vmContainerName'),'/',parameters('adVMName'),'-osdisk.vhd')]"
						},
						"caching": "ReadWrite",
						"createOption": "FromImage"
					},
					"dataDisks": [
						{
							"vhd": {
								"uri": "[concat('http://',parameters('newStorageAccountName'),'.blob.core.chinacloudapi.cn/',parameters('vmContainerName'),'/', variables('adDataDisk'),'-1.vhd')]"
							},
							"name": "[concat(parameters('adVMName'),'-data-disk1')]",
							"caching": "None",
							"createOption": "empty",
							"diskSizeGB": "[variables('adDataDiskSize')]",
							"lun": 0
						}
					]
				},
				"networkProfile": {
					"networkInterfaces": [
						{
							"id": "[resourceId('Microsoft.Network/networkInterfaces',variables('adNicName'))]"
						}
					]
				}
			},
			"resources": [
				{
					"type": "Microsoft.Compute/virtualMachines/extensions",
					"name": "[concat(parameters('adVMName'),'/InstallDomainController')]",
					"apiVersion": "2015-05-01-preview",
					"location": "[parameters('deploymentLocation')]",
					"dependsOn": [
						"[resourceId('Microsoft.Compute/virtualMachines', parameters('adVMName'))]"
					],
					"properties": {
						"publisher": "Microsoft.Powershell",
						"type": "DSC",
						"typeHandlerVersion": "1.7",
						"settings": {
							"ModulesUrl": "[variables('adModulesURL')]",
							"ConfigurationFunction": "[variables('adConfigurationFunction')]",
							"Properties": {
								"DomainName": "[parameters('domainName')]",
								"AdminCreds": {
									"UserName": "[parameters('adminUserName')]",
									"Password": "PrivateSettingsRef:AdminPassword"
								}
							}
						},
						"protectedSettings": {
							"Items": {
								"AdminPassword": "[parameters('adminPassword')]"
							}
						}
					}
				}
			]
		},

以 **"name": "UpdateVNetDNS"** 开头的域控制器的其他节配置虚拟网络的 DNS 服务器以使用域控制器的静态 IP 地址。

下一个 **"type":"Microsoft.Compute/virtualMachines"** 节在部署中创建 SQL Server 虚拟机并：

- 指定存储帐户、可用性集、负载平衡器、虚拟网络和网络接口。
- 添加额外的磁盘。

其他**“Microsoft.Compute/virtualMachines/extensions”**节调用 PowerShell 脚本以配置 SQL Server。

下一个 **"type":"Microsoft.Compute/virtualMachines"** 节在部署中创建 SharePoint 虚拟机，并指定存储帐户、可用性集、负载平衡器、虚拟网络和网络接口。其他**“Microsoft.Compute/virtualMachines/extensions”**节调用 PowerShell 脚本以配置 SharePoint 场。

请注意 JSON 文件的**“resources”**节的子节的整体结构：

1.	创建支持多个虚拟机所需的 Azure 基础结构元素（存储帐户、公共 IP 地址、可用性集、虚拟网络、网络接口和负载平衡器实例）。
2.	创建域控制器虚拟机，后者使用先前创建的 Azure 基础结构的公用和特定元素、添加数据磁盘，并运行 PowerShell 脚本。此外，还更新虚拟网络以使用域控制器的静态 IP 地址。
3.	创建 SQL Server 虚拟机，后者使用先前为域控制器创建的 Azure 基础结构的公用和特定元素、添加数据磁盘，并运行 PowerShell 脚本以配置 SQL Server。
4.	创建 SharePoint Server 虚拟机，后者使用先前创建的 Azure 基础结构的公用和特定元素并运行 PowerShell 脚本以配置 SharePoint 场。

你自己的用于在 Azure 中构建多层基础结构的 JSON 模板应遵循相同的步骤：

1.	创建部署所需的 Azure 基础结构的公用元素（存储帐户、虚拟网络）、特定于层的元素（可用性集）和特定于虚拟机的元素（公共 IP 地址、可用性集、网络接口、负载平衡器实例）。
2.	对于应用程序中的每个层（例如，身份验证、数据库、Web），使用公用元素（存储帐户、虚拟网络）、特定于层的元素（可用性集）和特定于虚拟机的元素（公共 IP 地址、网络接口、负载平衡器实例）在该层中创建并配置服务器。

有关详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-CN/library/azure/dn835138.aspx)。

## 其他资源

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

[Azure 资源管理器概述](/documentation/articles/resource-group-overview)

<!--[-->创作 Azure 资源管理器模板<!--](/documentation/articles/resource-group-authoring-templates)-->

[虚拟机文档](/home/features/virtual-machines/)

<!---HONumber=70-->