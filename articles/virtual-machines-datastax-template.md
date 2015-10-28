<properties pageTitle="在 Ubuntu 上使用资源管理器模板部署 DataStax" description="了解如何使用 Azure PowerShell 或 Azure CLI 以及资源管理器模板，在 Ubuntu VM 上轻松部署新的 DataStax 群集。" services="virtual-machines" documentationCenter="" authors="karthmut" manager="timlt" editor="tysonn"/>

<tags ms.service="virtual-machines" ms.date="04/29/2015" wacn.date="06/26/2015"/>

# 在 Ubuntu 上使用资源管理器模板部署 DataStax

DataStax 是知名的行业领导者，该公司根据 Apache Cassandra™ 开发和提供各种解决方案，这是一种可提供商业支持且符合企业需求的 NoSQL 分布式数据库技术，此技术广受市场认可、敏捷、永不停摆，并可依照未来的各种公司规模需求进行调整。DataStax 提供 Enterprise (DSE) 和 Community (DSC) 版本。它还内存中计算、企业级安全性、快速强大的集成分析，以及企业管理等功能。

除了提供 Azure 应用商店中已可供使用的功能之外，现在你还可以使用通过 [Azure PowerShell](/documentation/articles/powershell-install-configure) 或 [Azure CLI](/documentation/articles/xplat-cli) 部署的资源管理器模板，在 Ubuntu VM 上轻松部署新的 DataStax 群集。

根据此模板部署的新群集采用下图中所述的拓扑，不过，你可以自定义本文中所述的模板，轻松实现其他拓扑：

![cluster-architecture](./media/virtual-machines-datastax-template/cluster-architecture.png)

使用参数，你可以定义将在新 Apache Cassandra 群集上部署的节点数目。系统还会在同一个 VNET 中独立的 VM 上部署 Datastax Operation Center 服务的实例，使你能够监视群集和各个节点的状态、添加/删除节点，以及执行所有与该群集相关的管理任务。

部署完成之后，你可以使用配置的 DNS 地址来访问 Datastax Operations Center VM 实例。OpsCenter VM 将启用 SSH 端口 22 以及针对HTTPS 启用端口 8443。适用于运营中心的 DNS 地址将包括作为参数输入的 *dnsName* 和 *region*，生成的格式为 `{dnsName}.{region}.cloudapp.azure.com`。如果你在创建部署时在“中国北部”区域中将 *dnsName* 参数设为“datastax”，就可以访问 `https://datastax.westus.cloudapp.azure.com:8443` 上部署的 Datastax Operations Center VM。

> [AZURE.NOTE]部署时使用的证书是一种自签名证书，会造成浏览器发出警告。你可以遵照 [Datastax](http://www.datastax.com/) 网站上的过程，将此证书替换成自己的 SSL 证书。

在深入了解与 Azure 资源管理器和将针对此次部署使用的模板的详细信息之前，请确定你已正确配置 Azure PowerShell 或 Azure CLI。

[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell.md)]

[AZURE.INCLUDE [xplat-getting-set-up-arm](../includes/xplat-getting-set-up-arm.md)]

## 使用资源管理器模板创建基于 Datastax 的 Cassandra 群集

请遵照以下步骤，使用 Github 模板存储库中的资源管理器模板，来创建基于 DataStax 的 Apache Cassandra 群集。每个步骤将包含适用于 Azure PowerShell 和 Azure CLI 的指令。

### 步骤 1-a：使用 PowerShell 下载模板文件

为 JSON 模板和其他关联的文件创建本地文件夹（例如，C:\Azure\\Templates\\DataStax）。

将文件夹名称替换为你的本地文件夹，然后运行以下命令：

	$folderName="C:\Azure\Templates\DataStax"
	$webclient = New-Object System.Net.WebClient
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/azuredeploy.json"
	$filePath = $folderName + "\azuredeploy.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/azuredeploy-parameters.json"
	$filePath = $folderName + "\azuredeploy-parameters.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/dsenode.sh"
	$filePath = $folderName + "\dsenode.sh"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/ephemeral-nodes-resources.json"
	$filePath = $folderName + "\ephemeral-nodes-resources.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/metadata.json"
	$filePath = $folderName + "\metadata.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/opscenter-install-resources.json"
	$filePath = $folderName + "\opscenter-install-resources.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/opscenter-resources.json"
	$filePath = $folderName + "\opscenter-resources.json"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/opscenter.sh"
	$filePath = $folderName + "\opscenter.sh"
	$webclient.DownloadFile($url,$filePath)
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/datastax-on-ubuntu/shared-resources.json"
	$filePath = $folderName + "shared-resources.json"
	$webclient.DownloadFile($url,$filePath)

### 步骤 1-b：使用 Azure CLI 下载模板文件

使用选择的 git 客户端复制整个模板存储库，例如：

	git clone https://github.com/Azure/azure-quickstart-templates C:\Azure\Templates

完成后，查找 C:\Azure\\Templates 目录中的 **datastax-on-ubuntu** 文件夹。

### 步骤 2：（可选）了解模板参数

部署非简单的解决方案（例如基于 DataStax 的 DataStax Apache Cassandra 群集）时，必须指定一组配置参数来处理一些所需的设置。通过在模板定义中声明这些参数，就能在通过外部文件或命令行进行部署期间指定值。

在 **azuredeploy.json** 文件顶部的“parameters”节中，你会发现模板需要用来配置 DataStax 群集的参数集。以下示例来自此模板的 azuredeploy.json 文件的 parameters 节：

	"parameters": {
		"region": {
			"type": "string",
			"defaultValue": "China North",
			"metadata": {
				"Description": "Location where resources will be provisioned"
			}
		},
		"storageAccountPrefix": {
			"type": "string",
			"defaultValue": "uniqueStorageAccountName",
			"metadata": {
				"Description": "Unique namespace for the Storage Account where the Virtual Machine's disks will be placed"
			}
		},
		"dnsName": {
			"type": "string",
			"metadata" : {
				"Description": "DNS subname for the opserations center public IP"
			}
		},
		"virtualNetworkName": {
			"type": "string",
			"defaultValue": "myvnet",
			"metadata": {
				"Description": "Name of the virtual network provisioned for the cluster"
			}
		},
		"adminUsername": {
			"type": "string",
			"metadata": {
				"Description": "Administrator user name used when provisioning virtual machines"
			}
		},
		"adminPassword": {
			"type": "securestring",
			"metadata": {
				"Description": "Administrator password used when provisioning virtual machines"
			}
		},
		"opsCenterAdminPassword": {
			"type": "securestring",
			"metadata": {
				"Description": "Sets the operations center admin user password"
			}
		},
		"clusterVmSize": {
			"type": "string",
			"defaultValue": "Standard_D3",
			"allowedValues": [
				"Standard_D1",
				"Standard_D2",
				"Standard_D3",
				"Standard_D4",
				"Standard_D11",
				"Standard_D12",
				"Standard_D13",
				"Standard_D14"
			],
			"metadata": {
				"Description": "The size of the virtual machines used when provisioning cluster nodes"
			}
		},
		"clusterNodeCount": {
			"type": "int",
			"metadata": {
				"Description": "The number of nodes provisioned in the cluster"
			}
		},
		"clusterName": {
			"type": "string",
			"metadata": {
				"Description": "The name of the cluster provisioned"
			}
		}
	}

每个参数都具有数据类型和允许值之类的详细信息。这样，便可以验证在交互模式（例如 PowerShell 或 Azure CLI）下，以及在自我发现 UI（通过分析所需参数列表及其说明动态生成的 UI）中执行模板期间所传递的参数。

### 步骤 3-a：使用 PowerShell 基于模板部署 DataStax 群集

通过创建包含所有参数的运行时值的 JSON 文件，为部署准备参数文件。然后，将此文件作为单个实体传递给部署命令。如果未包含参数文件，PowerShell 将使用模板中指定的任何默认值，然后提示你填写剩余的值。

下面是 **azuredeploy-parameters.json** 文件中的一组示例参数：

	{
		"storageAccountPrefix": {
			"value": "scorianisa"
		},
		"dnsName": {
			"value": "scorianids"
		},
		"virtualNetworkName": {
			"value": "datastax"
		},
		"adminUsername": {
			"value": "scoriani"
		},
		"adminPassword": {
			"value": ""
		},
		"region": {
			"value": "China North"
		},
		"opsCenterAdminPassword": {
			"value": ""
		},
		"clusterVmSize": {
			"value": "Standard_D3"
		},
		"clusterNodeCount": {
			"value": 3
		},
		"clusterName": {
			"value": "cl1"
		}
	}

填写 Azure 部署名称、资源组名称、Azure 位置，以及存储 JSON 部署文件的文件夹。然后运行以下命令：

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as China North>"
	$folderName="<folder name, such as C:\Azure\Templates\DataStax>"
	$templateFile= $folderName + "\azuredeploy.json"
	$templateParameterFile= $folderName + "\azuredeploy-parameters.json"

	New-AzureResourceGroup –Name $RGName –Location $locName

	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile

运行 **New-AzureResourceGroupDeployment** 命令时，会从 JSON 参数文件中提取参数值，然后相应地开始执行模板。在不同的环境（例如测试、生产等）中定义和使用多个参数文件有利于重复使用模板，并简化复杂的多环境解决方案。

部署时，请记得需要创建新的 Azure 存储帐户，因此，提供用作存储帐户参数的名称必须唯一且符合 Azure 存储帐户的所有要求（仅限小写字母和数字）。

部署期间以及部署之后，你可以检查设置期间发出的所有要求，包括发生的任何错误。

为此，请转到 [Azure 门户](https://manage.windowsazure.cn)并执行以下操作：

- 单击左侧导航栏上的“浏览”，向下滚动，然后单击“资源组”。  
- 单击刚创建的资源组之后，系统会显示“资源组”边栏选项卡。  
- 在“资源组”边栏选项卡的“监视”部分中单击“事件”柱状图可以查看部署的事件：
- 单击各个事件可以深入到系统代表模板执行的各项操作。

测试之后，如果需要删除此资源组及其所有资源（存储帐户、虚拟机和虚拟网络），请使用这一个命令：

	Remove-AzureResourceGroup –Name "<resource group name>" -Force

### 步骤 3-b：使用 Azure CLI 基于模板部署 DataStax 群集

若要通过 Azure CLI 部署 DataStax 群集，请先通过指定名称和位置来创建资源组：

	azure group create dsc "China North"

将此资源组名称、JSON 模板文件的位置，以及参数文件的位置（有关详细信息，请参阅上面的 PowerShell 部分）传入以下命令：

	azure group deployment create dsc -f .\azuredeploy.json -e .\azuredeploy-parameters.json

你可以使用以下命令来检查单个资源部署的状态：

	azure group deployment list dsc

## Datastax 模板结构和文件组织概览

为了让资源管理器模板的设计更加完善且可重复使用，你必须在部署 DataStax 之类的复杂解决方案期间，考虑清楚如何安排一连串复杂但又彼此相关的任务。除了通过相关扩展执行脚本之外，还可以利用 ARM **模板链接**和**资源循环**，这样就能实现模块化方法，而所有基于模板的复杂部署基本上都能重复使用此方法。

下图描述了从 GitHub 中为此部署下载的所有文件彼此间的关系：

![datastax-files](./media/virtual-machines-datastax-template/datastax-files.png)

本部分将指导你逐步了解 DataStax 群集的 **azuredeploy.json** 文件结构。

### “parameters”节

**azuredeploy.json** 的“parameters”节指定此模板中使用的可修改参数。前面提到的 **azuredeploy-parameters.json** 文件用于在模板执行期间将值传入 azuredeploy.json 的“parameters”节。

### “variables”节

“variables”节指定可在这整个模板中使用的变量。这包含多个字段（JSON 数据类型或片段），在执行时，这些字段将设置为常量或计算值。下面是此 Datastax 模板的“variables”节：

	"variables": {
	"templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/datastax-on-ubuntu/",
	"sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
	"clusterNodesTemplateUrl": "[concat(variables('templateBaseUrl'), 'ephemeral-nodes-resources.json')]",
	"opsCenterTemplateUrl": "[concat(variables('templateBaseUrl'), 'opscenter-resources.json')]",
	"opsCenterInstallTemplateUrl": "[concat(variables('templateBaseUrl'), 'opscenter-install-resources.json')]",
	"opsCenterVmSize": "Standard_A1",
	"networkSettings": {
		"virtualNetworkName": "[parameters('virtualNetworkName')]",
		"addressPrefix": "10.0.0.0/16",
		"subnet": {
			"dse": {
				"name": "dse",
				"prefix": "10.0.0.0/24",
				"vnet": "[parameters('virtualNetworkName')]"
			}
		},
		"statics": {
			"clusterRange": {
				"base": "10.0.0.",
				"start": 5
			},
			"opsCenter": "10.0.0.240"
		}
	},
	"osSettings": {
		"imageReference": {
			"publisher": "Canonical",
			"offer": "UbuntuServer",
			"sku": "14.04.2-LTS",
			"version": "latest"
		},
		"scripts": [
			"[concat(variables('templateBaseUrl'), 'dsenode.sh')]",
			"[concat(variables('templateBaseUrl'), 'opscenter.sh')]",
			"https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/shared_scripts/ubuntu/vm-disk-utils-0.1.sh"
		]
	},
	"sharedStorageAccountName": "[concat(parameters('storageAccountPrefix'),'cmn')]",
	"nodeList": "[concat(variables('networkSettings').statics.clusterRange.base, variables('networkSettings').statics.clusterRange.start, '-', parameters('clusterNodeCount'))]"
	},

深入分析此示例之后，你可以看到两种不同的方法。在第一个片段中，“osSettings”变量是嵌套的 JSON 元素，其中包含 4 个键/值对：

	"osSettings": {
	      "imageReference": {
	        "publisher": "Canonical",
	        "offer": "UbuntuServer",
	        "sku": "14.04.2-LTS",
	        "version": "latest"
	      },

	 
在这第二个片段中，“scripts”变量是 JSON 数组，其中的每个元素将在运行时使用模板语言函数 (concat)、另一个变量的值以及字符串常量来计算：

	      "scripts": [
	        "[concat(variables('templateBaseUrl'), 'dsenode.sh')]",
	        "[concat(variables('templateBaseUrl'), 'opscenter.sh')]",
	        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/shared_scripts/ubuntu/vm-disk-utils-0.1.sh"
	      ]

### “resources”节

大多数操作是在**“resources”**节中发生的。仔细查看此节，你会立即找到两个不同的案例：第一个案例是定义为 `Microsoft.Resources/deployments` 类型的元素，它基本上表示调用主部署中的嵌套部署。通过“templateLink”元素（和相关的版本属性），可以指定链接的模板文件，并在调用此文件时传递一组参数作为输入，如同你在此片段中所看到的那样：

	{
	      "name": "shared",
	      "type": "Microsoft.Resources/deployments",
	      "apiVersion": "2015-01-01",
	      "properties": {
	        "mode": "Incremental",
	        "templateLink": {
	          "uri": "[variables('sharedTemplateUrl')]",
	          "contentVersion": "1.0.0.0"
	        },
	        "parameters": {
	          "region": {
	            "value": "[parameters('region')]"
	          },
	          "networkSettings": {
	            "value": "[variables('networkSettings')]"
	          },
	          "storageAccountName": {
	            "value": "[variables('sharedStorageAccountName')]"
	          }
	        }
	      }
	    },

从第一个示例我们清楚地了解此方案中的 **azuredeploy.json** 如何作为一种协调机制进行组织，并调用其他一些模板文件，其中每个文件都负责所需部署活动的一部分。

具体而言，以下链接模板将用于此部署：

-	**shared-resource.json**：包含要在整个部署中共享的所有资源的定义。例如，用来存储 VM 的操作系统磁盘和虚拟网络的存储帐户。
-	**opscenter-resources.json**：部署 OpsCenter VM 和所有相关资源，包括网络接口和公共 IP 地址。
-	**opscenter-install-resources.json**：部署 OpsCenter VM 扩展（适用于 Linux 的自定义脚本），该扩展将调用特定的 bash 脚本文件 (**opscenter.sh**)，当你在该 VM 中设置 OpsCenter 服务时需要用到此文件。
-	**ephemeral-nodes-resources.json**：部署所有群集节点 VM 和连接的资源（网卡、专用 IP 等）。此模板还将部署 VM 扩展（适用于 Linux 的自定义脚本），并调用 bash 脚本 (**dsenode.sh**)，以在每个节点上实际安装 Apache Cassandra 程序代码。

让我们深入了解这最后一个模板的用法，因为从模板开发角度来看，这是最有趣的模板之一。在此要强调一个重要的概念，那就是一个模板文件可以重复部署某一种资源类型，而且每一个实例都可以为所需的设置指定唯一的值。此概念称为**资源循环**。

从 **azuredeploy.json** 主文件内部调用 **ephemeral-nodes-resources.json** 时，系统会提供名为 **nodeCount** 的参数作为参数列表的一部分。在子模板中，将在每个需要部署于多个副本中的资源的**“copy”**元素内使用 nodeCount（要在群集中部署的节点数目），如以下片段中突出显示的内容所示。对于你需要为不同部署资源实例提供唯一值的所有设置，可使用 **copyindex()** 函数获取数字值，以指出当前用来创建此特定资源循环的索引。在以下片段中，可以在为 Datastax 群集节点创建的多个 VM 上看到此概念的运用：

			   {
			      "apiVersion": "2015-05-01-preview",
			      "type": "Microsoft.Compute/virtualMachines",
			      "name": "[concat(parameters('namespace'), 'vm', copyindex())]",
			      "location": "[parameters('region')]",
			      "copy": {
			        "name": "[concat(parameters('namespace'), 'vmLoop')]",
			        "count": "[parameters('nodeCount')]"
			      },
			      "dependsOn": [
			        "[concat('Microsoft.Network/networkInterfaces/', parameters('namespace'), 'nic', copyindex())]",
			        "[concat('Microsoft.Compute/availabilitySets/', parameters('namespace'), 'set')]",
			        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]"
			      ],
			      "properties": {
			        "availabilitySet": {
			          "id": "[resourceId('Microsoft.Compute/availabilitySets', concat(parameters('namespace'), 'set'))]"
			        },
			        "hardwareProfile": {
			          "vmSize": "[parameters('vmSize')]"
			        },
			        "osProfile": {
			          "computername": "[concat(parameters('namespace'), 'vm', copyIndex())]",
			          "adminUsername": "[parameters('adminUsername')]",
			          "adminPassword": "[parameters('adminPassword')]"
			        },
			        "storageProfile": {
			          "imageReference": "[parameters('osSettings').imageReference]",
			          "osDisk": {
			            "name": "osdisk",
			            "vhd": {
			              "uri": "[concat('http://',variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', variables('vmName'), copyindex(), '-osdisk.vhd')]"
			            },
			            "caching": "ReadWrite",
			            "createOption": "FromImage"
			          },
			          "dataDisks": [
			            {
			              "name": "datadisk1",
			              "diskSizeGB": 1023,
			              "lun": 0,
			              "vhd": {
			                "Uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/','vhds/', variables('vmName'), copyindex(), 'DataDisk1.vhd')]"
			              },
			              "caching": "None",
			              "createOption": "Empty"
			            }
			          ]
			        },
			        "networkProfile": {
			          "networkInterfaces": [
			            {
			              "id": "[resourceId('Microsoft.Network/networkInterfaces',concat(parameters('namespace'), 'nic', copyindex()))]"
			            }
			          ]
			        }
			      }
			    },

创建资源的另一个重要概念是能够指定资源间的依赖性和优先级，如同你在 **dependsOn** JSON 数组中所见。在此特定模板中，每个节点上还会附加 1 TB 磁盘（请参阅“dataDisks”），该磁盘可用于托管 Apache Cassandra 实例的备份和快照。

在通过 **dsenode.sh** 脚本文件触发节点准备活动期间，将格式化附加的磁盘。该脚本的第一行将调用另一个脚本：

	bash vm-disk-utils-0.1.sh

vm-disk-utils-0.1.sh 是 azure-quickstart-tempates github 存储库中 **shared_scripts\\ubuntu** 文件夹的一部分，其中包含用于磁盘装入、格式设置和脚本化的非常有用的函数。可以在存储库中的所有模板内使用这些函数。

另一个要探讨的有趣片段是与 CustomScriptForLinux VM 扩展相关的片段。这些片段以独立资源类型进行安装，并依赖于每个群集节点（以及 OpsCenter 实例）。它们利用针对虚拟机所述的相同资源循环机制：

	{
	"type": "Microsoft.Compute/virtualMachines/extensions",
	"name": "[concat(parameters('namespace'), 'vm', copyindex(), '/installdsenode')]",
	"apiVersion": "2015-05-01-preview",
	"location": "[parameters('region')]",
	"copy": {
		"name": "[concat(parameters('namespace'), 'vmLoop')]",
		"count": "[parameters('nodeCount')]"
	},
	"dependsOn": [
		"[concat('Microsoft.Compute/virtualMachines/', parameters('namespace'), 'vm', copyindex())]",
		"[concat('Microsoft.Network/networkInterfaces/', parameters('namespace'), 'nic', copyindex())]"
	],
	"properties": {
		"publisher": "Microsoft.OSTCExtensions",
		"type": "CustomScriptForLinux",
		"typeHandlerVersion": "1.2",
		"settings": {
			"fileUris": "[parameters('osSettings').scripts]",
			"commandToExecute": "bash dsenode.sh"
		}
	}
	}

熟悉此部署包含的其他文件后，你将能够了解有关如何利用 Azure 资源管理器模板来组织和协调适用于基于任何技术的多节点解决方案的复杂部署策略所需的所有详细信息和最佳实践。在此建议一种模板文件构建方法，请自行决定是否采用，如下图突出显示的部分所示：

![datastax-template-structure](./media/virtual-machines-datastax-template/datastax-template-structure.png)

本质上，这种方法会建议：

-	将你的核心模板文件定义成所有特定部署活动的中心协调点，并利用模板链接来调用子模板执行。
-	创建特定的模板文件，用于部署其他特定部署任务会共享的所有资源（例如存储帐户、vnet 配置，等等）。如果不同的部署具有类似的共同基础结构要求，你就可以频繁重复使用此方法。
-	根据特定资源的场地要求包含可选资源模板
-	针对资源组的相同成员（群集中的节点，等等）创建利用资源循环的特定模板，以便部署多个具有不同属性的实例。
-	对于所有部署后任务（例如产品安装、配置，等等），利用脚本部署扩展并创建特定于每种技术的脚本

有关详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-CN/library/azure/dn835138.aspx)。

<!---HONumber=61-->