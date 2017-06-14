<properties
    pageTitle="使用 Azure 流量管理器配置性能流量路由方法 | Azure"
    description="本文介绍了如何配置流量管理器以将流量路由到终结点并保证最低延迟"
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
    ms.openlocfilehash="3c1d0c85d6adab4edf38bd6aa1e0308396e57d59"
    ms.lasthandoff="04/22/2017" />

# <a name="configure-the-performance-traffic-routing-method"></a>配置性能流量路由方法

使用性能流量路由方法可将流量定向到客户端网络中延迟最低的终结点。 通常，延迟最低的数据中心就是地理距离最短的终结点。 此流量路由方法无法考虑网络配置或负载的实时变化。

##  <a name="to-configure-performance-routing-method"></a>配置性能路由方法

1. 在浏览器中，登录 [Azure 门户](http://portal.azure.cn)。 如果还没有帐户，可注册 [1 个月期限的免费试用版](/pricing/1rmb-trial/)。 
2. 在门户的搜索栏中，搜索“流量管理器配置文件”，然后单击要为其配置路由方法的配置文件名称。
3. 在“流量管理器配置文件”边栏选项卡中，检查要包含在配置中的云服务和网站是否都存在。
4. 在“设置”部分，单击“配置”，然后在“配置”边栏选项卡中完成如下操作：
    1. 对于“流量路由方法设置”和“路由方法”，请选择“性能”。
    2. 为此配置文件中的所有终结点设置相同的“终结点监视器设置”，如下所示：
        1. 选择相应的“协议”，并指定“端口”号。 
        2. 对于“路径”，请键入正斜杠 */*。 若要监视终结点，必须指定路径和文件名。 正斜杠“/”是有效的相对路径条目，表示文件位于根目录（默认位置）中。
        3. 在页面顶部，单击“保存”。
5.  按如下方式测试配置更改：
    1.    在门户的搜索栏中，搜索流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
    2.    在“流量管理器配置文件”边栏选项卡中，单击“概述”。
    3.    “流量管理器配置文件”边栏选项卡将显示新建的流量管理器配置文件的 DNS 名称。 任意客户端（例如通过 Web 浏览器导航到此名称）均可使用此名称路由到根据路由类型确定的相应终结点。 在此情况下，所有请求均通过客户端网络路由到终结点并保证最低延迟。
6. 流量管理器配置文件正常工作后，请在权威 DNS 服务器上编辑 DNS 记录，将公司域名指向流量管理器域名。

![使用流量管理器配置性能流量路由方法][1]

## <a name="next-steps"></a>后续步骤

- 了解[加权流量路由方法](/documentation/articles/traffic-manager-configure-weighted-routing-method/)。
- 了解[优先级路由方法](/documentation/articles/traffic-manager-configure-priority-routing-method/)。
- 了解如何[测试流量管理器设置](/documentation/articles/traffic-manager-testing-settings/)。

<!--Image references-->
[1]: ./media/traffic-manager-performance-routing-method/traffic-manager-performance-routing-method.png

<!--Update_Description: change from classic portal to new portal-->
