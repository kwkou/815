若要在部署过程中标记资源，请将 `tags` 元素添加到要部署的资源。提供标记名称和值。

### 将文本值应用于标记名称
以下示例显示了具有设置为文本值的两个标记（`Dept` 和 `Environment`）的存储帐户：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat('storage', uniqueString(resourceGroup().id))]",
          "location": "[resourceGroup().location]",
          "tags": {
            "Dept": "Finance",
            "Environment": "Production"
          },
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": { }
        }
        ]
    }

### 将对象应用于标记元素
可以定义一个存储多个标记的对象参数，并将该对象应用于标记元素。对象中的每个属性将成为资源的单独标记。以下示例有一个名为 `tagValues` 的参数，将应用于标记元素。

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "tagValues": {
          "type": "object",
          "defaultValue": {
            "Dept": "Finance",
            "Environment": "Production"
          }
        }
      },
      "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat('storage', uniqueString(resourceGroup().id))]",
          "location": "[resourceGroup().location]",
          "tags": "[parameters('tagValues')]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": {}
        }
      ]
    }

### 将 JSON 字符串应用于标记名称

若要将多个值存储在单个标记中，请应用表示这些值的 JSON 字符串。整个 JSON 字符串将存储为一个标记，该标记不能超过 256 个字符。以下示例有一个名为 `CostCenter` 的标记，其中包含 JSON 字符串中的多个值：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat('storage', uniqueString(resourceGroup().id))]",
          "location": "[resourceGroup().location]",
          "tags": {
            "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
          },
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": { }
        }
        ]
    }

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: new article about how to utilize the tag during the deployment -->