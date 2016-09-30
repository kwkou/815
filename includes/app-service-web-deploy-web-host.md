### App Service 计划

创建用于托管网站的服务计划。通过 **hostingPlanName** 参数提供计划的名称。计划的位置与用于资源组的位置相同。定价层和辅助角色大小在 **sku** 和 **workerSize** 参数中指定

    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('hostingPlanName')]",
      "type": "Microsoft.Web/serverfarms",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "[parameters('sku')]",
        "capacity": "[parameters('workerSize')]"
      },
      "properties": {
        "name": "[parameters('hostingPlanName')]"
      }
    },

<!---HONumber=Mooncake_0118_2016-->