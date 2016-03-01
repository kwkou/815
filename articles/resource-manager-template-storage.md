<properties
   pageTitle="用于存储的资源管理器模板 | Microsoft Azure"
   description="说明存储帐户的资源管理器架构。"
   services="azure-resource-manager,storage"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="10/25/2015"
   wacn.date="01/25/2016"/>

# 存储帐户 - 模板架构

创建存储帐户。

## 架构格式

若要创建存储帐户，请将以下架构添加到模板的资源节中。

    {
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2015-06-15",
        "name": string,
        "location": string,
        "properties": 
        {
        	"accountType": string
        }
    }

## 值

下表描述了需要在架构中设置的值。

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | 枚举 | 是 | **Microsoft.Storage/storageAccounts** | 要创建的资源类型。 |
| apiVersion | 枚举 | 是 | **2015-06-15** <br /> **2015-05-01-preview** | 要用于创建该资源的 API 版本。 | 
| name | 字符串 | 是 | 3 到 24 个字符，仅限数字和小写字母 | 要创建的存储帐户的名称该名称必须在全 Azure 中唯一。请考虑在你的命名约定中使用 [uniqueString](/documentation/articles/resource-group-template-functions/#uniquestring) 函数，如下面的示例所示。 |
| location | 字符串 | 是 | 若要确定有效的区域，请参阅[支持的区域](/documentation/articles/resource-manager-supported-services/#supported-regions)。 | 用于托管存储帐户的区域。 |
| properties | 对象 | 是 | （下面显示） | 一个对象，用于指定要创建的存储帐户的类型。

### 属性对象

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| accountType | 字符串 | 是 | **Standard\_LRS**<br />**Standard\_GRS**<br />**Standard\_RAGRS**<br />**Premium\_LRS** | 存储帐户的类型。允许的值对应于标准本地冗余、标准地域冗余、标准读取访问地域冗余和高级本地冗余。有关这些帐户类型的信息，请参阅 [Azure 存储复制](/documentation/articles/storage-redundancy)。 |

	
## 示例

以下示例使用基于资源组 ID 的唯一名称部署标准本地冗余存储帐户。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts",
                "apiVersion": "2015-06-15",
                "name": "[concat('storage', uniqueString(resourceGroup().id))]",
		         "location": "[resourceGroup().location]",
        	     "properties": 
        	     {
        		      "accountType": "Standard_LRS"
        	     }
	        }
	    ],
	    "outputs": {}
    }

## 后续步骤

- 有关存储的常规信息，请参阅 [Microsoft Azure 存储空间简介](/documentation/articles/storage-introduction)。

<!---HONumber=Mooncake_1207_2015-->