<properties
    pageTitle="在 Azure 流量管理器中管理终结点 | Azure"
    description="本文将帮助你从 Azure 流量管理器中添加、删除、启用和禁用终结点。"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date="11/07/2016"
    ms.author="sewhee"
/>  


# 添加、禁用、启用或删除终结点

无论网站模式如何，Azure 应用服务中的 Web 应用功能都已针对数据中心内的网站提供了故障转移和轮循机制流量路由功能。你可以使用 Azure 流量管理器为不同数据中心内的网站和云服务指定故障转移和轮询机制流量路由。提供该功能所需的第一个步骤是将云服务或网站终结点添加到流量管理器中。

>[AZURE.NOTE]  本文介绍如何使用经典管理门户。Azure 经典管理门户仅支持以终结点的形式创建和分配云服务和 Web 应用。新的 [Azure 门户预览](https://portal.azure.cn)是首选界面。

你还可以禁用属于流量管理器配置文件的一部分的个体终结点。禁用某个终结点会将其保留为配置文件的一部分，但是配置文件的行为就如同其中不包括该终结点一样。此操作对于临时删除处于维护模式或正在重新部署的终结点十分有用。终结点再次运行后，可以启用它。

>[AZURE.NOTE] 禁用某个终结点对其在 Azure 中的部署状态没有任何影响。正常的终结点保持运行并能够接收流量，即使在流量管理器中已将其禁用也是如此。此外，在一个配置文件中禁用某个终结点不会影响它在其他配置文件中的状态。

## 添加云服务或网站终结点

1. 在 Azure 经典管理门户的“流量管理器”窗格中，找到要修改其终结点设置的流量管理器配置文件。若要打开设置页，请单击配置文件名称右侧的箭头。
2. 在页面的顶部，单击“终结点”来查看已经属于你的配置的一部分的终结点。
3. 在页面的底部，单击“添加”来访问“添加服务终结点”页。默认情况下，该页面会在“服务终结点”下面列出云服务。
4. 对于云服务，请在列表中选择云服务，将其添加为此配置文件的终结点。清除云服务名称会将其从终结点列表中删除。
5. 对于网站，请单击“服务类型”下拉列表，然后选择“Web 应用”。
6. 在列表中选择网站以将其添加为此配置文件的终结点。清除网站名称会将其从终结点列表中删除。对于每个 Azure 数据中心（也称为区域），只能选择一个网站。选择第一个网站后，无法选择同一数据中心内的其他网站。另请注意，只会列出标准网站。
7. 在为此配置文件选择终结点后，单击右下角的复选标记来保存你的更改。

>[AZURE.NOTE] 使用“故障转移”流量路由方法在配置文件中添加或删除终结点后，故障转移优先级列表可能不会按所需的顺序列出。可以在“配置”页上调整故障转移优先级列表的顺序。有关详细信息，请参阅[配置故障转移流量路由](/documentation/articles/traffic-manager-configure-failover-routing-method/)。

## 禁用终结点

1. 在 Azure 经典管理门户的“流量管理器”窗格中，找到要修改其终结点设置的流量管理器配置文件。若要打开设置页，请单击配置文件名称右侧的箭头。
2. 在页面的顶部，单击“终结点”来查看你的配置中已包含的终结点。
3. 单击你要禁用的终结点，然后单击页面底部的“禁用”。
4. 在生存时间 (TTL) 的持续时间内，客户端会继续将流量发送到终结点。可以在流量管理器配置文件的“配置”页上更改 TTL。

## 启用终结点

1. 在 Azure 经典管理门户的“流量管理器”窗格中，找到要修改其终结点设置的流量管理器配置文件。若要打开设置页，请单击配置文件名称右侧的箭头。
2. 在页面的顶部，单击“终结点”来查看你的配置中已包含的终结点。
3. 单击你要启用的终结点，然后单击页面底部的“启用”。
4. 随后，将会根据配置文件的指示，将客户端定向到已启用的终结点。

## 删除云服务或网站终结点

1. 在 Azure 经典管理门户的“流量管理器”窗格中，找到要修改其终结点设置的流量管理器配置文件。若要打开设置页，请单击配置文件名称右侧的箭头。
2. 在页面的顶部，单击“终结点”来查看已经属于你的配置的一部分的终结点。
3. 在“终结点”页面上，单击你要从配置文件中删除的终结点的名称。
4. 在页面的底部，单击“删除”。

## 后续步骤

* [流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/traffic-manager-manage-profiles/)
* [配置故障转移路由方法](/documentation/articles/traffic-manager-configure-failover-routing-method/)
* [配置轮循机制路由方法](/documentation/articles/traffic-manager-configure-round-robin-routing-method/)
* [配置性能路由方法](/documentation/articles/traffic-manager-configure-performance-routing-method/)
* [流量管理器降级状态疑难解答](/documentation/articles/traffic-manager-troubleshooting-degraded/)
* [流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations/)
* [流量管理器上的操作（REST API 参考）](https://msdn.microsoft.com/zh-cn/library/hh758255.aspx)

<!---HONumber=Mooncake_1031_2016-->