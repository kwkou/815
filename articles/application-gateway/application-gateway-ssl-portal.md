<properties
    pageTitle="配置 SSL 卸载 - Azure 应用程序网关 - Azure 门户 | Azure"
    description="本页说明了如何使用门户创建支持 SSL 卸载的应用程序网关"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="8373379a-a26a-45d2-aa62-dd282298eff3"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="05/22/2017"
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="7a678155caeebcbf5978f97e82475e1b917f624f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="configure-an-application-gateway-for-ssl-offload-by-using-the-portal"></a>使用门户配置可以进行 SSL 卸载的应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/application-gateway-ssl-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-ssl-arm/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-ssl/)

可将 Azure 应用程序网关配置为在网关上终止安全套接字层 (SSL) 会话，以避免 Web 场中出现开销较高的 SSL 解密任务。 SSL 卸载还简化了 Web 应用程序的前端服务器设置与管理。

## <a name="scenario"></a>方案

以下方案演示了如何在现有应用程序网关中配置 SSL 卸载。 该方案假定已按相关步骤[创建应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)。

## <a name="before-you-begin"></a>开始之前

若要配置应用程序网关的 SSL 卸载，必须提供证书。 此证书在应用程序网关上加载，用于加密和解密通过 SSL 发送的流量。 证书需采用个人信息交换 (pfx) 格式。 此文件格式适用于导出私钥，后者是应用程序网关对流量进行加解密所必需的。

## <a name="add-an-https-listener"></a>添加 HTTPS 侦听器

HTTPS 侦听器根据配置来查找流量，并可将流量路由到后端池。

### <a name="step-1"></a>步骤 1

导航到 Azure 门户，然后选择现有的应用程序网关

### <a name="step-2"></a>步骤 2

单击“侦听器”，然后单击“添加”按钮即可添加侦听器。

![应用网关概述边栏选项卡][1]

### <a name="step-3"></a>步骤 3

填写侦听器的必需信息并上载 .pfx 证书，完成时单击“确定”。

**名称** - 该值为侦听器的友好名称。

**前端 IP 配置** - 该值为用于侦听器的前端 IP 配置。

**前端端口(名称/端口)** - 用在应用程序网关前端的端口的友好名称，以及所使用的实际端口。

**协议** - 一个开关，用于确定为前端使用了 https 还是 http。

**证书(名称/密码)** - 如果使用了 SSL 卸载，则需对此设置使用 .pfx 证书，并需提供友好名称和密码。

![添加侦听器边栏选项卡][2]

## <a name="create-a-rule-and-associate-it-to-the-listener"></a>创建规则并将其关联到侦听器

现在已创建侦听器。 此时需创建规则来处理来自侦听器的流量。 规则定义如何基于多个配置设置（包括是否使用了基于 Cookie 的会话相关性、协议、端口和运行状况探测）将流量路由到后端池。

### <a name="step-1"></a>步骤 1

单击应用程序网关的“规则”，然后单击“添加”。

![应用网关规则边栏选项卡][3]

### <a name="step-2"></a>步骤 2

在“添加基本规则”边栏选项卡中，键入规则的友好名称并选择在上一步创建的侦听器。 选择适当的后端池和 http 设置，然后单击“确定”

![https 设置窗口][4]

现在，设置将保存到应用程序网关。 保存这些设置可能需要一段时间，此时间过后才能通过门户或 PowerShell 查看这些设置。 保存完以后，应用程序网关将负责处理流量的加解密。 应用程序网关和后端 Web 服务器之间的所有流量将通过 http 来处理。 任何通过 https 启动的需要返回到客户端的通信都会返回到加密的客户端。

## <a name="next-steps"></a>后续步骤

若要了解如何配置 Azure 应用程序网关的自定义运行状况探测，请参阅[创建自定义运行状况探测](/documentation/articles/application-gateway-create-gateway-portal/)。

[1]: ./media/application-gateway-ssl-portal/figure1.png
[2]: ./media/application-gateway-ssl-portal/figure2.png
[3]: ./media/application-gateway-ssl-portal/figure3.png
[4]: ./media/application-gateway-ssl-portal/figure4.png

<!--Update_Description: wording update-->