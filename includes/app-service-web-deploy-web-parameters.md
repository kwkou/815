使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 Parameters 的部分，其中包含所有参数值。
你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

在定义参数时，请使用 **allowedValues** 字段来指定用户在部署过程中可以提供哪些值。如果在部署过程中未提供任何值，请使用 **defaultValue** 字段为该参数赋值。

下面介绍模板中的每个参数。

### siteName

要创建的网站的名称。

    "siteName":{
      "type":"string"
    }

### hostingPlanName

用于托管网站的 App Service 计划的名称。
    
    "hostingPlanName":{
      "type":"string"
    }

### sku

托管计划的定价层。

    "sku": {
      "type": "string",
      "allowedValues": [
        "F1",
        "D1",
        "B1",
        "B2",
        "B3",
        "S1",
        "S2",
        "S3"
      ],
      "defaultValue": "S1",
      "metadata": {
        "description": "The pricing tier for the hosting plan."
      }
    }

模板将定义此参数允许的值，如果未指定任何值，则分配默认值 (S1)。

### workerSize

托管计划的实例大小（small、medium 或 large）。

    "workerSize":{
      "type":"string",
      "allowedValues":[
        "0",
        "1",
        "2"
      ],
      "defaultValue":"0"
    }
    
模板将定义此参数允许的值（0、1 或 2），如果未指定任何值，则分配默认值 (0)。这些值分别对应于 small、medium 和 large。

<!---HONumber=Mooncake_0118_2016-->