<properties 
   pageTitle="配置性能负载平衡方法 | Azure"
   description="本文将帮助你在流量管理器中配置性能负载平衡方法"
   services="traffic-manager"
   documentationCenter=""
   authors="joaoma"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.date="12/09/2015"
   wacn.date="01/21/2016" />

# 配置性能负载平衡方法

若要对分布在全球不同数据中心（也称为区域）的云服务和 Web 应用（终结点）进行负载平衡，可以将传入的流量从发出请求的客户端定向到延迟最低的终结点。通常，延迟最低的数据中心对应于地理距离最短的终结点。使用“性能”负载平衡方法可以基于最低延迟进行分发，但无法考虑网络配置或负载中的实时变化。有关流量管理器提供的各种负载平衡方法的详细信息，请参阅[关于流量管理器负载平衡方法](/documentation/articles/traffic-manager-load-balancing-methods)。

## 根据最低延迟在一组终结点之间路由流量：

1. 在管理门户中，在左窗格中，单击流量管理器图标以打开“流量管理器”窗格。如果尚未创建流量管理器配置文件，请参阅[管理流量管理器配置文件](/documentation/articles/traffic-manager-manage-profiles)来了解创建基本的流量管理器配置文件的步骤。
2. 在管理门户的“流量管理器”窗格中，找到包含你要修改的设置的流量管理器配置文件，然后单击配置文件名称右侧的箭头。这将打开配置文件的设置页面。
3. 在你的配置文件页面上，单击页面顶部的“终结点”并验证你要在配置中包括的服务终结点是否都存在。有关在配置文件中添加或删除终结点的步骤，请参阅[在流量管理器中管理终结点](/documentation/articles/traffic-manager-endpoints)。
4. 在你的配置文件页面上，单击顶部的“配置”以打开配置页面。
5. 对于“负载平衡方法设置”，验证负载平衡方法是否是“性能”。如果不是，请在下拉列表中单击“性能”。
6. 验证是否已正确配置了“监视设置”。监视可确保不会向处于脱机状态的终结点发送流量。为了监视终结点，必须指定路径和文件名。注意，正斜杠“/”是有效的相对路径条目，表示该文件位于根目录（默认值）中。有关监视的详细信息，请参阅[关于流量管理器监视](/documentation/articles/traffic-manager-monitoring)。
7. 完成你的配置更改后，单击页面底部的“保存”。
8. 测试你的配置更改。有关详细信息，请参阅[测试流量管理器设置](/documentation/articles/traffic-manager-testing-settings)。
9. 在流量管理器配置文件完成设置并正常工作后，在你的权威 DNS 服务器上编辑 DNS 记录以将你的公司域名指向流量管理器域名。有关如何执行此操作的详细信息，请参阅[将公司 Internet 域指向流量管理器域](/documentation/articles/traffic-manager-point-internet-domain)。

## 后续步骤

[关于流量管理器负载平衡方法](/documentation/articles/traffic-manager-load-balancing-methods)

[流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/disable-enable-or-delete-a-profile)

[流量管理器 - 禁用或启用终结点](/documentation/articles/disable-or-enable-an-endpoint)

[什么是流量管理器？](/documentation/articles/traffic-manager-overview)

[云服务](https://msdn.microsoft.com/zh-CN/library/jj155995.aspx)

[ Web 应用](/home/features/web-site/)

[流量管理器上的操作（REST API 参考）](https://msdn.microsoft.com/zh-CN/library/hh758255.aspx)

[Azure 流量管理器 Cmdlet](https://msdn.microsoft.com/zh-CN/library/dn690250.aspx)

<!---HONumber=71-->