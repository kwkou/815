<properties
   pageTitle="流量管理器工作原理 | Azure"
   description="本文将帮助你理解 Azure 流量管理器工作原理"
   services="traffic-manager"
   documentationCenter=""
   authors="jtuliani"
   manager="carmonm"
   editor="tysonn"/>

<tags
	ms.service="traffic-manager"
	ms.date="06/07/2016"
	wacn.date="07/04/2016"/>

# 流量管理器的工作方式

可以使用 Azure 流量管理器来控制流量在应用程序终结点中的分布情况。终结点可以是任何面向 Internet 的终结点，托管在 Azure 内部或外部。

流量管理器具有两大优势：

1. 根据某个[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)对流量进行分布
2. [连续监视终结点运行状况](/documentation/articles/traffic-manager-monitoring/)，在终结点发生故障时自动进行故障转移

当最终用户尝试连接到服务终结点时，其客户端（电脑、手机等）必须首先将该终结点中的 DNS 名称解析为 IP 地址。然后，客户端就可以连接到该 IP 地址以访问相关服务。

**需要了解的最重要一点是，流量管理器在 DNS 级别工作。** 流量管理器会根据所选的流量路由方法和当前的终结点运行状况，使用 DNS 将最终用户引导到特定的服务终结点。然后，客户端即可**直接**连接到所选终结点。流量管理器不是代理，看不到流量在客户端和服务之间传递。

##<a name="traffic-manager-example"></a> 流量管理器示例

Contoso Corp 开发了一个新的合作伙伴门户。此门户的 URL 将是 https://partners.contoso.com/login.aspx 。该应用程序托管在 Azure 中。为了改进可用性并最大程度地提高全局性能，他们希望将应用程序部署到全世界的 3 个区域，并使用流量管理器将最终用户分配到最近的可用终结点。

为了实现该配置：

- 他们部署了服务的 3 个实例。这些部署的 DNS 名称为“contoso-us.chinacloudapp.cn”、“contoso-eu.chinacloudapp.cn”和“contoso-asia.chinacloudapp.cn”。
- 然后，他们创建了一个名为“contoso.trafficmanager.cn”的流量管理器配置文件，并将该文件配置为对上述 3 个终结点使用“性能”流量路由方法。
- 最后，他们使用 DNS CNAME 记录将其虚构域“partners.contoso.com”配置为指向“contoso.trafficmanager.cn”。

![流量管理器 DNS 配置][1]

> [AZURE.NOTE] 通过 Azure 流量管理器来使用虚构域时，必须使用 CNAME 将虚构域名指向流量管理器域名。

> 根据 DNS 标准的限制，不能在域的“顶点”（或根）位置创建 CNAME。因此，无法为“contoso.com”（有时称为“裸”域）创建 CNAME。只能为“contoso.com”下的域（例如“www.contoso.com”）创建 CNAME。

> 因此，你不能直接将流量管理器用于裸域。我们建议你通过简单的 HTTP 重定向将针对“contoso.com”的请求定向到某个备用名称（例如“www.contoso.com”），以便解决此问题。

##<a name="how-clients-connect-using-traffic-manager"></a> 客户端如何使用流量管理器进行连接

当最终用户请求页面 https://partners.contoso.com/login.aspx （如以上示例所述）时，其客户端将执行以下步骤，以便解析 DNS 名称并建立连接。

![使用流量管理器建立连接][2]

1.	客户端（电脑、手机等）向其配置的递归 DNS 服务发出针对“partners.contoso.com”的 DNS 查询。（递归 DNS 服务有时称为“本地 DNS”服务，并不直接托管 DNS 域，但客户端可以使用它将联系各种权威 DNS 服务的必要工作转移到整个 Internet，以便解析 DNS 名称。）
2.	递归 DNS 服务现在可以解析“partners.contoso.com”DNS 名称了。首先，递归 DNS 服务查找“contoso.com”域的名称服务器。然后，它会联系这些名称服务器以请求“partners.contoso.com”DNS 记录。将返回指向 contoso.trafficmanager.cn 的 CNAME。
3.	递归 DNS 服务此时会查找“trafficmanager.net”域的名称服务器，这些服务器由 Azure 流量管理器服务提供。它会联系这些名称服务器以请求“contoso.trafficmanager.cn”DNS 记录。
4.	流量管理器名称服务器接收该请求。然后，这些名称服务器会根据以下标准选择应返回的终结点：
a.	每个终结点的“已启用/已禁用”状态（不返回已禁用的终结点）
b.	每个终结点的当前运行状况，可通过流量管理器运行状况检查来确定。有关详细信息，请参阅“流量管理器终结点监视”。
c.	所选的流量路由方法。有关详细信息，请参阅“流量管理器流量路由方法”。
5.	所选终结点是作为另一 DNS CNAME 记录返回的，在本示例中，假设返回了 contoso-us.chinacloudapp.cn。
6.	现在，递归 DNS 服务会查找“chinacloudapp.cn”域的名称服务器。它会联系这些名称服务器以请求“contoso-us.chinacloudapp.cn”DNS 记录。返回的 DNS“A”记录包含位于美国的服务终结点的 IP 地址。
7.	递归 DNS 服务返回给客户端的是合并的结果，其中包含上述名称解析序列。
8.	客户端从递归 DNS 服务接收 DNS 结果，然后连接到给定 IP 地址。请注意，客户端直接连接到应用程序服务终结点，不通过流量管理器进行连接。由于是 HTTPS 终结点，因此客户端会执行必需的 SSL/TLS 握手，然后针对“/login.aspx”页进行 HTTP GET 请求。

请注意，递归 DNS 服务会缓存所接收的 DNS 响应，就像最终用户的设备上的 DNS 客户端一样。这样可以加快后续 DNS 查询的响应速度，因为使用的是缓存中的数据，不需查询其他名称服务器。缓存的持续时间取决于每个 DNS 记录的“生存时间”(TTL) 属性。时间值较小会导致缓存更快地到期，因此到流量管理器名称服务器的往返次数会更多；时间值较大意味着可能需要更长的时间才能将流量从发生故障的终结点引导开。使用流量管理器，你可以配置在流量管理器 DNS 响应中使用的 TTL，从而通过选择值对应用程序的需求进行最佳平衡。

## 常见问题

### 流量管理器使用什么 IP 地址？

如“流量管理器工作原理”中所述，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

因此，流量管理器不提供供客户端连接的终结点或 IP 地址。因此，举例来说，如果需要一个静态 IP 地址，则必须在服务中而不是流量管理器中进行相关配置。

### 流量管理器是否支持“粘滞”会话？

如[上](#how-clients-connect-using-traffic-manager)所述，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。因此，流量管理器看不到客户端和服务器之间的 HTTP 流量，包括 Cookie。

另请注意，流量管理器接收到的 DNS 查询的源 IP 地址是递归 DNS 服务的 IP 地址，而不是客户端的 IP 地址。

因此，流量管理器无法标识或跟踪各个客户端，因此无法实现“粘滞”会话。这在所有基于 DNS 的流量管理系统中很常见，不是使用流量管理器所带来的限制。

### 我在使用流量管理器时出现 HTTP 错误，这是什么原因？

如[上](#how-clients-connect-using-traffic-manager)所述，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

因此，流量管理器看不到客户端和服务器之间的 HTTP 流量，不可能生成 HTTP 级别错误。你看到的任何 HTTP 错误必定来自应用程序。由于客户端是连接到应用程序的，这也意味着 DNS 解析（包括流量管理器的角色）必定已完成。

因此，进一步的调查应着重于应用程序。

常见问题是，在使用流量管理器时，由浏览器传递给应用程序的“host”HTTP 标头将显示浏览器中使用的域名。该域名可能是流量管理器域名（例如 myprofile.trafficmanager.cn，如果你在测试中使用该域名），也可能是配置为指向流量管理器域名的虚构域 CNAME。不管是哪种情况，请确保将应用程序配置为接受该主机头。

如果你的应用程序是托管在 Azure Web 应用中的，请参阅[在 Azure 中使用流量管理器为 Web 应用配置自定义域名](/documentation/articles/web-sites-traffic-manager-custom-domain-name/)。

### 使用流量管理器对性能有什么影响？

如[上](#how-clients-connect-using-traffic-manager)所述，流量管理器在 DNS 级别工作。它使用 DNS 响应将客户端引导到相应的服务终结点。然后，客户端直接连接到服务终结点，不通过流量管理器进行连接。

由于客户端直接连接到你的服务终结点，因此在使用流量管理器时，一旦建立连接就没有性能影响。

由于流量管理器在 DNS 级别集成应用程序，因此需要将额外的 DNS 查找插入 DNS 解析链中（请参阅[流量管理器示例](#traffic-manager-example)）。流量管理器对 DNS 解析时间的影响微乎其微。流量管理器使用全局性的名称服务器网络，并使用任播网络来确保始终将 DNS 查询路由到最靠近的可用名称服务器。此外，对 DNS 响应进行缓存意味着，因使用流量管理器而导致的额外的 DNS 延迟仅出现在部分会话中。

最终结果是，将流量管理器加入应用程序中带来的总体性能影响将是微乎其微的。

此外，在使用流量管理器的[“性能”流量路由方法](/documentation/articles/traffic-manager-routing-methods/#performance-traffic-routing-method)时，可以通过将最终用户路由到最靠近的可用终结点来改进性能，从而部分抵消因 DNS 延迟增加而带来的影响。

### 流量管理器允许使用什么应用程序协议？
如[上](#how-clients-connect-using-traffic-manager)所述，流量管理器在 DNS 级别工作。完成 DNS 查找以后，客户端会直接连接到应用程序终结点，不通过流量管理器进行连接。因此，在进行这样的连接时，可以使用任何应用程序协议。

不过，流量管理器的终结点运行状况检查需要使用 HTTP 或 HTTPS 终结点。这可以与客户端所连接到的应用程序终结点隔离开来，只需在流量管理器配置文件运行状况检查设置中指定其他 TCP 端口或 URI 路径即可。

### 是否可以对“裸”域名（不带 www 的域名）使用流量管理器？

目前不可以。

DNS CNAME 记录类型用于创建从一个 DNS 名称到另一个名称的映射。如[流量管理器示例](#traffic-manager-example)中所述，流量管理器需要使用 DNS CNAME 记录将虚构 DNS 名称（例如 www.contoso.com ）映射到流量管理器配置文件 DNS 名称（例如 contoso.trafficmanager.cn）。此外，流量管理器配置文件还会自行返回另一个 DNS CNAME 来指示客户端应连接到的终结点。

DNS 标准不允许 CNAME 与同一类型的其他 DNS 记录共存。由于 DNS 区域的顶点（或根）始终包含两个预先存在的 DNS 记录（SOA 记录和权威性的 NS 记录），这意味着，在不违反 DNS 标准的情况下，无法在区域顶点位置创建 CNAME 记录。

为了解决此问题，我们建议，如果使用裸域（不带 www 的域）的服务需要使用流量管理器，则应让这些服务使用 HTTP 重定向将流量从裸域定向到另一 URL，然后再让后者使用流量管理器。例如，可以通过裸域“contoso.com”将用户重定向到“www.contoso.com”，然后再让后者使用流量管理器。

在流量管理器中实现对裸域的完全支持已列入我们的待开发功能中。如果你对此功能感兴趣，请[在我们的社区反馈站点上投票](https://feedback.azure.com/forums/217313-networking/suggestions/5485350-support-apex-naked-domains-more-seamlessly)以表达你对此功能的支持。

## 后续步骤

详细了解流量管理器[终结点监视和自动故障转移](/documentation/articles/traffic-manager-monitoring/)。

详细了解流量管理器[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

<!--Image references-->
[1]: ./media/traffic-manager-how-traffic-manager-works/dns-configuration.png
[2]: ./media/traffic-manager-how-traffic-manager-works/flow.png


<!---HONumber=Mooncake_0627_2016-->