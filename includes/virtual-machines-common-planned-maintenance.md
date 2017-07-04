## <a name="memory-preserving-updates"></a>内存保留更新
对于 Azure 中的某一类更新，客户发现它们不会对正在运行的虚拟机产生任何影响。 其中一些更新主要面向组件或服务，更新时不会干扰正在运行的实例。 还有一些更新是主机操作系统的平台基础结构更新，应用时无需完全重启虚拟机。

这些更新通过启用就地实时迁移的技术（也称为“内存保留”更新）实现。 更新时，虚拟机进入“暂停”状态，以保留 RAM 中的内存，而基础主机操作系统接收必要的更新和修补程序。 虚拟机在暂停后 30 秒内恢复正常。 恢复后，虚拟机的时钟将自动同步。

并非所有更新都可以通过使用此机制进行部署，但如果短暂时间较短，使用此方法部署更新可大大减少对虚拟机的影响。

多实例更新（针对可用性集中的虚拟机）一次应用一个更新域。  

## <a name="virtual-machine-configurations"></a>虚拟机配置
可选择两种虚拟机配置：多实例和单实例。 在多实例配置中，相似的虚拟机放置在可用性集中。

多实例配置可跨物理计算机、电源和网络提供冗余，若要确保应用程序的可用性，建议采用此配置。 可用性集中的所有虚拟机应对应用程序具有相同的用途。

有关配置虚拟机以实现高可用性的详细信息，请参阅[管理 Windows 虚拟机的可用性](/documentation/articles/virtual-machines-windows-manage-availability/)或[管理 Linux 虚拟机的可用性](/documentation/articles/virtual-machines-linux-manage-availability/)。

相反，单实例配置用于未放置在可用性集中的独立虚拟机。 这些虚拟机不符合服务级别协议 (SLA) 的要求，SLA 要求在同一可用性集中部署两个或更多虚拟机。

有关 SLA 的详细信息，请参阅[服务级别协议](/support/legal/sla/)中的“云服务和虚拟机”部分。

## <a name="multi-instance-configuration-updates"></a>多实例配置更新
在计划内维护期间，Azure 平台首先更新托管在多实例配置中的虚拟机集。 更新将导致这些虚拟机重新启动，从而有大约 15 分钟的停机时间。

多实例配置更新假定每个虚拟机在可用性集中发挥的功能与其他虚拟机类似。 在此设置中，虚拟机进行更新时采用的方式可确保整个过程中的可用性。

基础 Azure 平台为可用性集中的每个虚拟机分配一个更新域和一个容错域。 每个更新域是在同一时间内重新启动的一组虚拟机。 每个容错域是共享公共电源和网络交换机的一组虚拟机。

有关更新域和容错域的详细信息，请参阅[配置可用性集中的多个虚拟机以实现冗余](/documentation/articles/virtual-machines-windows-manage-availability/#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)。

为了通过更新保持可用性，Azure 通过更新域进行维护，一次更新一个域。 在更新域中进行维护时，其步骤包括：关闭域中的每个虚拟机，对主机应用更新，然后重新启动虚拟机。 当域中的维护完成以后，Azure 对下一个更新域重复此过程，继续对每个域进行操作，直至所有域都进行了更新。

在计划内维护期间，更新域的重启顺序可能不会按序进行，但一次只能重启一个更新域。 现在，Azure 可为多实例配置中虚拟机的计划内维护提供 1 周提前通知功能。

虚拟机还原后，下述示例演示了 Windows 事件查看器可能会显示的内容：

<!--Image reference-->
![][image2]

使用查看器报告在使用 Azure 门户、Azure PowerShell 或 Azure CLI 的多实例配置中配置的虚拟机。 例如，可以使用 Azure 门户将“可用性集”添加到“虚拟机(经典)”浏览器对话框。 用于报告同一可用性集的虚拟机是多实例配置的一部分。 在以下示例中，多实例配置包含虚拟机 SQLContoso01 和 SQLContoso02。

<!--Image reference-->
  ![Azure 门户中的“虚拟机(经典)”视图][image4]

## <a name="single-instance-configuration-updates"></a>单实例配置更新
完成多实例配置更新后，Azure 将执行单实例配置更新。 这些更新也会导致不在可用性集中运行的虚拟机重新启动。

> [AZURE.NOTE]
> 如果可用性集只有一个虚拟机实例在运行，Azure 平台会将其视作多实例配置更新。
>

在单实例配置中进行维护时，其步骤包括：关闭在主机上运行的每个虚拟机，更新主机，然后重新启动虚拟机。 维护需要大约 15 分钟的停机时间。 计划内维护事件在单个维护时段内跨某个区域中的所有虚拟机运行。

对于单实例配置，计划内维护事件可能会对应用程序的可用性产生影响。 Azure 可为单实例配置中虚拟机的计划内维护提供一周高级通知功能。

## <a name="email-notification"></a>电子邮件通知
Azure 会提前发送电子邮件通信，提醒用户即将执行计划内维护（提前一周），该功能仅适用于单实例和多实例虚拟机配置。 此电子邮件发送到订阅管理员和共同管理员的电子邮件帐户。 下面是这类电子邮件的示例：

<!--Image reference-->
![][image1]

## <a name="region-pairs"></a>区域对

执行维护时，Azure 只更新区域对中单个区域的虚拟机实例。 例如，更新中国北部的虚拟机时，Azure 不会同时更新中国东部的任何虚拟机。 后者会安排在其他时间进行，以便在区域之间进行故障转移或负载均衡。 但是，中国北部等其他区域可以与中国东部同时进行维护。

请参阅下表以了解当前区域对：

| 区域 1 | 区域 2 |
|:--- | ---:|
| 中国北部 |中国东部 |
| 中国东部 |中国北部 |

<!--Anchors-->
[image1]: ./media/virtual-machines-common-planned-maintenance/vmplanned1.png
[image2]: ./media/virtual-machines-common-planned-maintenance/EventViewerPostReboot.png
[image3]: ./media/virtual-machines-planned-maintenance/RegionPairs.PNG
[image4]: ./media/virtual-machines-common-planned-maintenance/AvailabilitySetExample.png

<!--Link references-->
[Virtual Machines Manage Availability]: /documentation/articles/virtual-machines-windows-hero-tutorial/

[Understand planned versus unplanned maintenance]: /documentation/articles/virtual-machines-windows-manage-availability/#Understand-planned-versus-unplanned-maintenance