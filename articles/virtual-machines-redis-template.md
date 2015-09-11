<properties
	pageTitle="Redis 群集资源管理器模板"
	description="了解如何使用 Azure PowerShell 或 Azure CLI 以及资源管理器模板，在 Ubuntu VM 上轻松部署新的 Redis 群集"
	services="virtual-machines"
	documentationCenter=""
	authors="timwieman"
	manager="timlt"
	editor="tysonn"/>

<tags 
	ms.service="virtual-machines"
	ms.date="05/04/2015"
	wacn.date="08/29/2015"/>

# 使用资源管理器模板的 Redis 群集

Redis 是开源键/值缓存和存储，其中键可以包含字符串、哈希、列表、集和经过排序的集等数据结构。Redis 支持对这些数据类型执行一组原子操作。随着 Redis 版本 3.0 的发布，Redis 群集现已在 Redis 的最新稳定版本中提供。Redis 群集是 Redis 的分布式实现，其中数据跨多个 Redis 节点自动分片，并且它可以在节点的子集遇到故障时继续运行。

Windows Azure Redis 缓存是专用 Redis 缓存服务，由 Microsoft 托管，但并非所有 Windows Azure 客户都想要使用 Azure Redis 缓存。一些客户想要在其自己的 Azure 部署中将 Redis 缓存保留在子网后面，而其他客户则更愿意在 Linux 虚拟机上托管其自己的 Redis 服务器，以便充分利用所有 Redis 功能。

本教程将指导你完成使用示例 Azure 资源管理器模板将 Redis 群集部署到 Windows Azure 中某个资源组的子网内的 Ubuntu VM 上。除了 Redis 3.0 群集外，此模板还支持使用 Redis Sentinel 部署 Redis 2.8。请注意，本教程将重点介绍 Redis 3.0 群集实现。

Redis 群集在子网后面创建，因此没有公共 IP 可访问 Redis 群集。在部署过程中，可部署可选的“跳转盒”。此“跳转盒”也是子网中部署的 Ubuntu VM，但它*确实*公开了你可以使用 SSH 连接到的开放 SSH 端口的公共 IP 地址。然后从“跳转盒”，就可以使用 SSH 连接到子网中的所有 Redis VM。

此模板利用“T恤大小”概念来指定“小型”、“中等”或“大型”Redis 群集安装。如果 Azure 资源管理器模板语言支持更多动态模板大小，则可以更改此项以指定 Redis 群集主节点数、从属节点数、VM 大小等。现在，你可以在文件 azuredeploy.json 中的变量 `tshirtSizeSmall`、`tshirtSizeMedium` 和 `tshirtSizeLarge` 中看到 VM 大小、主节点数和从属节点数。

“中等”T 恤大小的 Redis 群集模板可创建此配置：

![cluster-architecture](./media/virtual-machines-redis-template/cluster-architecture.png)

在深入了解与 Azure 资源管理器和将用于此部署的模板相关的详细信息之前，请确保你已正确配置 Azure PowerShell 或 Azure CLI。

[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell.md)]

[AZURE.INCLUDE [xplat-getting-set-up-arm](../includes/xplat-getting-set-up-arm.md)]

## 使用资源管理器模板部署 Redis 群集

按照以下步骤，使用 Github 模板存储库中的资源管理器模板创建 Redis 群集。每个步骤将同时提供 Azure PowerShell 和 Azure CLI 指令。

### 步骤 1-a：使用 Azure PowerShell 下载模板文件

为 JSON 模板和其他关联的文件创建本地文件夹（例如，C:\\Azure\\Templates\\RedisCluster）。

替换为你的本地文件夹的文件夹名称，并运行以下命令：

```powershell
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$webclient = New-Object System.Net.WebClient
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json"
$filePath = $folderName + "\azuredeploy.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy-parameters.json"
$filePath = $folderName + "\azuredeploy-parameters.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/empty-resources.json"
$filePath = $folderName + "\empty-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/jumpbox-resources.json"
$filePath = $folderName + "\jumpbox-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/node-resources.json"
$filePath = $folderName + "\node-resources.json"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-cluster-install.sh"
$filePath = $folderName + "\redis-cluster-install.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-cluster-setup.sh"
$filePath = $folderName + "\redis-cluster-setup.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/redis-sentinel-startup.sh"
$filePath = $folderName + "\redis-sentinel-startup.sh"
$webclient.DownloadFile($url,$filePath)

$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/shared-resources.json"
$filePath = $folderName + "\shared-resources.json"
$webclient.DownloadFile($url,$filePath)
```

### 步骤 1-b：使用 Azure CLI 下载模板文件

使用所选的 Git 客户端克隆整个模板存储库，例如：

```
git clone https://github.com/Azure/azure-quickstart-templates C:\Azure\Templates
```

完成克隆后，在 C:\\Azure\\Templates 目录中查找 **redis-high-availability** 文件夹。

### 步骤 2：（可选）了解模板参数

使用模板创建 Redis 群集时，必须指定一组配置参数。若要在运行命令以创建 Redis 群集之前查看需要为本地 JSON 文件中的模板指定的参数，请在所选的工具或文本编辑器中打开 JSON 文件。

在该文件顶部查找“parameters”节，其中列出了该模板配置 Redis 群集所需的参数集。下面是 azuredeploy.json 模板的“parameters”节：

```json
"parameters": {
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
	"storageAccountName": {
		"type": "string",
		"defaultValue": "",
		"metadata": {
			"Description": "Unique namespace for the Storage account where the virtual machine's disks will be placed"
		}
	},
	"location": {
		"type": "string",
		"metadata": {
			"Description": "Location where resources will be provisioned"
		}
	},
	"virtualNetworkName": {
		"type": "string",
		"defaultValue": "redisVirtNet",
		"metadata": {
			"Description": "The arbitrary name of the virtual network provisioned for the Redis cluster"
		}
	},
	"addressPrefix": {
		"type": "string",
		"defaultValue": "10.0.0.0/16",
		"metadata": {
			"Description": "The network address space for the virtual network"
		}
	},
	"subnetName": {
		"type": "string",
		"defaultValue": "redisSubnet1",
		"metadata": {
			"Description": "Subnet name for the virtual network that resources will be provisioned into"
		}
	},
	"subnetPrefix": {
		"type": "string",
		"defaultValue": "10.0.0.0/24",
		"metadata": {
			"Description": "Address space for the virtual network subnet"
		}
	},
	"nodeAddressPrefix": {
		"type": "string",
		"defaultValue": "10.0.0.1",
		"metadata": {
			"Description": "The IP address prefix that will be used for constructing a static private IP address for each node in the cluster"
		}
	},
	"jumpbox": {
		"type": "string",
		"defaultValue": "Disabled",
		"allowedValues": [
			"Enabled",
			"Disabled"
		],
		"metadata": {
			"Description": "The flag allowing to enable or disable provisioning of the jump-box VM that can be used to access the Redis nodes"
		}
	},
	"tshirtSize": {
		"type": "string",
		"defaultValue": "Small",
		"allowedValues": [
			"Small",
			"Medium",
			"Large"
		],
		"metadata": {
			"Description": "T-shirt size of the Redis deployment"
		}
	},
	"redisVersion": {
		"type": "string",
		"defaultValue": "stable",
		"metadata": {
			"Description": "The version of the Redis package to be deployed on the cluster (or use 'stable' to pull in the latest and greatest)"
		}
	},
	"redisClusterName": {
		"type": "string",
		"defaultValue": "redis-cluster",
		"metadata": {
			"Description": "The arbitrary name of the Redis cluster"
		}
	}
},
```

每个参数都具有数据类型和允许值之类的详细信息。这样，便可以验证在交互模式（例如 Azure PowerShell 或 Azure CLI）下，以及在自我发现 UI（可通过分析所需参数列表及其说明动态生成的 UI）中执行模板期间所传递的参数。

### 步骤 3-a：通过 Azure PowerShell 使用模板部署 Redis 群集

通过创建包含所有参数的运行时值的 JSON 文件，为部署准备参数文件。然后，将此文件作为单个实体传递给部署命令。如果未包含参数文件，Azure PowerShell 将使用模板中指定的任何默认值，然后提示你填写剩余的值。

下面是可在 azuredeploy-parameters.json 文件中找到的示例。请注意，你将需要为参数 `storageAccountName`、`adminUsername` 和 `adminPassword` 提供有效值，加上对其他参数的任何自定义：

```json
{
	"storageAccountName": {
		"value": "redisdeploy1"
	},
	"adminUsername": {
		"value": ""
	},
	"adminPassword": {
		"value": ""
	},
	"location": {
		"value": "West US"
	},
	"virtualNetworkName": {
		"value": "redisClustVnet"
	},
	"subnetName": {
		"value": "Subnet1"
	},
	"addressPrefix": {
		"value": "10.0.0.0/16"
	},
	"subnetPrefix": {
		"value": "10.0.0.0/24"
	},
	"nodeAddressPrefix": {
		"value": "10.0.0.1"
	},
	"redisVersion": {
		"value": "3.0.0"
	},
	"redisClusterName": {
		"value": "redis-arm-cluster"
	},
	"jumpbox": {
		"value": "Enabled"
	},
	"tshirtSize":  {
		"value": "Small"
	}
}
```

>[AZURE.NOTE]参数 `storageAccountName` 必须是不存在且符合 Windows Azure 存储空间帐户的命名要求的唯一存储空间帐户名称（仅限小写字母和数字）。此存储空间帐户将在部署过程中创建。

填写 Azure 部署名称、资源组名称、Azure 位置以及你保存 JSON 文件的文件夹。然后运行以下命令：

```powershell
$deployName="<deployment name>, such as TestDeployment"
$RGName="<resource group name>, such as TestRG"
$locName="<Azure location, such as West US>"
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$templateFile= $folderName + "\azuredeploy.json"
$templateParameterFile= $folderName + "\azuredeploy-parameters.json"

New-AzureResourceGroup –Name $RGName –Location $locName

New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile
```

>[AZURE.NOTE]`$RGName` 必须在你的订阅中唯一。

运行 **New-AzureResourceGroupDeployment** 命令时，会从参数 JSON 文件 (azuredeploy-parameters.json) 中提取参数值，然后相应地开始执行模板。在不同的环境（测试、生产等）中定义和使用多个参数文件有利于重复使用模板，并简化复杂的多环境解决方案。

部署时，请记得需要创建新的 Azure 存储空间帐户，因此，提供用作存储空间帐户参数的名称必须唯一且符合 Azure 存储空间帐户的所有要求（仅限小写字母和数字）。

在部署期间，你将看到如下内容：

	PS C:\> New-AzureResourceGroup –Name $RGName –Location $locName

	ResourceGroupName : TestRG
	Location          : westus
	ProvisioningState : Succeeded
	Tags              :
	Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

	ResourceId        : /subscriptions/1234abc1-abc1-1234-12a1-ab1ab12345ab/resourceGroups/TestRG

	PS C:\> New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile
	VERBOSE: 2:39:10 PM - Template is valid.
	VERBOSE: 2:39:14 PM - Create template deployment 'TestDeployment'.
	VERBOSE: 2:39:25 PM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is running
	VERBOSE: 2:40:04 PM - Resource Microsoft.Resources/deployments 'node-resources0' provisioning status is running
	VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is running
	VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'node-resources1' provisioning status is running
	VERBOSE: 2:40:05 PM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is succeeded
	VERBOSE: 2:42:23 PM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is succeeded
	VERBOSE: 2:47:44 PM - Resource Microsoft.Resources/deployments 'node-resources1' provisioning status is succeeded
	VERBOSE: 2:47:57 PM - Resource Microsoft.Resources/deployments 'node-resources0' provisioning status is succeeded
	VERBOSE: 2:48:01 PM - Resource Microsoft.Resources/deployments 'lastnode-resources' provisioning status is running
	VERBOSE: 2:58:24 PM - Resource Microsoft.Resources/deployments 'lastnode-resources' provisioning status is succeeded

	DeploymentName    : TestDeployment
	ResourceGroupName : TestRG
	ProvisioningState : Succeeded
	Timestamp         : 4/28/2015 7:58:23 PM
	Mode              : Incremental
	TemplateLink      :
	Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    adminUsername    String                     myadmin
                    adminPassword    SecureString
                    storageAccountName  String                     myuniqstgacct87
                    location         String                     West US
                    virtualNetworkName  String                     redisClustVnet
                    addressPrefix    String                     10.0.0.0/16
                    subnetName       String                     Subnet1
                    subnetPrefix     String                     10.0.0.0/24
                    nodeAddressPrefix  String                     10.0.0.1
                    jumpbox          String                     Enabled
                    tshirtSize       String                     Small
                    redisVersion     String                     3.0.0
                    redisClusterName  String                     redis-arm-cluster

	Outputs           :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    installCommand   String                     bash redis-cluster-install.sh -n redis-arm-cluster -v
                    3.0.0 -c 3 -m 3 -s 0 -p 10.0.0.1
                    setupCommand     String                     bash redis-cluster-install.sh -n redis-arm-cluster -v
                    3.0.0 -c 3 -m 3 -s 0 -p 10.0.0.1 -l
	...

部署期间以及部署之后，你可以检查预配期间发出的所有要求，包括发生的任何错误。

为此，请转到 [Azure 门户](https://manage.windowsazure.cn)并执行以下操作：

- 在左侧导航栏中单击“浏览”，向下滚动，然后单击“资源组”。
- 选择刚创建的资源组，以显示“资源组”边栏选项卡。
- 在“监视”部分中，选择“事件”条形图。这将显示你的部署事件。
- 单击各个事件可以进一步深化到系统代表模板执行的各项操作的详细信息中。

测试之后，如果需要删除此资源组及其所有资源（存储空间帐户、虚拟机和虚拟网络），请使用此命令：

```powershell
Remove-AzureResourceGroup –Name "<resource group name>"
```

### 步骤 3-b：通过 Azure CLI 使用模板部署 Redis 群集

若要通过 Azure CLI 部署 Redis 群集，请先通过指定名称和位置来创建资源组：

```powershell
azure group create TestRG "West US"
```

将此资源组名称、JSON 模板文件的位置，以及参数文件的位置（有关详细信息，请参阅上面的 Azure PowerShell 部分）传入以下命令：

```powershell
azure group deployment create TestRG -f .\azuredeploy.json -e .\azuredeploy-parameters.json
```

你可以使用以下命令来检查单个资源部署的状态：

```powershell
azure group deployment list TestRG
```

## Redis 群集模板结构和文件组织概览

为了创建资源管理器模板设计的功能强大且可重复使用的方法，你必须在部署 Redis 群集之类的复杂解决方案期间，考虑清楚如何安排一连串复杂但又彼此相关的任务。通过利用资源管理器模板链接功能、资源循环和通过相关扩展执行脚本，可以实现模块化方法，而所有基于模板的复杂部署基本上都能重复使用此方法。

下图描述了从 GitHub 中为此部署下载的所有文件彼此间的关系：

![redis-files](./media/virtual-machines-redis-template/redis-files.png)

本部分将指导你逐步了解 Redis 群集的 azuredeploy.json 模板的结构。

如果尚未下载模板文件的副本，请将本地文件夹指定为该文件的位置，然后创建该文件（例如，C:\\Azure\\Templates\\RedisCluster）。填写文件夹名称，并运行以下命令：

```powershell
$folderName="<folder name, such as C:\Azure\Templates\RedisCluster>"
$webclient = New-Object System.Net.WebClient
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json"
$filePath = $folderName + "\azuredeploy.json"
$webclient.DownloadFile($url,$filePath)
```

在所选的文本编辑器或工具中打开 azuredeploy.json 模板。以下信息介绍该模板文件的结构以及每个节的用途。或者，你可以在浏览器中从[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/redis-high-availability/azuredeploy.json)查看该模板的内容。

### “parameters”节

我们已提到 azuredeploy-parameters.json 的角色，执行模板期间将使用它来传递一组给定的参数值。azuredeploy.json 的“parameters”节指定用于将数据输入到此模板的参数。

下面是“T 恤大小”参数的示例：

```json
"tshirtSize": {
	"type": "string",
	"defaultValue": "Small",
	"allowedValues": [
		"Small",
		"Medium",
		"Large"
	],
	"metadata": {
		"Description": "T-shirt size of the Redis deployment"
	}
},
```

>[AZURE.NOTE]请注意，可以指定 `defaultValue` 以及 `allowedValues`。

### “variables”节

“variables”节指定可在这整个模板中使用的变量。基本上，这包含多个字段（JSON 数据类型或片段），在执行时，这些字段将设置为常量或计算值。下面是从简单到更复杂的一些示例：

```json
"vmStorageAccountContainerName": "vhd-redis",
"vmStorageAccountDomain": ".blob.core.chinacloudapi.cn",
"vnetID": "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
...
"machineSettings": {
	"adminUsername": "[parameters('adminUsername')]",
	"machineNamePrefix": "redisnode-",
	"osImageReference": {
		"publisher": "[variables('osFamilyUbuntu').imagePublisher]",
		"offer": "[variables('osFamilyUbuntu').imageOffer]",
		"sku": "[variables('osFamilyUbuntu').imageSKU]",
		"version": "latest"
	}
},
...
"vmScripts": {
	"scriptsToDownload": [
		"[concat(variables('scriptUrl'), 'redis-cluster-install.sh')]",
		"[concat(variables('scriptUrl'), 'redis-cluster-setup.sh')]",
		"[concat(variables('scriptUrl'), 'redis-sentinel-startup.sh')]"
	],
	"installCommand": "[concat('bash ', variables('installCommand'))]",
	"setupCommand": "[concat('bash ', variables('installCommand'), ' -l')]"
}
```

`vmStorageAccountContainerName` 和 `vmStorageAccountDomain` 变量属于简单的名称/值变量的示例。`vnetID` 是使用函数 `resourceId` 和 `parameters` 在运行时计算的变量的示例。`machineSettings` 通过在 `machineSettings` 变量中嵌套 JSON 对象 `osImageReference` 进一步基于这些概念生成。`vmScripts` 包含一个 JSON 数组 `scriptsToDownload`，在运行时使用 `concat` 和 `variables` 函数对其进行计算。

如果要自定义 Redis 群集部署的大小，可以更改 azuredeploy.json 模板中变量 `tshirtSizeSmall`、`tshirtSizeMedium` 和 `tshirtSizeLarge` 的属性。

```json
"tshirtSizeSmall": {
	"vmSizeMember": "Standard_A1",
	"numberOfMasters": 3,
	"numberOfSlaves": 0,
	"totalMemberCount": 3,
	"totalMemberCountExcludingLast": 2,
	"vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
"tshirtSizeMedium": {
	"vmSizeMember": "Standard_A2",
	"numberOfMasters": 3,
	"numberOfSlaves": 3,
	"totalMemberCount": 6,
	"totalMemberCountExcludingLast": 5,
	"vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
"tshirtSizeLarge": {
	"vmSizeMember": "Standard_A5",
	"numberOfMasters": 3,
	"numberOfSlaves": 6,
	"totalMemberCount": 9,
	"totalMemberCountExcludingLast": 8,
	"arbiter": "Enabled",
	"vmTemplate": "[concat(variables('templateBaseUrl'), 'node-resources.json')]"
},
```

注意：需要 `totalMemberCountExcludingLast` 和 `totalMemberCount` 属性，因为模板语言当前不具有“数学”运算功能。

有关模板语言的详细信息可在 MSDN 上的 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx)中找到。

### “resources”节

大多数操作是在“resources”节中发生的。仔细查看此节，你会立即找到两个不同的案例。第一个案例是定义为 `Microsoft.Resources/deployments` 类型的元素，它实质上调用主部署中的嵌套部署。第二个案例是 `templateLink` 属性（及相关 `contentVersion` 属性），使用它可以指定将通过传递作为输入的一组参数调用的链接模板文件。可以在此模板片段中看到这些内容：

```json
{
    "name": "shared-resources",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('sharedTemplateUrl')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "commonSettings": {
                "value": "[variables('commonSettings')]"
            },
            "storageSettings": {
                "value": "[variables('storageSettings')]"
            },
            "networkSettings": {
                "value": "[variables('networkSettings')]"
            }
        }
    }
},
```

从第一个示例我们清楚地了解此方案中的 azuredeploy.json 如何作为一种协调机制进行组织，并调用其他一些模板文件。每个文件都负责所需部署活动的一部分。

具体而言，以下链接模板将用于此部署：

- **shared-resource.json**：包含要在整个部署中共享的所有资源的定义。例如，用来存储 VM 的 OS 磁盘、虚拟网络和可用性集的存储空间帐户。
- **jumpbox-resources.json**：部署“跳转盒”VM 和所有相关资源，如网络接口、公共 IP 地址和用于使用 SSH 连接到环境的输入终结点。
- **node-resources.json**：部署所有 Redis 群集节点 VM 和连接的资源（网络适配器、专用 IP 等）。此模板还将部署 VM 扩展（适用于 Linux 的自定义脚本），并调用 bash 脚本，以在每个节点上实际安装和设置 Redis。要调用的脚本将传递到此模板 `commandToExecute` 属性的 `machineSettings` 参数中。所有 Redis 群集节点（但一个除外）都可以并行进行部署和编写脚本。一个节点需要保存直到结束，因为 Redis 群集设置只能在一个节点上运行，并且必须在所有节点都运行 Redis 服务器后完成。这就是为什么将要执行的脚本传递给此模板的原因；最后一个节点需要运行略有不同的脚本，该脚本不仅安装 Redis 服务器，而且还设置 Redis 群集。

让我们深入了解这最后一个模板 node-resources.json 的*用法*，因为从模板开发角度来看，这是最有趣的模板之一。在此要强调一个重要的概念，那就是一个模板文件如何可以部署单个资源类型的多个副本，并且对于每个实例，它都可以为所需设置指定唯一的值。此概念称为**资源循环**。

从主 azuredeploy.json 文件中调用 node-resources.json 时，它是从使用 `copy` 元素创建排序循环的资源内部调用。使用 `copy` 元素的资源将“复制”本身 `copy` 元素的 `count` 参数中指定的次数。对于你需要在不同部署资源实例之间指定唯一值的所有设置，可使用 **copyindex()** 函数获取数字值，以指示该特定资源循环创建中的当前索引。在 azuredeploy.json 的以下片段中，可以在为 Redis 群集节点创建的多个 VM 上看到此概念的运用：

```json
{
    "type": "Microsoft.Resources/deployments",
    "name": "[concat('node-resources', copyindex())]",
    "apiVersion": "2015-01-01",
    "dependsOn": [
        "[concat('Microsoft.Resources/deployments/', 'shared-resources')]"
    ],
    "copy": {
        "name": "memberNodesLoop",
        "count": "[variables('clusterSpec').totalMemberCountExcludingLast]"
    },
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('clusterSpec').vmTemplate]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "commonSettings": {
                "value": "[variables('commonSettings')]"
            },
            "storageSettings": {
                "value": "[variables('storageSettings')]"
            },
            "networkSettings": {
                "value": "[variables('networkSettings')]"
            },
            "machineSettings": {
                "value": {
                    "adminUsername": "[variables('machineSettings').adminUsername]",
                    "machineNamePrefix": "[variables('machineSettings').machineNamePrefix]",
                    "osImageReference": "[variables('machineSettings').osImageReference]",
                    "vmSize": "[variables('clusterSpec').vmSizeMember]",
                    "machineIndex": "[copyindex()]",
                    "vmScripts": "[variables('vmScripts').scriptsToDownload]",
                    "commandToExecute": "[concat(variables('vmScripts').installCommand, ' -i ', copyindex())]"
                }
            },
            "adminPassword": {
                "value": "[parameters('adminPassword')]"
            }
        }
    }
},
```

资源创建中的另一个重要概念是能够指定资源间的依赖关系和优先级，如你在 `dependsOn` JSON 数组中所见。在此特定模板中，你可以看到 Redis 群集节点依赖于先创建的共享资源。

如前所述，最后一个节点需要等到 Redis 群集中的所有其他节点均已预配为在其上运行 Redis 服务器后才进行预配。此功能在 azuredeploy.json 中完成，方法是在上面的模板片段中让名为 `lastnode-resources` 的资源依赖于名为 `memberNodesLoop` 的 `copy` 循环。完成 `memberNodesLoop` 的预配后，便可以预配 `lastnode-resources`：

```json
{
	"name": "lastnode-resources",
	"type": "Microsoft.Resources/deployments",
	"apiVersion": "2015-01-01",
	"dependsOn": [
		"memberNodesLoop"
	],
	"properties": {
		"mode": "Incremental",
		"templateLink": {
			"uri": "[variables('clusterSpec').vmTemplate]",
			"contentVersion": "1.0.0.0"
		},
		"parameters": {
			"commonSettings": {
				"value": "[variables('commonSettings')]"
			},
			"storageSettings": {
				"value": "[variables('storageSettings')]"
			},
			"networkSettings": {
				"value": "[variables('networkSettings')]"
			},
			"machineSettings": {
				"value": {
					"adminUsername": "[variables('machineSettings').adminUsername]",
					"machineNamePrefix": "[variables('machineSettings').machineNamePrefix]",
					"osImageReference": "[variables('machineSettings').osImageReference]",
					"vmSize": "[variables('clusterSpec').vmSizeMember]",
					"machineIndex": "[variables('clusterSpec').totalMemberCountExcludingLast]",
					"vmScripts": "[variables('vmScripts').scriptsToDownload]",
					"commandToExecute": "[concat(variables('vmScripts').setupCommand, ' -i ', variables('clusterSpec').totalMemberCountExcludingLast)]"
				}
			},
			"adminPassword": {
				"value": "[parameters('adminPassword')]"
			}
		}
	}
}
```

请注意，`lastnode-resources` 资源如何将略有不同的 `machineSettings.commandToExecute` 传递到链接模板。这是因为最后一个节点除了要安装 Redis 服务器外，还需要调用脚本来设置 Redis 群集（这必须在所有 Redis 服务器均已启动且正在运行后一次完成）。

另一个要探讨的有趣片段是与 `CustomScriptForLinux` VM 扩展相关的片段。这些扩展作为单独的资源类型安装，并依赖于每个群集节点。在此示例中，这用于在每个 VM 节点上安装和设置 Redis。让我们来看一下使用这些扩展的 node-resources.json 模板中的片段：

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat('vmMember', parameters('machineSettings').machineIndex, '/installscript')]",
    "apiVersion": "2015-05-01-preview",
    "location": "[parameters('commonSettings').location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', 'vmMember', parameters('machineSettings').machineIndex)]"
    ],
    "properties": {
        "publisher": "Microsoft.OSTCExtensions",
        "type": "CustomScriptForLinux",
        "typeHandlerVersion": "1.2",
        "settings": {
            "fileUris": "[parameters('machineSettings').vmScripts]",
            "commandToExecute": "[parameters('machineSettings').commandToExecute]"
        }
    }
}
```

你可以看到此资源依赖于已部署的资源 VM（`Microsoft.Compute/virtualMachines/vmMember<X>`，其中 `<X>` 是参数 `machineSettings.machineIndex`，它是已使用 **copyindex()** 函数传递给此脚本的 VM 索引）。

熟悉此部署包含的其他文件后，你将能够了解有关如何利用 Azure 资源管理器模板基于任何技术来组织和协调多节点解决方案的复杂部署策略所需的所有详细信息和最佳做法。在此建议一种构造模板文件的方法，请自行决定是否采用，如下图突出显示的部分所示：

![redis-template-structure](./media/virtual-machines-redis-template/redis-template-structure.png)

本质上，这种方法会建议：

- 将你的核心模板文件定义成所有特定部署活动的中心协调点，并利用模板链接来调用子模板执行。
- 创建特定的模板文件，用于部署所有其他特定部署任务会共享的所有资源（存储空间帐户、虚拟网络配置等）。如果不同的部署在公用基础结构方面具有类似的要求，你就可以对其频繁重复使用此方法。
- 针对特定于给定资源的场地要求提供可选资源模板。
- 针对资源组的相同成员（群集中的节点，等等），创建利用资源循环的特定模板，以便部署多个具有特有属性的实例。
- 对于所有部署后任务（产品安装、配置，等等），利用脚本部署扩展并创建特定于每种技术的脚本。

有关详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx)。

<!---HONumber=67-->