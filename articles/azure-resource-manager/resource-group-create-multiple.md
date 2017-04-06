<properties
    pageTitle="部署多个 Azure 资源实例 | Azure"
    description="在部署资源时使用 Azure 资源管理器模板中的复制操作和数组执行多次迭代。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="94d95810-a87b-460f-8e82-c69d462ac3ca"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/24/2017"
    wacn.date="03/31/2017"
    ms.author="tomfitz" />  

# 在 Azure Resource Manager 模板中部署多个资源实例
本主题演示如何在您的 Azure 资源管理器模板中进行迭代操作，以创建多个资源实例。

## copy、copyIndex 和 length
在要多次创建的资源中，您可以定义指定迭代次数的 **copy** 对象。copy 将采用以下格式：

    "copy": { 
        "name": "websitescopy", 
        "count": "[parameters('count')]" 
    } 

可以使用 **copyIndex()** 函数访问当前的迭代值。以下示例将 copyIndex 与 concat 函数配合使用来构造名称。

    [concat('examplecopy-', copyIndex())]

基于值数组创建多个资源时，可以使用 **length** 函数指定计数。请提供该数组作为 length 函数的参数。

    "copy": {
        "name": "websitescopy",
        "count": "[length(parameters('siteNames'))]"
    }

只能将 copy 对象应用于顶级资源，而不能应用于资源类型的属性或子资源。但是，本主题会介绍如何为属性指定多个项，以及如何创建子资源的多个实例。以下伪代码示例演示了可以在何处应用 copy：

    "resources": [
      {
        "type": "{provider-namespace-and-type}",
        "name": "parentResource",
        "copy": {  
          /* yes, copy can be applied here */
        },
        "properties": {
          "exampleProperty": {
            /* no, copy cannot be applied here */
          }
        },
        "resources": [
          {
            "type": "{provider-type}",
            "name": "childResource",
            /* copy can be applied if resource is promoted to top level */ 
          }
        ]
      }
    ] 

虽然不能将 **copy** 应用于属性，但该属性仍是所属资源的迭代的一部分。因此，可在属性内使用 **copyIndex()** 指定值。

多种情况下可能需要针对资源中的属性进行迭代操作。例如，可能需要为虚拟机指定多个数据磁盘。若要了解如何针对属性进行迭代操作，请参阅[当复制不可用时创建多个实例](#create-multiple-instances-when-copy-wont-work)。

若要使用子资源，请参阅[创建子资源的多个实例](#create-multiple-instances-of-a-child-resource)。

## 使用名称中的索引值
可以使用复制操作基于递增索引创建具有唯一名称的多个资源实例。例如，您可能想要将唯一编号添加到每个已部署的资源名称末尾。部署具有以下名称的三个网站：

* examplecopy-0
* examplecopy-1
* examplecopy-2

请使用以下模版：

    "parameters": { 
      "count": { 
        "type": "int", 
        "defaultValue": 3 
      } 
    }, 
    "resources": [ 
      { 
          "name": "[concat('examplecopy-', copyIndex())]", 
          "type": "Microsoft.Web/sites", 
          "location": "China East", 
          "apiVersion": "2015-08-01",
          "copy": { 
             "name": "websitescopy", 
             "count": "[parameters('count')]" 
          }, 
          "properties": {
              "serverFarmId": "hostingPlanName"
          }
      } 
    ]

## 偏移索引值
在前面的示例中，索引值范围为 0 到 2。若要偏移索引值，可以将某个值传递到 **copyIndex()** 函数中，如 **copyIndex(1)**。要执行的迭代次数仍被指定在 copy 元素中，但 copyIndex 的值已按指定的值发生了偏移。因此，使用前面的示例中的同一模板，但指定 **copyIndex(1)** 会部署具有以下名称的三个网站：

* examplecopy-1
* examplecopy-2
* examplecopy-3

## 对数组使用复制
处理数组时可以使用复制操作，因为可对数组中的每个元素执行迭代操作。部署具有以下名称的三个网站：

* examplecopy-Contoso
* examplecopy-Fabrikam
* examplecopy-Coho

请使用以下模版：

    "parameters": { 
      "org": { 
         "type": "array", 
         "defaultValue": [ 
             "Contoso", 
             "Fabrikam", 
             "Coho" 
          ] 
      }
    }, 
    "resources": [ 
      { 
          "name": "[concat('examplecopy-', parameters('org')[copyIndex()])]", 
          "type": "Microsoft.Web/sites", 
          "location": "China East", 
          "apiVersion": "2015-08-01",
          "copy": { 
             "name": "websitescopy", 
             "count": "[length(parameters('org'))]" 
          }, 
          "properties": {
              "serverFarmId": "hostingPlanName"
          } 
      } 
    ]

当然，不能将复制计数设置为数组的长度。例如，你可能要创建一个包含多个值的数组，然后传入一个参数值用于指定要部署多少个数组元素。在这种情况下，请按第一个示例中所示设置复制计数。

## 依赖于循环中的资源
可以使用 **dependsOn** 元素指定部署一个资源后再部署另一个资源。若要部署的资源依赖于循环中的资源集合，可在 **dependsOn** 元素中提供 copy 循环的名称。以下示例演示了如何在部署虚拟机之前部署三个存储帐户。此处并未显示完整的虚拟机定义。请注意，copy 元素的 **name** 设置为 **storagecopy**，而虚拟机的 **dependsOn** 元素也设置为 **storagecopy**。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "resources": [
            {
                "apiVersion": "2015-06-15",
                "type": "Microsoft.Storage/storageAccounts",
                "name": "[concat('storage', uniqueString(resourceGroup().id), copyIndex())]",
                "location": "[resourceGroup().location]",
                "properties": {
                    "accountType": "Standard_LRS"
                },
                "copy": { 
                    "name": "storagecopy", 
                    "count": 3 
                }
            },
            {
                "apiVersion": "2015-06-15", 
                "type": "Microsoft.Compute/virtualMachines", 
                "name": "[concat('VM', uniqueString(resourceGroup().id))]",  
                "dependsOn": ["storagecopy"],
                ...
            }
        ],
        "outputs": {}
    }

## <a name="create-multiple-instances-of-a-child-resource"></a> 创建子资源的多个实例
不能对子资源使用 copy 循环。若要创建子资源的多个实例，而该子资源通常在其他资源中定义为嵌套资源，则必须将该资源创建为顶级资源。可通过 **type** 和 **name** 属性定义与父资源的关系。

例如，假设用户通常会将某个数据集定义为数据工厂中的子资源。

    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "exampleDataFactory",
        ...
        "resources": [
        {
            "type": "datasets",
            "name": "exampleDataSet",
            "dependsOn": [
                "exampleDataFactory"
            ],
            ...
        }
    }]

若要创建数据集的多个实例，请将数据集移出数据工厂。数据集必须与数据工厂位于同一层级，但仍属数据工厂的子资源。可以通过 **type** 和 **name** 属性保留数据集和数据工厂之间的关系。由于类型不再可以从其在模板中的位置推断，因此必须按以下格式提供完全限定的类型：`{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`。

若要与数据工厂的实例建立父/子关系，提供的数据集的名称应包含父资源名称。使用以下格式：`{parent-resource-name}/{child-resource-name}`。

以下示例演示实现过程：

    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "exampleDataFactory",
        ...
    },
    {
        "type": "Microsoft.DataFactory/datafactories/datasets",
        "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
        "dependsOn": [
            "exampleDataFactory"
        ],
        "copy": { 
            "name": "datasetcopy", 
            "count": "3" 
        } 
        ...
    }]

## <a name="create-multiple-instances-when-copy-wont-work"></a> 当复制不可用时创建多个实例
只能对资源类型，而不能对资源类型中的属性使用 **copy**。需要创建属于资源一部分的某事物的多个实例时，此要求可能会造成问题。一种常见的情况是为一个虚拟机创建多个数据磁盘。无法对数据磁盘使用 **copy**，因为 **dataDisks** 是虚拟机的一个属性，而不是它的资源类型。可以根据需要使用足够多的数据磁盘创建数组，并传递要创建的数据磁盘的实际数量。在虚拟机定义中，使用 **take** 函数从数组中仅获取实际需要的元素数。

[使用动态选择的数据磁盘创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-dynamic-data-disks-selection/) 模板中显示了此模式的完整示例。

部署模板的相关节如以下示例所示。已删除模板中的大多数节，以便突出显示动态创建多个数据磁盘时所涉及的节。请注意参数 **numDataDisks**，用于传入要创建的磁盘数目。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        ...
        "numDataDisks": {
          "type": "int",
          "maxValue": 64,
          "metadata": {
            "description": "This parameter allows you to select the number of disks you want"
          }
        }
      },
      "variables": {
        "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'dynamicdisk')]",
        "sizeOfDataDisksInGB": 100,
        "diskCaching": "ReadWrite",
        "diskArray": [
          {
            "name": "datadisk1",
            "lun": 0,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk1.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          },
          {
            "name": "datadisk2",
            "lun": 1,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk2.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          },
          {
            "name": "datadisk3",
            "lun": 2,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk3.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          },
          {
            "name": "datadisk4",
            "lun": 3,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk4.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          },
          ...
          {
            "name": "datadisk63",
            "lun": 62,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk63.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          },
          {
            "name": "datadisk64",
            "lun": 63,
            "vhd": {
              "uri": "[concat('http://', variables('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', 'datadisk64.vhd')]"
            },
            "createOption": "Empty",
            "caching": "[variables('diskCaching')]",
            "diskSizeGB": "[variables('sizeOfDataDisksInGB')]"
          }
        ]
      },
      "resources": [
        ...
        {
          "type": "Microsoft.Compute/virtualMachines",
          "properties": {
            ...
            "storageProfile": {
              ...
              "dataDisks": "[take(variables('diskArray'),parameters('numDataDisks'))]"
            },
            ...
          }
          ...
        }
      ]
    }

需要使用属性的可变数目项创建资源的多个实例时，可以联合使用 **take** 函数和 **copy** 元素。例如，假设用户需创建多个虚拟机，但每个虚拟机的数据磁盘数不同。若要为每个数据磁盘提供一个名称来标识关联的虚拟机，可将数据磁盘阵列置于单独的模板中。包括虚拟机名称以及要返回的数据磁盘数等参数。在输出部分，返回指定项的数目。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "vmName": {
          "type": "string"
        },
        "storageAccountName": {
          "type": "string"
        },
        "numDataDisks": {
          "type": "int",
          "maxValue": 16,
          "metadata": {
            "description": "This parameter allows the user to select the number of disks they want"
          }
        }
      },
      "variables": {
        "diskArray": [
          {
            "name": "[concat(parameters('vmName'), '-datadisk1')]",
            "vhd": {
              "uri": "[concat('http://', parameters('storageAccountName'),'.blob.core.chinacloudapi.cn/vhds/', parameters('vmName'), '-datadisk1.vhd')]"
            },
            ...
          },
          ...
        ],
      },
      "resources": [
      ],
      "outputs": {
        "result": {
          "type": "array",
          "value": "[take(variables('diskArray'),parameters('numDataDisks'))]"
        }
      }
    }

在父模板中，包括虚拟机数目参数，以及针对每个虚拟机的数据磁盘数的数组。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        ...
        "numberOfInstances": {
          "type": "int",
          "defaultValue": 2,
          "metadata": {
            "description": "Number of VMs to deploy"
          }
        },
        "numberOfDataDisksPerVM": {
          "type": "array",
          "defaultValue": [1,2]
        }
      },

在资源部分，部署定义数据磁盘的模板的多个实例。

    {
      "apiVersion": "2016-09-01",
      "name": "[concat('nested-', copyIndex())]",
      "type": "Microsoft.Resources/deployments",
      "copy": {
        "name": "deploycopy",
        "count": "[parameters('numberOfInstances')]"
      },
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "{data-disk-template-uri}",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "vmName": { "value": "[concat('myvm', copyIndex())]" },
          "storageAccountName": { "value": "[variables('storageAccountName')]" },
          "numDataDisks": { "value": "[parameters('numberOfDataDisksPerVM')[copyIndex()]]" }
        }
      }
    },

在资源部分，部署虚拟机的多个实例。对于数据磁盘，可引用嵌套式部署，其中包含数据磁盘的正确数目和正确名称。

    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[concat('myvm', copyIndex())]",
      "copy": {
        "name": "virtualMachineLoop",
          "count": "[parameters('numberOfInstances')]"
      },
      "properties": {
        "storageProfile": {
          ...
          "dataDisks": "[reference(concat('nested-', copyIndex())).outputs.result.value]"
        },
        ...
      },
      ...
    }

## 从循环返回值
虽然创建某个资源类型的多个实例很方便，但从该循环返回值可能很困难。若要保留和返回值，一种方式是对嵌套式模板使用 **copy**，对包含所有待返回值的数组执行往返操作。例如，假设用户需要创建多个存储帐户，并返回每个帐户的主终结点。

首先，创建用于创建存储帐户的嵌套式模板。请注意，它接受的数组参数适用于 Blob URI。可以使用此参数对此前部署中的所有值执行往返操作。模板的输出是一个数组，该数组将新的 Blob URI 连接到旧的 URI。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "indexValue": {
          "type":"int"
        },
        "blobURIs": {
            "type": "array",
          "defaultValue": []
        }
      },
        "variables": {
        "storageName": "[concat('storage', uniqueString(resourceGroup().id), parameters('indexValue'))]"
      },
        "resources": [
        {
            "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[variables('storageName')]",
          "location": "[resourceGroup().location]",
          "sku": {
              "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": {  
          }
        }
        ],
        "outputs": {
          "result": {
            "type": "array",
          "value": "[concat(parameters('blobURIs'),split(reference(variables('storageName')).primaryEndpoints.blob, ','))]"
        }
      }
    }

现在可创建父模板，其中包含嵌套式模板的一个静态实例，并可对嵌套式模板的剩余实例执行循环操作。对于循环部署的每个实例，均可传递一个数组，该数组是旧部署的输出。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "numberofStorage": { "type": "int", "minValue": 2 }
      },
      "resources": [
        {
          "apiVersion": "2016-09-01",
          "name": "nestedTemplate0",
          "type": "Microsoft.Resources/deployments",
          "properties": {
            "mode": "incremental",
            "templateLink": {
              "uri": "{storage-template-uri}",
              "contentVersion": "1.0.0.0"
            },
            "parameters": {
              "indexValue": {"value": 0}
            }
          }
        },
        {
          "apiVersion": "2016-09-01",
          "name": "[concat('nestedTemplate', copyIndex(1))]",
          "type": "Microsoft.Resources/deployments",
          "copy": {
            "name": "storagecopy",
            "count": "[sub(parameters('numberofStorage'), 1)]"
          },
          "properties": {
            "mode": "incremental",
            "templateLink": {
              "uri": "{storage-template-uri}",
              "contentVersion": "1.0.0.0"
            },
            "parameters": {
              "indexValue": {"value": "[copyIndex(1)]"},
              "blobURIs": {"value": "[reference(concat('nestedTemplate', copyIndex())).outputs.result.value]"}
            }
          }
        }
      ],
      "outputs": {
        "result": {
          "type": "object",
          "value": "[reference(concat('nestedTemplate', sub(parameters('numberofStorage'), 1))).outputs.result]"
        }
      }
    }

## 后续步骤
* 若要了解有关模板区段的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
* 如需可在模板中使用的所有函数，请参阅 [Azure Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)。
* 若要了解如何部署模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: update meta properties; wording update -->