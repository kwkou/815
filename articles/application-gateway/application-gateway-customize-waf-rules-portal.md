<properties
    pageTitle="自定义 Azure 应用程序网关的 Web 应用程序防火墙规则 - 门户 | Azure"
    description="此页提供有关如何使用门户自定义应用程序网关的 Web 应用程序防火墙规则的信息。"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="1159500b-17ba-41e7-88d6-b96986795084"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.custom=""
    ms.workload="infrastructure-services"
    ms.date="03/28/2017"
    wacn.date=""
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="75890c3ffb1d1757de64a8b8344e9f2569f26273"
    ms.openlocfilehash="9d4c630ef6cfb51fad3b1887097aa860043d2eee"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="04/25/2017" />

# <a name="customize-web-application-firewall-rules-through-the-portal"></a>通过门户自定义 Web 应用程序防火墙规则

应用程序网关 Web 应用程序防火墙可为 Web 应用程序提供保护。 这些保护由 OWASP CRS 规则集提供。 某些规则可能会导致误报，并会阻止实际流量。  出于此原因，应用程序网关在启用了 Web 应用程序防火墙的应用程序网关上提供了自定义规则组和规则的功能。 有关特定规则组和规则的详细信息，请访问 [Web 应用程序防火墙 CRS 规则组和规则](/documentation/articles/application-gateway-crs-rulegroups-rules/)

>[AZURE.NOTE]
> 如果应用程序网关未使用 WAF 层，将显示“将应用程序网关升级到 WAF 层”选项，如下图所示：

![启用 waf][fig1]

## <a name="view-rule-groups-and-rules"></a>查看规则组和规则

导航到应用程序网关并选择“Web 应用程序防火墙”。  单击“高级规则配置”。  这将在随所选规则集提供的所有规则组页上显示一个表。

![配置已禁用的规则][1]

## <a name="search-for-rules-to-disable"></a>搜索要禁用的规则

“Web 应用程序防火墙设置”边栏选项卡提供通过文本搜索筛选规则的功能。 结果仅显示包含所搜索的文本的规则组和规则。

![搜索规则][2]

## <a name="disable-rule-groups-and-rules"></a>禁用规则组和规则

禁用规则时可以禁用整个规则组，也可以禁用一个或多个规则组下的特定规则。  取消选中要禁用的规则后，请单击“保存”。  这会将更改保存到应用程序网关。

![保存更改][3]

## <a name="next-steps"></a>后续步骤

配置已禁用的规则后，可通过访问[应用程序网关诊断](/documentation/articles/application-gateway-diagnostics/#diagnostic-logging)来了解如何查看 WAF 日志

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png