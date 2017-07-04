<properties
    pageTitle="在门户中的自动缩放云服务 | Azure"
    description="（经典）了解如何使用经典门户在 Azure 中为云服务 Web 角色或辅助角色配置自动缩放规则。"
    services="cloud-services"
    documentationCenter=""
    authors="Thraka"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="cloud-services"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/27/2017"
    ms.author="adegeo"
    wacn.date="04/24/2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="4ea38fa86e646cc71a006b5fa5f1e7d013ea5f7a"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-configure-auto-scaling-for-a-cloud-service-in-the-classic-portal"></a>如何在经典管理门户中为云服务配置自动缩放
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/cloud-services-how-to-scale-portal/)
- [Azure 经典管理门户](/documentation/articles/cloud-services-how-to-scale/)

在 Azure 经典管理门户的“缩放”页上，可以为 Web 角色或辅助角色配置自动缩放设置。 也可以配置手动缩放，不必配置基于规则的自动缩放。

>[AZURE.NOTE]
> 本文着重于云服务 Web 和辅助角色。 如果直接创建虚拟机（经典），该虚拟机将托管在云服务中。 其中有些信息适用于这些类型的虚拟机。 缩放虚拟机的可用性集其实只是根据配置的缩放规则将其关闭或打开。 有关虚拟机和可用性集的详细信息，请参阅 [Manage the Availability of Virtual Machines](/documentation/articles/virtual-machines-windows-classic-configure-availability/)

在配置应用程序的缩放之前，应考虑以下信息：

- 缩放受内核使用情况影响。

    角色实例越大，使用的核心越多。 只能在订阅的内核限制内缩放应用程序。 例如，假设订阅有 20 个核心的限制。 如果通过两个中等规模的云服务（一共 4 个核心）运行某个应用程序，则对于订阅中的其他云服务部署，只有 16 个核心的扩展空间。 有关大小的详细信息，请参阅[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)。

- 必须先创建队列并使其与角色关联，然后才能基于消息阈值缩放应用程序。 有关详细信息，请参阅[如何使用队列存储服务](/documentation/articles/storage-dotnet-how-to-use-queues/)。

- 可缩放链接到云服务的资源。 有关链接资源的更多信息，请参阅[如何：将资源链接到云服务](/documentation/articles/cloud-services-how-to-manage/#how-to-link-a-resource-to-a-cloud-service)。

- 若要使应用程序具有高可用性，应确保为其部署两个或更多角色实例。有关详细信息，请参阅[服务级别协议](/support/legal/sla)。

> [AZURE.WARNING]
> 自动缩放仅适用于经典 Azure 存储帐户， 而不适用于 Azure Resource Manager 存储帐户。

## <a name="schedule-scaling"></a>计划缩放
默认情况下，并非所有角色都遵循某一特定计划。 因此，所更改的任意设置都将应用到全年中的所有时间和日期。 如果需要，可针对以下模式之一设置手动或自动缩放：

- 工作日
- 周末
- 周中夜间
- 周中上午
- 特定日期
- 特定日期范围

在 [Azure 经典管理门户](https://manage.windowsazure.cn/) 中以下位置对计划设置进行配置：  
“云服务” > “\[你的云服务\]” > “缩放” > “\[生产或过渡\]”页。

单击要更改的每个角色对应的“设置计划时间”按钮。

![基于计划的云服务自动缩放][scale_schedules]  

## <a name="manual-scale"></a>手动缩放
在“缩放”页上，可手动增加或减少云服务中正在运行的实例数。 此设置针对已创建的每个计划进行配置，或者在尚未创建计划时随时进行。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中单击“云服务”，然后单击云服务名称以打开仪表板。

    > [AZURE.TIP]
    > 如果未看到云服务，可能需要从“生产”更改为“过渡”，或者进行相反的切换。

2. 单击“缩放”。
3. 选择要更改其缩放选项的计划。 如果尚未定义计划，则默认设置为“无计划时间”。
4. 找到“按度量值缩放”部分，然后选择“无”。 此设置是所有角色的默认设置。
5. 云服务中的每个服务角色都具有一个用于更改要使用的实例数的滑块。

    ![手动缩放云服务角色][manual_scale]

    如果需要更多实例，可能需要更改[云服务虚拟机大小](/documentation/articles/cloud-services-sizes-specs/)。

6. 单击“保存” 。  
   将根据用户的选择添加或删除角色实例。

>[AZURE.TIP]
> 看到 ![][tip_icon] 时，将鼠标移到其上可获取特定设置功能的相关帮助。

## <a name="automatic-scale---cpu"></a>自动缩放 - CPU
如果平均 CPU 使用量百分比高于或低于指定的阈值，此模式会进行缩放。 发生这种情况时，会创建或删除角色实例。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中单击“云服务”，然后单击云服务名称以打开仪表板。

    > [AZURE.TIP]
    > 如果未看到云服务，可能需要从“生产”更改为“过渡”，或者进行相反的切换。

2. 单击“缩放”。
3. 选择要更改其缩放选项的计划。 如果尚未定义计划，则默认设置为“无计划时间”。
4. 找到“按度量值缩放”部分，然后选择“CPU”。

5. 现在，可设置角色实例的最小和最大范围、（用于触发扩展的）目标 CPU 使用量，以及扩展和缩减的实例数目。

![按 CPU 负载缩放云服务角色][cpu_scale]  

>[AZURE.TIP]
> 看到 ![][tip_icon] 时，将鼠标移到其上可获取特定设置功能的相关帮助。

## <a name="automatic-scale---queue"></a>自动缩放 - 队列
如果队列中的消息数目高于或低于指定的阈值，此模式会自动缩放。 发生这种情况时，会创建或删除角色实例。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中单击“云服务”，然后单击云服务名称以打开仪表板。

    > [AZURE.TIP]
    > 如果未看到云服务，可能需要从“生产”更改为“过渡”，或者进行相反的切换。

2. 单击“缩放”。
3. 找到“按指标缩放”部分，然后选择“队列”。
4. 现在，可设置角色实例的最小和最大范围、每个实例要处理的队列和队列消息数，以及扩展和缩减的实例数目。

![按消息队列缩放云服务角色][queue_scale]  

>[AZURE.TIP]
> 看到 ![][tip_icon] 时，将鼠标移到其上可获取特定设置功能的相关帮助。

## <a name="scale-linked-resources"></a>缩放链接的资源
缩放角色时，通常最好同时缩放应用程序正在使用的数据库。 如果将数据库链接到云服务，可以单击相应的链接来访问该资源的缩放设置。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中单击“云服务”，然后单击云服务名称以打开仪表板。

    > [AZURE.TIP]
    > 如果未看到云服务，可能需要从“生产”更改为“过渡”，或者进行相反的切换。

2. 单击“缩放”。
3. 找到“链接的资源”部分，然后单击“管理此数据库的规模”。

    > [AZURE.NOTE]
    > 如果未看到“链接的资源”部分，则可能表示没有任何链接的资源。

![][linked_resource]

[manual_scale]: ./media/cloud-services-how-to-scale/manual-scale.png
[queue_scale]: ./media/cloud-services-how-to-scale/queue-scale.png
[cpu_scale]: ./media/cloud-services-how-to-scale/cpu-scale.png
[tip_icon]: ./media/cloud-services-how-to-scale/tip.png
[scale_schedules]: ./media/cloud-services-how-to-scale/schedules.png
[scale_popup]: ./media/cloud-services-how-to-scale/schedules-dialog.png
[linked_resource]: ./media/cloud-services-how-to-scale/linked-resources.png

<!---HONumber=Mooncake_Quality_Review_1118_2016-->
<!--Update_Description:update wording-->