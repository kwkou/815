<properties
    pageTitle="流量管理器工作原理 | Azure"
    description="本文介绍 Azure 流量管理器的工作原理"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date="01/03/2017"
    ms.author="sewhee"
/>  


# 流量管理器的工作方式

使用 Azure 流量管理器可以控制流量在应用程序终结点之间的分布。终结点可以是托管在 Azure 内部或外部的任何面向 Internet 的服务。

流量管理器具有两大优势：

1. 根据某个[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)对流量进行分布
2. [连续监视终结点运行状况](/documentation/articles/traffic-manager-monitoring/)，在终结点发生故障时自动进行故障转移

当客户端尝试连接到某个服务时，必须先将该服务的 DNS 名称解析成 IP 地址。然后，客户端就可以连接到该 IP 地址以访问相关服务。

**需要了解的最重要一点是，流量管理器在 DNS 级别工作。** 流量管理器根据流量路由方法的规则，使用 DNS 将客户端导向到特定的服务终结点。客户端**直接**连接到选定的终结点。流量管理器不是代理或网关。流量管理器看不到流量在客户端与服务之间传递。

## <a name="traffic-manager-example"></a>流量管理器示例

Contoso Corp 开发了一个新的合作伙伴门户。此门户的 URL 为 https://partners.contoso.com/login.aspx。该应用程序托管在三个 Azure 区域中。为了改善可用性并在全国最大程度地提高性能，他们使用流量管理器将客户端流量分布到最靠近的可用终结点。

为了实现该配置：

- 他们部署了服务的三个实例。这些部署的 DNS 名称为“contoso-us.chinacloudapp.cn”、“contoso-eu.chinacloudapp.cn”和“contoso-asia.chinacloudapp.cn”。
- 然后，他们创建了一个名为“contoso.trafficmanager.cn”的流量管理器配置文件，并将该文件配置为对三个终结点使用“性能”流量路由方法。
- 最后，他们使用 DNS CNAME 记录将其虚构域名“partners.contoso.com”配置为指向“contoso.trafficmanager.cn”。

![流量管理器 DNS 配置][1]  


> [AZURE.NOTE] 通过 Azure 流量管理器来使用虚构域时，必须使用 CNAME 将虚构域名指向流量管理器域名。DNS 标准不允许在域的“顶点”（或根）位置创建 CNAME。因此，无法为“contoso.com”（有时称为“裸”域）创建 CNAME。只能为“contoso.com”下的域（例如“www.contoso.com”）创建 CNAME。为了克服此限制，我们建议通过简单的 HTTP 重定向将针对“contoso.com”的请求定向到某个备用名称（例如“www.contoso.com”）。

## <a name="how-clients-connect-using-traffic-manager"></a>客户端如何使用流量管理器进行连接

沿用前面的示例，当客户端请求页面 https://partners.contoso.com/login.aspx 时，将会执行以下步骤来解析 DNS 名称并建立连接：

![使用流量管理器建立连接][2]  


1. 客户端向已配置的递归 DNS 服务发送 DNS 查询，以解析名称“partners.contoso.com”。递归 DNS 服务有时称为“本地 DNS”服务，并不直接托管 DNS 域。客户端将联系各种权威 DNS 服务的工作负荷转移到 Internet，以便解析 DNS 名称。
2. 为了解析 DNS 名称，递归 DNS 服务将查找“contoso.com”域的名称服务器。然后，它会联系这些名称服务器以请求“partners.contoso.com”DNS 记录。contoso.com DNS 服务器返回指向 contoso.trafficmanager.cn 的 CNAME 记录。
3. 接下来，递归 DNS 服务查找“trafficmanager.cn”域的名称服务器，这些服务器由 Azure 流量管理器服务提供。然后，将针对“contoso.trafficmanager.cn”DNS 记录发出的请求发送到这些 DNS 服务器。
4. 流量管理器名称服务器接收该请求。终结点的选择依据为：

    * 每个终结点的已配置状态（不返回已禁用的终结点）
    * 每个终结点的当前运行状况，可通过流量管理器运行状况检查来确定。有关详细信息，请参阅 [Traffic Manager Endpoint Monitoring](/documentation/articles/traffic-manager-monitoring/)（流量管理器终结点监视）。
    * 所选的流量路由方法。有关详细信息，请参阅 [Traffic Manager Routing Methods](/documentation/articles/traffic-manager-routing-methods/)（流量管理器路由方法）。

5. 选择的终结点以另一个 DNS CNAME 记录的形式返回。在本例中，假设返回的是 contoso-us.chinacloudapp.cn is returned。
6. 接下来，递归 DNS 服务将查找“chinacloudapp.cn”域的名称服务器。它会联系这些名称服务器以请求“contoso-us.chinacloudapp.cn”DNS 记录。返回的 DNS“A”记录包含位于美国的服务终结点的 IP 地址。
7. 递归 DNS 服务将结果合并，向客户端返回单个 DNS 响应。
8. 客户端接收 DNS 结果，然后连接到给定的 IP 地址。客户端直接连接到应用程序服务终结点，而不是通过流量管理器连接。由于这是一个 HTTPS 终结点，客户端将执行必要的 SSL/TLS 握手，然后针对“/login.aspx”页面发出 HTTP GET 请求。

递归 DNS 服务缓存它所收到的 DNS 响应。客户端设备上的 DNS 解析程序也会缓存结果。通过缓存可以加快后续 DNS 查询的响应速度，因为使用的是缓存中的数据，不需要查询其他名称服务器。缓存的持续时间取决于每个 DNS 记录的“生存时间”(TTL) 属性。该属性值越小，缓存过期时间就越短，因此访问流量管理器名称服务器所需的往返次数就越多。如果指定较大的值，则意味着从故障终结点定向流量需要更长的时间。使用流量管理器，你可以配置在流量管理器 DNS 响应中使用的 TTL，从而通过选择值对应用程序的需求进行最佳平衡。

## 常见问题

### 流量管理器使用什么 IP 地址？

如“流量管理器工作原理”中所述，流量管理器在 DNS 级别工作。它会发送 DNS 响应，将客户端定向到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

因此，流量管理器不提供供客户端连接的终结点或 IP 地址。因此，如果想要为服务使用静态 IP 地址，则必须在服务而不是流量管理器中配置该地址。

### 流量管理器是否支持“粘滞”会话？

[如上所述](#how-clients-connect-using-traffic-manager)，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。客户端直接连接到服务终结点，而不是通过流量管理器连接。因此，流量管理器看不到客户端与服务器之间的 HTTP 流量。

此外，流量管理器收到的 DNS 查询的源 IP 地址属于递归 DNS 服务而不是客户端。因此，流量管理器无法跟踪单个客户端，也无法实现“粘滞”会话。这种限制在所有基于 DNS 的流量管理系统中很常见，并不是流量管理器特有的限制。

### 使用流量管理器时为何出现 HTTP 错误？

[如上所述](#how-clients-connect-using-traffic-manager)，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。流量管理器看不到客户端与服务器之间的 HTTP 流量。因此，出现的任何 HTTP 错误必定来自应用程序。要使客户端连接到应用程序，必须完成所有 DNS 解析步骤。这包括流量管理器对应用程序流量流所做的任何交互。

因此，进一步的调查应着重于应用程序。

问题的最常见原因源自客户端浏览器发送的 HTTP 主机标头。请确保将应用程序配置为接受所要使用的域名的正确主机标头。对于使用 Azure 应用服务的终结点，请参阅 [configuring a custom domain name for a web app in Azure App Service using Traffic Manager](/documentation/articles/web-sites-traffic-manager-custom-domain-name/)（使用流量管理器为 Azure 应用服务中的 Web 应用配置自定义域名）。

### 使用流量管理器对性能有什么影响？

[如上所述](#how-clients-connect-using-traffic-manager)，流量管理器在 DNS 级别工作。由于客户端直接连接到你的服务终结点，因此在使用流量管理器时，一旦建立连接就没有性能影响。

由于流量管理器在 DNS 级别与应用程序集成，因此需要将额外的 DNS 查找插入 DNS 解析链中（请参阅 [Traffic Manager examples](#traffic-manager-example)（流量管理器示例））。流量管理器对 DNS 解析时间的影响微乎其微。流量管理器使用全局性的名称服务器网络，并使用[任播](https://en.wikipedia.org/wiki/Anycast)网络来确保始终将 DNS 查询路由到最靠近的可用名称服务器。此外，对 DNS 响应进行缓存意味着，因使用流量管理器而导致的额外的 DNS 延迟仅出现在部分会话中。

“性能”方法将流量路由到最靠近的可用终结点。最终结果是，与此方法相关的整体性能影响将会减到最小。通过减小与终结点之间的网络延迟，应该可以抵消 DNS 延迟增大造成的影响。

### 流量管理器允许使用什么应用程序协议？

[如上所述](#how-clients-connect-using-traffic-manager)，流量管理器在 DNS 级别工作。完成 DNS 查找以后，客户端会直接连接到应用程序终结点，不通过流量管理器进行连接。因此，连接可以使用任何应用程序协议。不过，流量管理器的终结点运行状况检查需要使用 HTTP 或 HTTPS 终结点。要执行运行状况检查的终结点可以不同于客户端连接到的应用程序终结点。

### 是否可以对“裸”域名使用流量管理器？

不可以。DNS 标准不允许 CNAME 与其他同名的 DNS 记录共存。DNS 区域的顶点（或根）始终包含两条预先存在的 DNS 记录：SOA 和权威 NS 记录。这意味着在不违反 DNS 标准的情况下，无法在区域顶点位置创建 CNAME 记录。

如 [Traffic Manager example](#traffic-manager-example)（流量管理器示例）中所述，流量管理器需要使用一条 DNS CNAME 记录来映射虚构 DNS 名称。例如，将 www.contoso.com 映射到流量管理器配置文件 DNS 名称 contoso.trafficmanager.cn。此外，流量管理器配置文件还会返回另一条 DNS CNAME 来指示客户端应连接到的终结点。

若要解决此问题，我们建议使用 HTTP 重定向将流量从裸域名定向到不同的 URL，然后即可使用流量管理器。例如，裸域“contoso.com”可将用户重定向到指向流量管理器 DNS 名称的 CNAME“www.contoso.com”。

在流量管理器中实现对裸域的完全支持已列入我们的待开发功能中。请[在我们的社区反馈站点上投票](https://feedback.azure.com/forums/217313-networking/suggestions/5485350-support-apex-naked-domains-more-seamlessly)，表达你对此项功能的支持。

## 后续步骤

详细了解流量管理器[终结点监视和自动故障转移](/documentation/articles/traffic-manager-monitoring/)。

详细了解流量管理器[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

<!--Image references-->

[1]: ./media/traffic-manager-how-traffic-manager-works/dns-configuration.png
[2]: ./media/traffic-manager-how-traffic-manager-works/flow.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->