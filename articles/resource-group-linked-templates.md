<properties
   pageTitle="将已链接的模版与 Azure 资源管理器配合使用"
   description="介绍如何使用 Azure 资源管理器模板中的链接模板创建一个模块化的模板的解决方案。演示如何传递参数值、指定参数文件和动态创建的 URL。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/15/2015"
   wacn.date="10/3/2015"/>

# 将已链接的模版与 Azure 资源管理器配合使用

从一个 Azure 资源管理器模板中，您可以链接到能使您将部署分解成一组有针对性并且有特定用途的模板的另一个模板。就像将一个应用程序分解为多个代码类那样，分解可提供测试、重用和可读性方面的好处。

可以将参数从主模板传递到链接的模板，并可以直接将这些参数映射到由调用模板公开提供的参数或变量。链接模板还可以将输出变量传递回源模板中，启用模板之间的双向数据交换。

## 链接到模板

通过在主模板内添加部署源，从而在两个模板间创建指向链接模板的链接。对链接模版的 URI 设置 **templateLink** 属性。您可以通过直接在您的模板中指定值或通过链接到参数文件，为链接模板提供参数值。以下示例使用 **parameters** 属性直接指定参数值。

    "resources": [ 
      { 
         "apiVersion": "2015-01-01", 
         "name": "nestedTemplate", 
         "type": "Microsoft.Resources/deployments", 
         "properties": { 
           "mode": "incremental", 
           "templateLink": {
              "uri": "https://www.contoso.com/AzureTemplates/newStorageAccount.json",
              "contentVersion": "1.0.0.0"
           }, 
           "parameters": { 
              "StorageAccountName":{"value": "[parameters('StorageAccountName')]"} 
           } 
         } 
      } 
    ] 

## 链接到参数文件

以下示例使用 **parametersLink** 属性链接到参数文件。

    "resources": [ 
      { 
         "apiVersion": "2015-01-01", 
         "name": "nestedTemplate", 
         "type": "Microsoft.Resources/deployments", 
         "properties": { 
           "mode": "incremental", 
           "templateLink": {
              "uri":"https://www.contoso.com/AzureTemplates/newStorageAccount.json",
              "contentVersion":"1.0.0.0"
           }, 
           "parametersLink": { 
              "uri":"https://www.contoso.com/AzureTemplates/parameters.json",
              "contentVersion":"1.0.0.0"
           } 
         } 
      } 
    ] 

## 使用变量来链接模板

前面的示例演示了用于模板链接的硬编码 URL 值。这种方法可能适用于简单的模板，但如果使用一组大型模块化模板时，将无法正常工作。相反，可以创建一个存储主模板的基 URL 的静态变量，然后从基 URL 动态创建用于链接模板的 URL。这种方法的好处是可以轻松地移动或派生模板，因为您只需在主模板中更改静态变量。主模板将在整个分解后的模板中传递正确的 URI。

以下示例演示如何使用基 URL 来创建两个用于链接模板的 URL（**sharedTemplateUrl** 和 **vmTemplate**）。

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

## 将值传递回链接模板

如果您需要将值从链接模板传递到主模板，则可以在链接模板的**输出**部分创建一个值。有关示例，请参阅[在 Azure 资源管理器模板中共享状态](/documentation/articles/best-practices-resource-manager-state)。

## 后续步骤
- [创作模板](/documentation/articles/resource-group-authoring-templates)
- [部署模板](/documentation/articles/resource-group-template-deploy)

<!---HONumber=71-->