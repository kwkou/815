<properties
   pageTitle="将链接模版与 Resource Manager 配合使用 | Azure"
   description="介绍如何使用 Azure Resource Manager 模板中的链接模板创建一个模块化的模板的解决方案。演示如何传递参数值、指定参数文件和动态创建的 URL。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.date="06/08/2016"
   wacn.date="07/11/2016"/>

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

资源管理器服务必须能够访问链接模板，这意味着，你不能为链接模板指定本地文件或只能在本地网络上使用的文件。只能提供包含 **http** 或 **https** 的 URI 值。一种做法是将链接模板放在存储帐户中并对该项使用 URI，如下所示。

    "templateLink": {
        "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
        "contentVersion": "1.0.0.0",
    }

尽管链接模板必须可从外部使用，但它无需向公众正式发布。你可以将模板添加到专用存储帐户，这样，只有存储帐户所有者可访问该模板，然后创建共享访问签名 (SAS) 令牌，以允许在部署期间访问。将该 SAS 令牌添加到链接模板的 URI。有关在存储帐户中设置模板和生成 SAS 令牌的步骤，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy)或[使用 Resource Manager 模板和 Azure CLI 部署资源](/documentation/articles/resource-group-template-deploy-cli)。

下面的示例演示了链接到其他模板的父模板。使用作为参数传入的 SAS 令牌访问嵌套模板。

    "parameters": {
        "sasToken": { "type": "securestring" }
    },
    "resources": [
        {
            "apiVersion": "2015-01-01",
            "name": "nestedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
              "mode": "incremental",
              "templateLink": {
                "uri": "[concat('https://storagecontosotemplates.blob.core.chinacloudapi.cn/templates/helloworld.json', parameters('sasToken'))]",
                "contentVersion": "1.0.0.0"
              }
            }
        }
    ],

即使令牌作为安全字符串传入，链接模板的 URI（包括 SAS 令牌）也将记录在该资源组的部署操作中。若要限制公开，请设置令牌的到期时间。

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

链接参数文件的 URI 值不能是本地文件，并且必须包含 **http** 或 **https**。当然，也可将参数文件限制为通过 SAS 令牌进行访问。

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

你还可以使用 [deployment()](/documentation/articles/resource-group-template-functions/#deployment) 获取当前模板的基 URL，并使用该 URL 来获取同一位置其他模板的 URL。如果你的模板位置发生变化（原因可能是改版）或者你想要避免对模板文件中的 URL 进行硬编码，则此操作非常有用。

    "variables": {
        "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
    }

## 完整示例

下面的示例模板显示了简化布置的链接模板以说明本文中的几个概念。它假定模板已添加到公共访问权限已关闭的存储帐户中的同一个容器。链接模板将一个值传递回 **outputs** 节中的主模板。

**Parent.json** 文件由以下部分组成：

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "containerSasToken": { "type": "string" }
      },
      "resources": [
        {
          "apiVersion": "2015-01-01",
          "name": "nestedTemplate",
          "type": "Microsoft.Resources/deployments",
          "properties": {
            "mode": "incremental",
            "templateLink": {
              "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
              "contentVersion": "1.0.0.0"
            }
          }
        }
      ],
      "outputs": {
        "result": {
          "type": "object",
          "value": "[reference('nestedTemplate').outputs.result]"
        }
      }
    }

**helloworld.json** 文件由以下部分组成：

    {
	  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
	  "contentVersion": "1.0.0.0",
	  "parameters": {},
	  "variables": {},
	  "resources": [],
	  "outputs": {
		"result": {
			"value": "Hello World",
			"type" : "string"
		}
	  }
    }
    
在 PowerShell 中，你使用以下命令获取容器的令牌并部署模板：

    Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
    $token = New-AzureStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ("https://storagecontosotemplates.blob.core.chinacloudapi.cn/templates/parent.json" + $token) -containerSasToken $token

在 Azure CLI 中，你使用以下代码获取容器的令牌并部署模板。目前，使用包括 SAS 令牌的模板 URI 时必须提供部署的名称。

    expiretime=$(date -I'minutes' --date "+30 minutes")  
    azure storage container sas create --container templates --permissions r --expiry $expiretime --json | jq ".sas" -r
    azure group deployment create -g ExampleGroup --template-uri "https://storagecontosotemplates.blob.core.chinacloudapi.cn/templates/parent.json?{token}" -n tokendeploy  

系统将提示你提供 SAS 令牌作为参数。你需要在令牌的前面加 **?**。

## 后续步骤
- 若要了解如何为资源定义部署顺序，请参阅 [Defining dependencies in Azure Resource Manager templates（在 Azure Resource Manager 模板中定义依赖关系）](/documentation/articles/resource-group-define-dependencies/)
- 若要了解如何定义一个资源但要创建其多个实例，请参阅 [Create multiple instances of resources in Azure Resource Manager（在 Azure Resource Manager 中创建多个资源实例）](/documentation/articles/resource-group-create-multiple/)

<!---HONumber=Mooncake_0704_2016-->