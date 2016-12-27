<properties
    pageTitle="Resource Manager 模板最佳实践 | Azure"
    description="有关简化 Azure Resource Manager 模板的指导。"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="31b10deb-0183-47ce-a5ba-6d0ff2ae8ab3"
    ms.service="azure-resource-manager"
    ms.workload="multiple"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/25/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />

# 创建 Azure Resource Manager 模板的最佳实践
以下指导帮助用户创建可靠且易于使用的 Resource Manager 模板。这些指导仅供参考，并不一定要你遵循。你可能需要根据方案对此做出变通。

## <a name="resource-names"></a> 资源名称
通常处理三种类型的资源名称：

1. 必须唯一的资源名称。
2. 不需要唯一的资源名称，不过，提供的名称应可帮助识别上下文。
3. 通用的资源名称。

有关建立命名约定的帮助，请参阅 [Infrastructure naming guidelines](/documentation/articles/virtual-machines-windows-infrastructure-naming-guidelines/)（基础结构命名指南）。

### 唯一的资源名称
对于具有数据访问终结点的任何资源类型，必须提供唯一的资源名称。需要唯一名称的一些常见类型包括：

* 存储帐户
* 网站
* SQL 服务器
* 密钥保管库
* Redis 缓存
* 批处理帐户
* 流量管理器
* 搜索服务
* HDInsight 群集

此外，存储帐户名必须使用小写字母，包含 24 个或更少的字符，并且不包含任何连字符。

如果为这些资源名称提供参数，则必须在部署过程中构思唯一的名称。与此相反，可以创建一个变量，使用 [uniqueString()](/documentation/articles/resource-group-template-functions#uniquestring) 函数生成名称。通常，还需要在 **uniqueString** 中添加一个前缀或后缀，这样，只需查看名称就能轻松确定资源类型。例如，可以使用以下变量生成存储帐户的唯一名称：

    "variables": {
        "storageAccountName": "[concat(uniqueString(resourceGroup().id),'storage')]"
    }

包含 uniqueString 前缀的存储帐户不在同一机架中组建群集。

### 用于标识的资源名称
对于想要命名但不提供唯一性保证的资源类型，只需提供一个名称用于标识其上下文和资源类型。需要提供一个描述性名称，用于在资源名称列表中识别该资源。如果需要在部署期间改变资源名称，请使用该名称的参数：

    "parameters": {
        "vmName": { 
            "type": "string",
            "defaultValue": "demoLinuxVM",
            "metadata": {
                "description": "The name of the VM to create."
            }
        }
    }

如果在部署期间不需要传入名称，请使用一个变量：

    "variables": {
        "vmName": "demoLinuxVM"
    }

或者使用硬编码值：

    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "demoLinuxVM",
      ...
    }

### 通用资源名称
对于主要通过其他资源访问的资源类型，可以在模板中使用一个经过硬编码的通用名称。例如，你可能不想要为 SQL Server 上的防火墙规则提供可自定义的名称。

    {
        "type": "firewallrules",
        "name": "AllowAllWindowsAzureIps",
        ...
    }

## Parameters
1. 尽量不要使用参数，而是尽可能地使用变量或文本。只为以下各项提供参数：
   
    * 想要根据环境更改的设置（如 SKU、大小或容量）。
    * 想要方便识别而指定的资源名称。
    * 通常用于完成其他任务的值（如用户名）。
    * 机密（如密码）
    * 创建资源类型的多个实例时要使用的值的数目或数组。
2. 参数名称应遵循 **camelCasing**。
3. 在每个参数的元数据中提供说明。
   
        "parameters": {
            "storageAccountType": {
                "type": "string",
                "metadata": {
                    "description": "The type of the new storage account created to store the VM disks"
                }
            }
        }
4. 定义参数（密码和 SSH 密钥除外）的默认值。
   
        "parameters": {
            "storageAccountType": {
                "type": "string",
                "defaultValue": "Standard_GRS",
                "metadata": {
                    "description": "The type of the new storage account created to store the VM disks"
                }
            }
        }
5. 为所有密码和机密使用 **securestring**。
   
        "parameters": {
            "secretValue": {
                "type": "securestring",
                "metadata": {
                    "description": "Value of the secret to store in the vault"
                }
            }
        }
6. 尽量避免使用参数来指定**位置**。改用资源组的位置属性。如果为所有资源使用 **resourceGroup().location** 表达式，模板中的资源将部署在与资源组相同的位置。
   
        "resources": [
          {
              "name": "[variables('storageAccountName')]",
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2016-01-01",
              "location": "[resourceGroup().location]",
              ...
          }
        ]
   
     如果只有有限数量的位置支持某种资源类型，请考虑在模板中直接指定有效的位置。如果必须使用位置参数，请尽量与可能需要位于同一位置的资源共享该参数值。这种方法可以最大程度地减少用户为每个资源类型提供位置的需要。
7. 避免对资源类型的 API 版本使用参数或变量。资源的属性和值可能会因版本号的不同而异。如果将 API 版本设置为参数或变量，代码编辑器中的 Intellisense 将无法确定正确的架构，并且会在模板中将 API 版本硬编码。

## 变量
1. 针对需要在模板中多次使用的值使用变量。如果一次只使用一个值，则硬编码值可使模板更易于阅读。
2. 不能在 variables 节中使用 [reference](/documentation/articles/resource-group-template-functions#reference) 函数。reference 函数从资源的运行时状态中派生其值，但变量是在初始模板分析期间解析的。应直接在模板的 **resources** 或 **outputs** 节中构造需要 **reference** 函数的值。
3. 根据[资源名称](#resource-names)中所述，针对需要保持唯一的资源名称包含变量。
4. 可以将变量组合成复杂对象。可以使用 **variable.subentry** 格式，从复杂对象引用值。组合变量有助于跟踪相关变量，并提高模板的易读性。
   
        "variables": {
            "storage": {
                "name": "[concat(uniqueString(resourceGroup().id),'storage')]",
                "type": "Standard_LRS"
            }
        },
        "resources": [
          {
              "type": "Microsoft.Storage/storageAccounts",
              "name": "[variables('storage').name]",
              "apiVersion": "2016-01-01",
              "location": "[resourceGroup().location]",
              "sku": {
                  "name": "[variables('storage').type]"
              },
              ...
          }
        ]
   
    > [AZURE.NOTE]
    复杂对象不能包含从复杂对象引用值的表达式。若要进行这种引用，可以定义一个单独的变量。
    > 
    > 
   
    有关使用复杂对象作为变量的更高级示例，请参阅[Sharing state in Azure Resource Manager templates](/documentation/articles/best-practices-resource-manager-state/)（在 Azure Resource Manager 模板中共享状态）。

## 资源
1. 为模板中的每个资源指定**注释**，以帮助其他参与者理解该资源的用途。
   
        "resources": [
          {
              "name": "[variables('storageAccountName')]",
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2016-01-01",
              "location": "[resourceGroup().location]",
              "comments": "This storage account is used to store the VM disks",
              ...
          }
        ]
2. 使用标记将元数据添加到可让你添加有关资源的其他信息的资源。例如，可以将元数据添加到某个资源以显示计费详细信息。有关详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags/)。
3. 如果在模板中使用**公共终结点**（例如 Blob 存储公共终结点），请**不要**将命名空间硬编码。使用 **reference** 函数可动态检索命名空间。使用此方法可以将模板部署到不同的公共命名空间环境，而无需在模板中手动更改终结点。在模板中将 apiVersion 设置为用于 storageAccount 的同一版本。
   
        "osDisk": {
            "name": "osdisk",
            "vhd": {
                "uri": "[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2016-01-01').primaryEndpoints.blob, variables('vmStorageAccountContainerName'), '/',variables('OSDiskName'),'.vhd')]"
            }
        }
   
     如果在同一模板中部署了存储帐户，则引用资源时不需要指定提供程序命名空间。简化的语法为：
   
        "osDisk": {
            "name": "osdisk",
            "vhd": {
                "uri": "[concat(reference(variables('storageAccountName'), '2016-01-01').primaryEndpoints.blob, variables('vmStorageAccountContainerName'), '/',variables('OSDiskName'),'.vhd')]"
            }
        }
   
     如果在使用公共命名空间配置的模板中包含了其他值，请更改这些值以反映相同的 reference 函数。例如，虚拟机 diagnosticsProfile 的 storageUri 属性。
   
        "diagnosticsProfile": {
            "bootDiagnostics": {
                "enabled": "true",
                "storageUri": "[reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2016-01-01').primaryEndpoints.blob]"
            }
        }
   
     你还可以在不同资源组中**引用**现有的存储帐户。

        "osDisk": {
            "name": "osdisk", 
            "vhd": {
                "uri":"[concat(reference(resourceId(parameters('existingResourceGroup'), 'Microsoft.Storage/storageAccounts/', parameters('existingStorageAccountName')), '2016-01-01').primaryEndpoints.blob,  variables('vmStorageAccountContainerName'), '/', variables('OSDiskName'),'.vhd')]"
            }
        }

1. 仅当应用程序有需要时，才将 publicIPAddresses 分配到虚拟机。若要建立连接以进行调试或管理，请使用 inboundNatRules、virtualNetworkGateways 或 jumpbox。
   
     有关连接到虚拟机的详细信息，请参阅：
   
    * [为 Azure Resource Manager 中的虚拟机设置 WinRM 访问权限](/documentation/articles/virtual-machines-windows-winrm/)
    * [使用 Azure 门户预览实现对 VM 的外部访问](/documentation/articles/virtual-machines-windows-nsg-quickstart-portal/)
    * [使用 PowerShell 对 VM 实现外部访问](/documentation/articles/virtual-machines-windows-nsg-quickstart-powershell/)
    * [打开端口和终结点](/documentation/articles/virtual-machines-linux-nsg-quickstart/)
2. publicIPAddresses 的 **domainNameLabel** 属性必须唯一。domainNameLabel 必须包含 3 到 63 个字符，并遵循正则表达式 `^[a-z][a-z0-9-]{1,61}[a-z0-9]$` 指定的规则。由于 uniqueString 函数生成 13 个字符的字符串，因此 dnsPrefixString 参数不得超过 50 个字符。
   
        "parameters": {
            "dnsPrefixString": {
                "type": "string",
                "maxLength": 50,
                "metadata": {
                    "description": "DNS Label for the Public IP. Must be lowercase. It should match with the following regular expression: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$ or it will raise an error."
                }
            }
        },
        "variables": {
            "dnsPrefix": "[concat(parameters('dnsPrefixString'),uniquestring(resourceGroup().id))]"
        }
3. 将密码添加到 **customScriptExtension** 时，请在 protectedSettings 中使用 **commandToExecute** 属性。
   
        "properties": {
            "publisher": "Microsoft.Azure.Extensions",
            "type": "CustomScript",
            "typeHandlerVersion": "2.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "fileUris": [
                    "[concat(variables('template').assets, '/lamp-app/install_lamp.sh')]"
                ]
            },
            "protectedSettings": {
                "commandToExecute": "[concat('sh install_lamp.sh ', parameters('mySqlPassword'))]"
            }
        }
   
    > [AZURE.NOTE]
    为了确保作为参数传递给 virtualMachines/扩展的机密经过加密，请使用相关扩展的 protectedSettings 属性。
    > 
    > 

## Outputs
如果模板创建 **publicIPAddresses**，则应有一个 **outputs** 节，以便返回 IP 地址的详细信息和完全限定的域名。用户可以使用这些输出值在部署后轻松检索这些详细信息。引用资源时，请使用创建该资源时所用的 API 版本。

    "outputs": {
        "fqdn": {
            "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName')), '2016-07-01').dnsSettings.fqdn]",
            "type": "string"
        },
        "ipaddress": {
            "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName')), '2016-07-01').ipAddress]",
            "type": "string"
        }
    }

## 单个模板或嵌套模板
若要部署解决方案，可以使用单个模板，或者使用包含多个嵌套模板的主模板。嵌套模板在较高级方案中很常见。嵌套模板具有以下优点：

1. 可将解决方案分解为目标组件
2. 可在不同的主模板中重复使用嵌套模板

如果确定要将模板设计分解为多个嵌套模板，可通过以下指导将设计标准化。建议的设计包括以下模板：

* **主模板** (azuredeploy.json)。用于输入参数。
* **共享的资源模板**。部署其他所有资源使用的共享资源（例如虚拟网络、可用性集）。表达式 dependsOn 强制在其他模板之前部署此模板。
* **可选资源模板**。根据某个参数（例如 jumpbox）有条件地部署资源
* **成员资源模板**。应用程序层中的每个实例类型都有其自身的配置。在层中，可以定义不同的实例类型（例如，第一个实例创建群集，其他实例将添加到现有群集）。每个实例类型都有其自身的部署模板。
* **脚本**。广泛可重用的脚本适用于每个实例类型（例如，初始化和格式化其他磁盘）。为特定自定义目的创建的自定义脚本根据实例类型的不同而异。

![嵌套模板](./media/resource-manager-template-best-practices/nestedTemplateDesign.png)

有关详细信息，请参阅[将链接的模板与 Azure 资源管理器配合使用](/documentation/articles/resource-group-linked-templates/)。

## 有条件地链接到嵌套模板
可以使用属于模板 URI 一部分的参数，有条件地链接到嵌套模板。

    "parameters": {
        "newOrExisting": {
            "type": "String",
            "allowedValues": [
                "new",
                "existing"
            ]
        }
    },
    "variables": {
        "templatelink": "[concat('https://raw.githubusercontent.com/Contoso/Templates/master/',parameters('newOrExisting'),'StorageAccount.json')]"
    },
    "resources": [
        {
            "apiVersion": "2015-01-01",
            "name": "nestedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "incremental",
                "templateLink": {
                    "uri": "[variables('templatelink')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters": {
                }
            }
        }
    ]

## 模板格式
1. 一种不错的做法是通过 JSON 验证程序传递模板来删除多余的逗号、圆括号和方括号，以免部署期间出错。尝试对你偏好编辑环境（Visual Studio Code、Atom、Sublime Text、Visual Studio 等）使用 [JSONlint](http://jsonlint.com/) 或 linter 包
2. 另外，一个不错的想法是设置 JSON 的格式以以提高可读性。可以为本地编辑器使用 JSON 格式化程序包。在 Visual Studio 中，使用 **Ctrl+K、Ctrl+D** 设置文档的格式。在 VS Code 中，使用 **Alt+Shift+F**。如果你的本地编辑器无法设置文档格式，你可以使用[联机格式化程序](https://www.bing.com/search?q=json+formatter)。

## 后续步骤
* 有关设置存储帐户的指导，请参阅 [Azure Storage Performance and Scalability Checklist](/documentation/articles/storage-performance-checklist/)（Azure 存储空间的性能和可缩放性清单）。
* 有关虚拟网络的帮助，请参阅 [Networking infrastructure guidelines](/documentation/articles/virtual-machines-windows-infrastructure-networking-guidelines/)（网络基础结构指南）。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_1219_2016-->