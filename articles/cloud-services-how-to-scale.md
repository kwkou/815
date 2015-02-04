<properties linkid="manage-services-how-to-scale-a-cloud-service" urlDisplayName="How to scale" pageTitle="如何缩放云服务 - Azure" metaKeywords="Azure link resource, scaling cloud service" description="了解如何在 Azure 中缩放云服务和链接的资源。" metaCanonical="" services="cloud-services" documentationCenter="" title="How to Scale an Application" authors="davidmu" solutions="" manager="jeffreyg" editor="mattshel" />






#如何缩放应用程序


在 Azure 管理门户的"缩放"页上，你可以手动缩放应用程序，也可以设置参数使其自动缩放。你可以缩放运行 Web 角色、辅助角色或虚拟机的应用程序。若要缩放运行 Web 角色或辅助角色实例的应用程序，你需要添加或删除角色实例以适应工作负载。

当你缩放运行虚拟机的应用程序时，不会创建新的虚拟机，也不会删除虚拟机，但会根据先前创建的虚拟机的可用性集打开或关闭虚拟机。你可以根据 CPU 使用率的平均百分比或基于队列中的消息数指定缩放。

在配置应用程序的缩放之前，应考虑以下信息：

- 你必须将你创建的虚拟机添加到可用性集，才能缩放使用它们的应用程序。你添加的虚拟机最初可能处于打开或关闭状态，但它们在扩展操作中将打开，在缩减操作中将关闭。有关虚拟机和可用性集的详细信息，请参阅[管理虚拟机的可用性](/zh-cn/documentation/articles/virtual-machines-manage-availability/)。
- 缩放受内核使用情况影响。较大的角色实例或虚拟机使用更多内核。你只能在你的订阅的内核限制内缩放应用程序。例如，如果你的订阅的上限是二十个内核，并且你通过两个中等规模的虚拟机（一共四个内核）运行某个应用程序，则对于订阅中的其他云服务部署，你只能扩展十六个内核。可用性集中用于缩放应用程序的所有虚拟机必须具有相同大小。有关内核使用情况和虚拟机大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小](http://msdn.microsoft.com/zh-cn/library/dn197896.aspx)。
- 你必须先创建队列并使其与角色或可用性集关联，然后才能基于消息阈值缩放应用程序。有关详细信息，请参阅[如何使用队列存储服务]。(/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/)。
- 你可以缩放链接到云服务的资源。有关链接资源的更多信息，请参见[如何：将资源链接到云服务](/zh-cn/documentation/articles/cloud-services-how-to-manage/#linkresources).
- To enable high availability of your application, you should ensure that it is deployed with two or more role instances or Virtual Machines. For more information, see [Service Level Agreements](/support/legal/sla/)。

你可以对云服务执行下列缩放操作：

- [手动缩放运行 Web 角色或辅助角色的应用程序](#manualscale)
- [自动缩放运行 Web 角色、辅助角色或虚拟机的应用程序](#autoscale)
- [缩放链接的资源](#scalelink)
- [计划应用程序的缩放](#schedule)


##<a id="manualscale"></a>手动缩放运行 Web 角色或辅助角色的应用程序##

在"缩放"页上，你可以手动增加或减少云服务中正在运行的实例数。

1. 在[管理门户](https://manage.windowsazure.cn/)中单击"云服务"****，然后单击云服务名称以打开仪表板。

2. 单击"缩放"****。默认情况下，对所有角色启用自动缩放，这意味着你可以手动更改应用程序使用的实例数。

	![Scale page][manual_scale]

3. 云服务中的每个服务角色都具有一个用于更改要使用的实例数的滑块。若要添加角色实例，请将竖线向右拖动。若要删除实例，请将竖线向左拖动。

	![Scale role][slider_role] 


	仅当有适当的内核数可以支持实例时，才能增加要使用的实例数。滑块的颜色表示订阅中使用的内核数和可用的内核数：

	- 蓝色表示所选角色使用的内核数
	- 深灰色表示订阅中的所有角色和虚拟机使用的内核数
	- 浅灰色表示可用来进行缩放的内核数目
	- 粉红色表示尚未保存的更改

4. 单击"保存"****。将基于你的选择添加或删除角色实例。

##<a id="autoscale"></a>自动缩放运行 Web 角色、辅助角色或虚拟机的应用程序##

在"缩放"页上，你可以将云服务配置为自动增加或减少应用程序使用的实例或虚拟机的数目。你可以基于以下参数配置扩展：

- [平均 CPU 使用率](#averagecpu) - 如果 CPU 使用率的平均百分比高于或低于指定的阈值，则将创建或删除角色实例，或者从可用性集中打开或关闭虚拟机。
- [队列消息数](#queuemessages) - 如果队列中的消息数高于或低于指定的阈值，则将创建或删除角色实例，或者从可用性集中打开或关闭虚拟机。

<h3><a id="averagecpu"></a>平均 CPU 使用率</h3>

1. 在[管理门户](https://manage.windowsazure.cn/)中单击"云服务"****，然后单击云服务名称以打开仪表板。
2. 单击"缩放"****。
3. 滚动到角色或可用性集部分，然后单击"CPU"****。这样，你就可以根据应用程序使用的 CPU 资源的平均百分比来自动缩放应用程序。

	![Autoscale on][autoscale_on]

4. 每个角色或可用性集都包含一个滑块，用于更改可以使用的实例数。要设置可以使用的最大实例数，请向右拖动右侧的竖线。若要设置可以使用的最小实例数，请向左拖动左侧的竖线。

	**注意：**在"缩放"页上，"实例"****表示角色实例或虚拟机实例。

	![Instance range][instance_range]

	最大实例数量受订阅中可用的内核数目限制。滑块的颜色表示订阅中使用的内核数和可用的内核数：

	- 蓝色表示角色可以使用的最大内核数目。
	- 深灰色表示订阅中的所有角色和虚拟机使用的内核数。当此值与角色使用的内核数重叠时，颜色将变为深蓝色。
	- 浅灰色表示可用来进行缩放的内核数目。
	- 粉红色表示尚未保存的已进行的更改。

5. 滑块用于指定 CPU 使用率的平均百分比范围。当 CPU 使用率的平均百分比高于最大设置时，将创建更多的角色实例或打开更多的虚拟机。当 CPU 使用率的平均百分比低于最小设置时，将删除角色实例或关闭虚拟机。若要设置最大平均 CPU 百分比，请将右侧的竖线向右侧拖动。若要设置最小平均 CPU 百分比，请将左侧的竖线向左侧拖动。

	![Target cpu][target_cpu]

6. 每次扩展应用程序时，你都可以指定要添加或打开的实例数。要在扩展应用程序时增加创建或打开的实例数，则向右拖动竖线。若要减少此数目，请将竖线向左拖动。

	![Scale cpu up][scale_cpuup]

7. 设置上一个缩放操作与下一个扩展操作之间等待的分钟数。上一个缩放操作可以是扩展或缩减。

	![Up time][scale_uptime]

	计算 CPU 使用率的平均百分比时将包括所有实例，平均值基于前一个小时的使用情况。根据应用程序使用的实例数，如果等待时间设置为非常低，则等待缩放操作发生的时间可能比指定的等待时间更长。缩放操作之间的最小时间是五分钟。如果任一实例处于过渡状态，则无法执行缩放操作。

8. 你还可以指定在缩减应用程序时要删除或关闭的实例数。要在缩减应用程序时增加要删除或关闭的实例数，则向右拖动竖线。若要减少此数目，请将竖线向左拖动。

	![Scale cpu down][scale_cpudown]

	如果应用程序的 CPU 使用率突然增加，必须确保你具有足够的最小实例数目来处理它们。

9. 设置上一个缩放操作与下一个缩减操作之间等待的分钟数。上一个缩放操作可以是扩展或缩减。

	![Down time][scale_downtime]

10. 单击"保存"****。缩放操作可能需要长达五分钟才能完成。

<h3><a id="queuemessages"></a>队列消息数</h3>

1. 在[管理门户](https://manage.windowsazure.cn/)中单击"云服务"****，然后单击云服务名称以打开仪表板。
2. 单击"缩放"****。
3. 滚动到角色或可用性集部分，然后单击"队列"****。这样，你就可以根据队列消息的目标数目来自动缩放应用程序。

	![Scale queue][scale_queue]

4. 云服务中的每个角色或可用性集都包含一个滑块，用于更改可以使用的实例数。要设置可以使用的最大实例数，请向右拖动右侧的竖线。若要设置可以使用的最小实例数，请向左拖动左侧的竖线。

	![Queue range][queue_range]

	**注意：**在"缩放"页上，"实例"****表示角色实例或虚拟机实例。
	
	最大实例数量受订阅中可用的内核数目限制。滑块的颜色表示订阅中使用的内核数和可用的内核数：
	- 蓝色表示角色可以使用的最大内核数目。
	- 深灰色表示订阅中的所有角色和虚拟机使用的内核数。当此值与角色使用的内核数重叠时，颜色将变为深蓝色。
	- 浅灰色表示可用来进行缩放的内核数目。
	- 粉红色表示尚未保存的已进行的更改。

5. 选择与你要使用的队列关联的存储帐户。

	![Storage name][storage_name]	

6. 选择队列。

	![Queue name][queue_name]

7. 指定你希望每个实例支持的消息数。实例将基于总消息数除以每台计算机的目标消息数而进行缩放。

	![Message number][message_number]

8. 每次扩展应用程序时，你都可以指定要添加或打开的实例数。要在扩展应用程序时增加要添加或打开的实例数，则向右拖动竖线。若要减少此数目，请将竖线向左拖动。

	![Scale cpu up][scale_cpuup]

9. 设置上一个缩放操作与下一个扩展操作之间等待的分钟数。上一个缩放操作可以是扩展或缩减。

	![Up time][scale_uptime]

	缩放操作之间的最小时间是五分钟。如果任一实例处于过渡状态，则无法执行缩放操作。

10. 你还可以指定在缩减应用程序时要删除或不使用的实例数。滑块用于指定缩放增量。要在缩减应用程序时增加要删除或不使用的实例数，则向右拖动竖线。若要减少此数目，请将竖线向左拖动。

	![Scale cpu down][scale_cpudown]

11.	设置上一个缩放操作与下一个缩减操作之间等待的分钟数。上一个缩放操作可以是扩展或缩减。

	![Down time][scale_downtime]

12. 单击"保存"****。缩放操作可能需要长达五分钟才能完成。

##<a id="scalelink"></a>缩放链接的资源##

通常当你缩放角色时，最好同时缩放应用程序正在使用的数据库。如果将数据库链接到云服务，则可更改 SQL Database 版本并在"缩放"页上调整数据库的大小。

1. 在[管理门户](https://manage.windowsazure.cn/)中单击"云服务"****，然后单击云服务名称以打开仪表板。
2. 单击"缩放"****。
3. 在"链接的资源"部分中，选择用于此数据库的版本。

	![Linked resources][linked_resources]

4. 选择数据库的大小。
5. 单击"保存"****以更新链接的资源。

##<a id="schedule"></a>计划应用程序的缩放##

你可以通过配置针对不同时间的计划，计划你的应用程序的自动缩放。你可以使用以下选项进行自动缩放：

- **无计划** - 这是默认选项，使你的应用程序能够随时以相同方式自动缩放。

- **白天和夜晚** - 此选项使你能够为白天和夜晚的具体时间指定缩放。

**注意：**计划目前不可用于使用虚拟机的应用程序。

1. 在[管理门户](https://manage.windowsazure.cn/)中单击"云服务"****，然后单击云服务名称以打开仪表板。
2. 单击"缩放"****。
3. 在"缩放"页上，单击"设置计划时间"****。

	![Schedule scaling][scale_schedule]

4. 指定要设置的缩放计划的类型。

5. 指定白天的开始和结束的时间并且设置时区。对于白天和夜晚计划，这些时间表示白天天的开始和结束时间，剩余时间表示夜晚。

6. 单击页面底部的复选标记可保存计划。

7. 在保存计划后，它们将出现在列表中。你可以选择要使用的时间表，然后修改你的缩放设置。缩放设置仅在你选择的计划过程中适用。你可以通过单击"设置计划时间"****编辑计划。

[manual_scale]: ./media/cloud-services-how-to-scale/CloudServices_ManualScaleRoles.png
[slider_role]: ./media/cloud-services-how-to-scale/CloudServices_SliderRole.png
[autoscale_on]: ./media/cloud-services-how-to-scale/CloudServices_AutoscaleOn.png
[instance_range]: ./media/cloud-services-how-to-scale/CloudServices_InstanceRange.png
[target_cpu]: ./media/cloud-services-how-to-scale/CloudServices_TargetCPURange.png
[scale_cpuup]: ./media/cloud-services-how-to-scale/CloudServices_ScaleUpBy.png
[scale_uptime]: ./media/cloud-services-how-to-scale/CloudServices_ScaleUpWaitTime.png
[scale_cpudown]: ./media/cloud-services-how-to-scale/CloudServices_ScaleDownBy.png
[scale_downtime]: ./media/cloud-services-how-to-scale/CloudServices_ScaleDownWaitTime.png
[scale_queue]: ./media/cloud-services-how-to-scale/CloudServices_QueueScale.png
[queue_range]: ./media/cloud-services-how-to-scale/CloudServices_QueueRange.png
[storage_name]: ./media/cloud-services-how-to-scale/CloudServices_StorageAccountName.png
[queue_name]: ./media/cloud-services-how-to-scale/CloudServices_QueueName.png
[message_number]: ./media/cloud-services-how-to-scale/CloudServices_TargetMessageNumber.png
[linked_resources]: ./media/cloud-services-how-to-scale/CloudServices_ScaleLinkedResources.png
[scale_schedule]: ./media/cloud-services-how-to-scale/CloudServices_SetUpSchedule.png
<!--HONumber=39-->
