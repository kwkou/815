<properties
    pageTitle="什么是流量管理器  | Azure"
    description="本文将有助于你了解什么是流量管理器，以及流量管理器是否是适合你应用程序的流量路由选择"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="75d5ff9a-f4b9-4b05-af32-700e7bdfea5a"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/16/2017"
    wacn.date="05/31/2017"
    ms.author="kumud"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="820fe65b1f0fe93497bbbdc811d05f9b8550e712"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="overview-of-traffic-manager"></a>流量管理器概述

使用 Azure 流量管理器可以控制用户流量在不同数据中心内的服务终结点上的分布。 流量管理器支持的服务终结点包括 Azure VM、Web 应用和云服务。 也可将流量管理器用于外部的非 Azure 终结点。

流量管理器根据[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)和终结点的运行状况，使用域名系统 (DNS) 将客户端请求定向到最合适的终结点。 流量管理器提供多种流量路由方法来满足不同的应用程序需求、终结点运行状况[监视](/documentation/articles/traffic-manager-monitoring/)和自动故障转移。 流量管理器能够灵活应对故障，包括整个 Azure 区域的故障。

## <a name="traffic-manager-benefits"></a>流量管理器优点

流量管理器可帮助你：

* **提高关键应用程序的可用性**

    流量管理器可以监视终结点，在终结点发生故障时提供自动故障转移，实现应用程序的高可用性。

* **改进高性能应用程序的响应能力**

    在 Azure 中，可以运行位于世界各地的数据中心内的云服务或网站。 流量管理器通过将流量定向到客户端网络延迟最低的终结点，改进应用程序的响应能力。

* **在不停机的情况下执行服务维护**

    无需停机即可在应用程序上执行计划内的维护操作。 在维护过程中，流量管理器会将流量定向到备用的终结点。

* **合并本地应用程序和基于云的应用程序**

    流量管理器支持外部非 Azure 终结点，因此可以用于混合云部署和本地部署，包括“云爆发”、“云迁移”和“云故障转移”方案。

* **分发大型复杂部署的流量**

    使用[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)可以合并流量路由方法，创建复杂、灵活的规则来满足更大、更复杂部署的需求。

## <a name="how-traffic-manager-works"></a>流量管理器的工作方式

使用 Azure 流量管理器可以控制流量在应用程序终结点之间的分布。 终结点可以是托管在 Azure 内部或外部的任何面向 Internet 的服务。

流量管理器具有两大优势：

1. 根据某个[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)对流量进行分布
2. [连续监视终结点运行状况](/documentation/articles/traffic-manager-monitoring/)，在终结点发生故障时自动进行故障转移

当客户端尝试连接到某个服务时，必须先将该服务的 DNS 名称解析成 IP 地址。 然后，客户端就可以连接到该 IP 地址以访问相关服务。

**需要了解的最重要一点是，流量管理器在 DNS 级别工作。**  流量管理器根据流量路由方法的规则，使用 DNS 将客户端导向到特定的服务终结点。 客户端 **直接**连接到选定的终结点。 流量管理器不是代理或网关。 流量管理器看不到流量在客户端与服务之间传递。

### <a name="traffic-manager-example"></a>流量管理器示例

Contoso Corp 开发了一个新的合作伙伴门户。 此门户的 URL 是 https://partners.contoso.com/login.aspx。 该应用程序托管在三个 Azure 区域中。 为了改善可用性并在全球最大程度地提高性能，他们使用流量管理器将客户端流量分布到最靠近的可用终结点。

为了实现该配置：

* 他们部署了服务的三个实例。 这些部署的 DNS 名称为“contoso-us.chinacloudapp.cn”、“contoso-eu.chinacloudapp.cn”和“contoso-asia.chinacloudapp.cn”。
* 然后，他们创建了一个名为“contoso.trafficmanager.cn”的流量管理器配置文件，并将该文件配置为对三个终结点使用“性能”流量路由方法。
* 最后，他们使用 DNS CNAME 记录将其虚构域名“partners.contoso.com”配置为指向“contoso.trafficmanager.cn”。

![流量管理器 DNS 配置][1]

> [AZURE.NOTE]
> 通过 Azure 流量管理器来使用虚构域时，必须使用 CNAME 将虚构域名指向流量管理器域名。 DNS 标准不允许在域的“顶点”（或根）位置创建 CNAME。 因此，无法为“contoso.com”（有时称为“裸”域）创建 CNAME。 只能为“contoso.com”下的域（例如“www.contoso.com”）创建 CNAME。 为了克服此限制，我们建议通过简单的 HTTP 重定向将针对“contoso.com”的请求定向到某个备用名称（例如“www.contoso.com”）。

### <a name="how-clients-connect-using-traffic-manager"></a>客户端如何使用流量管理器进行连接

沿用前面的示例，当客户端请求页面 https://partners.contoso.com/login.aspx 时，将会执行以下步骤来解析 DNS 名称并建立连接：

![使用流量管理器建立连接][2]

1. 客户端向已配置的递归 DNS 服务发送 DNS 查询，以解析名称“partners.contoso.com”。 递归 DNS 服务有时称为“本地 DNS”服务，并不直接托管 DNS 域。 客户端将联系各种权威 DNS 服务的工作负荷转移到 Internet，以便解析 DNS 名称。
2. 为了解析 DNS 名称，递归 DNS 服务将查找“contoso.com”域的名称服务器。 然后，它会联系这些名称服务器以请求“partners.contoso.com”DNS 记录。 contoso.com DNS 服务器返回指向 contoso.trafficmanager.cn 的 CNAME 记录。
3. 接下来，递归 DNS 服务查找“trafficmanager.cn”域的名称服务器，这些服务器由 Azure 流量管理器服务提供。 然后，将针对“contoso.trafficmanager.cn”DNS 记录发出的请求发送到这些 DNS 服务器。
4. 流量管理器名称服务器接收该请求。 终结点的选择依据为：

    - 每个终结点的已配置状态（不返回已禁用的终结点）
    - 每个终结点的当前运行状况，可通过流量管理器运行状况检查来确定。 有关详细信息，请参阅[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/)。
    - 所选的流量路由方法。 有关详细信息，请参阅[流量管理器路由方法](/documentation/articles/traffic-manager-routing-methods/)。

5. 选择的终结点以另一个 DNS CNAME 记录的形式返回。 在本例中，假设返回的是 contoso-us.chinacloudapp.cn is returned。
6. 接下来，递归 DNS 服务将查找“chinacloudapp.cn”域的名称服务器。 它会联系这些名称服务器以请求“contoso-us.chinacloudapp.cn”DNS 记录。 返回的 DNS“A”记录包含位于美国的服务终结点的 IP 地址。
7. 递归 DNS 服务将结果合并，向客户端返回单个 DNS 响应。
8. 客户端接收 DNS 结果，然后连接到给定的 IP 地址。 客户端直接连接到应用程序服务终结点，而不是通过流量管理器连接。 由于这是一个 HTTPS 终结点，客户端将执行必要的 SSL/TLS 握手，然后针对“/login.aspx”页面发出 HTTP GET 请求。

递归 DNS 服务缓存它所收到的 DNS 响应。 客户端设备上的 DNS 解析程序也会缓存结果。 通过缓存可以加快后续 DNS 查询的响应速度，因为使用的是缓存中的数据，不需要查询其他名称服务器。 缓存的持续时间取决于每个 DNS 记录的“生存时间”(TTL) 属性。 该属性值越小，缓存过期时间就越短，因此访问流量管理器名称服务器所需的往返次数就越多。 如果指定较大的值，则意味着从故障终结点定向流量需要更长的时间。 使用流量管理器，你可以配置在流量管理器 DNS 响应中使用的 TTL，从而通过选择值对应用程序的需求进行最佳平衡。

## <a name="pricing"></a>定价

有关定价信息，请参阅[流量管理器定价](/pricing/details/traffic-manager/)。

## <a name="faq"></a>常见问题

有关流量管理器的常见问题，请参阅[流量管理器常见问题解答](/documentation/articles/traffic-manager-FAQs/)

## <a name="next-steps"></a>后续步骤

详细了解流量管理器[终结点监视和自动故障转移](/documentation/articles/traffic-manager-monitoring/)。

详细了解流量管理器[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

<!--Image references-->
[1]: ./media/traffic-manager-how-traffic-manager-works/dns-configuration.png
[2]: ./media/traffic-manager-how-traffic-manager-works/flow.png

<!--Update_Description: add FAQ-->