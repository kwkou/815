<properties
    pageTitle="Azure 流量管理器 - 流量路由方法 | Azure"
    description="本文将帮助你了解流量管理器使用的各种流量路由方法。"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="db1efbf6-6762-4c7a-ac99-675d4eeb54d0"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/22/2017"
    wacn.date="05/02/2017"
    ms.author="kumud"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="6fd97863aa9e103b3691f2e212a5aee86159e599"
    ms.lasthandoff="04/22/2017" />

# <a name="traffic-manager-routing-methods"></a>流量管理器路由方法

Azure 流量管理器支持使用三种流量路由方法来确定如何将网络流量路由到不同的服务终结点。 流量管理器将流量路由方法应用于它收到的每个 DNS 查询。 流量路由方法确定要在 DNS 响应中返回哪个终结点。

流量管理器中提供了四种流量路由方法：

* **优先级：**如果想要使用主服务终结点来处理所有流量，并提供备份来防范主终结点或备份终结点不可用的情况，可以选择“优先级”。
* **加权：** 如果想要跨一组终结点来分配流量，不管是平均分配还是根据所定义的权重进行分配，可以选择“加权”。
* **性能：**如果终结点位于不同的地理位置，并且你希望最终用户依据最低网络延迟使用“最近的”终结点，可以选择“性能”。

所有流量管理器配置文件都包括监视终结点运行状况以及终结点自动故障转移的设置。 有关详细信息，请参阅[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/)。 一个流量管理器配置文件只能使用一种流量路由方法。 你可以随时为配置文件选择其他流量路由方法。 一分钟内即可应用所做的更改，不会导致停机。 可以通过嵌套式流量管理器配置文件来组合使用多种流量路由方法。 使用嵌套可以启用复杂且灵活的流量路由配置，满足更大、更复杂应用程序的需求。 有关详细信息，请参阅[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)。

## <a name="priority-traffic-routing-method"></a> “优先级”流量路由方法

组织往往会提供一个或多个备份服务来防范主服务发生故障，从而确保其服务的可靠性。 Azure 客户可以通过“优先级”流量路由方法来轻松实现此故障转移模式。

![Azure 流量管理器的“优先级”流量路由方法][1]

流量管理器配置文件包含服务终结点的优先顺序列表。 默认情况下，流量管理器将所有流量发送到主终结点（优先级最高）。 如果主终结点不可用，流量管理器会将流量路由到第二个终结点。 如果主终结点和辅助终结点都不可用，流量将转到第三个终结点，依此类推。 终结点的可用性取决于配置的状态（已启用或已禁用）和正在进行的终结点监视。

### <a name="configuring-endpoints"></a>配置终结点

在 Azure Resource Manager 中，可以使用每个终结点的“priority”属性显式配置终结点优先级。 此属性是一个介于 1 和 1000 之间的值。 值越小，优先级越高。 终结点不能共享优先级值。 该属性的设置是可选的。 如果省略该属性，将会根据终结点顺序使用默认优先级。

## <a name="weighted-traffic-routing-method"></a> “加权”流量路由方法
使用“加权”流量路由方法可以均匀分布流量，或使用预定义的权重。

![Azure 流量管理器的“加权”流量路由方法][2]

在“加权”流量路由方法中，需要为流量管理器配置文件中的每个终结点分配一个权重。 该权重是从 1 到 1000 的整数。 此参数是可选的。 如果省略此参数，流量管理器将使用默认权重“1”。

对于收到的每个 DNS 查询，流量管理器会随机选择一个可用终结点。 选择哪个终结点取决于分配到所有可用终结点的权重。 对所有终结点使用相同的权重会导致均匀分布流量。 对特定的终结点使用较高或较低的权重会导致这些终结点在 DNS 响应中的返回次数较多或较少。

加权方法可以实现一些有用的方案：

* 应用程序逐步升级：分配要路由到新终结点的流量百分比，并随着时间的推移逐渐将流量增加到 100％。
* 将应用程序迁移到 Azure：创建包含 Azure 终结点和外部终结点的配置文件。 调整终结点的权重，优先选择新终结点。
* 适用于更多容量的云爆发：通过将本地部署放在流量管理器配置文件之后，快速将本地部署扩展到云中。 当你需要在云中获得额外的容量时，可以添加或启用更多终结点，并指定哪部分流量将流向每个终结点。

Azure 门户支持加权流量路由的配置。  也可以使用 Resource Manager 版本的 Azure PowerShell、CLI 和 REST API 配置权重。

必须知道，客户端及其用来解析 DNS 名称的递归 DNS 服务器会缓存 DNS 响应。 这种缓存可能会影响到加权流量分布。 如果客户端和递归 DNS 服务器的数目较大，流量分布将按预期工作。 但是，如果客户端或递归 DNS 服务器的数目较小，缓存可能会严重影响流量分布。

常见用例包括：

* 开发和测试环境
* 应用程序间通信
* 目标用户群体较小、共享一个公用递归 DNS 基础结构的应用程序（例如，公司的员工通过代理进行连接）

这些 DNS 缓存影响常见于所有基于 DNS 的流量路由系统，不只存在于 Azure 流量管理器上。 在某些情况下，显式清除 DNS 缓存可能会解决问题。 在另外一些情况下，可能更适合使用替代性流量路由方法。

## <a name="performance-traffic-routing-method"></a> “性能”流量路由方法

在全国两个位置部署终结点，将流量路由到“最靠近”用户的位置，即可改善许多应用程序的响应能力。 “性能”流量路由方法提供这种能力。

![Azure 流量管理器的“性能”流量路由方法][3]

“最靠近”的终结点不一定是地理距离最近的终结点。 “性能”流量路由方法通过测试网络延迟来确定最靠近的终结点。 流量管理器维护一份 Internet 延迟表，用于跟踪 IP 地址范围与每个 Azure 数据中心之间的往返时间。

流量管理器在 Internet 延迟表中查找传入 DNS 请求的源 IP 地址。 流量管理器在处理该 IP 地址范围的请求时保持最低延迟的 Azure 数据中心内选择一个可用终结点，然后在 DNS 响应中返回该终结点。

如[流量管理器工作原理](/documentation/articles/traffic-manager-overview/#how-traffic-manager-works)中所述，流量管理器不会直接从客户端接收 DNS 查询。 DNS 查询来自客户端配置使用的递归 DNS 服务。 因此，用于确定“最靠近”终结点的 IP 地址不是客户端的 IP 地址，而是递归 DNS 服务的 IP 地址。 在实践中，此 IP 地址是客户端的适当代理。

流量管理器定期更新 Internet 延迟表，反映全国 Internet 的变化以及新的 Azure 区域。 但是，由于 Internet 上的负载会实时变化，应用程序性能也会随之变化。 “性能”流量路由不会监视给定服务终结点上的负载。 但是，如果某个终结点变得不可用，则流量管理器不会在 DNS 查询响应中包括该终结点。

需要注意的要点：

* 如果配置文件包含同一 Azure 区域中的多个终结点，流量管理器会在该区域中的可用终结点之间均匀分布流量。 如果想要在某个区域中采用不同的流量分布，可以使用[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)。
* 如果给定 Azure 区域中的所有已启用终结点已降级，流量管理器会将流量分布到其他所有可用终结点，而不是下一个最靠近的终结点。 这种逻辑不会使下一个最靠近的终结点过载，从而可以防止发生连锁故障。 如果想要定义首选故障转移顺序，请使用[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)。
* 对外部终结点或嵌套式终结点使用“性能”流量路由方法时，需要指定这些终结点的位置。 请选择离你的部署最近的 Azure 区域。 这些位置是 Internet 延迟表支持的值。
* 选择终结点的算法是确定性的。 同一个客户端重复发出的 DNS 查询会定向到同一个终结点。 通常，客户端在遍历时使用不同的递归 DNS 服务器。 客户端可以路由到不同的终结点。 更新 Internet 延迟表也可能会影响路由。 因此，“性能”流量路由方法不保证始终将客户端路由到同一个终结点。
* 更改 Internet 延迟表时，可能会注意到某些客户端被定向到其他终结点。 这种路由更改基于当前延迟数据，因此更加准确。 进行这些更新很重要，可以确保在 Internet 持续发展的情况下“性能”流量路由的准确性。

## <a name="next-steps"></a>后续步骤

了解如何使用[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/)开发高可用性应用程序

了解如何[创建流量管理器配置文件](/documentation/articles/traffic-manager-create-profile/)

<!--Image references-->
[1]: ./media/traffic-manager-routing-methods/priority.png
[2]: ./media/traffic-manager-routing-methods/weighted.png
[3]: ./media/traffic-manager-routing-methods/performance.png

<!--Update_Description: wording update-->