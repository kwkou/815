<properties
   pageTitle="部署多个资源实例 | Azure"
   description="在部署资源时使用 Azure 资源管理器模板中的复制操作和数组执行多次迭代。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/09/2016"
   wacn.date="04/18/2016"/>

# 在 Azure 资源管理器中创建多个资源实例

本主题演示如何在您的 Azure 资源管理器模板中进行迭代操作，以创建多个资源实例。

## copy、copyIndex 和 length

在要多次创建的资源中，您可以定义指定迭代次数的 **copy** 对象。copy 将采用以下格式：

    "copy": { 
        "name": "websitescopy", 
        "count": "[parameters('count')]" 
    } 

您可以使用 **copyIndex()** 函数访问当前的迭代值，如以下 concat 函数中所示。

    [concat('examplecopy-', copyIndex())]

基于值数组创建多个资源时，可以使用 **length** 函数指定计数。请提供该数组作为 length 函数的参数。

    "copy": {
        "name": "websitescopy",
        "count": "[length(parameters('siteNames'))]"
    }

## 使用名称中的索引值

可以使用复制操作基于递增索引创建具有唯一名称的多个资源实例。例如，您可能想要将唯一编号添加到每个已部署的资源名称末尾。部署具有以下名称的三个网站：

- examplecopy-0
- examplecopy-1
- examplecopy-2。

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
          "location": "East US", 
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

您会在前面的示例中注意到索引值为从 0 到 2。若要偏移索引值，可以将某个值传递到 **copyIndex()** 函数中，如 **copyIndex(1)**。要执行的迭代次数仍被指定在 copy 元素中，但 copyIndex 的值已按指定的值发生了偏移。因此，使用前面的示例中的同一模板，但指定 **copyIndex(1)** 会部署具有以下名称的三个网站：

- examplecopy-1
- examplecopy-2
- examplecopy-3

## 与数组一起使用
   
当使用数组时，复制操作十分有用，因为可迭代数组中的每个元素。部署具有以下名称的三个网站：

- examplecopy-Contoso
- examplecopy-Fabrikam
- examplecopy-Coho

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
          "location": "East US", 
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

当然，你不能将复制计数设置为数组的长度。例如，你可能要创建一个包含多个值的数组，然后传入一个参数值用于指定要部署多少个数组元素。在这种情况下，请按第一个示例中所示设置复制计数。

## 依赖于循环中的资源

你可以使用 **dependsOn** 元素来指定部署一个资源后再部署另一个资源。如果需要部署的资源依赖于循环中的资源集合，则你可以在 **dependsOn** 元素中提供 copy 循环的名称。以下示例演示了如何在部署虚拟机之前部署 3 个存储帐户。此处并未显示完整的虚拟机定义。请注意，copy 元素的 **name** 设置为 **storagecopy**，而虚拟机的 **dependsOn** 元素也设置为 **storagecopy**。

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

## 对嵌套资源的循环

不能对嵌套资源使用 copy 循环。如果你需要创建某个资源的多个实例，而该资源通常定义为另一个资源中的嵌套资源，那么，你必须将该资源创建为顶级资源，并通过 **type** 和 **name** 属性定义该资源与其父资源的关系。

例如，假设你通常要将某个数据集定义为数据工厂中的嵌套资源。

    "parameters": {
        "dataFactoryName": {
            "type": "string"
         },
         "dataSetName": {
            "type": "string"
         }
    },
    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "[parameters('dataFactoryName')]",
        ...
        "resources": [
        {
            "type": "datasets",
            "name": "[parameters('dataSetName')]",
            "dependsOn": [
                "[parameters('dataFactoryName')]"
            ],
            ...
        }
    }]
    
若要创建数据集的多个实例，你需要按如下所示更改模板。请注意完全限定的类型，名称包括数据工厂名称。

    "parameters": {
        "dataFactoryName": {
            "type": "string"
         },
         "dataSetName": {
            "type": "array"
         }
    },
    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "[parameters('dataFactoryName')]",
        ...
    },
    {
        "type": "Microsoft.DataFactory/datafactories/datasets",
        "name": "[concat(parameters('dataFactoryName'), '/', parameters('dataSetName')[copyIndex()])]",
        "dependsOn": [
            "[parameters('dataFactoryName')]"
        ],
        "copy": { 
            "name": "datasetcopy", 
            "count": "[length(parameters('dataSetName'))]" 
        } 
        ...
    }]

## 为一个虚拟机创建多个数据磁盘

一种常见的情况是为一个虚拟机创建多个数据磁盘。你不能将 **copy** 与数据磁盘配合使用，因为 **dataDisks** 是虚拟机上的属性，而不是它自己的资源类型。**copy** 只能配合资源使用。你可以创建链接模板，在其中定义所需的数据磁盘数目，然后输出具有指定数据磁盘数目的数组。你可以从部署模板链接到此模板，并传入要返回的数据磁盘数目。在部署模板中，将 **dataDisks** 属性设置为来自链接模板的输出值。

[使用动态选择的数据磁盘创建 VM](https://azure.microsoft.com/documentation/templates/201-vm-dynamic-data-disks-selection/) 模板中显示了此模式的完整示例。

部署模板的相关节如下所示。已删除模板中的大多数节，以便突出显示动态创建多个数据磁盘时所涉及的节。请注意参数 **numDataDisks**，它可让你传入要创建的磁盘数目。名为 **diskSelection** 的链接模板在 resources 节中指定。该链接模板的输出将分配到虚拟机上的 **dataDisks** 属性。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        ...
        "numDataDisks": {
          "type": "string",
          "allowedValues": [
            "1",
            "2",
            "3",
            "4",
            "5",
            "6",
            "7",
            "8",
            "9",
            "10",
            "11",
            "12",
            "13",
            "14",
            "15",
            "16",
            "32"
          ],
          "metadata": {
            "description": "This parameter allows the user to select the number of disks they want"
          }
        }
      },
      "variables": {
        "artifactsLocation": "https://raw.githubusercontent.com/singhkay/azure-quickstart-templates/master/201-vm-dynamic-data-disks-selection/", 
        "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'dynamicdisk')]",
        "sizeOfDataDisksInGB": 100,
        "diskCaching": "ReadWrite",
        ...
      },
      "resources": [
        {
          "apiVersion": "2015-01-01",
          "name": "diskSelection",
          "type": "Microsoft.Resources/deployments",
          "properties": {
            "mode": "Incremental",
            "templateLink": {
              "uri": "[concat(variables('artifactsLocation'), 'disksSelector', '.json')]",
              "contentVersion": "1.0.0.0"
            },
            "parameters": {
              "numDataDisks": {
                "value": "[parameters('numDataDisks')]"
              },
              "diskStorageAccountName": {
                "value": "[variables('storageAccountName')]"
              },
              "diskCaching": {
                "value": "[variables('diskCaching')]"
              },
              "diskSizeGB": {
                "value": "[variables('sizeOfDataDisksInGB')]"
              }
            }
          }
        },
        ...
        {
          "type": "Microsoft.Compute/virtualMachines",
          "properties": {
            "storageProfile": {
              ...
              "dataDisks": "[reference('diskSelection').outputs.dataDiskArray.value]"
            },
            ...
          }
        }
      ]
    }

链接的模板定义要返回的数组。下面所示的模板省略了介于 3 和 32 之间的重复磁盘定义，但在实际模板中需要包含所有这些定义。如果你需要 32 个以上的数据磁盘，可以继续使用此模式。请注意，不会通过此模板部署任何资源；它只返回一个数组，其中包含定义数据磁盘的对象的请求数目。

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "numDataDisks": {
      "type": "string",
      "allowedValues": [
        "1",
        "2",
        "3",
        "4",
        "5",
        "6",
        "7",
        "8",
        "9",
        "10",
        "11",
        "12",
        "13",
        "14",
        "15",
        "16",
        "32"
      ],
      "metadata": {
        "description": "This parameter allows the user to select the number of disks they want"
      }
    },
    "diskStorageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the storage account where the data disks are stored"
      }
    },
    "diskCaching": {
      "type": "string",
      "allowedValues": [
        "None",
        "ReadOnly",
        "ReadWrite"
      ],
      "metadata": {
        "description": "Caching type for the data disks"
      }
    },
    "diskSizeGB": {
      "type": "int",
      "minValue": 1,
      "maxValue": 1023,
      "metadata": {
        "description": "Size of the data disks"
      }
    }
  },
  "variables": {
    "disksArray": {
      "1": "[variables('dataDisks')['1']]",
      "2": "[concat(variables('dataDisks')['1'], variables('dataDisks')['2'])]",
      "3": "[concat(variables('dataDisks')['1'], variables('dataDisks')['2'], variables('dataDisks')['3'])]",
      "4": "[variables('diskDeltas')['4delta']]",
      "5": "[concat(variables('diskDeltas')['4delta'], variables('dataDisks')['5'])]",
      "6": "[concat(variables('diskDeltas')['4delta'], variables('dataDisks')['5'], variables('dataDisks')['6'])]",
      "7": "[concat(variables('diskDeltas')['4delta'], variables('dataDisks')['5'], variables('dataDisks')['6'], variables('dataDisks')['7'])]",
      "8": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'])]",
      "9": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('dataDisks')['9'])]",
      "10": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('dataDisks')['9'], variables('dataDisks')['10'])]",
      "11": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('dataDisks')['9'], variables('dataDisks')['10'], variables('dataDisks')['11'])]",
      "12": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'])]",
      "13": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'], variables('dataDisks')['13'])]",
      "14": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'], variables('dataDisks')['13'], variables('dataDisks')['14'])]",
      "15": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'], variables('dataDisks')['13'], variables('dataDisks')['14'], variables('dataDisks')['15'])]",
      "16": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'], variables('diskDeltas')['16delta'])]",
      "32": "[concat(variables('diskDeltas')['4delta'], variables('diskDeltas')['8delta'], variables('diskDeltas')['12delta'], variables('diskDeltas')['16delta'], variables('diskDeltas')['32delta'])]"
    },
    "dataDisks": {
      "1": [
        {
          "name": "datadisk1",
          "lun": 0,
          "vhd": {
            "uri": "[concat('http://', parameters('diskStorageAccountName'),'.blob.core.windows.net/vhds/', 'datadisk1.vhd')]"
          },
          "createOption": "Empty",
          "caching": "[parameters('diskCaching')]",
          "diskSizeGB": "[parameters('diskSizeGB')]"
        }
      ],
      "2": [
        {
          "name": "datadisk2",
          "lun": 1,
          "vhd": {
            "uri": "[concat('http://', parameters('diskStorageAccountName'),'.blob.core.windows.net/vhds/', 'datadisk2.vhd')]"
          },
          "createOption": "Empty",
          "caching": "[parameters('diskCaching')]",
          "diskSizeGB": "[parameters('diskSizeGB')]"
        }
      ],
      "3": [
        {
          "name": "datadisk3",
          "lun": 2,
          "vhd": {
            "uri": "[concat('http://', parameters('diskStorageAccountName'),'.blob.core.windows.net/vhds/', 'datadisk3.vhd')]"
          },
          "createOption": "Empty",
          "caching": "[parameters('diskCaching')]",
          "diskSizeGB": "[parameters('diskSizeGB')]"
        }
      ],
      ...
      "32": [
        {
          "name": "datadisk32",
          "lun": 31,
          "vhd": {
            "uri": "[concat('http://', parameters('diskStorageAccountName'),'.blob.core.windows.net/vhds/', 'datadisk32.vhd')]"
          },
          "createOption": "Empty",
          "caching": "[parameters('diskCaching')]",
          "diskSizeGB": "[parameters('diskSizeGB')]"
        }
      ]
    },
    "_comment2": "The delta arrays below build the difference from 0 to 4, 4 to 8, 8 to 12 disks and so on",
    "diskDeltas": {
      "4delta": [
        "[variables('dataDisks')['1'][0]]",
        "[variables('dataDisks')['2'][0]]",
        "[variables('dataDisks')['3'][0]]",
        "[variables('dataDisks')['4'][0]]"
      ],
      "8delta": [
        "[variables('dataDisks')['5'][0]]",
        "[variables('dataDisks')['6'][0]]",
        "[variables('dataDisks')['7'][0]]",
        "[variables('dataDisks')['8'][0]]"
      ],
      "12delta": [
        "[variables('dataDisks')['9'][0]]",
        "[variables('dataDisks')['10'][0]]",
        "[variables('dataDisks')['11'][0]]",
        "[variables('dataDisks')['12'][0]]"
      ],
      "16delta": [
        "[variables('dataDisks')['13'][0]]",
        "[variables('dataDisks')['14'][0]]",
        "[variables('dataDisks')['15'][0]]",
        "[variables('dataDisks')['16'][0]]"
      ],
      "32delta": [
        "[variables('dataDisks')['17'][0]]",
        "[variables('dataDisks')['18'][0]]",
        "[variables('dataDisks')['19'][0]]",
        "[variables('dataDisks')['20'][0]]",
        "[variables('dataDisks')['21'][0]]",
        "[variables('dataDisks')['22'][0]]",
        "[variables('dataDisks')['23'][0]]",
        "[variables('dataDisks')['24'][0]]",
        "[variables('dataDisks')['25'][0]]",
        "[variables('dataDisks')['26'][0]]",
        "[variables('dataDisks')['27'][0]]",
        "[variables('dataDisks')['28'][0]]",
        "[variables('dataDisks')['29'][0]]",
        "[variables('dataDisks')['30'][0]]",
        "[variables('dataDisks')['31'][0]]",
        "[variables('dataDisks')['32'][0]]"
      ]
    }
  },
  "resources": [],
  "outputs": {
    "dataDiskArray": {
      "type": "array",
      "value": "[variables('disksArray')[parameters('numDataDisks')]]"
    }
  }
}

```

## 后续步骤
- 若要了解有关模板区段的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
- 有关可在模板中使用的函数列表，请参阅 [Azure 资源管理器模板函数](/documentation/articles/resource-group-template-functions/)
- 若要了解如何部署模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0411_2016-->