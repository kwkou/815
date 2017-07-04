<properties
    pageTitle="管理 Azure 流量管理器配置文件 | Azure"
    description="本文有助于你创建、禁用、启用和删除 Azure 流量管理器配置文件。"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f06e0365-0a20-4d08-b7e1-e56025e64f66"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/10/2017"
    wacn.date="05/31/2017"
    ms.author="kumud"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="66cd7dfff55f26a738084fd9d0b7aa098e827293"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="manage-an-azure-traffic-manager-profile"></a>管理 Azure 流量管理器配置文件

流量管理器配置文件使用流量路由方法控制云服务或网站终结点的流量分布。 本文介绍如何创建和管理这些配置文件。

## <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

可以使用 Azure 门户创建流量管理器配置文件。 在创建配置文件后，可以在 Azure 门户中配置终结点、监视以及其他设置。 流量管理器支持每个配置文件最多 200 个终结点。 但是，大多数使用方案只需要少量的终结点。

### <a name="to-create-a-traffic-manager-profile"></a>创建流量管理器配置文件

1. 在浏览器中，登录 [Azure 门户](http://portal.azure.cn)。 如果还没有帐户，可注册 [1 个月期限的免费试用版](/pricing/1rmb-trial/)。 
2. 在“中心”菜单上，单击“新建” > “网络” > “全部查看”，单击“流量管理器配置文件”，打开“创建流量管理器配置文件”边栏选项卡，然后单击“创建”。
3. 在“创建流量管理器配置文件”边栏选项卡上，按如下所示完成输入：
    1. 在**名称**中，提供配置文件的名称。 此名称必须在 trafficmanager.cn 区域中唯一，并将生成 DNS 名称 (<name>,trafficmanager.cn)，该名称用于访问流量管理器配置文件。
    2. 在**路由方法**中，选择“优先级”路由方法。
    3. 在**订阅**中，选择要创建此配置文件的订阅
    4. 在**资源组**中，创建新的资源组，以在其下放置此配置文件。
    5. 在**资源组位置**中，选择资源组的位置。 此设置指的是资源组的位置，对将全局部署的流量管理器配置文件没有影响。
    6. 单击“创建” 。
    7. 流量管理器配置文件的全局部署完成后，它将在相应的资源组中作为资源之一列出。

## <a name="disable-enable-or-delete-a-profile"></a>禁用、启用或删除配置文件

可以禁用现有的配置文件，使流量管理器不会将用户请求路由到配置的终结点。 禁用流量管理器配置文件后，配置文件及其中包含的信息将保持不变，并且可以在流量管理器界面中进行编辑。  重新启用配置文件时，路由将会恢复。 在 Azure 经典管理门户中创建流量管理器配置文件时，该配置文件会自动启用。 如果确定不再需要某个配置文件，可以将其删除。

### <a name="to-disable-a-profile"></a>禁用配置文件

1. 如果使用自定义域名，请更改 Internet DNS 服务器上的 CNAME 记录，使它不再指向流量管理器配置文件。
2. 流量将不再通过流量管理器配置文件设置定向到终结点。
3. 在浏览器中，登录 [Azure 门户](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡中，单击“概述”，在“概述”边栏选项卡中单击“禁用”，然后确认禁用流量管理器配置文件。

### <a name="to-enable-a-profile"></a>启用配置文件

1. 在浏览器中，登录 [Azure 门户](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡中，单击“概述”，然后在“概述”边栏选项卡中单击“启用”。
5. 如果使用自定义域名，请在 Internet DNS 服务器上创建一条指向流量管理器配置文件域名的 CNAME 资源记录。
6. 然后，流量将再次定向到终结点。

### <a name="to-delete-a-profile"></a>删除配置文件

1. 确保你的 Internet DNS 服务器上的 DNS 资源记录不再使用指向你的流量管理器配置文件域名的 CNAME 资源记录。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡中，单击“概述”，在“概述”边栏选项卡中单击“删除”，然后确认删除流量管理器配置文件。

## <a name="next-steps"></a>后续步骤

* [添加终结点](/documentation/articles/traffic-manager-endpoints/)
* [配置优先级路由方法](/documentation/articles/traffic-manager-configure-priority-routing-method/)
* [配置加权路由方法](/documentation/articles/traffic-manager-configure-weighted-routing-method/)
* [配置性能路由方法](/documentation/articles/traffic-manager-configure-performance-routing-method/)

<!--Update_Description: change to the new portal-->