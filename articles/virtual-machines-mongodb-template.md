<properties pageTitle="在 Ubuntu 上使用资源管理器模板创建 MongoDB 群集" description="通过 PowerShell 或 Azure CLI 使用资源管理器模板在 Ubuntu 上创建 MongoDB 群集" services="virtual-machines" documentationCenter="" authors="karthmut" manager="timlt" editor="tysonn"/>

<tags ms.service="virtual-machines" ms.date="04/29/2015" wacn.date="08/29/2015"/>

# 在 Ubuntu 上使用资源管理器模板创建 MongoDB 群集

MongoDB 是一种可提供高性能、高可用性和自动缩放的开源文档数据库。MongoDB 可独立安装或利用内置的复制功能在群集内安装。在某些情况下，你可使用复制来提高读取容量。客户端能够将读取和写入操作发送到不同的服务器。你还可以在不同数据中心中维护副本，以增加分布式应用程序的的数据位置和可用性。使用 MongoDB，复制还提供了冗余，并提高数据可用性。通过将数据的多个副本保存在不同数据库服务器上，复制还可保护数据库免遭丢失一个服务器的危害。复制还可以让你从硬件故障和服务中断中恢复。有了附加数据副本，你可以将其中一个副本专用于灾难恢复、报告或备份。

除了提供 Azure 应用商店中已可供使用的各种功能之外，现在你还可以使用通过 [Azure PowerShell](/documentation/articles/powershell-install-configure) 或 [Azure CLI](/documentation/articles/xplat-cli) 部署的资源管理器模板，在 Ubuntu VM 上轻松部署新的 MongoDB 群集。

根据此模板新部署的群集采用下图中所述的拓扑，不过，你可以通过自定义本文中所述的模板，轻松实现其他拓扑：

![cluster-architecture](./media/virtual-machines-mongodb-template/cluster-architecture.png)

通过一个参数你可以定义要在新的 MongoDB 群集中部署的节点数，而基于另一个参数，也可以将一个具有公共 IP 地址的 VM 实例 (Jumpbox) 部署在同一个虚拟网络内，从而使你能够从公共 Internet 连接到该群集并执行任何种类的与该群集相关的管理任务。使用作为参数提供的另一个选项可以向副本集添加一个仲裁节点，当副本集具有偶数个成员时通常会建议这样做。有关 MongoDB 复制拓扑和细节的详细信息，应参阅正式的 [MongoDB 文档](http://docs.mongodb.org/manual/core/replication-introduction/)。

部署完成之后，你可以使用 SSH 端口 22 上配置的 DNS 地址来访问 Jumpbox。

在深入了解与 Azure 资源管理器和你将用于此部署的模板相关的详细信息之前，请确保你已正确配置 Azure PowerShell 或 Azure CLI。

[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell)]

[AZURE.INCLUDE [xplat-getting-set-up-arm](../includes/xplat-getting-set-up-arm)]

## 使用资源管理器模板创建 MongoDB 群集

按照以下步骤，使用 Github 模板存储库中的资源管理器模板创建 MongoDB 群集。每个步骤将同时提供 Azure PowerShell 和 Azure CLI 指令。

### 步骤 1-a：使用 PowerShell 下载模板文件

为 JSON 模板和其他关联的文件创建本地文件夹（例如，C:\Azure\\Templates\\MongoDB）。

替换为你的本地文件夹的文件夹名称，并运行以下命令：

    $folderName="C:\Azure\Templates\MongoDB"
    $webclient = New-Object System.Net.WebClient
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/azuredeploy.json"
    $filePath = $folderName + "\azuredeploy.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/azuredeploy-parameters.json"
    $filePath = $folderName + "\azuredeploy-parameters.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/arbiter-resources.json"
    $filePath = $folderName + "\arbiter-resources.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/empty-resources.json"
    $filePath = $folderName + "\empty-resources.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/jumpbox-resources.json"
    $filePath = $folderName + "\jumpbox-resources.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D1.json"
    $filePath = $folderName + "\member-resources-D1.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D2.json"
    $filePath = $folderName + "\member-resources-D2.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D3.json"
    $filePath = $folderName + "\member-resources-D3.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D4.json"
    $filePath = $folderName + "\member-resources-D4.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D11.json"
    $filePath = $folderName + "\member-resources-D11.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D12.json"
    $filePath = $folderName + "\member-resources-D12.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D13.json"
    $filePath = $folderName + "\member-resources-D13.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/member-resources-D14.json"
    $filePath = $folderName + "\member-resources-D14.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/mongodb-ubuntu-install.sh"
    $filePath = $folderName + "\mongodb-ubuntu-install.sh"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/metadata.json"
    $filePath = $folderName + "\metadata.json"
    $webclient.DownloadFile($url,$filePath)
    $url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/mongodb-high-availability/shared-resources.json"
    $filePath = $folderName + "\shared-resources.json"
    $webclient.DownloadFile($url,$filePath)  

### 步骤 1-b：使用 Azure CLI 下载模板文件

使用所选的 git 客户端克隆整个模板存储库，例如：

    git clone https://github.com/Azure/azure-quickstart-templates C:\Azure\Templates

完成后，在 C:\Azure\\Templates 目录中查找 **mongodb-high-availability** 文件夹。

### 步骤 2：（可选）了解模板参数

部署非简单的解决方案（例如，MongoDB 群集）时，必须指定一组配置参数来处理一些所需的设置。通过在模板定义中声明这些参数，就能在部署期间通过外部文件或命令行指定值。

在 **azuredeploy.json** 文件顶部的“parameters”节中，你会发现模板需要用来配置 MongoDB 群集的参数集。以下示例来自此模板的 azuredeploy.json 文件的 parameters 节：

    "parameters": {
      "adminUsername": {
          "type": "string",
          "metadata": {
            "Description": "Administrator user name used when provisioning virtual machines (which also becomes a system user administrator in MongoDB)"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "metadata": {
            "Description": "Administrator password used when provisioning virtual machines (which is also a password for the system administrator in MongoDB)"
          }
        },
        "storageAccountName": {
          "type": "string",
          "defaultValue": "",
          "metadata": {
            "Description": "Unique namespace for the Storage Account where the Virtual Machine's disks will be placed (this name will be used as a prefix to create one or more storage accounts as per t-shirt size)"
          }
        },
        "region": {
          "type": "string",
          "metadata": {
            "Description": "Location where resources will be provisioned"
          }
        },
        "virtualNetworkName": {
          "type": "string",
          "defaultValue": "mongodbVnet",
          "metadata": {
            "Description": "The arbitrary name of the virtual network provisioned for the MongoDB deployment"
          }
        },
        "subnetName": {
          "type": "string",
          "defaultValue": "mongodbSubnet",
          "metadata": {
            "Description": "Subnet name for the virtual network that resources will be provisioned in to"
          }
        },
        "addressPrefix": {
          "type": "string",
          "defaultValue": "10.0.0.0/16",
          "metadata": {
            "Description": "The network address space for the virtual network"
          }
        },
        "subnetPrefix": {
          "type": "string",
          "defaultValue": "10.0.0.0/24",
          "metadata": {
            "Description": "The network address space for the virtual subnet"
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
            "Description": "The flag allowing to enable or disable provisioning of the jumpbox VM that can be used to access the MongoDB environment"
          }
        },
        "tshirtSize": {
          "type": "string",
          "defaultValue": "XSmall",
          "allowedValues": [
          "XSmall",
          "Small",
          "Medium",
          "Large",
          "XLarge",
          "XXLarge"
          ],
          "metadata": {
            "Description": "T-shirt size of the MongoDB deployment"
          }
        },
        "osFamily": {
          "type": "string",
          "defaultValue": "Ubuntu",
          "allowedValues": [
          "Ubuntu"
          ],
          "metadata": {
            "Description": "The target OS for the virtual machines running MongoDB"
          }
        },
        "mongodbVersion": {
          "type": "string",
          "defaultValue": "3.0.2",
          "metadata": {
            "Description": "The version of the MongoDB packages to be deployed"
          }
        },
        "replicaSetName": {
          "type": "string",
          "defaultValue": "rs0",
          "metadata": {
            "Description": "The name of the MongoDB replica set"
          }
        },
        "replicaSetKey": {
          "type": "string",
          "metadata": {
            "Description": "The shared secret key for the MongoDB replica set"
          }
        }
      },

每个参数都具有数据类型和允许值之类的详细信息。这样，便可以验证在交互模式（例如 PowerShell 或 Azure CLI）下，以及在自我发现 UI（通过分析所需参数列表及其说明动态生成的 UI）中执行模板期间所传递的参数。

### 步骤 3-a：使用 PowerShell 基于模板部署 MongoDB 群集

通过创建包含所有参数的运行时值的 JSON 文件，为部署准备参数文件。然后，将此文件作为单个实体传递给部署命令。如果未包含参数文件，PowerShell 将使用模板中指定的任何默认值，然后提示你填写剩余的值。

下面是 **azuredeploy-parameters.json** 文件中的一组示例参数：

    {
      "adminUsername": {
          "value": "MongoAdmin"
      },
      "adminPassword": {
          "value": ""
      },
      "storageAccountName": {
          "value": ""
      },
      "region": {
          "value": "China North"
      },
      "virtualNetworkName": {
          "value": "mongodbVnet"
      },
      "subnetName": {
          "value": "mongodbSubnet"
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
      "jumpbox": {
          "value": "Disabled"
      },
      "tshirtSize": {
          "value": "XSmall"
      },
      "osFamily": {
          "value": "Ubuntu"
      },
      "mongodbVersion": {
          "value": "3.0.2"
      },
      "replicaSetName": {
          "value": "rs0"
      },
      "replicaSetKey":  {
          "value": ""
      }
    }

填写 Azure 部署名称、资源组名称、Azure 位置，以及存储 JSON 部署文件的文件夹。然后运行以下命令：

    $deployName="<deployment name>"
    $RGName="<resource group name>"
    $locName="<Azure location, such as China North>"
    $folderName="<folder name, such as C:\Azure\Templates\MongoDB>"
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
- 在“资源组”边栏选项卡的“监视”部分中单击“事件”条形图可以查看部署的事件：
- 单击各个事件可以进一步细化到系统代表模板执行的各项操作的详细信息中。

测试之后，如果需要删除此资源组及其所有资源（存储帐户、虚拟机和虚拟网络），请使用这一命令：

    Remove-AzureResourceGroup –Name "<resource group name>" -Force

### 步骤 3-b：使用 Azure CLI 基于模板部署 MongoDB 群集

若要通过 Azure CLI 部署 MongoDB 群集，请先通过指定名称和位置来创建资源组：

    azure group create mdbc "China North"

将此资源组名称、JSON 模板文件的位置，以及参数文件的位置（有关详细信息，请参阅上面的 PowerShell 部分）传入以下命令：

    azure group deployment create mdbc -f .\azuredeploy.json -e .\azuredeploy-parameters.json

你可以使用以下命令来检查单个资源部署的状态：

    azure group deployment list mdbc

## MongoDB 模板结构和文件组织概览

为了让资源管理器模板的设计更加完善且可重复使用，你必须在部署 MongoDB 之类的复杂解决方案期间，考虑清楚如何安排一连串复杂但又彼此相关的任务。除了通过相关扩展执行脚本之外，还可以利用 ARM **模板链接**和**资源循环**，这样就能实现模块化方法，而所有基于模板的复杂部署基本上都能重复使用此方法。

下图描述了从 GitHub 中为此部署下载的所有文件彼此间的关系：

![mongodb-files](./media/virtual-machines-mongodb-template/mongodb-files.png)

本部分将指导你逐步了解 MongoDB 群集的 **azuredeploy.json** 文件结构。

### “parameters”节

**azuredeploy.json** 的“parameters”节指定此模板中使用的可修改参数。本文前面所述的 **azuredeploy-parameters.json** 文件用于在模板执行期间将值传入 azuredeploy.json 的“parameters”节。

### “variables”节

“variables”节指定可在这整个模板中使用的变量。这包含多个字段（JSON 数据类型或片段），在执行时，这些字段将设置为常量或计算值。下面是此 MongoDB 模板的“variables”节：

    "variables": {
          "_comment0": "/* T-shirt sizes may vary for different reasons, and some customers may want to modify these - so feel free to go ahead and define your favorite t-shirts */",
          "tshirtSizeXSmall": {
              "vmSizeMember": "Standard_D1",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 1,
              "totalMemberCount": 2,
              "arbiter": "Enabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D1.json')]",
              "storageAccountCount": 1,
              "dataDiskSize": 100
          },
          "tshirtSizeSmall": {
              "vmSizeMember": "Standard_D1",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 2,
              "totalMemberCount": 3,
              "arbiter": "Disabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D1.json')]",
              "storageAccountCount": 1,
              "dataDiskSize": 100
          },
          "tshirtSizeMedium": {
              "vmSizeMember": "Standard_D2",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 3,
              "totalMemberCount": 4,
              "arbiter": "Enabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D2.json')]",
              "storageAccountCount": 2,
              "dataDiskSize": 250
          },
          "tshirtSizeLarge": {
              "vmSizeMember": "Standard_D2",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 7,
              "totalMemberCount": 8,
              "arbiter": "Enabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D2.json')]",
              "storageAccountCount": 4,
              "dataDiskSize": 250
          },
          "tshirtSizeXLarge": {
              "vmSizeMember": "Standard_D3",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 7,
              "totalMemberCount": 8,
              "arbiter": "Enabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D3.json')]",
              "storageAccountCount": 4,
              "dataDiskSize": 500
          },
          "tshirtSizeXXLarge": {
              "vmSizeMember": "Standard_D3",
              "vmSizeArbiter": "Standard_A1",
              "numberOfMembers": 15,
              "totalMemberCount": 16,
              "arbiter": "Disabled",
              "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D3.json')]",
              "storageAccountCount": 8,
              "dataDiskSize": 500
          },
          "osFamilyUbuntu": {
              "osName": "ubuntu",
              "installerBaseUrl": "http://repo.mongodb.org/apt/ubuntu",
              "installerPackages": "mongodb-org",
              "imagePublisher": "Canonical",
              "imageOffer": "UbuntuServer",
              "imageSKU": "14.04.2-LTS"
          },
          "vmStorageAccountContainerName": "vhd-mongodb",
          "vmStorageAccountDomain": ".blob.core.chinacloudapi.cn",
          "vnetID": "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
          "sharedScriptUrl": "[concat('https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/shared_scripts/', variables('osFamilySpec').osName, '/')]",
          "scriptUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-high-availability/",
          "templateBaseUrl": "[variables('scriptUrl')]",
          "jumpboxTemplateEnabled": "jumpbox-resources.json",
          "jumpboxTemplateDisabled": "empty-resources.json",
          "arbiterTemplateEnabled": "arbiter-resources.json",
          "arbiterTemplateDisabled": "empty-resources.json",
          "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
          "jumpboxTemplateUrl": "[concat(variables('templateBaseUrl'), variables(concat('jumpboxTemplate', parameters('jumpbox'))))]",
          "arbiterTemplateUrl": "[concat(variables('templateBaseUrl'), variables(concat('arbiterTemplate', variables('clusterSpec').arbiter)))]",
          "commonSettings": {
              "availabilitySetName": "mongodbAvailSet",
              "region": "[parameters('region')]"
          },
          "storageSettings": {
              "vhdStorageAccountName": "[parameters('storageAccountName')]",
              "vhdContainerName": "[variables('vmStorageAccountContainerName')]",
              "destinationVhdsContainer": "[concat('https://', parameters('storageAccountName'), variables('vmStorageAccountDomain'), '/', variables('vmStorageAccountContainerName'), '/')]",
              "storageAccountCount": "[variables('clusterSpec').storageAccountCount]"
          },
          "networkSettings": {
              "virtualNetworkName": "[parameters('virtualNetworkName')]",
              "addressPrefix": "[parameters('addressPrefix')]",
              "subnetName": "[parameters('subnetName')]",
              "subnetPrefix": "[parameters('subnetPrefix')]",
              "subnetRef": "[concat(variables('vnetID'), '/subnets/', parameters('subnetName'))]",
              "machineIpPrefix": "[parameters('nodeAddressPrefix')]"
          },
          "machineSettings": {
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]",
              "machineNamePrefix": "mongodb-",
              "osImageReference": {
                  "publisher": "[variables('osFamilySpec').imagePublisher]",
                  "offer": "[variables('osFamilySpec').imageOffer]",
                  "sku": "[variables('osFamilySpec').imageSKU]",
                  "version": "latest"
              }
          },
          "clusterSpec": "[variables(concat('tshirtSize', parameters('tshirtSize')))]",
          "osFamilySpec": "[variables(concat('osFamily', parameters('osFamily')))]",
          "installCommand": "[concat('bash mongodb-', variables('osFamilySpec').osName, '-install.sh', ' -i ', variables('osFamilySpec').installerBaseUrl, ' -b ', variables('osFamilySpec').installerPackages, ' -r ', parameters('replicaSetName'), ' -k ', parameters('replicaSetKey'), ' -u ', parameters('adminUsername'), ' -p ', parameters('adminPassword'), ' -x ', variables('networkSettings').machineIpPrefix, ' -n ', variables('clusterSpec').totalMemberCount)]",
          "vmScripts": {
              "scriptsToDownload": [
                  "[concat(variables('scriptUrl'), 'mongodb-', variables('osFamilySpec').osName, '-install.sh')]",
                  "[concat(variables('sharedScriptUrl'), 'vm-disk-utils-0.1.sh')]"
              ],
              "regularNodeInstallCommand": "[variables('installCommand')]",
              "lastNodeInstallCommand": "[concat(variables('installCommand'), ' -l')]",
              "arbiterNodeInstallCommand": "[concat(variables('installCommand'), ' -a')]"
          },
          "_comment1": "/* The list of values below helps partition VM disks across multiple storage accounts to achieve a better throughput */",
          "_comment2": "/* Feel free to modify the default allocations if you are comfortable and understand what you are doing */",
          "storageAccountForXSmall_0": "0",
          "storageAccountForXSmall_1": "0",
          "storageAccountForSmall_0": "0",
          "storageAccountForSmall_1": "0",
          "storageAccountForSmall_2": "0",
          "storageAccountForMedium_0": "0",
          "storageAccountForMedium_1": "0",
          "storageAccountForMedium_2": "0",
          "storageAccountForMedium_3": "0",
          "storageAccountForLarge_0": "0",
          "storageAccountForLarge_1": "1",
          "storageAccountForLarge_2": "2",
          "storageAccountForLarge_3": "3",
          "storageAccountForLarge_4": "0",
          "storageAccountForLarge_5": "1",
          "storageAccountForLarge_6": "2",
          "storageAccountForLarge_7": "3",
          "storageAccountForXLarge_0": "0",
          "storageAccountForXLarge_1": "1",
          "storageAccountForXLarge_2": "2",
          "storageAccountForXLarge_3": "3",
          "storageAccountForXLarge_4": "0",
          "storageAccountForXLarge_5": "1",
          "storageAccountForXLarge_6": "2",
          "storageAccountForXLarge_7": "3",
          "storageAccountForXXLarge_0": "0",
          "storageAccountForXXLarge_1": "1",
          "storageAccountForXXLarge_2": "2",
          "storageAccountForXXLarge_3": "3",
          "storageAccountForXXLarge_4": "4",
          "storageAccountForXXLarge_5": "5",
          "storageAccountForXXLarge_6": "6",
          "storageAccountForXXLarge_7": "7",
          "storageAccountForXXLarge_8": "0",
          "storageAccountForXXLarge_9": "1",
          "storageAccountForXXLarge_10": "2",
          "storageAccountForXXLarge_11": "3",
          "storageAccountForXXLarge_12": "4",
          "storageAccountForXXLarge_13": "5",
          "storageAccountForXXLarge_14": "6",
          "storageAccountForXXLarge_15": "7"
      },

深入分析此示例之后，你可以看到两种不同的方法。在这第一个片段中，“osFamilyUbuntu”变量将设为 JSON 元素，其中包含 6 个键/值对：

    "osFamilyUbuntu": {
      "osName": "ubuntu",
      "installerBaseUrl": "http://repo.mongodb.org/apt/ubuntu",
      "installerPackages": "mongodb-org",
      "imagePublisher": "Canonical",
      "imageOffer": "UbuntuServer",
      "imageSKU": "14.04.2-LTS"
    },

在这第二个片段中，将“vmScripts”变量分配给 JSON 数组，其中的单个元素将在运行时使用模板语言函数 (concat)、另一个变量的值以及字符串常量来计算：

    "vmScripts": {
      "scriptsToDownload": [
      "[concat(variables('scriptUrl'), 'mongodb-', variables('osFamilySpec').osName, '-install.sh')]",
      "[concat(variables('sharedScriptUrl'), 'vm-disk-utils-0.1.sh')]"
    ],

此模板中的一个重要概念是针对 MongoDB 群集定义不同“T 恤大小”的方式。看一下这些“tshirtSizeXXXX”变量中的一个，可以注意到它描述了群集的部署方式的重要特征。让我们以“中等”大小为例：

    "tshirtSizeMedium": {
      "vmSizeMember": "Standard_D2",
      "vmSizeArbiter": "Standard_A1",
      "numberOfMembers": 3,
      "totalMemberCount": 4,
      "arbiter": "Enabled",
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'member-resources-D2.json')]",
      "storageAccountCount": 2,
      "dataDiskSize": 250
    },

“中等”MongoDB 群集将使用 D2 作为托管数据的 3 个 MongoDB 节点，以及将出于复制目的用作仲裁器的第四个 A1 VM 的 VM 大小。调用以部署数据节点的相应子模板将是 **member-resources-D2.json**，并且数据文件（每个 250 GB）将存储在 2 个存储帐户中。在“resources”节中将使用此变量来安排节点部署和其他任务。

### “resources”节

大多数操作是在**“resources”**节中发生的。仔细查看此节，你会立即找到两个不同的案例：第一个案例是定义为 `Microsoft.Resources/deployments` 类型的元素，它基本上表示调用主部署中的嵌套部署。通过“templateLink”元素（和相关的版本属性），可以指定链接的模板文件，并在调用此文件时传递一组参数作为输入，如同你在此片段中所看到的那样：

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

从第一个示例我们清楚地了解此方案中的 **azuredeploy.json** 如何作为一种协调机制进行组织，并调用其他一些模板文件，其中每个文件都负责所需部署活动的一部分。

具体而言，以下链接模板将用于此部署：

-	**shared-resource.json**：包含要在整个部署中共享的所有资源的定义。例如，用来存储 VM 的操作系统磁盘和虚拟网络的存储帐户。
-	**jumpbox-resources.json**：如果启用，将负责部署与 Jumpbox VM 相关的所有资源，该 VM 具有公共 IP 地址可用于从公共网络访问 MongoDB 群集。
-	**arbiter-resources.json**：如果启用，此模板会在 MongoDB 群集中部署仲裁器成员。仲裁器不包含数据，但在副本集包含偶数个用于管理主选举的节点时使用。
-	**member-resources-Dx.json**：这些资源模板有效地部署 MongoDB 节点。将根据所选的 T 恤大小定义使用特定文件，其中每个文件将因每个节点的附加磁盘数而异。
-	**mongodb-ubuntu-install.sh**：CustomScriptForLinux 扩展对群集中每个节点调用的 bash 脚本文件。负责装载并格式化数据磁盘，以及在节点上安装 MongoDB 各部分。

若要部署 MongoDB 群集，需要特定逻辑才能正确设置副本集。在部署过程中必须按照的特定顺序如下：

部署数据成员（以并行方式）= > 部署最后一个数据成员 = >（可选）部署仲裁器

在此顺序中，部署多个数据节点以并行方式进行，但最后一个节点除外。这是将形成群集和新副本集将部署到的位置，因此以前的所有节点需要在该时刻之前已启动并在运行。最后一步将部署可选的仲裁节点（仅适用于需要此节点的 T 恤大小）。

在我们的主模板 (azuredeploy.json) 内再看一下，让我们看一下如何实现此逻辑，从所有数据成员开始：

    {
      "type": "Microsoft.Resources/deployments",
      "name": "[concat('member-resources', copyindex())]",
      "apiVersion": "2015-01-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/', 'shared-resources')]"
      ],
      "copy": {
        "name": "memberNodesLoop",
        "count": "[variables('clusterSpec').numberOfMembers]"
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
            "value": {
              "vhdStorageAccountName": "[concat(variables('storageSettings').vhdStorageAccountName, variables(concat('storageAccountFor', parameters('tshirtSize'), '_', copyindex())))]",
                              "vhdContainerName": "[variables('storageSettings').vhdContainerName]",
                              "destinationVhdsContainer": "[concat('https://', variables('storageSettings').vhdStorageAccountName, variables(concat('storageAccountFor', parameters('tshirtSize'), '_', copyindex())), variables('vmStorageAccountDomain'), '/', variables('storageSettings').vhdContainerName, '/')]"
            }
          },
          "networkSettings": {
            "value": "[variables('networkSettings')]"
          },
          "machineSettings": {
            "value": {
              "adminUsername": "[variables('machineSettings').adminUsername]",
                              "adminPassword": "[variables('machineSettings').adminPassword]",
                              "machineNamePrefix": "[variables('machineSettings').machineNamePrefix]",
                              "osImageReference": "[variables('machineSettings').osImageReference]",
                              "vmSize": "[variables('clusterSpec').vmSizeMember]",
                              "dataDiskSize": "[variables('clusterSpec').dataDiskSize]",
                              "machineIndex": "[copyindex()]",
                              "vmScripts": "[variables('vmScripts').scriptsToDownload]",
                              "commandToExecute": "[variables('vmScripts').regularNodeInstallCommand]"
            }
          }
        }
      }
    },

在此要强调一个重要的概念，那就是如何可以部署单个资源类型的多个副本，而且可以为每一个实例指定所需设置的唯一值。此概念称为**资源循环**。

在上一片段中，参数（要在群集中部署的节点数）将用于设置变量 (“numberOfMembers”)，然后将该变量传递给 **“copy”**元素以触发多个子部署（环），每个子部署将导致群集中的每个成员的模板实例化。若要能够设置在实例之间需要唯一值的所有设置，可使用 **copyindex()** 函数获取数字值，以指示该特定资源循环创建中的当前索引。

资源创建中的另一个重要概念是能够指定资源间的依赖关系和优先级，如你在 **dependsOn** JSON 数组中所注意到的。在此特定模板中，部署每个节点取决于**共享资源**的前次成功部署。

在通过执行 **mongodb-ubuntu-install.sh** 脚本文件触发的节点准备活动中，将格式化附加的磁盘。在该文件中，实际上你可以找到此调用的实例：

    bash ./vm-disk-utils-0.1.sh -b $DATA_DISKS -s

**vm-disk-utils-0.1.sh** 是 azure-quickstart-tempates github 存储库中 **shared_scripts\\ubuntu** 文件夹的一部分，其中包含用于磁盘装入、格式化和条带化的非常有用的函数，每次需要在模板创建过程中执行类似任务时都可以使用。

另一个要探讨的有趣片段是与 CustomScriptForLinux VM 扩展相关的片段。这些扩展作为单独的资源类型安装，并依赖于每个群集节点部署模板，请在每个 **member-resources-Dx.json** 文件末尾查看此片段：

    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat('vmMember', parameters('machineSettings').machineIndex, '/installmongodb')]",
      "apiVersion": "2015-05-01-preview",
      "location": "[parameters('commonSettings').region]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', 'vmMember', parameters('machineSettings').machineIndex)]"
      ],
      "properties": {
        "publisher": "Microsoft.OSTCExtensions",
        "type": "CustomScriptForLinux",
        "typeHandlerVersion": "1.2",
        "settings": {
          "fileUris": "[parameters('machineSettings').vmScripts]", "commandToExecute":"[parameters('machineSettings').commandToExecute]"
        }
      }
    }

熟悉此部署包含的其他文件后，你将能够了解有关如何利用 Azure 资源管理器模板基于任何技术来组织和协调多节点解决方案的复杂部署策略所需的所有详细信息和最佳实践。在此建议一种构造模板文件的方法，请自行决定是否采用，如下图突出显示的部分所示：

![mongodb-template-structure](./media/virtual-machines-mongodb-template/mongodb-template-structure.png)

本质上，这种方法会建议：

-	将你的核心模板文件定义成所有特定部署活动的中心协调点，并利用模板链接来调用子模板执行
-	创建特定的模板文件，用于部署所有其他特定部署任务会共享的所有资源（例如存储帐户、虚拟网络配置，等等）。如果不同的部署在公用基础结构方面具有类似的要求，你就可以对其频繁重复使用此方法。
-	针对特定于给定资源的场地要求提供可选资源模板
-	针对资源组的相同成员（群集中的节点，等等）创建利用资源循环的特定模板，以便部署多个具有特有属性的实例。
-	对于所有部署后任务（例如产品安装、配置，等等），利用脚本部署扩展并创建特定于每种技术的脚本

有关详细信息，请参阅 [Azure 资源管理器模板语言](https://msdn.microsoft.com/zh-CN/library/azure/dn835138.aspx)。

<!---HONumber=67-->