<properties
	pageTitle="管理虚拟机的可用性 | Azure"
	description="了解如何使用多个虚拟机来确保你的 Azure 应用程序的高可用性。"
	services="virtual-machines"
	documentationCenter=""
	authors="kenazk"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/23/2015"
	wacn.date="03/28/2016"/>

#管理虚拟机的可用性

## 了解计划内与计划外维护
有两种类型的 Azure 平台事件可能影响到虚拟机的可用性：计划内维护和计划外维护。

- **计划内维护事件**是指由 21vnet 对底层 Azure平台进行定期更新，以改进虚拟机运行时所在的平台基础结构的总体可靠性、性能和安全性。大多数此类更新在执行时不会影响虚拟机或云服务。但有时候，这些更新要求重启你的虚拟机，否则无法将这些必需更新应用到平台基础结构。

- **计划外维护事件**见于虚拟机所在硬件或物理基础结构出现某类故障的情况。此类故障可能包括：本地网络故障、本地磁盘故障，或者其他机架级别的故障。检测到此类故障时，Azure 平台会自动将虚拟机从所在的不健康物理机迁移到健康的物理机。此类事件很少见，但也会导致虚拟机重启。

## 当你设计可用性高的应用程序时，请遵循最佳做法
若要减轻一个或多个此类事件引发的停机所造成的影响，我们建议你遵循以下最佳做法以提高虚拟机的可用性：

* [在可用性集中配置多个虚拟机以确保冗余]
* [将每个应用程序层配置到不同的可用性集中]
* [将负载平衡器与可用性集组合在一起]
* [避免可用性集中出现单实例虚拟机]

###<a name="configure-multiple-virtual-machines-in-an-availability-set-for-redundancy"></a> 在可用性集中配置多个虚拟机以确保冗余
若要为应用程序提供冗余，我们建议你将两个或两个以上的虚拟机组合到一个可用性集中。这种配置可以确保在发生计划内或计划外维护事件时，至少有一个虚拟机可用，因此满足 99.95% Azure SLA 的要求。有关服务级别协议要求的更多信息，请参阅[服务级别协议](/support/legal/sla/)中的“云服务、虚拟机和虚拟网络”一节。

基础 Azure 平台为可用性集中的每个虚拟机分配一个更新域 (UD) 和一个容错域 (FD)。对于给定的可用性集，将会分配 5 个非用户可配置 UD，以指示能够同时重启的虚拟机组和底层物理硬件。在单个可用性集中配置 5 个以上的虚拟机时，第 6 个虚拟机将放置在第 1 个虚拟机所在的 UD 中，第 7 个虚拟机将放置在第 2 个虚拟机所在的 UD 中，依此类推。在计划内维护期间，UD 的重启顺序可能不会按序进行，但一次只重启一个 UD。

根据定义，FD 是一组共用一个通用电源和网络交换机的虚拟机。默认情况下，在可用性集中配置的虚拟机分布在两个 FD 中。虽然将虚拟机置于可用性集中并不能让你的应用程序免受特定于操作系统或应用程序的故障的影响，但可以限制潜在物理硬件故障、网络中断或电源中断的影响。

<!--Image reference-->
   ![UD FD 配置](./media/virtual-machines-manage-availability/ud-fd-configuration.png)

>[AZURE.NOTE]有关说明，请参阅[如何为虚拟机配置可用性集][]。

###<a name="configure-each-application-tier-into-separate-availability-sets"></a> 将每个应用程序层配置到不同的可用性集中
如果可用性集中的虚拟机几乎都是相同的，并且对你的应用程序的用途是一样的，我们建议你针对每个应用程序层配置可用性集。如果将两个不同的层置于同一可用性集中，则同一应用程序层中的所有虚拟机可以同时重启。通过在可用性集中为每个层配置至少两个虚拟机，可以确保每个层中至少有一个虚拟机可用。

例如，你可以将运行 IIS、Apache、Nginx 等服务器的应用程序前端的所有虚拟机置于单个可用性集中。请确保仅将前端虚拟机置于同一可用性集中。同样，请确保仅将数据层虚拟机置于其自身的可用性集中，例如已复制的 SQL 服务器虚拟机或 MySQL 虚拟机。

<!--Image reference-->
   ![应用程序层](./media/virtual-machines-manage-availability/application-tiers.png)


###<a name="combine-the-load-balancer-with-availability-sets"></a> 将负载平衡器与可用性集组合在一起
将 Azure 负载平衡器与可用性集组合在一起，以获取最大的应用程序复原能力。Azure 负载平衡器将流量分布到多个虚拟机中。对于标准层虚拟机来说，Azure 负载平衡器已包括在内。请注意，并非所有虚拟机层都包括 Azure 负载平衡器。有关对虚拟机进行负载平衡的更多信息，请阅读[对虚拟机进行负载平衡](/documentation/articles/load-balance-virtual-machines)。

如果没有将负载平衡器配置为对多个虚拟机上的流量进行平衡，则任何计划内维护事件都会影响唯一的那个处理流量的虚拟机，导致应用程序层中断。将同一层的多个虚拟机置于相同的负载平衡器和可用性集下可以确保至少有一个虚拟机实例能够持续处理流量。

###<a name="avoid-single-instance-virtual-machines-in-availability-sets"></a> 避免可用性集中出现单实例虚拟机
避免将单实例虚拟机置于可用性集中。此配置中的虚拟机并不符合 SLA 保证，在出现 Azure 计划内维护事件时就会停机。请注意，如果系统发送多实例虚拟机计划内维护通知，则可用性集中的单个虚拟机实例也会收到提前发出的电子邮件通知。

<!-- Link references -->
[在可用性集中配置多个虚拟机以确保冗余]: #configure-multiple-virtual-machines-in-an-availability-set-for-redundancy
[将每个应用程序层配置到不同的可用性集中]: #configure-each-application-tier-into-separate-availability-sets
[将负载平衡器与可用性集组合在一起]: #combine-the-load-balancer-with-availability-sets
[避免可用性集中出现单实例虚拟机]: #avoid-single-instance-virtual-machines-in-availability-sets
[如何为虚拟机配置可用性集]: /documentation/articles/virtual-machines-how-to-configure-availability
