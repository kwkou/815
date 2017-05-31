<properties
    pageTitle="在 Azure 流量管理器中管理终结点 | Azure"
    description="本文将帮助你从 Azure 流量管理器中添加、删除、启用和禁用终结点。"
    services="traffic-manager"
    documentationcenter=""
    author="kumudd"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ade2bbc2-35a7-43c5-8001-4698f7254526"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/08/2017"
    wacn.date="05/31/2017"
    ms.author="kumud"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="526746bd790a6eea154e62d20aa112fd6a778b8b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="add-disable-enable-or-delete-endpoints"></a>添加、禁用、启用或删除终结点

无论网站模式如何，Azure 应用服务中的 Web 应用功能都已针对数据中心内的网站提供了故障转移和轮循机制流量路由功能。 你可以使用 Azure 流量管理器为不同数据中心内的网站和云服务指定故障转移和轮询机制流量路由。 提供该功能所需的第一个步骤是将云服务或网站终结点添加到流量管理器中。

你还可以禁用属于流量管理器配置文件的一部分的个体终结点。 禁用某个终结点会将其保留为配置文件的一部分，但是配置文件的行为就如同其中不包括该终结点一样。 此操作对于临时删除处于维护模式或正在重新部署的终结点十分有用。 终结点再次运行后，可以启用它。

> [AZURE.NOTE]
> 禁用某个终结点对其在 Azure 中的部署状态没有任何影响。 正常的终结点保持运行并能够接收流量，即使在流量管理器中已将其禁用也是如此。 此外，在一个配置文件中禁用某个终结点不会影响它在其他配置文件中的状态。

## <a name="to-add-a-cloud-service-or-an-app-service-endpoint-to-a-traffic-manager-profile"></a>将云服务或应用服务终结点添加到流量管理器配置文件

1. 在浏览器中，登录 [Azure 门户预览](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡的“设置”部分中，单击“终结点”。
4. 在显示的“终结点”边栏选项卡中，单击“添加”。
5. 在“添加终结点”边栏选项卡中，按如下所示完成输入：
    1. 对于**类型**，单击“Azure 终结点”。
    2. 提供要用于识别此终结点的**名称**。
    3. 从下拉列表中选择适当的资源类型作为“目标资源类型”。
    4. 从下拉列表中选择适当的目标资源作为“目标资源”，以便在“资源”边栏选项卡的同一订阅下显示列出的资源。 在显示的“资源”边栏选项卡中，选取要添加为第一个终结点的服务。
    5. 对于**优先级**，选为 **1**。 如果此终结点处于正常状态，这会导致所有流量转到此终结点。
    6. 使“添加为已禁用”保持未选中状态。
    7. 单击 **“确定”**
6.    重复步骤 4 和 5，添加下一个 Azure 终结点。 确保添加该终结点时将其**优先级**值设为 **2**。
7.    添加完这两个终结点后，这两个终结点将显示在“流量管理器配置文件”边栏选项卡中，并且其监视状态为“联机”。

> [AZURE.NOTE]
> 使用“故障转移”流量路由方法在配置文件中添加或删除终结点后，故障转移优先级列表可能不会按所需的顺序列出。 可以在“配置”页上调整故障转移优先级列表的顺序。 有关详细信息，请参阅[配置故障转移流量路由](/documentation/articles/traffic-manager-configure-priority-routing-method/)。

## <a name="to-disable-an-endpoint"></a>禁用终结点

1. 在浏览器中，登录 [Azure 门户预览](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的**流量管理器配置文件**名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡的“设置”部分中，单击“终结点”。 
4. 单击要禁用的终结点，然后在显示的“终结点”边栏选项卡中，单击“编辑”。
5. 在“终结点”边栏选项卡中，将终结点的状态更改为“已禁用”，然后单击“保存”。
6. 在生存时间 (TTL) 的持续时间内，客户端会继续将流量发送到终结点。 可以在流量管理器配置文件的“配置”页上更改 TTL。

## <a name="to-enable-an-endpoint"></a>启用终结点

1. 在浏览器中，登录 [Azure 门户预览](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的**流量管理器配置文件**名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡的“设置”部分中，单击“终结点”。 
4. 单击要禁用的终结点，然后在显示的“终结点”边栏选项卡中，单击“编辑”。
5. 在“终结点”边栏选项卡中，将终结点的状态更改为“已启用”，然后单击“保存”。
6. 在生存时间 (TTL) 的持续时间内，客户端会继续将流量发送到终结点。 可以在流量管理器配置文件的“配置”页上更改 TTL。

## <a name="to-delete-an-endpoint"></a>删除终结点

1. 在浏览器中，登录 [Azure 门户预览](http://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的**流量管理器配置文件**名称，然后在显示的结果中单击该流量管理器配置文件。
3. 在“流量管理器配置文件”边栏选项卡的“设置”部分中，单击“终结点”。 
4. 单击要禁用的终结点，然后在显示的“终结点”边栏选项卡中，单击“编辑”。
5. 在“终结点”边栏选项卡中，将终结点的状态更改为“已启用”，然后单击“保存”。

## <a name="next-steps"></a>后续步骤

* [管理流量管理器配置文件](/documentation/articles/traffic-manager-manage-profiles/)
* [配置路由方法](/documentation/articles/traffic-manager-configure-priority-routing-method/)
* [流量管理器降级状态疑难解答](/documentation/articles/traffic-manager-troubleshooting-degraded/)
* [流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations/)
* [流量管理器上的操作（REST API 参考）](https://msdn.microsoft.com/zh-cn/library/hh758255.aspx)

<!--Update_Description: change to the new portal-->