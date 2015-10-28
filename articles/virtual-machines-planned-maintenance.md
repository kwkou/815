<properties
	pageTitle="Azure 虚拟机的计划内维护"
	description="了解什么是 Azure 计划内维护以及它如何影响正在 Azure 中运行的虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="kenazk"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.date="07/23/2015"
	wacn.date="09/18/2015"/>


# Azure 虚拟机的计划内维护

## Azure 为何要执行计划内维护
<p> Windows Azure 在全球范围内定期执行更新，以提高虚拟机所基于的主机基础结构的可靠性、性能及安全性。其中许多更新在执行时不会对虚拟机或云服务产生任何影响，其中包括内存保留更新。

不过，有些更新就需要重新启动虚拟机，以便向基础结构应用所需的更新。虚拟机会在我们修补基础结构时关闭，之后再重新启动。

请注意，有两类维护可能会影响虚拟机的可用性：计划内维护和计划外维护。本页介绍 Windows Azure 如何执行计划内维护。有关计划外维护的详细信息，请参阅[了解计划内与计划外维护]。

## 内存保留更新
对于 Windows Azure 中的某一类更新，客户会发现它们对正在运行的虚拟机没有任何影响。其中一些更新主要面向组件或服务，更新时不会干扰正在运行的实例。还有一些更新是主机操作系统上的平台基础结构更新，应用时无需完全重启虚拟机。

这些更新借助支持实时迁移的技术（“内存保留”更新）来实现。更新时，虚拟机进入“暂停”状态，以保留 RAM 中的内存，基础主机操作系统则接收必要的更新和补丁。虚拟机在暂停后 30 秒内恢复正常。恢复后，虚拟机的时钟将自动同步。

并非所有更新都可以通过使用此机制来部署，但是如果提供短暂的暂停期，采用此方法部署更新将大幅减少对虚拟机的影响。

多实例更新（针对可用性集中的虚拟机）一次只应用于一个更新域。

## 虚拟机配置
虚拟机配置分两种：多实例和单实例。在多实例配置中，相似的虚拟机放在一个可用性集中。

多实例配置可提供冗余，若要确保应用程序的可用性，建议采用此配置。可用性集中的所有虚拟机应几乎相同，并且对应用程序具有相同的用途。

有关配置虚拟机的高可用性的详细信息，请参阅[管理虚拟机的可用性](/documentation/articles/virtual-machines-manage-availability)。

相反，单实例配置用于不在一个可用性集中的独立虚拟机。这些虚拟机不符合服务级别协议 (SLA) 的要求，SLA 要求在同一个可用性集下部署两个或更多虚拟机。

有关 SLA 的详细信息，请参阅[服务级别协议](/support/legal/sla)中的“云服务、虚拟机和虚拟网络”一节。


## 多实例配置更新
在计划内维护期间，Azure 平台会先更新多实例配置中托管的虚拟机集。这将导致这些虚拟机重新启动。

在多实例配置更新中，虚拟机在整个更新过程中都将保持可用性，但前提是可用性集中每个虚拟机的用途都与其他虚拟机相似。

基础 Azure 平台为可用性集中的每个虚拟机分配一个更新域和一个容错域。每个更新域是一组在相同时间范围内重新启动的虚拟机。每个容错域是一组共用一个通用电源和网络交换机的虚拟机。

有关更新域和容错域的详细信息，请参阅[配置可用性集中的多个虚拟机以实现冗余](/documentation/articles/virtual-machines-manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)。

为防止更新域同时脱机，将通过以下方式执行维护：关闭某个更新域中的所有虚拟机，向主机应用更新，重新启动虚拟机，然后继续对下一个更新域执行相同操作。更新完所有更新域之后，计划内维护事件即结束。

在计划内维护期间，更新域的重启顺序可能不会按序进行，但一次只重启一个更新域。现在，Azure 为多实例配置中虚拟机的计划内维护提供 48 小时高级通知功能。

虚拟机还原后，下面的示例演示了 Windows 事件查看器可能会显示的内容：

<!--Image reference-->
![][image2]

## 单实例配置更新
完成多实例配置更新后，Azure 将执行单实例配置更新。此更新也会导致不在可用性集中运行的虚拟机重新启动。

请注意，即使可用性集中只有一个实例在运行，Azure 平台也会将其视作多实例配置更新。

对于单实例配置中的虚拟机，将通过以下方式更新虚拟机：关闭虚拟机，向主机应用更新，然后重新启动虚拟机。这些更新将在单一维护时段内跨某个区域中的所有虚拟机运行。

对于这一类型的虚拟机配置，此计划内维护事件将影响应用程序的可用性。Azure 为单实例配置中虚拟机的计划内维护提供 1 周高级通知功能。

### 电子邮件通知
Azure 会提前发送电子邮件通信，提醒你即将执行计划内维护（单实例提前 1 周，多实例提前 48 小时），该功能仅适用于单实例和多实例虚拟机配置。此电子邮件将发送到订阅提供的主电子邮件帐户。下面是这类电子邮件的示例：

<!--Image reference-->
![][image1]

## 区域对
Azure 整理了一组区域对。在采用单实例配置的虚拟机的计划内维护期间，Azure 不会对配对区域同时执行更新。

有关当前区域对的信息，请参阅下表：

区域 1 | 区域 2
:----- | ------:
美国中北部 | 美国中南部
美国东部 | 美国西部
美国东部 2 | 美国中部
欧洲北部 | 欧洲西部
东南亚 | 东亚
中国东部 | 中国北部
日本东部 | 日本西部
巴西南部 | 美国中南部
澳大利亚东南部 | 澳大利亚东部
美国政府爱荷华州 | 美国政府弗吉尼亚州

例如，在计划内维护期间，如果美国东部正在进行维护，Azure 就不会同时对美国西部执行更新。但是，北欧等其他区域可以与美国东部同时进行维护。

<!--Anchors-->
[image1]: ./media/virtual-machines-planned-maintenance/vmplanned1.png
[image2]: ./media/virtual-machines-planned-maintenance/EventViewerPostReboot.png
[image3]: ./media/virtual-machines-planned-maintenance/RegionPairs.PNG


<!--Link references-->
[Virtual Machines Manage Availability]: /documentation/articles/virtual-machines-windows-tutorial
[了解计划内与计划外维护]: /documentation/articles/virtual-machines-manage-availability#Understand-planned-versus-unplanned-maintenance
<!---HONumber=70-->