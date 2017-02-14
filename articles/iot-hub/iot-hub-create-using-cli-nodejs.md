<properties
    pageTitle="使用 Azure CLI (azure.js) 创建 IoT 中心 | Azure"
    description="如何使用跨平台的 Azure CLI (azure.js) 创建 Azure IoT 中心。"
    services="iot-hub"
    documentationcenter=".net"
    author="BeatriceOltean"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="46a17831-650c-41d9-b228-445c5bb423d3"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/04/2017"
    wacn.date="02/10/2017"
    ms.author="boltean" />  


# 使用 Azure CLI 创建 IoT 中心
[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍
可使用 Azure CLI (azure.js) 以编程方式创建和管理 Azure IoT 中心。本文介绍如何使用 Azure CLI (azure.js) 创建 IoT 中心。

可使用以下 CLI 版本之一完成任务：

- Azure CLI (azure.js)：用于经典部署模型和资源管理部署模型的 CLI（如本文中所述）。
- [Azure CLI 2.0（预览版）(az.py)](/documentation/articles/iot-hub-create-using-cli/)：用于资源管理部署模型的下一代 CLI。

若要完成本教程，您需要以下各项：

- 有效的 Azure 帐户。如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
- [Azure CLI 0.10.4][lnk-CLI-install] 或更高版本。如果已安装 Azure CLI，则可在命令提示符处使用以下命令验证当前版本：
  
        azure --version
  

> [AZURE.NOTE]
Azure 具有用于创建和处理资源的两个不同的部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。Azure CLI 必须处于 Azure Resource Manager 模式：
> 
> `azure config mode arm`
> 

## 设置 Azure 帐户和订阅
1. 在命令提示符处键入以下命令登录
   
   
        azure login -e AzureChinaCloud
   
    使用建议的 Web 浏览器和代码进行身份验证。
    
2. 如果有多个 Azure 订阅，则连接到 Azure 即有权访问与凭据关联的所有 Azure 订阅。可以使用以下命令查看 Azure 订阅并确定哪个订阅是默认订阅：
   
   
        azure account list 
   

    若要设置订阅上下文，以便在其下运行其余命令，请使用：

   
        azure account set <subscription name>
   

3. 如果没有资源组，则可创建一个，将其命名为 **exampleResourceGroup**
   
        azure group create -n exampleResourceGroup -l westus
   

> [AZURE.TIP]
[Use the Azure CLI to manage Azure resources and resource groups][lnk-CLI-arm]（使用 Azure CLI 管理 Azure 资源和资源组）一文详细介绍了如何使用 Azure CLI 管理 Azure 资源。
> 
> 

## 创建 IoT 中心
所需参数：


     azure iothub create -g <resource-group> -n <name> -l <location> -s <sku-name> -u <units>  
        - <resourceGroup> The resource group name (case insensitive alphanumeric, underscore and hyphen, 1-64 length)
        - <name> (The name of the IoT hub to be created. The format is case insensitive alphanumeric, underscore and hyphen, 3-50 length )
        - <location> (The location (azure region/datacenter) where the IoT hub will be provisioned.
        - <sku-name> (The name of the sku, one of: [F1, S1, S2, S3] etc. For the latest full list refer to the pricing page for IoT Hub.
        - <units> (The number of provisioned units. Range : F1 [1-1] : S1, S2 [1-200] : S3 [1-10]. IoT Hub units are based on your total message count and the number of devices you want to connect.)

若要查看所有可以创建的参数，可以在命令提示符处使用帮助命令


    azure iothub create -h 

简单示例：

 若要在资源组 **exampleResourceGroup** 中创建名为 **exampleIoTHubName** 的 IoT 中心，请运行以下命令：


    azure iothub create -g exampleResourceGroup -n exampleIoTHubName -l chinaeast -k s1 -u 1


> [AZURE.NOTE]
此 Azure CLI 命令为用户创建付费的 S1 标准 IoT 中心。可以使用以下命令删除 IoT 中心 **exampleIoTHubName**
> 
> `azure iothub delete -g exampleResourceGroup -n exampleIoTHubName`
> 

## 后续步骤
若要深入了解如何开发 IoT 中心，请参阅以下内容：

- [IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 Azure 门户管理 IoT 中心][lnk-portal]

<!-- Links -->

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-CLI-install]: /documentation/articles/xplat-cli-install/
[lnk-rest-api]: https://msdn.microsoft.com/zh-cn/library/mt589014.aspx
[lnk-CLI-arm]: /documentation/articles/xplat-cli-azure-resource-manager/

[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-portal]: /documentation/articles/iot-hub-create-through-portal/

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update wording-->