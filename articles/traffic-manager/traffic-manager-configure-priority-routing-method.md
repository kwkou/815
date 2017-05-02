<properties
    pageTitle="使用 Azure 流量管理器配置优先级流量路由方法 | Azure"
    description="本文介绍如何在流量管理器中配置优先级流量路由方法"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="6dca6de1-18f7-4962-bd98-6055771fab22"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/20/2017"
    wacn.date="05/02/2017"
    ms.author="kumud"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="6137f622a01d4f768c051a86d667ac9a6886c231"
    ms.lasthandoff="04/22/2017" />

# <a name="configure-priority-traffic-routing-method-in-traffic-manager"></a>在流量管理器中配置优先级流量路由方法

无论网站模式如何，Azure 网站都已针对数据中心（也称为区域）内的网站提供了故障转移功能。 流量管理器为不同的数据中心内的网站提供故障转移。

服务故障转移的常见模式是将流量发送到主服务，并提供一组相同的备份服务进行故障转移。 以下步骤说明如何对 Azure 云服务和网站配置这种具有优先顺序的故障转移：

## <a name="to-configure-the-priority-traffic-routing-method"></a>配置优先级流量路由方法

1. 在浏览器中，登录 [Azure 门户预览](http://portal.azure.cn)。 如果还没有帐户，可注册 [1 个月期限的免费试用版](/pricing/1rmb-trial/)。 
2. 在门户的搜索栏中，搜索“流量管理器配置文件”，然后单击要为其配置路由方法的配置文件名称。
3. 在“流量管理器配置文件”边栏选项卡中，检查要包含在配置中的云服务和网站是否都存在。
4. 在“设置”部分，单击“配置”，然后在“配置”边栏选项卡中完成如下操作：
    1. 对于“流量路由方法设置”，验证流量路由方法是否是“优先级”。 如果不是，请在下拉列表中单击“优先级”。
    2. 为此配置文件中的所有终结点设置相同的“终结点监视器设置”，如下所示：
        1. 选择相应的“协议”，并指定“端口”号。 
        2. 对于“路径”，请键入正斜杠 */*。 若要监视终结点，必须指定路径和文件名。 正斜杠“/”是有效的相对路径条目，表示文件位于根目录（默认位置）中。
        3. 在页面顶部，单击“保存”。
5. 在“设置”部分，单击“终结点”。
6. 在“终结点”边栏选项卡中，查看终结点的优先级顺序。 如果选择了“优先级”流量路由方法，则所选终结点的顺序很重要。 验证终结点的优先级顺序。  主终结点位于顶部。 请仔细检查显示的顺序。 所有请求均会路由到第一个终结点；如果流量管理器检测到其处于不正常状态，则流量会自动故障转移到下一终结点。 
7. 若要更改终结点优先级顺序，请单击该终结点，然后在显示的“终结点”边栏选项卡中，单击“编辑”并根据需要更改“优先级”值。 
8. 单击“保存”以保存更改的终结点设置。
9. 完成你的配置更改后，单击页面底部的“**保存**”。
10. 按如下方式测试配置更改：
    1.    在门户的搜索栏中，搜索流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
    2.    在“流量管理器配置文件”边栏选项卡中，单击“概述”。
    3.    “流量管理器配置文件”边栏选项卡将显示新建的流量管理器配置文件的 DNS 名称。 任意客户端（例如通过 Web 浏览器导航到此名称）均可使用此名称路由到根据路由类型确定的相应终结点。 此情况下，所有请求均会路由到第一个终结点；如果流量管理器检测到其处于不正常状态，则流量会自动故障转移到下一终结点。
11. 流量管理器配置文件正常工作后，请在权威 DNS 服务器上编辑 DNS 记录，将公司域名指向流量管理器域名。

![使用流量管理器配置优先级流量路由方法][1]

## <a name="next-steps"></a>后续步骤

- 了解[加权流量路由方法](/documentation/articles/traffic-manager-configure-weighted-routing-method/)。
- 了解[性能路由方法](/documentation/articles/traffic-manager-configure-performance-routing-method/)。
- 了解如何[测试流量管理器设置](/documentation/articles/traffic-manager-testing-settings/)。

<!--Image references-->
[1]: ./media/traffic-manager-priority-routing-method/traffic-manager-priority-routing-method.png