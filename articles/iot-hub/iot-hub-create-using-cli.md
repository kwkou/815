<properties
    pageTitle="使用 Azure CLI (az.py) 创建 IoT 中心 | Azure"
    description="如何使用跨平台的 Azure CLI 2.0（预览版）(az.py) 创建 Azure IoT 中心。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ms.service: iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/15/2016"
    wacn.date="01/13/2017"
    ms.author="dobett" />  


# 使用 Azure CLI 2.0（预览版）创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍

可使用 Azure CLI 2.0（预览版）\(az.py\) 以编程方式创建和管理 Azure IoT 中心。本文说明如何使用 Azure CLI 2.0（预览版）\(az.py\) 创建 IoT 中心。

可使用以下 CLI 版本之一完成任务：

* [Azure CLI \(azure.js\)](/documentation/articles/iot-hub-create-using-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI。
* Azure CLI 2.0（预览版）\(az.py\)：用于本文中所述的资源管理部署模型的下一代 CLI。

要完成本教程，需要具备以下先决条件：

* 有效的 Azure 帐户。如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
* [Azure CLI 2.0（预览版）][lnk-CLI-install]。

## 登录并设置 Azure 帐户

登录 Azure 帐户，并将 Azure CLI 配置为与 IoT 中心资源配合使用。

1. 在命令提示符中，运行 [login 命令][lnk-login-command]：
    
        az login

    按照说明使用代码进行身份验证，并通过 Web 浏览器登录 Azure 帐户。

2. 如果有多个 Azure 订阅，登录 Azure 可获得与凭据关联的所有 Azure 帐户的访问权限。使用以下[命令，列出可供使用的 Azure 帐户][lnk-az-account-command]：
    
    
        az account list 
    

    使用以下命令，选择想要用于运行命令以创建 IoT 中心的订阅。可使用上一命令输出中的订阅名称或 ID：

    
        az account set --subscription {your subscription name or id}
    

3. 安装 Azure CLI *iot 组件* 。运行以下[命令，添加 iot 组件][lnk-az-addcomponent-command]：
    
    
        az component update --add iot
    

4. 必须注册 IoT 提供程序才能部署 IoT 资源。运行以下[命令，注册 IoT 提供程序][lnk-az-register-command]：
    
    
        az provider register -namespace Microsoft.Devices
    

## 创建 IoT 中心

使用 Azure CLI 创建资源组，然后添加 IoT 中心。

1. 创建 IoT 中心时，必须在资源组中创建它。使用现有资源组，或运行以下[命令创建资源组][lnk-az-resource-command]：
    
    
         az group create --name {your resource group name} --location chinaeast
    

    > [AZURE.TIP]
    上一示例在中国东部位置创建资源组。可运行 `az account list-locations -o table` 命令，查看可用位置的列表。
    >
    >

2. 运行以下[命令，在资源组中创建 IoT 中心][lnk-az-iot-command]：
    
    
        az iot hub create --name {your iot hub name} --resource-group {your resource group name} --sku S1
    

> [AZURE.NOTE]
IoT 中心的名称必须全局唯一。上一命令在计费的 S1 定价层中创建 IoT 中心。有关详细信息，请参阅 [Azure IoT 中心定价][lnk-iot-pricing]。
>
>

## 删除 IoT 中心

可使用 Azure CLI [删除单个资源][lnk-az-resource-command]（例如 IoT 中心），或删除资源组及其所有资源（包括任何 IoT 中心）。

若要删除 IoT 中心，请运行以下命令：


	az resource delete --name {your iot hub name} --resource-group {your resource group name} --resource-type Microsoft.Devices/IotHubs


若要删除资源组及其所有资源，请运行以下命令：


	az group delete --name {your resource group name}


## 后续步骤
若要详细了解如何开发 IoT 中心，请参阅以下文章：

* [IoT 中心开发人员指南][lnk-devguide]

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 Azure 门户管理 IoT 中心][lnk-portal]

<!-- Links -->

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-CLI-install]: https://docs.microsoft.com/cli/azure/install-az-cli2
[lnk-login-command]: https://docs.microsoft.com/cli/azure/get-started-with-az-cli2
[lnk-az-account-command]: https://docs.microsoft.com/cli/azure/account
[lnk-az-register-command]: https://docs.microsoft.com/cli/azure/provider
[lnk-az-addcomponent-command]: https://docs.microsoft.com/cli/azure/component
[lnk-az-resource-command]: https://docs.microsoft.com/cli/azure/resource
[lnk-az-iot-command]: https://docs.microsoft.com/cli/azure/iot
[lnk-iot-pricing]: /pricing/details/iot-hub/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-portal]: /documentation/articles/iot-hub-create-through-portal/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and code-->