<properties
	pageTitle="在 Ubuntu 上使用资源管理器模板部署 Spark"
	description="了解如何使用 Azure PowerShell 或 Azure CLI 以及资源管理器模板在 Ubuntu VM 上轻松部署新的 Spark 群集。"
	services="virtual-machines"
	documentationCenter=""
	authors="paolosalvatori"
	manager="timlt"
	editor="tysonn"/>

<tags 
	ms.service="virtual-machines"
	ms.date="05/16/2015"
	wacn.date="08/29/2015"/>

# 在 Ubuntu 上使用资源管理器模板部署 Spark

Apache Spark 是一种适用于大规模数据处理的快速引擎。Spark 具有高级的 DAG 执行引擎，它支持循环的数据流和内存中计算，并且可以访问各种数据源，包括 HDFS、Spark、HBase 和 S3。

除了在 Mesos 或 YARN 群集管理器上运行，Spark 提供简单的独立部署模式。本教程将指导你如何使用示例 Azure 资源管理器模板通过 [Azure PowerShell](/documentation/articles/powershell-install-configure) 或 [Azure CLI](/documentation/articles/xplat-cli) 在 Ubuntu VM 上部署 Spark 群集。

此模板可在 Ubuntu 虚拟机上部署 Spark 群集。此模板还设置存储帐户、虚拟网络、可用性集、公共 IP 地址和安装所需的网络接口。Spark 群集在子网后面创建，因此没有公共 IP 可访问此群集。在部署过程中，可部署可选的“跳转盒”。此“跳转盒”也是子网中部署的 Ubuntu VM，但它*确实*公开了你可以连接到的开放 SSH 端口的公共 IP 地址。然后通过“跳转盒”，就可以使用 SSH 连接到子网中的所有 Spark VM。

此模板利用“T恤大小”概念来指定“小型”、“中等”或“大型”Spark 群集安装。当模板语言支持更多动态模板大小时，可以更改此项以指定 Spark 群集主节点数、从属节点数、VM 大小等。现在，你可以在文件 azuredeploy.json 中的变量 **tshirtSizeS**、**tshirtSizeM** 和 **tshirtSizeL** 中看到 VM 大小、主节点数和从属节点数。

- S：1 个主节点，1 个从属节点
- M：1 个主节点，4 个从属节点
- L：1 个主节点，6 个从属节点

根据此模板新部署的群集采用下图中所述的拓扑，不过，你可以通过自定义本文中所述的模板，轻松实现其他拓扑：

![cluster-architecture](./media/virtual-machines-spark-template/cluster-architecture.png)

如上面的图像所示，部署拓扑由以下元素组成：

-	托管新创建的虚拟机的 OS 磁盘的新存储帐户。
-	包含子网的虚拟网络。由模板创建的所有虚拟机都在此虚拟网络中进行设置。
-	主节点和从属节点的可用性集。
-	新创建的可用性集中的主节点。
-	在相同虚拟子网中运行的四个从属节点和可作为主节点的可用性集。
-	位于可用于访问群集的相同虚拟网络和子网中的“跳转盒”VM。

Spark 3.0.0 版是默认版本，并可以更改为适用于 Spark 存储库中的任何预生成的二进制文件。还提供脚本中取消从源生成注释的设置。静态 IP 地址将分配到每个 Spark 主节点：10.0.0.10。静态 IP 地址将分配到每个 Spark 从属节点，以便解决当前不能在模板内动态编写 IP 地址列表的限制。（默认情况下，第一个节点将分配给专用 IP 地址 10.0.0.30，第二个节点将分配给 10.0.0.31，依次类推。） 若要检查部署错误，请转到新的 Azure 门户并通过“资源组”>“上次部署”>“检查操作详细信息”查看。

在深入了解与 Azure 资源管理器和将用于此部署的模板相关的详细信息之前，请确保你已正确配置 Azure PowerShell 或 Azure CLI。

[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell.md)]

[AZURE.INCLUDE [xplat-getting-set-up-arm](../includes/xplat-getting-set-up-arm.md)]

## 使用资源管理器模板创建 Spark 群集

请按照下列步骤，通过 Azure PowerShell 或 Azure CLI，使用 GitHub 模板存储库中的资源管理器模板创建 Spark 群集。

### 步骤 1-a：使用 Azure PowerShell 下载模板文件

为 JSON 模板和其他关联的文件创建本地文件夹（例如，C:\\Azure\\Templates\\Spark）。

替换为你的本地文件夹的文件夹名称，并运行以下命令：

```powershell
# Define variables
$folderName="C:\Azure\Templates\Spark"
$baseAddress = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/spark-on-ubuntu/"
$webclient = New-Object System.Net.WebClient
$files = $("azuredeploy.json", `
         "azuredeploy-parameters.json", `
         "jumpbox-resources-disabled.json", `
         "jumpbox-resources-enabled.json",
         "metadata.json", `
         "shared-resources.json", `
         "spark-cluster-install.sh", `
         "jumpbox-resources-enabled.json")

# Download files
foreach ($file in $files)
{
    $url = $baseAddress + $file
    $filePath = $folderName + $file
    $webclient.DownloadFile($baseAddress + $file, $folderName + $file)
    Write-Host $url "downloaded to" $filePath
}
```

### 步骤 1-b：使用 Azure CLI 下载模板文件

使用你选择的 Git 客户端克隆整个模板存储库，例如：

	git clone https://github.com/Azure/azure-quickstart-templates C:\Azure\Templates

完成克隆后，在 C:\\Azure\\Templates 目录中查找 **spark-on-ubuntu** 文件夹。

### 步骤 2：（可选）了解模板参数

当通过使用模板创建 Spark 群集时，必须指定一组配置参数来处理大量所需设置。通过声明模板定义中的这些参数，则可能通过外部文件或命令行在部署执行过程中指定值。

在 azuredeploy.json 文件顶部的“parameters”节中，你会发现模板需要用来配置 Spark 群集的一组参数。下面的示例来自此模板的 azuredeploy.json 文件的“parameters”节：

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
	"imagePublisher": {
		"type": "string",
		"defaultValue": "Canonical",
		"metadata": {
			"Description": "Image publisher"
		}
	},
	"imageOffer": {
		"type": "string",
		"defaultValue": "UbuntuServer",
		"metadata": {
			"Description": "Image offer"
		}
	},
	"imageSKU": {
		"type": "string",
		"defaultValue": "14.04.2-LTS",
		"metadata": {
			"Description": "Image SKU"
		}
	},
	"storageAccountName": {
		"type": "string",
		"defaultValue": "uniquesparkstoragev12",
		"metadata": {
			"Description": "Unique namespace for the Storage account where the virtual machine's disks will be placed"
		}
	},
	"region": {
		"type": "string",
		"defaultValue": "West US",
		"metadata": {
			"Description": "Location where resources will be provisioned"
		}
	},
	"virtualNetworkName": {
		"type": "string",
		"defaultValue": "myVNETspark",
		"metadata": {
			"Description": "The arbitrary name of the virtual network provisioned for the cluster"
		}
	},
	"dataDiskSize": {
		"type": "int",
		"defaultValue": 100,
		"metadata": {
			"Description": "Size of the data disk attached to Spark nodes (in GB)"
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
		"defaultValue": "Subnet-1",
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
	"sparkVersion": {
		"type": "string",
		"defaultValue": "stable",
		"metadata": {
			"Description": "The version of the Spark package to be deployed on the cluster (or use 'stable' to pull in the latest and greatest)"
		}
	},
	"sparkClusterName": {
		"type": "string",
		"metadata": {
			"Description": "The arbitrary name of the Spark cluster (maps to cluster's configuration file name)"
		}
	},
	"sparkNodeIPAddressPrefix": {
		"type": "string",
		"defaultValue": "10.0.0.1",
		"metadata": {
			"Description": "The IP address prefix that will be used for constructing a static private IP address for each node in the cluster"
		}
	},
	"sparkSlaveNodeIPAddressPrefix": {
		"type": "string",
		"defaultValue": "10.0.0.3",
		"metadata": {
			"Description": "The IP address prefix that will be used for constructing a static private IP address for each node in the cluster"
		}
	},
	"jumpbox": {
		"type": "string",
		"defaultValue": "enabled",
		"allowedValues": [
		"enabled",
		"disabled"
		],
		"metadata": {
			"Description": "The flag allowing to enable or disable provisioning of the jump-box VM that can be used to access the Spark nodes"
		}
	},
	"tshirtSize": {
		"type": "string",
		"defaultValue": "S",
		"allowedValues": [
		"S",
		"M",
		"L"
		],
		"metadata": {
			"Description": "T-shirt size of the Spark cluster"
		}
	}
}
```

每个参数都具有数据类型和允许值之类的详细信息。这样，便可以在交互模式（例如 Azure PowerShell 或 Azure CLI）下，以及在自我发现 UI（可通过分析所需参数列表及其说明动态生成的 UI）中，在执行模板期间验证所传递的参数。

### 步骤 3-a：通过 Azure PowerShell 使用模板部署 Spark 群集

通过创建包含所有参数的运行时值的 JSON 文件，为部署准备参数文件。然后，将此文件作为单个实体传递给部署命令。如果不包括参数文件，Azure PowerShell 将使用模板中指定的任何默认值，然后提示你填写其余的值。

下面是 azuredeploy-parameters.json 文件中的一组示例参数。请注意，你将需要为参数 **storageAccountName**、**adminUsername** 和 **adminPassword** 提供有效值，并自定义其他参数：

```json
{
  "storageAccountName": {
    "value": "paolosspark"
  },
  "adminUsername": {
    "value": "paolos"
  },
  "adminPassword": {
    "value": "Passw0rd!"
  },
  "region": {
    "value": "West US"
  },
  "virtualNetworkName": {
    "value": "sparkClustVnet"
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
  "sparkVersion": {
    "value": "3.0.0"
  },
  "sparkClusterName": {
    "value": "spark-arm-cluster"
  },
  "sparkNodeIPAddressPrefix": {
		"value": "10.0.0.1"
  },
  "sparkSlaveNodeIPAddressPrefix": {
    "value": "10.0.0.3"
  },
  "jumpbox": {
    "value": "enabled"
  },
  "tshirtSize": {
    "value": "M"
  }
}
```
填写 Azure 部署名称、资源组名称、Azure 位置，以及存储 JSON 部署文件的文件夹。然后运行以下命令：

```powershell
$deployName="<deployment name>"
$RGName="<resource group name>"
$locName="<Azure location, such as West US>"
$folderName="<folder name, such as C:\Azure\Templates\Spark>"
$templateFile= $folderName + "\azuredeploy.json"
$templateParameterFile= $folderName + "\azuredeploy-parameters.json"
New-AzureResourceGroup -Name $RGName -Location $locName
New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParamterFile $templateParameterFile -TemplateFile $templateFile
```
> [AZURE.NOTE]**$RGName** 必须在你的订阅中具有唯一性。

运行 **New-AzureResourceGroupDeployment** 命令时，会从 JSON 参数文件中提取参数值，然后相应地开始执行模板。在不同的环境（测试、生产等）中定义和使用多个参数文件有利于重复使用模板，并简化复杂的多环境解决方案。

部署时，请记得需要创建新的 Azure 存储帐户，因此，提供用作存储帐户参数的名称必须唯一且符合 Azure 存储帐户的所有要求（仅限小写字母和数字）。

在部署期间，你将看到如下内容：

```powershell
PS C:\> New-AzureResourceGroup -Name $RGName -Location $locName

ResourceGroupName : SparkResourceGroup
Location          : westus
ProvisioningState : Succeeded
Tags              :
Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

ResourceId        : /subscriptions/2018abc3-dbd9-4437-81a8-bb3cf56138ed/resourceGroups/sparkresourcegroup

PS C:\> New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateParameterFile $templateParameterFile -TemplateFile $templateFile
VERBOSE: 10:08:28 AM - Template is valid.
VERBOSE: 10:08:28 AM - Create template deployment 'SparkTestDeployment'.
VERBOSE: 10:08:37 AM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is running
VERBOSE: 10:09:11 AM - Resource Microsoft.Resources/deployments 'shared-resources' provisioning status is succeeded
VERBOSE: 10:09:13 AM - Resource Microsoft.Network/networkInterfaces 'nicsl0' provisioning status is succeeded
VERBOSE: 10:09:16 AM - Resource Microsoft.Network/networkInterfaces 'nic0' provisioning status is succeeded
VERBOSE: 10:09:16 AM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is running
VERBOSE: 10:09:24 AM - Resource Microsoft.Compute/virtualMachines 'mastervm0' provisioning status is running
VERBOSE: 10:11:04 AM - Resource Microsoft.Compute/virtualMachines/extensions 'mastervm0/installsparkmaster'
provisioning status is running
VERBOSE: 10:11:04 AM - Resource Microsoft.Compute/virtualMachines 'mastervm0' provisioning status is succeeded
VERBOSE: 10:11:11 AM - Resource Microsoft.Compute/virtualMachines 'slavevm0' provisioning status is running
VERBOSE: 10:11:42 AM - Resource Microsoft.Resources/deployments 'jumpbox-resources' provisioning status is succeeded
VERBOSE: 10:13:10 AM - Resource Microsoft.Compute/virtualMachines 'slavevm0' provisioning status is succeeded
VERBOSE: 10:13:15 AM - Resource Microsoft.Compute/virtualMachines/extensions 'slavevm0/installsparkslave' provisioning
status is running
VERBOSE: 10:16:58 AM - Resource Microsoft.Compute/virtualMachines/extensions 'mastervm0/installsparkmaster'
provisioning status is succeeded
VERBOSE: 10:19:05 AM - Resource Microsoft.Compute/virtualMachines/extensions 'slavevm0/installsparkslave' provisioning
status is succeeded


DeploymentName    : SparkTestDeployment
ResourceGroupName : SparkResourceGroup
ProvisioningState : Succeeded
Timestamp         : 5/5/2015 5:19:05 PM
Mode              : Incremental
TemplateLink      :
Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    adminUsername    String                     Paolos
                    adminPassword    SecureString
                    imagePublisher   String                     Canonical
                    imageOffer       String                     UbuntuServer
                    imageSKU         String                     14.04.2-LTS
                    storageAccountName  String                  paolosspark
                    region           String                     West US
                    virtualNetworkName  String                  sparkClustVnet
                    addressPrefix    String                     10.0.0.0/16
										subnetName       String                     Subnet1
                    subnetPrefix     String                     10.0.0.0/24
                    sparkVersion     String                     3.0.0
                    sparkClusterName  String                    spark-arm-cluster
                    sparkNodeIPAddressPrefix  String            10.0.0.1
                    sparkSlaveNodeIPAddressPrefix  String       10.0.0.3
                    jumpbox          String                     enabled
                    tshirtSize       String                     M
```

部署期间以及部署之后，你可以检查设置期间发出的所有要求，包括发生的任何错误。

为此，请转到 [Azure 门户](https://manage.windowsazure.cn)并执行以下操作：

- 在左侧导航栏中单击“浏览”，向下滚动，然后单击“资源组”。
- 选择刚创建的资源组，以显示“资源组”边栏选项卡。
- 在“资源组”边栏选项卡的“监视”部分中单击“事件”条形图，可以查看部署的事件。
- 单击各个事件，可以进一步深化到系统代表模板执行的各项操作的详细信息中。

![portal-events](./media/virtual-machines-spark-template/portal-events.png)

测试之后，如果需要删除此资源组及其所有资源（存储帐户、虚拟机和虚拟网络），请使用这一命令：

```powershell
Remove-AzureResourceGroup -Name "<resource group name>" -Force
```

### 步骤 3-b：通过 Azure CLI 使用模板部署 Spark 群集

若要通过 Azure CLI 部署 Spark 群集，请先通过指定名称和位置来创建资源组：

	azure group create SparkResourceGroup "West US"

将此资源组名称、JSON 模板文件的位置，以及参数文件的位置（有关详细信息，请参阅上面的 Azure PowerShell 部分）传入以下命令：

	azure group deployment create SparkResourceGroup -f .\azuredeploy.json -e .\azuredeploy-parameters.json

你可以使用以下命令来检查单个资源部署的状态：

	azure group deployment list SparkResourceGroup

## Spark 模板结构和文件组织概览

为了让资源管理器模板的设计更加完善且可重复使用，你必须在部署 Spark 之类的复杂解决方案期间，考虑清楚如何安排一连串复杂但又彼此相关的任务。除了通过相关扩展执行脚本之外，还可以利用资源管理器模板链接和资源循环，这样就能实现模块化方法，而所有基于模板的复杂部署基本上都能重复使用此方法。

下图描述了从 GitHub 中为此部署下载的所有文件彼此间的关系：

![spark-files](./media/virtual-machines-spark-template/spark-files.png)

本部分将指导你逐步了解 Spark 群集的 azuredeploy.json 文件结构。

如果尚未下载模板文件的副本，请将本地文件夹指定为该文件的位置，然后创建该文件（例如，C:\\Azure\\Templates\\Spark）。填写文件夹名称，并运行这些命令：

```powershell
$folderName="<folder name, such as C:\Azure\Templates\Spark>"
$webclient = New-Object System.Net.WebClient
$url = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/spark-on-ubuntu/azuredeploy.json"
$filePath = $folderName + "\azuredeploy.json"
$webclient.DownloadFile($url,$filePath)
```

在所选的文本编辑器或工具中打开 azuredeploy.json 模板。以下信息介绍该模板文件的结构以及每个节的用途。或者，你可以在浏览器中从[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/spark-on-ubuntu/azuredeploy.json)查看该模板的内容 。

### “parameters”节

azuredeploy.json 的“parameters”节指定此模板中使用的可修改参数。前面提到的 azuredeploy-parameters.json 文件用于在模板执行期间将值传入 azuredeploy.json 的“parameters”节。

下面是“T 恤大小”参数的示例：

```json
"tshirtSize": {
    "type": "string",
    "defaultValue": "S",
    "allowedValues": [
        "S",
        "M",
        "L"
    ],
    "metadata": {
        "Description": "T-shirt size of the Spark deployment"
    }
},
```

> [AZURE.NOTE]请注意，**defaultValue** 以及 **allowedValues** 可以指定。

### “variables”节

“variables”节指定可在这整个模板中使用的变量。这包含多个字段（JSON 数据类型或片段），在执行时，这些字段将设置为常量或计算值。下面是从简单到更复杂的一些示例：

```json
"variables": {
	"vmStorageAccountContainerName": "vhd",
	"vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
	"subnetRef": "[concat(variables('vnetID'),'/subnets/',parameters('subnetName'))]",
	"computerNamePrefix": "sparknode",
	"scriptUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/spark-on-ubuntu/",
	"templateUrl": "[variables('scriptUrl')]",
	"sharedTemplateName": "shared-resources",
	"jumpboxTemplateName": "jumpbox-resources-",
	"tshirtSizeS": {
		"numberOfMasterInstances": 1,
		"numberOfSlavesInstances" : 1,
		"vmSize": "Standard_A2"
	},
	"tshirtSizeM": {
		"numberOfMasterInstances": 1,
		"numberOfSlavesInstances" : 4,
		"vmSize": "Standard_A4"
	},
	"tshirtSizeL": {
		"numberOfMasterInstances": 1,
		"numberOfSlavesInstances" : 6,
		"vmSize": "Standard_A7"
	},
	"numberOfMasterInstances": "[variables(concat('tshirtSize', parameters('tshirtSize'))).numberOfMasterInstances]",
	"numberOfSlavesInstances": "[variables(concat('tshirtSize', parameters('tshirtSize'))).numberOfSlavesInstances]",
	"vmSize": "[variables(concat('tshirtSize', parameters('tshirtSize'))).vmSize]"
},
```

**vmStorageAccountContainerName** 变量是简单的名称/值变量的示例。**vnetID** 是在运行时使用函数 **resourceId** 和 **parameters** 计算的变量示例。值 **numberOfMasterInstances** 和 **vmSize** 变量在运行时使用 **concat**、**variables** 和 **parameters** 函数进行计算。

如果要自定义 Spark 群集部署的大小，则可以更改 azuredeploy.json 模板中变量 **tshirtSizeS**、**tshirtSizeM** 和 **tshirtSizeL** 的属性。

有关模板语言的详细信息可在 MSDN 上的 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx)中找到。


### “resources”节

大多数操作是在“resources”节中发生的。仔细查看此节，你会立即找到两个不同的案例。第一个案例是定义为 `Microsoft.Resources/deployments` 类型的元素，它实质上调用主部署中的嵌套部署。第二个案例是 **templateLink** 属性（及相关 **contentVersion** 属性），使用它可以指定将通过传递作为输入的一组参数调用的链接模板文件。可以在此模板片段中看到这些内容：

```json
"resources": [
{
	"name": "shared-resources",
	"type": "Microsoft.Resources/deployments",
	"apiVersion": "2015-01-01",
	"properties": {
		"mode": "Incremental",
		"templateLink": {
			"uri": "[concat(variables('templateUrl'), variables('sharedTemplateName'), '.json')]",
			"contentVersion": "1.0.0.0"
		},
		"parameters": {
			"region": {
				"value": "[parameters('region')]"
			},
			"storageAccountName": {
				"value": "[parameters('storageAccountName')]"
			},
			"virtualNetworkName": {
				"value": "[parameters('virtualNetworkName')]"
			},
			"addressPrefix": {
				"value": "[parameters('addressPrefix')]"
			},
			"subnetName": {
				"value": "[parameters('subnetName')]"
			},
			"subnetPrefix": {
				"value": "[parameters('subnetPrefix')]"
			}
		}
	}
},
```

在第一个示例中，我们很清楚地知道此方案中的 azuredeploy.json 用作一种协调机制，负责调用其他一些模板文件。每个文件都负责所需部署活动的一部分。

具体而言，以下链接模板将用于此部署：

-	**shared-resource.json**：包含要在整个部署中共享的所有资源的定义。例如，用来存储 VM 的操作系统磁盘和虚拟网络的存储帐户。
-	**jumpbox-resources-enabled.json**：部署“跳转盒”VM 和所有相关资源，例如网络接口、公共 IP 地址和用于使用 SSH 连接到环境的输入终结点。

在调用这两个模板之后，azuredeploy.json 设置所有 Spark 群集节点 VM 和连接资源（网络适配器、专用 IP 等）。此模板还将部署 VM 扩展（适用于 Linux 的自定义脚本），并调用 bash 脚本 (spark-cluster-install.sh)，以在每个节点上实际安装和设置 Spark。

让我们深入了解这最后一个模板 azuredeploy.json 的*用法*，因为从模板开发角度来看，这是最有趣的模板之一。在此要强调一个重要的概念，那就是一个模板文件如何可以部署单个资源类型的多个副本，并且对于每个实例，它都可以为所需设置指定唯一的值。此概念称为**资源循环**。

使用 **copy** 元素的资源将“复制”本身 **copy** 元素的 **count** 参数中指定的次数。对于你需要在不同部署资源实例之间指定唯一值的所有设置，可使用 **copyindex()** 函数获取数字值，以指示该特定资源循环创建中的当前索引。在 azuredeploy.json 的以下片段中，可以在为 Spark 群集创建的多个网络适配器、VM 和 VM 扩展上看到此概念的运用：

```json
{
	"apiVersion": "2015-05-01-preview",
	"type": "Microsoft.Network/networkInterfaces",
	"name": "[concat('nic', copyindex())]",
	"location": "[parameters('region')]",
	"copy": {
		"name": "nicLoop",
		"count": "[variables('numberOfMasterInstances')]"
	},
	"dependsOn": [
	"[concat('Microsoft.Resources/deployments/', 'shared-resources')]"
	],
	"properties": {
		"ipConfigurations": [
		{
			"name": "ipconfig1",
			"properties": {
				"privateIPAllocationMethod": "Static",
				"privateIPAddress": "[concat(parameters('sparkNodeIPAddressPrefix'), copyindex())]",
				"subnet": {
					"id": "[variables('subnetRef')]"
				}
			}
		}
		]
	}
},
{
	"apiVersion": "2015-05-01-preview",
	"type": "Microsoft.Network/networkInterfaces",
	"name": "[concat('nicsl', copyindex())]",
	"location": "[parameters('region')]",
	"copy": {
		"name": "nicslLoop",
		"count": "[variables('numberOfSlavesInstances')]"
	},
	"dependsOn": [
	"[concat('Microsoft.Resources/deployments/', 'shared-resources')]"
	],
	"properties": {
		"ipConfigurations": [
		{
			"name": "ipconfig1",
			"properties": {
				"privateIPAllocationMethod": "Static",
				"privateIPAddress": "[concat(parameters('sparkSlaveNodeIPAddressPrefix'), copyindex())]",
				"subnet": {
					"id": "[variables('subnetRef')]"
				}
			}
		}
		]
	}
},
{
	"apiVersion": "2015-05-01-preview",
	"type": "Microsoft.Compute/virtualMachines",
	"name": "[concat('mastervm', copyindex())]",
	"location": "[parameters('region')]",
	"copy": {
		"name": "virtualMachineLoopMaster",
		"count": "[variables('numberOfMasterInstances')]"
	},
	"dependsOn": [
	"[concat('Microsoft.Resources/deployments/', 'shared-resources')]",
	"[concat('Microsoft.Network/networkInterfaces/', 'nic', copyindex())]"
	],
	"properties": {
		"availabilitySet": {
			"id": "[resourceId('Microsoft.Compute/availabilitySets', 'sparkCluserAS')]"
		},
		"hardwareProfile": {
			"vmSize": "[variables('vmSize')]"
		},
		"osProfile": {
			"computername": "[concat(variables('computerNamePrefix'), copyIndex())]",
			"adminUsername": "[parameters('adminUsername')]",
			"adminPassword": "[parameters('adminPassword')]",
			"linuxConfiguration": {
				"disablePasswordAuthentication": "false"
			}
		},
		"storageProfile": {
			"imageReference": {
				"publisher": "[parameters('imagePublisher')]",
				"offer": "[parameters('imageOffer')]",
				"sku" : "[parameters('imageSKU')]",
				"version":"latest"
			},
			"osDisk" : {
				"name": "osdisk",
				"vhd": {
					"uri": "[concat('https://',parameters('storageAccountName'),'.blob.core.chinacloudapi.cn/',variables('vmStorageAccountContainerName'),'/','osdisk', copyindex() ,'.vhd')]"
					},
					"caching": "ReadWrite",
					"createOption": "FromImage"
				}
			},
			"networkProfile": {
				"networkInterfaces": [
				{
					"id": "[resourceId('Microsoft.Network/networkInterfaces',concat('nic', copyindex()))]"
				}
				]
			}
		}
	},
	{
		"apiVersion": "2015-05-01-preview",
		"type": "Microsoft.Compute/virtualMachines",
		"name": "[concat('slavevm', copyindex())]",
		"location": "[parameters('region')]",
		"copy": {
			"name": "virtualMachineLoop",
			"count": "[variables('numberOfSlavesInstances')]"
		},
		"dependsOn": [
		"[concat('Microsoft.Resources/deployments/', 'shared-resources')]",
		"[concat('Microsoft.Network/networkInterfaces/', 'nicsl', copyindex())]",
		"virtualMachineLoopMaster"
		],
		"properties": {
			"availabilitySet": {
				"id": "[resourceId('Microsoft.Compute/availabilitySets', 'sparkCluserAS')]"
			},
			"hardwareProfile": {
				"vmSize": "[variables('vmSize')]"
			},
			"osProfile": {
				"computername": "[concat(variables('computerNamePrefix'),'sl', copyIndex())]",
				"adminUsername": "[parameters('adminUsername')]",
				"adminPassword": "[parameters('adminPassword')]",
				"linuxConfiguration": {
					"disablePasswordAuthentication": "false"
				}
			},
			"storageProfile": {
				"imageReference": {
					"publisher": "[parameters('imagePublisher')]",
					"offer": "[parameters('imageOffer')]",
					"sku" : "[parameters('imageSKU')]",
					"version":"latest"
				},
				"osDisk" : {
					"name": "osdisksl",
					"vhd": {
						"uri": "[concat('https://',parameters('storageAccountName'),'.blob.core.chinacloudapi.cn/',variables('vmStorageAccountContainerName'),'/','osdisksl', copyindex() ,'.vhd')]"
					},
					"caching": "ReadWrite",
					"createOption": "FromImage"
				}
			},
			"networkProfile": {
				"networkInterfaces": [
				{
					"id": "[resourceId('Microsoft.Network/networkInterfaces',concat('nicsl', copyindex()))]"
				}
				]
			}
		}
	},
	{
		"type": "Microsoft.Compute/virtualMachines/extensions",
		"name": "[concat('mastervm', copyindex(), '/installsparkmaster')]",
		"apiVersion": "2015-05-01-preview",
		"location": "[parameters('region')]",
		"copy": {
			"name": "virtualMachineExtensionsLoopMaster",
			"count": "[variables('numberOfMasterInstances')]"
		},
		"dependsOn": [
		"[concat('Microsoft.Compute/virtualMachines/', 'mastervm', copyindex())]",
		"[concat('Microsoft.Network/networkInterfaces/', 'nic', copyindex())]"
		],
		"properties": {
			"publisher": "Microsoft.OSTCExtensions",
			"type": "CustomScriptForLinux",
			"typeHandlerVersion": "1.2",
			"settings": {
				"fileUris": [
				"[concat(variables('scriptUrl'), 'spark-cluster-install.sh')]"
				],
				"commandToExecute": "[concat('bash spark-cluster-install.sh -k ',parameters('sparkVersion'),' -d ', reference('nic0').ipConfigurations[0].properties.privateIPAddress,' -s ',variables('numberOfSlavesInstances'),' -m ', ' 1 ')]"
			}
		}
	},
	{
		"type": "Microsoft.Compute/virtualMachines/extensions",
		"name": "[concat('slavevm', copyindex(), '/installsparkslave')]",
		"apiVersion": "2015-05-01-preview",
		"location": "[parameters('region')]",
		"copy": {
			"name": "virtualMachineExtensionsLoopSlave",
			"count": "[variables('numberOfSlavesInstances')]"
		},
		"dependsOn": [
		"[concat('Microsoft.Compute/virtualMachines/', 'slavevm', copyindex())]",
		"[concat('Microsoft.Network/networkInterfaces/', 'nicsl', copyindex())]"
		],
		"properties": {
			"publisher": "Microsoft.OSTCExtensions",
			"type": "CustomScriptForLinux",
			"typeHandlerVersion": "1.2",
			"settings": {
				"fileUris": [
				"[concat(variables('scriptUrl'), 'spark-cluster-install.sh')]"
				],
				"commandToExecute": "[concat('bash spark-cluster-install.sh -k ',parameters('sparkVersion'),' -m ', ' 0 ')]"
			}
		}
	}
```

创建资源的另一个重要概念是能够指定资源间的依赖性和优先级，如同你在 **dependsOn** JSON 数组中所见。在此特定模板中，你可以看到 Spark 群集节点依赖于先创建的共享资源和 **networkInterfaces** 资源。

另一个要探讨的有趣片段是与 **CustomScriptForLinux** VM 扩展相关的片段。这些扩展以独立资源类型进行安装，并依赖于每个群集节点。在此示例中，这用于在每个 VM 节点上安装和设置 Spark。让我们来看一下使用这些扩展的 azuredeploy.json 模板中的代码段：

```json
{
	"type": "Microsoft.Compute/virtualMachines/extensions",
	"name": "[concat('mastervm', copyindex(), '/installsparkmaster')]",
	"apiVersion": "2015-05-01-preview",
	"location": "[parameters('region')]",
	"copy": {
		"name": "virtualMachineExtensionsLoopMaster",
		"count": "[variables('numberOfMasterInstances')]"
	},
	"dependsOn": [
		"[concat('Microsoft.Compute/virtualMachines/', 'mastervm', copyindex())]",
		"[concat('Microsoft.Network/networkInterfaces/', 'nic', copyindex())]"
	],
	"properties": {
		"publisher": "Microsoft.OSTCExtensions",
		"type": "CustomScriptForLinux",
		"typeHandlerVersion": "1.2",
		"settings": {
			"fileUris": [
				"[concat(variables('scriptUrl'), 'spark-cluster-install.sh')]"
			],
			"commandToExecute": "[concat('bash spark-cluster-install.sh -k ',parameters('sparkVersion'),' -d ', reference('nic0').ipConfigurations[0].properties.privateIPAddress,' -s ',variables('numberOfSlavesInstances'),' -m ', ' 1 ')]"
		}
	}
},
{
	"type": "Microsoft.Compute/virtualMachines/extensions",
	"name": "[concat('slavevm', copyindex(), '/installsparkslave')]",
	"apiVersion": "2015-05-01-preview",
	"location": "[parameters('region')]",
	"copy": {
		"name": "virtualMachineExtensionsLoopSlave",
		"count": "[variables('numberOfSlavesInstances')]"
	},
	"dependsOn": [
		"[concat('Microsoft.Compute/virtualMachines/', 'slavevm', copyindex())]",
		"[concat('Microsoft.Network/networkInterfaces/', 'nicsl', copyindex())]"
	],
	"properties": {
		"publisher": "Microsoft.OSTCExtensions",
		"type": "CustomScriptForLinux",
		"typeHandlerVersion": "1.2",
		"settings": {
			"fileUris": [
				"[concat(variables('scriptUrl'), 'spark-cluster-install.sh')]"
			],
			"commandToExecute": "[concat('bash spark-cluster-install.sh -k ',parameters('sparkVersion'),' -m ', ' 0 ')]"
		}
	}
}
```

请注意，主节点和从属节点的扩展执行 **commandToExecute** 属性中定义的不同命令，作为设置过程的一部分。

如果你看一下最新虚拟机扩展的 JSON 代码段，就可以看到此资源依赖于虚拟机资源及其网络接口。这表明设置和运行此 VM 扩展之前需要已部署这两个资源。此外请注意，使用 **copyindex()** 函数对每个从属虚拟机重复此步骤。

熟悉此部署包含的其他文件后，你将能够通过利用 Azure 资源管理器模板，基于任何技术了解组织和协调多节点解决方案的复杂部署策略所需的所有详细信息和最佳做法。在此建议一种构造模板文件的方法，请自行决定是否采用，如下图突出显示的部分所示：

![spark-template-structure](./media/virtual-machines-spark-template/spark-template-structure.png)

本质上，这种方法会建议：

-	将你的核心模板文件定义成所有特定部署活动的中心协调点，并利用模板链接调用子模板执行
-	创建特定的模板文件，用于部署所有其他特定部署任务会共享的所有资源（存储帐户、虚拟网络配置等）。如果不同的部署在公用基础结构方面具有类似的要求，你就可以对其频繁重复使用此方法。
-	针对特定于给定资源的场地要求提供可选资源模板。
-	针对资源组的相同成员（群集中的节点等），创建利用资源循环的特定模板，以便部署多个具有唯一属性的实例。
-	针对所有部署后任务（产品安装、配置等），利用脚本部署扩展并创建特定于每种技术的脚本。

有关详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx)。

## 后续步骤

了解关于<!--[-->部署模板<!--](/documentation/articles/resource-group-template-deploy)-->的更多详细信息。

了解关于<!--[-->应用程序框架<!--](/documentation/articles/virtual-machines-app-frameworks)-->的更多信息。

<!--[-->对模板部署进行故障排除<!--](/documentation/articles/resource-group-deploy-debug)-->。
<!---HONumber=67-->