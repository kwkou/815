<properties
   pageTitle="创建多个资源实例"
   description="介绍如何在部署资源时使用 Azure 资源管理器模板中的复制操作执行多次迭代。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/14/2015"
   wa.date="10/3/2015"/>

# 在 Azure 资源管理器中创建多个资源实例

本主题演示如何在您的 Azure 资源管理器模板中进行迭代操作，以创建多个资源实例。

## copy 和 copyIndex()

在要多次创建的资源中，您可以定义指定迭代次数的 **copy** 对象。copy 将采用以下格式：

    "copy": { 
        "name": "websitescopy", 
        "count": "[parameters('count')]" 
    } 

您可以使用 **copyIndex()** 函数访问当前的迭代值，如以下 concat 函数中所示。

    [concat('examplecopy-', copyIndex())]

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
          "apiVersion": "2014-06-01",
          "copy": { 
             "name": "websitescopy", 
             "count": "[parameters('count')]" 
          }, 
          "properties": {} 
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
      },
      "count": { 
         "type": "int", 
         "defaultValue": 3 
      } 
    }, 
    "resources": [ 
      { 
          "name": "[concat('examplecopy-', parameters('org')[copyIndex()])]", 
          "type": "Microsoft.Web/sites", 
          "location": "East US", 
          "apiVersion": "2014-06-01",
          "copy": { 
             "name": "websitescopy", 
             "count": "[parameters('count')]" 
          }, 
          "properties": {} 
      } 
    ]

## 后续步骤
- [创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)
- [Azure 资源管理器模板函数](/documentation/articles/resource-group-template-functions)
- [使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy)

<!---HONumber=71-->