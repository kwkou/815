域名系统 (DNS) 用于在 Internet 上查找内容。例如，当您在浏览器中输入一个地址或单击 Web 页面上的某个链接时，它使用 DNS 将域名转换为 IP 地址。IP 地址有点像街道地址，但其用户友好性并不是很好。例如，记住 **contoso.com** 这样的 DNS 名称比较容易，而记住 192.168.1.88 或 2001:0:4137:1f67:24a2:3888:9cce:fea3 这样的 IP 地址则要困难得多。

DNS 系统基于 *records*。记录将特定的 *name*（例如 **contoso.com**）与一个 IP 地址或其他的 DNS 名称相关联。当某个应用程序（例如 Web 浏览器）在 DNS 中查找某个名称，它将找到该记录，并将它所指向的内容用作地址。如果它所指向的值是 IP 地址，则浏览器将使用该值。如果它指向另一个 DNS 名称，则应用程序必须再次执行解析。最终，所有名称解析都将以 IP 地址的形式结束。

当您创建 Azure Web 应用时，DNS 名称将自动分配到站点。此名称采用 **&lt;yoursitename&gt;.chinacloudsites.cn** 的格式。当您将您的 Web 应用添加为 Azure Traffic Manager 终结点时，将可以通过 **&lt;yourtrafficmanagerprofile&gt;.trafficmanager.cn** 域访问您的 Web 应用。

> [WACOM.NOTE] 当您的 Web 应用配置为 Traffic Manager 终结点时，您将在创建 DNS 记录时使用 **.trafficmanager.cn** 地址。

> [WACOM.NOTE] 您只能为 Traffic Manager 使用 CNAME 记录。

此外还有多种类型的记录，每种类型都有其自己的功能和限制，但是对于配置为 Traffic Manager 终结点的 Web 应用，我们只关心一种；即  *CNAME* 记录。

###CNAME 或别名记录

CNAME 记录将  *specific* DNS 名称（例如 **mail.contoso.com** 或 **www.contoso.com**）映射到另一个（规范）域名。对于使用 Traffic Manager 的 Azure Web 应用；规范域名是您的 Traffic Manager 配置文件的 **&lt;myapp>.trafficmanager.cn** 域名。创建后，CNAME 将为 **&lt;myapp>.trafficmanager.cn** 域名创建一个别名。CNAME 条目将自动解析为您的 **&lt;myapp>.trafficmanager.cn** 的 IP 地址，因此，如果 Web 应用的 IP 地址发生更改，您不必采取任何操作。

一旦流量到达 Traffic Manager，后者随后会使用它为流量配置的负载平衡方法，将该流量路由到您的 Web 应用。这对您的 Web 应用访问者完全透明。他们将只在浏览器中看到自定义域。

> [WACOM.NOTE] 某些域注册机构只允许您在使用 CNAME 记录（例如 **www.contoso.com**）而不是根名称（例如 **contoso.com**）时映射子域。有关 CNAME 记录的详细信息，请参阅由您的注册机构提供的文档、<a href="http://en.wikipedia.org/wiki/CNAME_record">关于 CNAME 记录的 Wikipedia 条目</a>或 <a href="http://tools.ietf.org/html/rfc1035">IETF 域名 - 实现和规范</a>文档。
<!--HONumber=41-->
