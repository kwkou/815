<properties
    pageTitle="在 Resource Manager 模板中处理状态 | Azure"
    description="显示了如何通过建议的方法来使用复杂对象，以便与 Azure 资源管理器模板和已链接模板共享状态数据"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="fd2f5e2d-d56d-4e01-a57d-34f3eaead4a9"
    ms.service="azure-resource-manager"
    ms.workload="multiple"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/26/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />  


# 在 Azure 资源管理器模板中共享状态
本主题介绍有关在模板中管理和共享状态的最佳实践。本主题中所示的参数和变量是用户为了方便组织部署要求而可以定义的对象类型的示例。通过这些示例，你可以使用适合环境的属性值实现自己的对象。

本主题是包含更多内容的白皮书的一部分。若要阅读完整的白皮书，请下载[一流的 Resource Manager 模板注意事项和成熟的做法]（http://download.microsoft.com/download/8/E/1/8E1DBEFA-CECE-4DC9-A813-93520A5D7CFE/World Class ARM Templates - Considerations and Proven Practices.pdf）。

## 提供标准配置设置
通常会提供已知配置供用户选择，而不是提供一个过于灵活可变的模板。实际上，用户可以选择标准 T 恤尺寸，例如沙盒、小、中、大。T 恤尺寸的其他示例包括产品，例如社区版本或企业版本。在其他情况下，这可能是某种技术的工作负荷特定配置，例如，映射化简或 No SQL。

如果使用复杂对象，你可以创建包含数据集合的变量（有时称为“属性包”），并使用使用数据驱动模板中的资源声明。这种方法可对于预先为客户配置好的各种大小提供正常且已知的配置。如果没有已知配置，模板用户就必须自行确定群集大小、整合平台资源约束，以及执行数学运算来识别存储帐户的生成分区和其他资源（因群集大小和资源约束而导致）。除了为客户提供更好的体验，少量的已知配置可让你更轻松地提供支持，并帮助你提供较高的密度级别。

以下示例显示了如何定义包含复杂对象（代表数据聚合）的变量。这些集合定义的值可用于虚拟机大小、网络设置、操作系统设置和可用性设置。

    "variables": {
      "tshirtSize": "[variables(concat('tshirtSize', parameters('tshirtSize')))]",
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
    }

请注意，**tshirtSize** 变量连接通过参数（**Small**、**Medium**、**Large**）提供给文本 **tshirtSize** 的 T 恤大小。可以使用此变量来检索该 T 恤大小关联的复杂对象变量。

然后，你可以在将来引用模板中的这些变量。能够引用指定变量及其属性可以简化模板语法，从而轻松地理解上下文。下面的示例定义的资源在部署时可以使用前述对象来设置值。例如，VM 大小是通过检索 `variables('tshirtSize').vmSize` 的值来设置的，而磁盘大小的值是从 `variables('tshirtSize').diskSize` 检索的。此外，已链接模板的 URI 是通过 `variables('tshirtSize').vmTemplate` 的值来设置的。

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

## 将状态传递给模板
通过在部署期间直接提供的参数，可以在模板中共享状态。

下表列出了模板中的常用参数。

| 名称 | 值 | 说明 |
| --- | --- | --- |
| location |Azure 区域的约束列表中的字符串 |资源的部署位置。 |
| storageAccountNamePrefix |String |在其中放置 VM 磁盘的存储帐户的唯一 DNS 名称 |
| domainName |String |可公开访问的 jumpbox VM 的域名采用的格式：**{domainName}.{location}.cloudapp.com** 例如：**mydomainname.chinanorth.chinacloudapp.cn** |
| adminUsername |String |VM 的用户名 |
| adminPassword |String |VM 的密码 |
| tshirtSize |所提供 T 恤尺寸的约束列表中的字符串 |要预配的指定的缩放单位大小。例如，“小”、“中”、“大” |
| virtualNetworkName |String |使用者想要使用的虚拟网络的名称。 |
| enableJumpbox |约束列表中的字符串 (enabled/disabled) |一个参数，标识是否启用针对环境的 jumpbox。值：“enabled”、“disabled” |

上一部分中使用的 **tshirtSize** 参数定义为：

    "parameters": {
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
    }


## 将状态传递给链接模板
连接到已链接模板时，通常混合使用静态变量和生成的变量。

### 静态变量
静态变量通常用于提供基础值，例如 URL，可用于整个模板。

在以下模板摘录中，`templateBaseUrl` 指定模板在 GitHub 中的根位置。下一行构建新变量 `sharedTemplateUrl`，该变量将基 URL 的值与共享资源模板的已知名称连接起来。在该行下面使用了一个复杂的对象变量来存储 T 恤大小，其中的基 URL 将连接到已知配置模板位置并存储在 `vmTemplate` 属性中。

这样做的好处是，如果模板位置发生更改，则只需在一个位置更改静态变量，并将静态变量传递到所有链接模板。

    "variables": {
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
    }

### 生成的变量
除了静态变量之外，多个变量是动态生成的。本部分介绍所生成变量的部分常见类型。

#### tshirtSize
你已熟悉基于上述示例生成的这个变量。

#### networkSettings
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

#### availabilitySettings
在链接的模板中创建的资源通常都放在可用性集中。在下面的示例中，指定了可用性集名称，以及要使用的容错域和更新域计数。

    "availabilitySetSettings": {
      "name": "pgsqlAvailabilitySet",
      "fdCount": 3,
      "udCount": 5
    }

如果你需要多个可用性集（例如，一个用于主节点，另一个用于数据节点），你可以使用某个名称作为前缀，并指定多个可用性集，或者遵循此前显示的用于创建变量（针对特定 T 恤大小）的模型。

#### storageSettings
通常会与链接的模板共享存储详细信息。在以下示例中， *storageSettings* 对象提供了有关存储帐户和容器名称的详细信息。

    "storageSettings": {
        "vhdStorageAccountName": "[parameters('storageAccountName')]",
        "vhdContainerName": "[variables('vmStorageAccountContainerName')]",
        "destinationVhdsContainer": "[concat('https://', parameters('storageAccountName'), variables('vmStorageAccountDomain'), '/', variables('vmStorageAccountContainerName'), '/')]"
    }

#### osSettings
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

#### machineSettings
生成的变量 *machineSettings* 是复杂的对象，其中包含各种用于创建 VM 的核心变量。变量包括管理员用户名和密码、VM 名称前缀，以及操作系统映像引用。

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

请注意， *osImageReference* 会检索主模板中定义的 *osSettings* 变量的值。这意味着你可以轻松更改 VM 的操作系统 — 完全进行更改或基于模板使用者的首选项进行更改。

#### vmScripts
*vmScripts* 对象包含可以下载并可在 VM 实例上执行的脚本的详细信息，包括外部和内部引用。外部引用包含基础结构。内部引用包含已按照的软件和配置。

使用 *scriptsToDownload* 属性可以列出要下载到 VM 的脚本。此对象还包含对命令行参数的引用，可以执行不同类型的操作。这些操作包括为单个节点执行默认安装（在所有节点部署后即可运行的一种安装），以及执行任何其他可能特定于给定模板的脚本。

此示例来自用于部署 MongoDB（需要仲裁器才能提供高可用性）的模板。已将 *arbiterNodeInstallCommand* 添加到 *vmScripts* 以安装仲裁器。

在变量部分，可以找到用于定义特定文本以使用适当值执行脚本的变量。

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

    "[reference('master-node').outputs.masterip.value]"

可以在主模板的 outputs 节或 resources 节中使用此表达式。由于该表达式依赖于运行时状态，因此不能在 variables 节中使用。若要从主模板返回此值，请使用：

    "outputs": {
      "masterIpAddress": {
        "value": "[reference('master-node').outputs.masterip.value]",
        "type": "string"
      }

有关使用链接模板的 outputs 节返回虚拟机数据磁盘的示例，请参阅[为一个虚拟机创建多个数据磁盘](/documentation/articles/resource-group-create-multiple/)。

## 为虚拟机定义身份验证设置
可以使用与前面所示相同的配置设置模式来指定虚拟机的身份验证设置。需创建要在身份验证类型中传递的参数。

    "parameters": {
      "authenticationType": {
        "allowedValues": [
          "password",
          "sshPublicKey"
        ],
        "defaultValue": "password",
        "metadata": {
          "description": "Authentication type"
        },
        "type": "string"
      }
    }

需要为不同的身份验证类型添加变量，并使用一个变量根据参数值存储此部署使用的类型。

    "variables": {
      "osProfile": "[variables(concat('osProfile', parameters('authenticationType')))]",
      "osProfilepassword": {
        "adminPassword": "[parameters('adminPassword')]",
        "adminUsername": "notused",
        "computerName": "[parameters('vmName')]",
        "customData": "[base64(variables('customData'))]"
      },
      "osProfilesshPublicKey": {
        "adminUsername": "notused",
        "computerName": "[parameters('vmName')]",
        "customData": "[base64(variables('customData'))]",
        "linuxConfiguration": {
          "disablePasswordAuthentication": "true",
          "ssh": {
            "publicKeys": [
              {
                "keyData": "[parameters('sshPublicKey')]",
                "path": "/home/notused/.ssh/authorized_keys"
              }
            ]
          }
        }
      }
    }

定义虚拟机时，请将 **osProfile** 设置为已创建的变量。

    {
      "type": "Microsoft.Compute/virtualMachines",
      ...
      "osProfile": "[variables('osProfile')]"
    }


## 后续步骤
* 若要了解模板的章节，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)
* 若要查看模板中可用的函数，请参阅 [Azure Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)

<!---HONumber=Mooncake_1219_2016-->