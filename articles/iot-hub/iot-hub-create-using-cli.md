<properties
    pageTitle="使用 Azure CLI (az.py) 创建 IoT 中心 | Azure"
    description="如何使用跨平台的 Azure CLI 2.0 (az.py) 创建 Azure IoT 中心。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="iot-hub"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/23/2017"
    wacn.date="05/08/2017"
    ms.author="dobett"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="5cf453781736a41a4db49d68c21a5da1257d227e"
    ms.lasthandoff="04/28/2017" />

# <a name="create-an-iot-hub-using-the-azure-cli-20"></a>使用 Azure CLI 2.0 创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>介绍

可使用 Azure CLI 2.0 (az.py) 以编程方式创建和管理 Azure IoT 中心。 本文介绍如何使用 Azure CLI 2.0 (az.py) 创建 IoT 中心。

可以使用以下 CLI 版本之一完成任务：

* [Azure CLI (azure.js)](/documentation/articles/iot-hub-create-using-cli-nodejs/) - 适用于经典部署模型和资源管理部署模型的 CLI。
* Azure CLI 2.0 (az.py) - 适用于本文中所述的资源管理部署模型的下一代 CLI。

若要完成本教程，需要以下各项：

* 有效的 Azure 帐户。如果没有帐户，只需几分钟即可创建一个[试用帐户][lnk-free-trial]。
* [Azure CLI 2.0][lnk-CLI-install]。

## <a name="sign-in-and-set-your-azure-account"></a>登录并设置 Azure 帐户

登录到 Azure 帐户，然后选择订阅。

1. 在命令提示符中，运行 [login 命令][lnk-login-command]：

        az login

    按照说明使用代码进行身份验证，并通过 Web 浏览器登录 Azure 帐户。
    
    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

2. 如果有多个 Azure 订阅，登录 Azure 可获得与凭据关联的所有 Azure 帐户的访问权限。 使用以下 [命令，列出可供使用的 Azure 帐户][lnk-az-account-command] ：

        az account list 

    使用以下命令，选择想要用于运行命令以创建 IoT 中心的订阅。 可使用上一命令输出中的订阅名称或 ID：

        az account set --subscription {your subscription name or id}

## <a name="create-an-iot-hub"></a>创建 IoT 中心

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
> IoT 中心的名称必须全局唯一。 上一命令在计费的 S1 定价层中创建 IoT 中心。 有关详细信息，请参阅 [Azure IoT 中心定价][lnk-iot-pricing]。
>
>

## <a name="remove-an-iot-hub"></a>删除 IoT 中心

可使用 Azure CLI [删除单个资源][lnk-az-resource-command]（例如 IoT 中心），或删除资源组及其所有资源（包括任何 IoT 中心）。

若要删除 IoT 中心，请运行以下命令：

    az iot hub delete --name {your iot hub name} --resource-group {your resource group name}

若要删除资源组及其所有资源，请运行以下命令：

    az group delete --name {your resource group name}

## <a name="next-steps"></a>后续步骤
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

<!--Update_Description:update wording and code-->