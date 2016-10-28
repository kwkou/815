<!-- Remove load-balancer, vm-scale -->
<properties
   pageTitle="高可用性清单 | Azure"
   description="为确保在 Azure 中提高应用程序可用性而可以采用的设置和操作的快速清单。"
   services=""
   documentationCenter="na"
   authors="adamglick"
   manager="hongfeig"
   editor=""/>

<tags
   ms.service="resiliency"
   ms.date="06/15/2016"
   wacn.date="07/04/2016"/>

#高可用性清单
使用 Azure 的好处之一是能够使用云提高应用程序的可用性（和可缩放性）。为了确保你可以充分利用这些选项，以下清单将帮助你了解一些重要的基础结构基本知识，以确保应用程序具有复原能力。

>[AZURE.NOTE] 以下大多数建议可随时在应用程序中实施，因此很适合用于“快速修复”。最佳的长期解决方案通常涉及到为云构建的应用程序设计。有关这些功能的清单（更多面向设计的领域），请参阅 [Availability checklist（可用性清单）](/documentation/articles/best-practices-availability-checklist/)。

###是否在资源前面使用流量管理器？
使用流量管理器有助于跨 Azure 区域或者在 Azure 与本地位置之间路由 Internet 流量。这样做有几项原因，包括延迟和可用性。

__不使用流量管理器会发生什么情况？__ 如果在应用程序前面不使用流量管理器，则你只能在单个区域中访问资源。这会限制缩放能力，远离所选区域的用户将感受到更长的延迟，而在发生区域范围的服务中断时将缺乏保护。

###你是否避免使用单个虚拟机作为任何角色？
良好的设计会避免任何单点故障。这在所有服务设计（本地或云）中都很重要，尤其在云中特别有用，因为可以通过横向扩展（添加虚拟机）而不是纵向扩展（使用功能更强大的虚拟机）来提高可缩放性与复原能力。有关可缩放应用程序设计的详细信息，请参阅[构建在 Azure 基础之上的应用程序高可用性](/documentation/articles/resiliency-high-availability-azure-applications/)。

__如果使用单个虚拟机作为角色，会发生什么情况？__ 单个计算机是单点故障，不适用于 [Azure 虚拟机服务级别协议](/support/sla/virtual-machines)。在最好的情况下，应用程序将正常运行，但这不是具有复原能力的设计，并且不受 Azure 虚拟机 SLA 的保障，如果发生故障，任何单点故障将增大停机的可能性。

###是否在应用程序的面向 Internet 的 VM 前面使用负载均衡器？
负载均衡器可让你将应用程序的传入流量分散到任意数目的计算机。你可以随时在负载均衡器中添加/删除计算机，这适用于虚拟机（以及配合虚拟机规模集来自动缩放），让你轻松处理流量增加或 VM 故障的情况。<!-- 若要详细了解负载均衡器，请阅读 [Azure Load Balancer overview（Azure Load Balancer 概述）](/documentation/articles/load-balancer-overview/)和 [Running multiple VMs on Azure for scalability and availability（在 Azure 上运行多个 VM 以提高可缩放性和可用性）](/documentation/articles/guidance-compute-multi-vm/)。-->

__如果在面向 Internet 的 VM 前面不使用负载均衡器，会发生什么情况？__ 如果没有负载均衡器，将无法横向扩展（添加更多计算机），而只能选择纵向扩展（增加面向 Web 的虚拟机的大小）。该虚拟机也面临着单点故障。你还需要编写 DNS 代码，以注意是否丢失面向 Internet 的计算机，并将 DNS 条目重新映射到开始接管的新计算机。

###你是否对无状态应用程序和 Web 服务器使用可用性集？
将计算机包含在可用性集中的同一个应用程序层内可让 VM 符合 Azure VM SLA 的条件。成为可用性集的一部分还可确保将计算机放入不同的更新域（即，在不同时间修补的不同主机）和容错域（即，共享通用电源和网络交换机的主机）。不在可用性集中的 VM 可能位于同一主机，因此，可能潜藏着看不到的单点故障。若要了解有关使用可用性集提高 VM 可用性的详细信息，请参阅 [Manage the availability of virtual machines（管理虚拟机的可用性）](/documentation/articles/virtual-machines-windows-manage-availability/)。

__如果不对无状态应用程序和 Web 服务器使用可用性集，会发生什么情况？__ 不使用可用性集意味着无法利用 Azure VM SLA。这也意味着，如果主机进行更新（托管你使用的 VM 的计算机）或发生常见的硬件故障，该应用程序层中的计算机将全部脱机。

<!-- ###是否对无状态应用程序和 Web 服务器使用虚拟机规模集 (VMSS)？
可缩放且有复原能力的良好设计使用 VMSS，以确保可以扩展/缩减应用程序层（例如 Web 层）中的计算机数目。VMSS 允许你定义应用程序层如何缩放（根据所选的条件来添加或删除服务器）。若要了解如何使用 Azure 虚拟机规模集来灵活应对流量高峰的详细信息，请参阅 [Virtual Machine Scale Sets Overview（虚拟机规模集概述）](/documentation/articles/virtual-machine-scale-sets-overview/)。
__如果不对无状态应用程序或 Web 服务器使虚拟机规模集，会发生什么情况？__ 如果不使用 VMSS，则就有点难以做到无限制缩放和优化资源用法。缺少 VMSS 的设计有其缩放上限，必须以额外的代码来处理（或手动）。缺少 VMSS 还意味着应用程序无法轻松添加和删除计算机（无论规模如何），因而无法帮助你处理较大的流量高峰（例如在促销期间，或者站点/应用/产品变得流行时）。 -->

###是否对每个虚拟机使用高级存储和独立的存储帐户？
生产虚拟机最好使用高级存储。此外，应确保每个虚拟机使用不同的存储帐户（在小规模部署中就应如此。对于大型部署，多个计算机可以重复使用存储帐户，但需要保持平衡，以确保更新域之间和应用程序层之间达到平衡）。若要了解有关 Azure 存储空间性能和可缩放性的详细信息，请阅读 [Azure Storage Performance and Scalability Checklist（Azure 存储空间性能和可缩放性清单）](/documentation/articles/storage-performance-checklist/)。

__如果不对每个虚拟机使用独立的存储帐户，会发生什么情况？__ 就像其他许多资源一样，存储帐户也是单点故障。尽管 Azure 存储空间提供许多保护措施和复原能力，但存在单点故障绝对不是良好的设计。例如，如果该帐户的访问权限受损、达到存储限制或达到 [IOPS 限制](/documentation/articles/azure-subscription-service-limits/#virtual-machine-disk-limits)，则使用该存储帐户的所有虚拟机将受到影响。此外，如果服务中断影响到包含该特定存储帐户的存储戳记，则多个虚拟机会受到影响。

###是否在应用程序的每层之间使用负载均衡器或队列？
在应用程序的每层之间使用负载均衡器或队列，可以轻松地单独缩放应用程序的每一层。应该根据延迟、复杂性和分布（即，应用分发的广度）需求，在这些技术之间做出选择。一般而言，队列的延迟和复杂性通常较高，但提供更高的复原能力，并且可让你将应用程序分发到更大的区域（例如跨区域）。若要了解有关如何使用内部负载均衡器或队列的详细信息，请阅读 <!-- [Internal Load balancer Overview（内部负载均衡器概述）](/documentation/articles/load-balancer-internal-overview/)和 --> [Azure Queues and Service Bus queues - compared and contrasted（Azure 队列和服务总线队列 - 比较与对照）](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)。

__如果在应用程序的每层之间不使用负载均衡器或队列，会发生什么情况？__ 如果应用程序的每层之间没有负载均衡器或队列，则难以扩展或缩减应用程序并将其负载分散到多个计算机。不这样做可能会导致资源预配过度或不足，如果流量出现意外的变化或发生系统故障，还可能造成停机或用户体验不佳。
 
###SQL 数据库是否使用活动异地复制？ 
活动异地复制可让你在相同或不同区域中最多配置 4 个可读的辅助数据库。在发生服务中断或无法连接到主数据库时，可以使用辅助数据库。若要了解有关 SQL 数据库活动异地复制的详细信息，请参阅 [Overview: SQL Database Active Geo-Replication（概述：SQL 数据库活动异地复制）](/documentation/articles/sql-database-geo-replication-overview/)。
 
 __如果 SQL 数据库不使用活动异地复制，会发生什么情况？__ 在没有活动异地复制的情况下，如果主数据库脱机（计划内维护、服务中断、硬件故障等），应用程序数据库将会脱机，直到主数据库重新联机且运行正常为止。
 
###是否在数据库前面使用缓存（Azure Redis 缓存）？
如果应用程序的数据库负载很高，其中大多数数据库调用都是读取操作，则可以通过在数据库前面实施缓存层来卸载这些读取操作，以提高应用程序的速度并降低数据库的负载。可以通过在数据库前面放置缓存层，来提高应用程序的速度并降低数据库负载（因而提高它可以处理的规模）。若要了解有关 Azure Redis 缓存的详细信息，请阅读 [Caching guidance（缓存指南）](/documentation/articles/best-practices-caching/)。
 
 __如果不在数据库前面使用缓存，会发生什么情况？__ 如果数据库计算机的能力足以处理施加于其上的流量负载，则应用程序将正常做出响应，但这也可能意味着在负载较低时，数据库计算机的开销相对较不划算。如果数据库计算机的能力不足以处理负载，则用户的应用程序体验将开始恶化（延迟、超时，甚至服务中断）。
 
###如果预期会出现大规模事件，你是否联系过 Azure 支持部门？
Azure 支持部门可帮助你提高服务限制以应对计划内的高流量事件（例如新产品推出或特殊节假日）。Azure 支持部门还可帮助你联系专家，从而帮助你与帐户团队一起审查设计，并找到可满足大规模事件需求的最佳解决方案。若要了解有关如何联系 Azure 支持部门的详细信息，请阅读 [Azure Support FAQs（Azure 支持常见问题）](https://azure.microsoft.com/support/faq/)。

__如果未联系 Azure 支持部门来帮助处理大规模事件，会发生什么情况？__ 如果未针对高流量事件进行沟通或规划，你可能会达到特定的 [Azure 服务限制](/documentation/articles/azure-subscription-service-limits/)，因而在事件期间造成用户体验不佳（更糟的是停机）。在高峰出现之前做好体系结构审查和沟通有助于缓解这些风险。

###是否在面向 Web 的存储 Blob 和静态资产前面使用内容交付网络 (Azure CDN)？
使用 CDN 可在世界各地的 CDN POP/边缘位置缓存内容，从而帮助减轻服务器的负载。这样做可以缩短延迟、提高可缩放性、减少服务器负载，并采用保护策略来防范拒绝服务 (DOS) 攻击。若要了解有关如何使用 Azure CDN 提高复原能力并缩短客户延迟的详细信息，请阅读 [Overview of the Azure Content Delivery Network (CDN)（Azure 内容交付网络 (CDN) 概述）](/documentation/articles/cdn-overview/)。

__不使用 CDN 会发生什么情况？__ 如果不使用 CDN，则所有客户流量将直接定向到你的资源。这意味着服务器的负载会升高，导致其可缩放性降低。此外，客户可能会遇到更高的延迟。相比之下，CDN 在世界各地的位置可能更靠近你的客户。

##后续步骤：
若要了解有关如何设计应用程序以实现高可用性的详细信息，请阅读[构建在 Azure 基础之上的应用程序高可用性](/documentation/articles/resiliency-high-availability-azure-applications/)。
<!---HONumber=Mooncake_0627_2016-->