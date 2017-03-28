<properties
    pageTitle="创建自定义探测 - Azure 应用程序网关 - Azure 门户预览 | Azure"
    description="了解如何使用门户创建应用程序网关的自定义探测"
    services="application-gateway"
    documentationcenter="na"
    author="georgewallace"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="33fd5564-43a7-4c54-a9ec-b1235f661f97"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="03/28/2017"
    ms.author="gwallace" />

# 使用门户创建应用程序网关的自定义探测
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-create-probe-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-probe-ps/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-create-probe-classic-ps/)

[AZURE.INCLUDE [azure-probe-intro-include](../../includes/application-gateway-create-probe-intro-include.md)]

## 方案

以下方案演示了如何在现有应用程序网关中创建自定义运行状况探测。该方案假定你已按相关步骤[创建应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)。

## <a name="createprobe"></a>创建探测

可通过门户分两步配置探测。第一步是创建探测。第二步是将探测添加到应用程序网关的后端 http 设置。

### 步骤 1

导航到 [Azure 门户预览](http://portal.azure.cn)，然后选择现有的应用程序网关。

![应用程序网关概述][1]  


### 步骤 2

单击“探测”，然后单击“添加”按钮添加探测。

![添加填写了信息的“探测”边栏选项卡][2]  


### 步骤 3

填写探测的必需信息，完成时单击“确定”。

* **名称** - 此值是可在门户中访问的探测的友好名称。
* **主机** - 此值是用于探测的主机名。仅在应用程序网关上配置了多站点的情况下适用，否则使用“127.0.0.1”。此值与 VM 主机名不同。
* **路径** - 自定义探测的完整 URL 的其余部分。有效路径以“/”开头
* **时间间隔(秒)** - 运行探测来检查运行状况的频率。不建议将时间间隔设置为不到 30 秒。
* **超时(秒)** - 超时之前探测的等待时间。超时间隔必须足够长，以便进行 http 调用，确保后端运行状况页可用。
* **不正常阈值** - 系统认为不正常的失败尝试次数。阈值为 0 意味着，如果运行状况检查失败，则会立即将后端确定为不正常。

> [AZURE.IMPORTANT]
主机名不同于服务器名。此值是运行在应用程序服务器上的虚拟主机的名称。探测发送到 http://(host 名):(httpsetting 中的端口)/urlPath

![探测配置设置][3]

## 向网关添加探测

创建探测以后，即可将其添加到网关。探测设置在应用程序网关的后端 http 设置中设置。

### 步骤 1

单击应用程序网关的“HTTP 设置”，然后单击窗口中当前的后端 http 设置，以便显示配置边栏选项卡。

![https 设置窗口][4]  


### 步骤 2

在 **appGatewayBackEndHttp** 设置边栏选项卡上，单击“使用自定义探测”，然后选择在[创建探测](#createprobe)部分创建的探测。完成后，单击“确定”即可应用相关设置。

![appgatewaybackend 设置边栏选项卡][5]  


默认探测检查对 Web 应用程序的默认访问权限。现在已创建自定义探测，因此应用程序网关可以使用定义的自定义路径来监视所选后端的运行状况。应用程序网关根据所定义的条件检查探测中指定的文件。如果调用 host:Port/path 时没有返回“Http 200 正常”状态响应，则在达到不正常阈值后，服务器将不再进行轮换。将继续对不正常的实例进行探测，看其何时恢复正常。将实例添加回正常的服务器池以后，其流量即可恢复，同时也会继续按用户指定的正常时间间隔对实例进行探测。

## 后续步骤

若要了解如何通过 Azure 应用程序网关配置 SSL 卸载，请参阅[配置 SSL 卸载](/documentation/articles/application-gateway-ssl-portal/)

[1]: ./media/application-gateway-create-probe-portal/figure1.png
[2]: ./media/application-gateway-create-probe-portal/figure2.png
[3]: ./media/application-gateway-create-probe-portal/figure3.png
[4]: ./media/application-gateway-create-probe-portal/figure4.png
[5]: ./media/application-gateway-create-probe-portal/figure5.png

<!---HONumber=Mooncake_1226_2016-->