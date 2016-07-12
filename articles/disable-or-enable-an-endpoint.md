<properties
   pageTitle="禁用或启用流量管理器终结点 | Azure"
   description="本文将帮助你禁用或启用流量管理器配置文件终结点。"
   services="traffic-manager"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
   ms.date="03/17/2016"
	wacn.date="04/26/2016"/>

# 禁用或启用流量管理器终结点

你还可以禁用属于流量管理器配置文件的一部分的个体终结点。终结点包括云服务和 Web 应用。禁用某个终结点会将其保留为配置文件的一部分，但是配置文件的行为就如同其中不包括该终结点一样。此操作对于临时删除处于维护模式或正在重新部署的终结点非常有用。终结点再次运行后，可以启用它。

>[AZURE.NOTE]**禁用某个终结点对其在 Azure 中的部署状态没有任何影响。正常的终结点将保持运行并能够接收流量，即使在流量管理器中已将其禁用也是如此。此外，在一个配置文件中禁用某个终结点不会影响它在其他配置文件中的状态。**

## 禁用终结点

1. 在经典管理门户的“流量管理器”窗格中，找到包含你要修改的终结点设置的流量管理器配置文件，然后单击配置文件名称右侧的箭头。这将打开配置文件的设置页面。
1. 在页面的顶部，单击“终结点”来查看你的配置中已包含的终结点。 
1. 单击你要禁用的终结点，然后单击页面底部的“禁用”。
1. 流量将根据为流量管理器域名配置的 DNS 生存时间 (TTL) 停止流向该终结点。你可以从流量管理器配置文件的“配置”页面更改 TTL。

## 启用终结点


1. 在经典管理门户的“流量管理器”窗格中，找到包含你要修改的终结点设置的流量管理器配置文件，然后单击配置文件名称右侧的箭头。这将打开配置文件的设置页面。
1. 在页面的顶部，单击“终结点”来查看你的配置中已包含的终结点。
1. 单击你要启用的终结点，然后单击页面底部的“启用”。
1. 流量将重新开始流向配置文件所指定的服务。

## 后续步骤

[流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/disable-enable-or-delete-a-profile/)

[流量管理器降级状态疑难解答](/documentation/articles/traffic-manager-troubleshooting-degraded/)

[流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations/)

<!---HONumber=Mooncake_1221_2015-->