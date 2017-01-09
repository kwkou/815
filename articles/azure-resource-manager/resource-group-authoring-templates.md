<properties
    pageTitle="创作 Azure Resource Manager 模板 | Azure"
    description="使用声明性 JSON 语法创建 Azure 资源管理器模板，以将应用程序部署到 Azure。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="19694cb4-d9ed-499a-a2cc-bcfc4922d7f5"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/01/2016"
    wacn.date="01/06/2017"
    ms.author="tomfitz" />

# 创作 Azure 资源管理器模板
本主题介绍 Azure Resource Manager 模板的结构，演示了模板的不同部分，以及可在相应部分使用的属性。模板中包含可用于为部署构造值的 JSON 和表达式。

若要查看已部署资源的模板，请参阅[从现有资源导出 Azure Resource Manager 模板](/documentation/articles/resource-manager-export-template/)。有关创建模板的指导，请参阅 [Resource Manager Template Walkthrough](/documentation/articles/resource-manager-template-walkthrough/)（Resource Manager 模板演练）。有关创建模板的建议，请参阅 [Best practices for creating Azure Resource Manager templates](/documentation/articles/resource-manager-template-best-practices/)（创建 Azure Resource Manager 模板的最佳实践）。

好的 JSON 编辑器可以简化模板创建任务。有关使用 Visual Studio 处理模板的信息，请参阅 [Creating and deploying Azure resource groups through Visual Studio](/documentation/articles/vs-azure-tools-resource-groups-deployment-projects-create-deploy/)（通过 Visual Studio 创建和部署 Azure 资源组）。有关如何使用 VS Code 的信息，请参阅 [Working with Azure Resource Manager Templates in Visual Studio Code](/documentation/articles/resource-manager-vs-code/)（在 Visual Studio Code 中使用 Azure Resource Manager 模板）。

将模板大小限制为 1 MB 以内，每个参数文件大小限制为 64 KB 以内。通过迭代资源定义及变量和参数的值扩展模板后，1 MB 的限制适用于模板的最终状态。

## <a name="template-format"></a> 模板格式
使用最简单的结构时，模板包含以下元素：

    {
       "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "",
       "parameters": {  },
       "variables": {  },
       "resources": [  ],
       "outputs": {  }
    }

| 元素名称 | 必选 | 说明 |
|:--- |:--- |:--- |
| $schema |是 |描述模板语言版本的 JSON 架构文件所在的位置。使用前面的示例中显示的 URL。 |
| contentVersion |是 |模板的版本（例如 1.0.0.0）。可为此元素提供任意值。使用模板部署资源时，此值可用于确保使用正确的模板。 |
| 参数 |否 |执行部署以自定义资源部署时提供的值。 |
| variables |否 |在模板中用作 JSON 片段以简化模板语言表达式的值。 |
| 资源 |是 |已在资源组中部署或更新的资源类型。 |
| outputs |否 |部署后返回的值。 |

本主题稍后更详细地介绍模板的各个节。

## 表达式和函数
模板的基本语法为 JSON。但是，表达式和函数扩展了模板中的可用 JSON。使用表达式，可以创建不是严格文本值的值。表达式括在方括号（[ 和 ]）中，并在部署模板时进行求值。表达式可以出现在 JSON 字符串值中的任何位置，并始终返回另一个 JSON 值。如果需要使用以方括号 [ 开头的文本字符串，则必须使用两个方括号 [[。

通常，你会将表达式与函数一起使用，以执行用于配置部署的操作。如同在 JavaScript 中一样，函数调用的格式为 **functionName(arg1,arg2,arg3)**。使用点和 [index] 运算符引用属性。

以下示例演示如何在构造值时使用一些函数：

    "variables": {
       "location": "[resourceGroup().location]",
       "usernameAndPassword": "[concat('parameters('username'), ':', parameters('password'))]",
       "authorizationHeader": "[concat('Basic ', base64(variables('usernameAndPassword')))]"
    }

有关模板函数的完整列表，请参阅 [Azure 资源管理器模板函数](/documentation/articles/resource-group-template-functions/)。

## <a name="parameters"></a> Parameters
在模板的 parameters 节中，你可以指定在部署资源时能够输入的值。提供针对特定环境（例如开发、测试和生产环境）定制的参数值可以自定义部署。无需在模板中提供参数，但如果没有参数，模板始终部署具有相同名称、位置和属性的相同资源。

你可以在整个模板中使用这些参数值，来为部署的资源设置值。在模板的其他节中，只能使用 parameters 节中声明的参数。

使用以下结构定义参数：

    "parameters": {
       "<parameter-name>" : {
         "type" : "<type-of-parameter-value>",
         "defaultValue": "<default-value-of-parameter>",
         "allowedValues": [ "<array-of-allowed-values>" ],
         "minValue": <minimum-value-for-int>,
         "maxValue": <maximum-value-for-int>,
         "minLength": <minimum-length-for-string-or-array>,
         "maxLength": <maximum-length-for-string-or-array-parameters>,
         "metadata": {
             "description": "<description-of-the parameter>" 
         }
       }
    }

| 元素名称 | 必选 | 说明 |
|:--- |:--- |:--- |
| parameterName |是 |参数的名称。必须是有效的 JavaScript 标识符。 |
| type |是 |参数值的类型。请参阅以下允许类型的列表。 |
| defaultValue |否 |参数的默认值，如果没有为参数提供任何值。 |
| allowedValues |否 |用来确保提供正确值的参数的允许值数组。 |
| minValue |否 |int 类型参数的最小值，此值是包容性的。 |
| maxValue |否 |int 类型参数的最大值，此值是包容性的。 |
| minLength |否 |字符串、secureString 和数组类型参数的最小长度（含）。 |
| maxLength |否 |字符串、secureString 和数组类型参数的最大长度（含）。 |
| description |否 |通过门户自定义模板界面向模板用户显示的参数的描述。 |

允许的类型和值是：

* **字符串**
* **secureString**
* **int**
* **bool**
* **对象**
* **secureObject**
* **数组**

若要将某个参数指定为可选，请提供 defaultValue（可以为空字符串）。

如果指定的参数名称与部署模板的命令中的某个参数匹配，系统会提示用户为后缀为 **FromTemplate** 的参数提供值。例如，如果在模板中提供的名为 **ResourceGroupName** 的参数与 [New-AzureRmResourceGroupDeployment][deployment2cmdlet] cmdlet 中的 **ResourceGroupName** 参数相同，系统会提示用户提供 **ResourceGroupNameFromTemplate** 的值。通常，不应将参数命名为与用于部署操作的参数的名称相同以避免这种混乱。

> [AZURE.NOTE]
所有密码、密钥和其他机密信息应使用 **secureString** 类型。若要将敏感数据传入 JSON 对象，请使用 **secureObject** 类型。部署资源后，无法读取 secureString 或 secureObject 类型的模板参数。
><p> 
> 例如，部署历史记录中的以下条目将显示字符串和对象的值，但不会显示 secureString 和 secureObject 的值。
><p>
> ![显示部署值](./media/resource-group-authoring-templates/show-parameters.png)  

>

以下示例演示如何定义参数：

    "parameters": {
      "siteName": {
        "type": "string",
        "defaultValue": "[concat('site', uniqueString(resourceGroup().id))]"
      },
      "hostingPlanName": {
        "type": "string",
        "defaultValue": "[concat(parameters('siteName'),'-plan')]"
      },
      "skuName": {
        "type": "string",
        "defaultValue": "F1",
        "allowedValues": [
          "F1",
          "D1",
          "B1",
          "B2",
          "B3",
          "S1",
          "S2",
          "S3",
          "P1",
          "P2",
          "P3",
          "P4"
        ]
      },
      "skuCapacity": {
        "type": "int",
        "defaultValue": 1,
        "minValue": 1
      }
    }

若要了解如何在部署过程中输入参数值，请参阅 [Deploy an application with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署应用程序）。

## <a name="variables"></a> 变量
在 variables 节中构造可在整个模板中使用的值。通常，变量基于通过参数提供的值。不需要定义变量，但使用变量可以减少复杂的表达式，从而简化模板。

使用以下结构定义变量：

    "variables": {
       "<variable-name>": "<variable-value>",
       "<variable-name>": { 
           <variable-complex-type-value> 
       }
    }

以下示例演示如何定义从两个参数值构造出的变量：

    "variables": {
        "connectionString": "[concat('Name=', parameters('username'), ';Password=', parameters('password'))]"
    }

下一个示例演示一个属于复杂的 JSON 类型的变量，以及从其他变量构造出的变量：

    "parameters": {
       "environmentName": {
         "type": "string",
         "allowedValues": [
           "test",
           "prod"
         ]
       }
    },
    "variables": {
       "environmentSettings": {
         "test": {
           "instancesSize": "Small",
           "instancesCount": 1
         },
         "prod": {
           "instancesSize": "Large",
           "instancesCount": 4
         }
       },
       "currentEnvironmentSettings": "[variables('environmentSettings')[parameters('environmentName')]]",
       "instancesSize": "[variables('currentEnvironmentSettings').instancesSize]",
       "instancesCount": "[variables('currentEnvironmentSettings').instancesCount]"
    }

## <a name="resources"></a> 资源
在 resources 节，可以定义部署或更新的资源。此节可能比较复杂，因为用户必须了解要部署哪些类型才能提供正确的值。

使用以下结构定义资源：

    "resources": [
       {
         "apiVersion": "<api-version-of-resource>",
         "type": "<resource-provider-namespace/resource-type-name>",
         "name": "<name-of-the-resource>",
         "location": "<location-of-resource>",
         "tags": "<name-value-pairs-for-resource-tagging>",
         "comments": "<your-reference-notes>",
         "dependsOn": [
           "<array-of-related-resource-names>"
         ],
         "properties": "<settings-for-the-resource>",
         "copy": {
           "name": "<name-of-copy-loop>",
           "count": "<number-of-iterations>"
         }
         "resources": [
           "<array-of-child-resources>"
         ]
       }
    ]

| 元素名称 | 必选 | 说明 |
|:--- |:--- |:--- |
| apiVersion |是 |用于创建资源的 REST API 版本。 |
| type |是 |资源的类型。此值是资源提供程序的命名空间以及资源类型的组合（例如 **Microsoft.Storage/storageAccounts**）。 |
| 名称 |是 |资源的名称。该名称必须遵循 RFC3986 中定义的 URI 构成部分限制。此外，向第三方公开资源名称的 Azure 服务将验证名称，以确保它不是尝试窃取另一个身份。请参阅[检查资源名称](https://msdn.microsoft.com/zh-cn/library/azure/mt219035.aspx)。 |
| location |多种多样 |提供的资源支持的地理位置。可以选择任何可用位置，但通常选取靠近用户的位置。通常还会将彼此交互的资源置于同一区域。大多数资源类型需要一个位置，但某些类型 （如角色分配）不需要位置。 |
| 标记 |否 |与资源关联的标记。 |
| 注释 |否 |用于描述模板中资源的注释 |
| dependsOn |否 |部署此资源之前必须部署的资源。Resource Manager 将评估资源之间的依赖关系，并按正确的顺序部署资源。如果资源不相互依赖，则可并行部署资源。该值可以是资源名称或资源唯一标识符的逗号分隔列表。在此模板中仅部署列出的资源。此模板中未定义的资源必须已存在。有关详细信息，请参阅[在 Azure 资源管理器模板中定义依赖关系](/documentation/articles/resource-group-define-dependencies/)。 |
| 属性 |否 |特定于资源的配置设置。properties 的值与用户在创建资源时，在 REST API 操作（PUT 方法）的请求正文中提供的值相同。有关资源架构文档或 REST API 的链接，请参阅 [Resource Manager providers, regions, API versions and schemas](/documentation/articles/resource-manager-supported-services/)（Resource Manager 提供程序、区域、API 版本和架构）。 |
| 复制 |否 |如果需要多个实例，则为要创建的资源数。有关详细信息，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。 |
| 资源 |否 |依赖于所定义的资源的子资源。只能提供父资源的架构允许的资源类型。子资源类型的完全限定名称包含父资源类型，例如 **Microsoft.Web/sites/extensions**。对父资源的依赖性不是隐式的；你必须显式定义该依赖性。 |

了解哪些值可以指定，因为 **apiVersion**、**type** 和 **location** 不会立即显示。幸运的是，可以通过 Azure PowerShell 或 Azure CLI 确定这些值。

若要通过 **PowerShell** 获取所有资源提供程序，请使用：

    Get-AzureRmResourceProvider -ListAvailable

从返回的列表中，找到感兴趣的资源提供程序。若要获取资源提供程序（如存储）的资源类型，请使用：

    (Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Storage).ResourceTypes

若要获取资源类型（如存储帐户）的 API 版本，请使用：

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Storage).ResourceTypes | Where-Object ResourceTypeName -eq storageAccounts).ApiVersions

若要获取资源类型支持的位置，请使用：

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Storage).ResourceTypes | Where-Object ResourceTypeName -eq storageAccounts).Locations

若要通过 **Azure CLI** 获取所有资源提供程序，请使用：

    azure provider list

从返回的列表中，找到感兴趣的资源提供程序。若要获取资源提供程序（如存储）的资源类型，请使用：

    azure provider show Microsoft.Storage

若要获取支持的位置和 API 版本，请使用：

    azure provider show Microsoft.Storage --details --json

若要详细了解资源提供程序，请参阅 [Resource Manager 提供程序、区域、API 版本和架构](/documentation/articles/resource-manager-supported-services/)。

resources 节包含要部署的资源数组。在每个资源内，还可以定义子资源数组。因此，resources 节的结构可能类似于：

    "resources": [
       {
           "name": "resourceA",
       },
       {
           "name": "resourceB",
           "resources": [
               {
                   "name": "firstChildResourceB",
               },
               {   
                   "name": "secondChildResourceB",
               }
           ]
       },
       {
           "name": "resourceC",
       }
    ]

以下示例演示了 **Microsoft.Web/serverfarms** 资源，以及一个包含 **Extensions** 子资源的 **Microsoft.Web/sites** 资源。请注意，站点标记为依赖于服务器场，因为只有该服务器场存在，才能部署该站点。另请注意，**Extensions** 资源是站点的子级。

    "resources": [
      {
        "apiVersion": "2015-08-01",
        "name": "[parameters('hostingPlanName')]",
        "type": "Microsoft.Web/serverfarms",
        "location": "[resourceGroup().location]",
        "tags": {
          "displayName": "HostingPlan"
        },
        "sku": {
          "name": "[parameters('skuName')]",
          "capacity": "[parameters('skuCapacity')]"
        },
        "properties": {
          "name": "[parameters('hostingPlanName')]",
          "numberOfWorkers": 1
        }
      },
      {
        "apiVersion": "2015-08-01",
        "type": "Microsoft.Web/sites",
        "name": "[parameters('siteName')]",
        "location": "[resourceGroup().location]",
        "tags": {
          "environment": "test",
          "team": "Web"
        },
        "dependsOn": [
          "[concat(parameters('hostingPlanName'))]"
        ],
        "properties": {
          "name": "[parameters('siteName')]",
          "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
        },
        "resources": [
          {
            "apiVersion": "2015-08-01",
            "type": "extensions",
            "name": "MSDeploy",
            "dependsOn": [
              "[concat('Microsoft.Web/sites/', parameters('siteName'))]"
            ],
            "properties": {
              "packageUri": "https://auxmktplceprod.blob.core.windows.net/packages/StarterSite-modified.zip",
              "dbType": "None",
              "connectionString": "",
              "setParameters": {
                "Application Path": "[parameters('siteName')]"
              }
            }
          }
        ]
      }
    ]

## Outputs
在 Outputs 节中，可以指定从部署返回的值。例如，可能会返回用于访问已部署资源的 URI。

以下示例演示了输出定义的结构：

    "outputs": {
       "<outputName>" : {
         "type" : "<type-of-output-value>",
         "value": "<output-value-expression>"
       }
    }

| 元素名称 | 必选 | 说明 |
|:--- |:--- |:--- |
| outputName |是 |输出值的名称。必须是有效的 JavaScript 标识符。 |
| type |是 |输出值的类型。输出值支持的类型与模板输入参数相同。 |
| value |是 |要求值并作为输出值返回的模板语言表达式。 |

以下示例演示了 Outputs 节中返回的值。

    "outputs": {
       "siteUri" : {
         "type" : "string",
         "value": "[concat('http://',reference(resourceId('Microsoft.Web/sites', parameters('siteName'))).hostNames[0])]"
       }
    }

有关如何处理输出的详细信息，请参阅 [Sharing state in Azure Resource Manager templates](/documentation/articles/best-practices-resource-manager-state/)（在 Azure Resource Manager 模板中共享状态）。

## 后续步骤
* 若要查看许多不同类型的解决方案的完整模型，请参阅 [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates/)（Azure 快速入门模板）。
* 有关用户可以使用的来自模板中的函数的详细信息，请参阅 [Azure Resource Manager Template Functions](/documentation/articles/resource-group-template-functions/)（Azure Resource Manager 模板函数）。
* 若要在部署期间合并多个模板，请参阅 [Using linked templates with Azure Resource Manager](/documentation/articles/resource-group-linked-templates/)（将已链接的模板与 Azure Resource Manager 配合使用）。
* 你可能需要使用不同资源组中的资源。使用跨多个资源组共享的存储帐户或虚拟网络时，此方案很常见。有关详细信息，请参阅 [resourceId 函数](/documentation/articles/resource-group-template-functions/#resourceid)。

[deployment2cmdlet]: https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.2.0/new-azurermresourcegroupdeployment

<!---HONumber=Mooncake_0103_2017-->