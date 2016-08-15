<properties
   pageTitle="配置流量管理器轮循机制流量路由方法 | Azure"
   description="本文将帮助你为流量管理器终结点配置“循环”负载平衡。"
   services="traffic-manager"
   documentationCenter=""
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
   ms.date="03/17/2016"
	wacn.date="04/26/2016"/>

# 配置轮循机制路由方法

一种常见的流量路由方法模式是提供一组相同的终结点（包括云服务和 Web 应用），并以循环方式向每个终结点发送流量。下面的步骤概述了如何对流量管理器进行配置以执行此类型的流量路由方法。有关不同流量路由方法的详细信息，请参阅[关于流量管理器流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

>[AZURE.NOTE]Azure Web 应用已经针对数据中心（也称为区域）内的 Web 应用提供了“循环”负载平衡功能。你可以使用流量管理器为不同数据中心内的 Web 应用指定轮询机制流量路由方法。

## 均衡地（轮循机制）在一组终结点之间路由流量：

1. 在经典管理门户中，在左窗格中，单击流量管理器图标以打开“流量管理器”窗格。如果尚未创建流量管理器配置文件，请参阅[管理流量管理器配置文件](/documentation/articles/traffic-manager-manage-profiles/)来了解创建基本的流量管理器配置文件的步骤。
2. 在经典管理门户的“流量管理器”窗格中，找到包含你要修改的设置的流量管理器配置文件，然后单击配置文件名称右侧的箭头。这将打开配置文件的设置页面。
3. 在你的配置文件页面上，单击页面顶部的“终结点”并验证你要在配置中包括的服务终结点是否都存在。有关添加或删除终结点的步骤，请参阅[在流量管理器中管理终结点](/documentation/articles/traffic-manager-endpoints/)。
4. 在你的配置文件页面上，单击顶部的“配置”以打开配置页面。
5. 对于“流量路由方法设置”，验证流量路由方法是否是“轮循机制”。如果不是，请在下拉列表中单击“轮循机制”。
6. 验证是否已正确配置了“监视设置”。监视可确保不会向处于脱机状态的终结点发送流量。为了监视终结点，必须指定路径和文件名。注意，正斜杠“/”是有效的相对路径条目，表示该文件位于根目录（默认值）中。有关监视的详细信息，请参阅[关于流量管理器监视](/documentation/articles/traffic-manager-monitoring/)。
7. 完成你的配置更改后，单击页面底部的“保存”。
8. 测试你的配置更改。有关详细信息，请参阅[测试流量管理器设置](/documentation/articles/traffic-manager-testing-settings/)。
9. 在流量管理器配置文件完成设置并正常工作后，在你的权威 DNS 服务器上编辑 DNS 记录以将你的公司域名指向流量管理器域名。有关如何执行此操作的详细信息，请参阅[将公司 Internet 域指向流量管理器域](/documentation/articles/traffic-manager-point-internet-domain/)。

## 后续步骤


[将公司 Internet 域指向流量管理器域](/documentation/articles/traffic-manager-point-internet-domain/)

[流量管理器路由方法](/documentation/articles/traffic-manager-routing-methods/)

[配置故障转移路由方法](/documentation/articles/traffic-manager-configure-failover-routing-method/)

[配置性能路由方法](/documentation/articles/traffic-manager-configure-performance-routing-method/)

[流量管理器降级状态疑难解答](/documentation/articles/traffic-manager-troubleshooting-degraded/)

[流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/disable-enable-or-delete-a-profile/)

[流量管理器 - 禁用或启用终结点](/documentation/articles/disable-or-enable-an-endpoint/)

 

<!---HONumber=Mooncake_1221_2015-->