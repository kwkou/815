<properties
    pageTitle="Azure 流量管理器 - 常见问题 | Azure"
    description="本文提供有关流量管理器的常见问题解答"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="75d5ff9a-f4b9-4b05-af32-700e7bdfea5a"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2017"
    wacn.date="04/17/2017"
    ms.author="kumud"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="aeee509b6c582b22fce6ec451c68732dc9441b23"
    ms.lasthandoff="04/06/2017" />

# <a name="traffic-manager-frequently-asked-questions-faq"></a>流量管理器常见问题解答 (FAQ)

## <a name="traffic-manager-basics"></a>流量管理器基础知识

### <a name="what-ip-address-does-traffic-manager-use"></a>流量管理器使用什么 IP 地址？

如“流量管理器工作原理”中所述，流量管理器在 DNS 级别工作。 它会发送 DNS 响应，将客户端定向到相应的服务终结点。 然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

因此，流量管理器不提供供客户端连接的终结点或 IP 地址。 因此，如果想要为服务使用静态 IP 地址，则必须在服务而不是流量管理器中配置该地址。

### <a name="does-traffic-manager-support-sticky-sessions"></a>流量管理器是否支持“粘滞”会话？

[如此文所述](/documentation/articles/traffic-manager-overview/#how-traffic-manager-works)，流量管理器在 DNS 级别工作。 它使用 DNS 响应将客户端引导到相应的服务终结点。 客户端直接连接到服务终结点，而不是通过流量管理器连接。 因此，流量管理器看不到客户端与服务器之间的 HTTP 流量。

此外，流量管理器收到的 DNS 查询的源 IP 地址属于递归 DNS 服务而不是客户端。 因此，流量管理器无法跟踪单个客户端，也无法实现“粘滞”会话。 这种限制在所有基于 DNS 的流量管理系统中很常见，并不是流量管理器特有的限制。

### <a name="why-am-i-seeing-an-http-error-when-using-traffic-manager"></a>使用流量管理器时为何出现 HTTP 错误？

[如此文所述](/documentation/articles/traffic-manager-overview/#how-traffic-manager-works)，流量管理器在 DNS 级别工作。 它使用 DNS 响应将客户端引导到相应的服务终结点。 然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。 流量管理器看不到客户端与服务器之间的 HTTP 流量。 因此，出现的任何 HTTP 错误必定来自应用程序。 要使客户端连接到应用程序，必须完成所有 DNS 解析步骤。 这包括流量管理器对应用程序流量流所做的任何交互。

因此，进一步的调查应着重于应用程序。

问题的最常见原因源自客户端浏览器发送的 HTTP 主机标头。 请确保将应用程序配置为接受所要使用的域名的正确主机标头。 对于使用 Azure 应用服务的终结点，请参阅[使用流量管理器为 Azure 应用服务中的 Web 应用配置自定义域名](/documentation/articles/web-sites-traffic-manager-custom-domain-name/)。

### <a name="what-is-the-performance-impact-of-using-traffic-manager"></a>使用流量管理器对性能有什么影响？

[如此文所述](/documentation/articles/traffic-manager-overview/#how-traffic-manager-works)，流量管理器在 DNS 级别工作。 由于客户端直接连接到你的服务终结点，因此在使用流量管理器时，一旦建立连接就没有性能影响。

由于流量管理器在 DNS 级别与应用程序集成，因此需要将额外的 DNS 查找插入 DNS 解析链中。 流量管理器对 DNS 解析时间的影响微乎其微。 流量管理器使用全局性的名称服务器网络，并使用 [任播](https://en.wikipedia.org/wiki/Anycast) 网络来确保始终将 DNS 查询路由到最靠近的可用名称服务器。 此外，对 DNS 响应进行缓存意味着，因使用流量管理器而导致的额外的 DNS 延迟仅出现在部分会话中。

“性能”方法将流量路由到最靠近的可用终结点。 最终结果是，与此方法相关的整体性能影响将会减到最小。 通过减小与终结点之间的网络延迟，应该可以抵消 DNS 延迟增大造成的影响。

### <a name="what-application-protocols-can-i-use-with-traffic-manager"></a>流量管理器允许使用什么应用程序协议？

[如此文所述](/documentation/articles/traffic-manager-overview/#how-traffic-manager-works)，流量管理器在 DNS 级别工作。 完成 DNS 查找以后，客户端会直接连接到应用程序终结点，不通过流量管理器进行连接。 因此，连接可以使用任何应用程序协议。 不过，流量管理器的终结点运行状况检查需要使用 HTTP 或 HTTPS 终结点。 要执行运行状况检查的终结点可以不同于客户端连接到的应用程序终结点。

### <a name="can-i-use-traffic-manager-with-a-naked-domain-name"></a>是否可以对“裸”域名使用流量管理器？

不可以。 DNS 标准不允许 CNAME 与其他同名的 DNS 记录共存。 DNS 区域的顶点（或根）始终包含两条预先存在的 DNS 记录：SOA 和权威 NS 记录。 这意味着在不违反 DNS 标准的情况下，无法在区域顶点位置创建 CNAME 记录。

流量管理器需要使用一条 DNS CNAME 记录来映射虚构 DNS 名称。 例如，将 www.contoso.com 映射到流量管理器配置文件 DNS 名称 contoso.trafficmanager.cn。 此外，流量管理器配置文件还会返回另一条 DNS CNAME 来指示客户端应连接到的终结点。

若要解决此问题，我们建议使用 HTTP 重定向将流量从裸域名定向到不同的 URL，然后即可使用流量管理器。 例如，裸域“contoso.com”可将用户重定向到指向流量管理器 DNS 名称的 CNAME“www.contoso.com”。

在流量管理器中实现对裸域的完全支持已列入我们的待开发功能中。 请 [在我们的社区反馈站点上投票](https://feedback.azure.com/forums/217313-networking/suggestions/5485350-support-apex-naked-domains-more-seamlessly)，表达你对此项功能的支持。

## <a name="traffic-manager-endpoints"></a>流量管理器终结点

### <a name="can-i-use-traffic-manager-with-endpoints-from-multiple-subscriptions"></a>能否将流量管理器用于多个订阅的终结点？

不能对 Azure Web 应用使用多个订阅中的终结点。 Web 应用要求其所用的任何自定义域名只能在单个订阅中使用。 无法对多个订阅中的 Web 应用使用同一个域名。

对于其他终结点类型，可在多个订阅中结合使用流量管理器和终结点。 流量管理器的配置方式取决于使用的是经典部署模型还是 Resource Manager 体验。

* 在 Resource Manager 中，只要配置流量管理器配置文件的人员具有终结点的读取访问权限，任何订阅的终结点就都可添加到流量管理器中。 可使用 [Azure Resource Manager 基于角色的访问控制 (RBAC)](/documentation/articles/role-based-access-control-configure/) 授予这些权限。
* 在经典部署模型接口中，流量管理器要求配置为 Azure 终结点的云服务或 Web 应用必须与流量管理器配置文件位于同一个订阅中。 可以将其他订阅中的云服务终结点作为“外部”终结点添加到流量管理器。 这些外部终结点按 Azure 终结点而不是外部费率计费。

### <a name="can-i-use-traffic-manager-with-cloud-service-staging-slots"></a>能否将流量管理器用于云服务的“过渡”槽？

是的。 可以在流量管理器中将云服务的“过渡”槽配置为“外部”终结点。 运行状况检查仍按 Azure 终结点费率计费。 由于使用的是“外部”终结点类型，系统不会自动拾取对基础服务所做的更改。 使用外部终结点时，流量管理器无法检测停止或删除云服务的时间。 因此，在禁用或删除终结点之前，流量管理器会一直对运行状况检查计费。

### <a name="does-traffic-manager-support-ipv6-endpoints"></a>流量管理器是否支持 IPv6 终结点？

流量管理器当前不提供 IPv6 可寻址的名称服务器。 但是，IPv6 客户端仍可使用流量管理器连接到 IPv6 终结点。 客户端不会直接向流量管理器发出 DNS 请求， 而是使用递归 DNS 服务。 仅使用 IPv6 的客户端通过 IPv6 向递归 DNS 服务发送请求。 然后，该递归服务应该能够使用 IPv4 联系流量管理器名称服务器。

流量管理器使用终结点的 DNS 名称做出响应。 若要支持 IPv6 终结点，必须有一条可将终结点 DNS 名称指向 IPv6 地址的 DNS AAAA 记录。 流量管理器运行状况检查仅支持 IPv4 地址。 服务需要在相同的 DNS 名称中公开 IPv4 终结点。

### <a name="can-i-use-traffic-manager-with-more-than-one-web-app-in-the-same-region"></a>流量管理器是否可用于同一区域中的多个 Web 应用？

通常情况下，流量管理器用于将流量引导到部署在不同区域中的应用程序。 不过，流量管理器也可用于一个应用程序在同一区域中存在多个部署的情形。 流量管理器 Azure 终结点不允许将同一 Azure 区域中的多个 Web 应用终结点添加到同一流量管理器配置文件。

执行以下步骤即可解除该约束：

1. 检查终结点是否在不同的 Web 应用“缩放单位”中。 域名必须映射到给定缩放单位中的单个站点。 因此，同一缩放单位中的两个 Web 应用不能共享一个流量管理器配置文件。
2. 将虚构域名作为自定义主机名添加到每个 Web 应用。 每个 Web 应用必须位于不同的缩放单位中。 所有 Web 应用必须属于同一个订阅。
3. 将一个（仅限一个）Web 应用终结点作为 Azure 终结点添加到流量管理器配置文件。
4. 将每个附加的 Web 应用终结点作为外部终结点添加到流量管理器配置文件。 只能使用 Resource Manager 部署模型添加外部终结点。
5. 在虚构域中创建一条指向流量管理器配置文件 DNS 名称 (<…>.trafficmanager.cn) 的 DNS CNAME 记录。
6. 通过虚构域名而不是流量管理器配置文件 DNS 名称来访问你的站点。

##  <a name="traffic-manager-endpoint-monitoring"></a>流量管理器终结点监视

### <a name="is-traffic-manager-resilient-to-azure-region-failures"></a>流量管理器能否灵活应对 Azure 区域故障？

流量管理器是在 Azure 中提供高可用性应用程序的关键组件。
若要提供高可用性，流量管理器必须具有超高的可用性，并且能够灵活应对区域故障。

在设计上，流量管理器组件能够灵活应对任何 Azure 区域的全面故障。 这样的应对能力见于所有流量管理器组件：DNS 名称服务器、API、存储层、终结点监视服务。

在整个 Azure 区域发生服务中断的情况下（这种情况很少见），流量管理器仍可继续正常运行。 在多个 Azure 区域中部署的应用程序可以依赖流量管理器将流量定向到这些区域中的可用应用程序实例。

### <a name="how-does-the-choice-of-resource-group-location-affect-traffic-manager"></a>选择资源组位置会如何影响流量管理器？

流量管理器属于单一的全局性服务， 而不是区域性服务。 选择资源组位置对部署在该资源组中的流量管理器配置文件没有影响。

Azure Resource Manager 要求所有资源组指定一个位置，这决定了部署在该资源组中的资源的默认位置。 创建流量管理器配置文件时，将在资源组中创建该配置文件。 所有流量管理器配置文件使用“**全局**”作为位置，覆盖资源组的默认值。

### <a name="how-do-i-determine-the-current-health-of-each-endpoint"></a>如何确定每个终结点的当前运行状况？

除了整个配置文件以外，每个终结点的当前监视状态也会显示在 Azure 门户预览中。 此信息也可通过流量监视器 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163667.aspx)、[PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt125941.aspx) 和[跨平台 Azure CLI](/documentation/articles/cli-install-nodejs/) 获取。

Azure 不提供有关过去终结点运行状况的历史信息，也不提供在终结点运行状况发生变化时引发警报的功能。

### <a name="can-i-monitor-https-endpoints"></a>能否监视 HTTPS 终结点？

是的。 流量管理器支持通过 HTTPS 进行探测。 在监视配置中将 **HTTPS** 配置为协议。

流量管理器无法提供任何证书验证，包括：

* 不验证服务器端证书
* 不支持 SNI 服务器端证书
* 不支持客户端证书

### <a name="what-host-header-do-endpoint-health-checks-use"></a>终结点运行状况检查使用什么主机头？

流量管理器在 HTTP 和 HTTPS 运行状况检查中使用 host 标头。 流量管理器使用的 host 标头是在配置文件中配置的终结点目标名称。 在主机头中使用的值不能通过目标属性单独指定。

## <a name="traffic-manager-nested-profiles"></a>流量管理器嵌套式配置文件

### <a name="how-do-i-configure-nested-profiles"></a>如何配置嵌套式配置文件？

可以使用 Azure Resource Manager、经典 Azure REST API、Azure PowerShell cmdlet 和跨平台 Azure CLI 命令配置嵌套式流量管理器配置文件。 也支持通过新 Azure 门户预览配置这些配置文件。 不支持使用经典管理门户。

### <a name="how-many-layers-of-nesting-does-traffic-manger-support"></a>流量管理器支持多少层嵌套？

配置文件的嵌套最深可达 10 层。 不允许使用“循环”。

### <a name="can-i-mix-other-endpoint-types-with-nested-child-profiles-in-the-same-traffic-manager-profile"></a>能否在同一流量管理器配置文件中将其他终结点类型与嵌套式子配置文件混合在一起使用？

是的。 至于如何在一个配置文件中组合使用不同类型的终结点，并无任何限制。

### <a name="how-does-the-billing-model-apply-for-nested-profiles"></a>嵌套式配置文件如何应用计费模型？

使用嵌套式配置文件在价格方面不会给你带来负面影响。

流量管理器计费分两部分：终结点运行状况检查以及数百万的 DNS 查询

* 终结点运行状况检查：在父配置文件中将子配置文件配置为终结点时，不对子配置文件收费。 在子配置文件中监视终结点按正常方式计费。
* DNS 查询：每个查询仅统计一次。 对父配置文件执行查询时，如果返回了子配置文件中的终结点，则仅统计父配置文件。

如需完整的详细信息，请参阅[流量管理器定价页](/pricing/details/traffic-manager/)。

### <a name="is-there-a-performance-impact-for-nested-profiles"></a>嵌套式配置文件是否会造成性能影响？

否。 使用嵌套式配置文件不会造成性能影响。

在处理每个 DNS 查询时，流量管理器名称服务器会在内部遍历配置文件层次结构。 对父配置文件执行 DNS 查询可能会收到终结点来自子配置文件的 DNS 响应。 不管使用的是单个配置文件还是嵌套式配置文件，都只使用一条 CNAME 记录。 不需要在层次结构中为每个配置文件创建一条 CNAME 记录。

### <a name="how-does-traffic-manager-compute-the-health-of-a-nested-endpoint-in-a-parent-profile"></a>在父配置文件中，流量管理器如何计算嵌套式终结点的运行状况？

父配置文件不会针对子配置文件直接执行运行状况检查。 子配置文件的终结点运行状况用于计算子配置文件的总体运行状况。 此信息在嵌套式配置文件层次结构中向上传播，确定嵌套式终结点的运行状况。 父配置文件使用此聚合运行状况信息确定是否可将流量定向到子配置文件。

下表描述了针对嵌套式终结点执行的流量管理器运行状况检查的行为。

| 子配置文件监视器状态 | 父级终结点监视器状态 | 说明 |
| --- | --- | --- |
| 已禁用。 子配置文件已禁用。 |已停止 |父级终结点状态为“已停止”，而不是“已禁用”。 “已禁用”状态保留用于指示你已在父配置文件中禁用了终结点。 |
| 已降级。 至少一个子配置文件终结点处于“已降级”状态。 |联机：子配置文件中处于“联机”状态的终结点的数目至少是 MinChildEndpoints 的值。<BR>正在检查终结点：子配置文件中处于“联机”和“正在检查终结点”状态的终结点的数目至少是 MinChildEndpoints 的值。<BR>已降级：其他。 |流量将路由到状态为“正在检查终结点”的终结点。 如果将 MinChildEndpoints 设置得过高，终结点将始终处于降级状态。 |
| 联机。 至少一个子配置文件终结点处于“联机”状态。 没有任何终结点处于“已降级”状态。 |见上。 | |
| 正在检查终结点。 至少一个子配置文件终结点处于“正在检查终结点”状态。 没有任何终结点处于“联机”或“已降级”状态 |同上。 | |
| 非活动。 所有子配置文件终结点都处于“Disabled”或“Stopped”状态，除非这是没有终结点的配置文件。 |已停止 | |