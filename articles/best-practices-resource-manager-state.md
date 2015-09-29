<properties
	pageTitle="在 Azure 资源管理器模板中处理状态的最佳做法"
	description="显示了如何通过建议的方法来使用复杂对象，以便与 Azure 资源管理器模板和已链接模板共享状态数据"
	services="azure-resource-manager"
	documentationCenter=""
	authors="mmercuri"
	manager="georgem"
	editor="tysonn"/>

<tags
	ms.service="azure-resource-manager"
	ms.date="07/15/2015"
	wacn.date="08/29/2015"/>

# 在 Azure 资源管理器模板中共享状态

本主题介绍如何在 Azure 资源管理器模板中以及多个链接的模板中管理和共享状态。

## 使用复杂的对象来共享状态

除了单值参数，你还可以在 Azure 资源管理器模板中使用复杂对象作为参数。使用复杂对象时，你可以实现和引用特定区域的数据集合，例如 T 恤大小（用于描述虚拟机）、网络设置、操作系统 (OS) 设置和可用性设置。

以下示例显示了如何定义包含复杂对象（代表数据聚合）的变量。这些集合定义的值可用于虚拟机大小、网络设置、操作系统设置和可用性设置。

    "tshirtSizeLarge": {
      "vmSize": "Standard_A4",
      "diskSize": 1023,
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-16disk-resources.json')]",
      "vmCount": 3,
      "slaveCount": 2,
      "storage": {
        "name": "[parameters('storageAccountNamePrefix')]",
        "count": 2,
        "pool": "db",
        "map": [0,1,1],
        "jumpbox": 0
      }
    },
    "osSettings": {
      "scripts": [
        "[concat(variables('templateBaseUrl'), 'install_postgresql.sh')]",
        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/shared_scripts/ubuntu/vm-disk-utils-0.1.sh"
      ],
      "imageReference": {
				"publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "14.04.2-LTS",
        "version": "latest"
      }
    },
    "networkSettings": {
      "vnetName": "[parameters('virtualNetworkName')]",
      "addressPrefix": "10.0.0.0/16",
      "subnets": {
        "dmz": {
          "name": "dmz",
          "prefix": "10.0.0.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        },
        "data": {
          "name": "data",
          "prefix": "10.0.1.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        }
      }
    },
    "availabilitySetSettings": {
      "name": "pgsqlAvailabilitySet",
      "fdCount": 3,
      "udCount": 5
    }

然后，你可以在将来引用模板中的这些变量。能够引用指定变量及其属性可以简化模板语法，从而轻松地理解上下文。下面的示例定义的资源在部署时可以使用上述对象来设置值。例如，你可以注意到，VM 大小是通过检索 `variables('tshirtSize').vmSize` 的值来设置的，而磁盘大小的值是从 `variables('tshirtSize').diskSize` 检索的。此外，已链接模板的 URI 是通过 `variables('tshirtSize').vmTemplate` 的值来设置的。

    "name": "master-node",
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2015-01-01",
    "dependsOn": [
        "[concat('Microsoft.Resources/deployments/', 'shared')]"
    ],
    "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('tshirtSize').vmTemplate]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "adminPassword": {
            "value": "[parameters('adminPassword')]"
          },
          "replicatorPassword": {
            "value": "[parameters('replicatorPassword')]"
          },
          "osSettings": {
	    "value": "[variables('osSettings')]"
          },
          "subnet": {
            "value": "[variables('networkSettings').subnets.data]"
          },
          "commonSettings": {
            "value": {
              "region": "[parameters('region')]",
              "adminUsername": "[parameters('adminUsername')]",
              "namespace": "ms"
            }
          },
          "storageSettings": {
            "value":"[variables('tshirtSize').storage]"
          },
          "machineSettings": {
            "value": {
              "vmSize": "[variables('tshirtSize').vmSize]",
              "diskSize": "[variables('tshirtSize').diskSize]",
              "vmCount": 1,
              "availabilitySet": "[variables('availabilitySetSettings').name]"
            }
          },
          "masterIpAddress": {
            "value": "0"
          },
          "dbType": {
            "value": "MASTER"
          }
        }
      }
    }

## 将状态传递给模板及其链接的模板

你可以通过以下参数将状态信息共享到模板及其链接的模板中：

- 在部署过程中直接向主模板提供的参数
- 主模板与链接的模板共享的参数、静态变量和生成的变量

### 向主模板提供的公用参数

下表列出了模板中的常用参数。

**传递给主模板的常用参数**

Name | 值 | 说明
---- | ----- | -----------
location | Azure 区域的约束列表中的字符串 | 将在其中部署资源的位置。
storageAccountNamePrefix | String | 将在其中放置 VM 磁盘的存储帐户的唯一 DNS 名称
domainName | String | 可公开访问的 jumpbox VM 的域名采用的格式：**{domainName}.{location}.chinacloudapp.cn** 例如：**mydomainname.westus.chinacloudapp.azure.cn**
adminUsername | String | VM 的用户名
adminPassword | String | VM 的密码
tshirtSize | 所提供 T 恤尺寸的约束列表中的字符串 | 要预配的指定的缩放单位大小。例如，“小”、“中”、“大”
virtualNetworkName | String | 使用者想要使用的虚拟网络的名称。
enableJumpbox | 约束列表中的字符串 (enabled/disabled) | 一个参数，标识是否启用针对环境的 jumpbox。值：“enabled”、“disabled”

### 发送到已链接模板的参数

连接到已链接模板时，你会经常混合使用静态变量和生成的变量。

#### 静态变量

静态变量通常用于提供基础值，例如 URL，可用于整个模板，或者用来为动态变量组合各种值。

在下面的模板摘录中，*templateBaseUrl* 指定模板在 GitHub 中的根位置。下一行构建新变量 *sharedTemplateUrl*，该变量将 *templateBaseUrl* 的值与共享资源模板的已知名称连接起来。在其下面使用了一个复杂的对象变量来存储 T 恤大小，其中会连接 *templateBaseUrl* 来指定存储在 *vmTemplate* 属性中的已知配置模板位置。

此方法的好处是可以轻松地移动模板、使模板分叉，或者将模板作为基础来创建新的模板。如果模板位置发生更改，则只需在一个位置更改静态变量，该位置就是主模板。主模板可将静态变量传递到所有模板中。

    "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
    "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
    "tshirtSizeSmall": {
      "vmSize": "Standard_A1",
      "diskSize": 1023,
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]",
      "vmCount": 2,
      "slaveCount": 1,
      "storage": {
        "name": "[parameters('storageAccountNamePrefix')]",
        "count": 1,
        "pool": "db",
        "map": [0,0],
        "jumpbox": 0
      }
    }

#### 生成的变量

除了静态变量之外，许多变量是动态生成的。本部分介绍所生成变量的部分常见类型。

##### tshirtSize

在调用主模板时，你可以从固定数目的选项中选择 T 恤大小，这些选项通常包括各种值，例如“小”、“中”、“大”。

在主模板中，该选项显示为参数，例如 *tshirtSize*：

    "tshirtSize": {
      "type": "string",
      "defaultValue": "Small",
      "allowedValues": [
        "Small",
        "Medium",
        "Large"
      ],
      "metadata": {
        "Description": "T-shirt size of the MongoDB deployment"
      }
    }

在主模板中，每种大小都有相应的变量。例如，如果提供的大小为小、中、大，则变量部分会包含名为 *tshirtSizeSmall*、*tshirtSizeMedium* 和 *tshirtSizeLarge* 的变量。

如以下示例所示，这些变量用于定义特定 T 恤大小的属性。单个变量可用于标识 VM 类型、磁盘大小、关联的要链接到的缩放单元资源模板、实例数、存储帐户详细信息，以及 jumpbox 状态。

存储帐户名称前缀是取自用户提供的参数，链接的模板是将模板的基 URL 与特定缩放单元资源模板的文件名相连接。

    "tshirtSizeSmall": {
      "vmSize": "Standard_A1",
			"diskSize": 1023,
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]",
      "vmCount": 2,
      "storage": {
        "name": "[parameters('storageAccountNamePrefix')]",
        "count": 1,
        "pool": "db",
        "map": [0,0],
        "jumpbox": 0
      }
    },
    "tshirtSizeMedium": {
      "vmSize": "Standard_A3",
      "diskSize": 1023,
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-8disk-resources.json')]",
      "vmCount": 2,
      "storage": {
        "name": "[parameters('storageAccountNamePrefix')]",
        "count": 2,
        "pool": "db",
        "map": [0,1],
        "jumpbox": 0
      }
    },
    "tshirtSizeLarge": {
      "vmSize": "Standard_A4",
      "diskSize": 1023,
      "vmTemplate": "[concat(variables('templateBaseUrl'), 'database-16disk-resources.json')]",
      "vmCount": 3,
      "storage": {
        "name": "[parameters('storageAccountNamePrefix')]",
        "count": 2,
        "pool": "db",
        "map": [0,1,1],
        "jumpbox": 0
      }
    }

*tshirtSize* 变量显示在变量部分的更下面。你提供的 T 恤大小（*小*、*中*、*大*）的末端与文本 *tshirtSize* 相连，可检索该 T 恤大小所关联的复杂对象变量：

    "tshirtSize": "[variables(concat('tshirtSize', parameters('tshirtSize')))]",

此变量传递给链接的缩放单元资源模板。

##### networkSettings

在容量、功能或端到端范围解决方案模板中，链接的模板通常创建存在于网络上的资源。一个简单的方法是使用复杂的对象来存储网络设置并将其传递给链接的模板。

下面是网络设置通信的示例。

    "networkSettings": {
      "vnetName": "[parameters('virtualNetworkName')]",
      "addressPrefix": "10.0.0.0/16",
      "subnets": {
        "dmz": {
          "name": "dmz",
          "prefix": "10.0.0.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        },
        "data": {
          "name": "data",
          "prefix": "10.0.1.0/24",
          "vnet": "[parameters('virtualNetworkName')]"
        }
      }
    }

##### availabilitySettings

在链接的模板中创建的资源通常都放在可用性集中。在下面的示例中，指定了可用性集名称，以及要使用的容错域和更新域计数。

    "availabilitySetSettings": {
      "name": "pgsqlAvailabilitySet",
      "fdCount": 3,
      "udCount": 5
    }

如果你需要多个可用性集（例如，一个用于主节点，另一个用于数据节点），你可以使用某个名称作为前缀，并指定多个可用性集，或者遵循此前显示的用于创建变量（针对特定 T 恤大小）的模型。

##### storageSettings

通常会与链接的模板共享存储详细信息。在以下示例中，*storageSettings* 对象提供了有关存储帐户和容器名称的详细信息。

    "storageSettings": {
        "vhdStorageAccountName": "[parameters('storageAccountName')]",
        "vhdContainerName": "[variables('vmStorageAccountContainerName')]",
        "destinationVhdsContainer": "[concat('https://', parameters('storageAccountName'), variables('vmStorageAccountDomain'), '/', variables('vmStorageAccountContainerName'), '/')]"
    }

##### osSettings

使用链接的模板，可能需要通过不同的已知配置类型将操作系统设置传递给各种节点类型。使用复杂对象可以轻松地存储和共享操作系统信息，还可以更轻松地支持多个操作系统部署选项。

下面的示例演示了 *osSettings* 的一个对象：

    "osSettings": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "14.04.2-LTS",
        "version": "latest"
      }
    }

##### machineSettings

生成的变量 *machineSettings* 是一个复杂的对象，其中包含各种用于创建新的 VM 的核心变量：管理员用户名和密码、VM 名称前缀，以及操作系统映像引用，如下所示：

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

请注意，*osImageReference* 会检索主模板中定义的 *osSettings* 变量的值。这意味着你可以轻松更改 VM 的操作系统 — 完全进行更改或基于模板使用者的首选项进行更改。

##### vmScripts

*vmScripts* 对象包含可以下载并可在 VM 实例上执行的脚本的详细信息，包括外部和内部引用。外部引用包含基础结构。内部引用包含已按照的软件和配置。

使用 *scriptsToDownload* 属性可以列出要下载到 VM 的脚本。

如以下示例所示，此对象还包含对命令行参数的引用，可以执行不同类型的操作。这些操作包括为单个节点执行默认安装（在所有节点部署后即可运行的一种安装），以及执行任何其他可能特定于给定模板的脚本。

此示例来自用于部署 MongoDB（需要仲裁器才能提供高可用性）的模板。已将 *arbiterNodeInstallCommand* 添加到 *vmScripts* 以安装仲裁器。

在变量部分，你可以找到用于定义特定文本以使用适当值执行脚本的变量。

    "vmScripts": {
        "scriptsToDownload": [
            "[concat(variables('scriptUrl'), 'mongodb-', variables('osFamilySpec').osName, '-install.sh')]",
            "[concat(variables('sharedScriptUrl'), 'vm-disk-utils-0.1.sh')]"
        ],
        "regularNodeInstallCommand": "[variables('installCommand')]",
        "lastNodeInstallCommand": "[concat(variables('installCommand'), ' -l')]",
        "arbiterNodeInstallCommand": "[concat(variables('installCommand'), ' -a')]"
    },


## 从模板返回状态

不仅可以传递数据到模板中，还可以反过来将数据共享给调用的模板。在已链接模板的**“输出”**部分，你可以提供允许源模板使用的键/值对。

以下示例演示如何传递在已链接模板中生成的专用 IP 地址。

    "outputs": {
        "masterip": {
            "value": "[reference(concat(variables('nicName'),0)).ipConfigurations[0].properties.privateIPAddress]",
            "type": "string"
         }
    }

在主模板中，可通过以下语法来使用该数据：

    "masterIpAddress": {
        "value": "[reference('master-node').outputs.masterip.value]"
    }

<!--## 后续步骤
- [创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)
- [Azure 资源管理器模板函数](/documentation/articles/resource-group-template-functions)-->

<!---HONumber=67-->