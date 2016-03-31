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
   ms.date="12/01/2015"
   wacn.date="01/14/2016"/>

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

    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "[variables('dataFactoryName')]",
        ...
        "resources": [
        {
            "type": "datasets",
            "name": "[variables('dataSetName')]",
            "dependsOn": [
                "[variables('dataFactoryName')]"
            ],
            ...
        }
    }]
    
若要创建数据集的多个实例，你需要按如下所示更改模板。请注意完全限定的类型，名称包括数据工厂名称。

    "resources": [
    {
        "type": "Microsoft.DataFactory/datafactories",
        "name": "[variables('dataFactoryName')]",
        ...
    },
    {
        "type": "Microsoft.DataFactory/datafactories/datasets",
        "name": "[concat(variables('dataFactoryName'), '/', variables('dataSetName'), copyIndex())]",
        "dependsOn": [
            "[variables('dataFactoryName')]"
        ],
        "copy": { 
            "name": "datasetcopy", 
            "count": "[parameters('count')]" 
        } 
        ...
    }]

## 后续步骤
- 若要了解有关模板区段的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/./resource-group-authoring-templates)。
- 有关可在模板中使用的函数列表，请参阅 [Azure 资源管理器模板函数](/documentation/articles/./resource-group-template-functions)
- 若要了解如何部署模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy)。

<!---HONumber=Mooncake_0104_2016-->