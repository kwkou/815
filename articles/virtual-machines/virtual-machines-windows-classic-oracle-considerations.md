<properties
    pageTitle="使用 Oracle VM 映像的注意事项 | Azure"
    description="在部署之前，了解 Azure 中 Windows Server 上的 Oracle VM 支持的配置以及限制。"
    services="virtual-machines-windows"
    documentationcenter=""
    manager="timlt"
    author="rickstercdn"
    tags="azure-service-management" />
<tags
    ms.assetid="5d71886b-463a-43ae-b61f-35c6fc9bae25"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="04/11/2017"
    wacn.date="05/15/2017"
    ms.author="rclaus"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="0eefb9f3a7153ccf8cd65805756f30d48f593903"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="miscellaneous-considerations-for-oracle-virtual-machine-images"></a>有关 Oracle 虚拟机映像的其他注意事项
本文介绍 Azure 中基于 Oracle 软件映像的 Oracle 虚拟机的注意事项，这些映像用户可以自行上传，使用 Windows Server 作为操作系统。你也可以本地创建 VHD 然后上传到 Azure 来创建虚拟机。

* Oracle 数据库虚拟机映像
* Oracle WebLogic Server 虚拟机映像
* Oracle JDK 虚拟机映像

## <a name="oracle-database-virtual-machine-images"></a>Oracle 数据库虚拟机映像

### <a name="no-static-internal-ip"></a>没有静态内部 IP
Azure 为每个虚拟机分配内部 IP 地址。 除非虚拟机是虚拟网络的一部分，否则虚拟机的 IP 地址就是动态的，在虚拟机重启后可能会更改。 这会导致问题，因为 Oracle 数据库需要静态的 IP 地址。 若要避免此问题，请考虑将虚拟机添加到 Azure 虚拟网络中。 有关详细信息，请参阅[虚拟网络](/documentation/services/networking/)和[在 Azure 中创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)。

### <a name="attached-disk-configuration-options"></a>附加磁盘配置选项

附加磁盘依赖于 Azure Blob 存储服务。 每个标准磁盘理论上每秒最多能够完成大约 500 个输入/输出操作 (IOPS)。 高级磁盘产品主要针对高性能数据库工作负荷，每个磁盘 IOPS 高达 5000。 尽管可使用单个磁盘（如果这可满足性能需求），但是如果使用多个附加磁盘，将数据库数据分散到这些磁盘上，然后使用 Oracle 自动存储管理 (ASM)，通常可以提高有效 IOPS 性能。 有关详细信息，请参阅 [Oracle 自动存储概述](http://www.oracle.com/technetwork/database/index-100339.html)。 尽管可以在操作系统级别使用多个磁盘的条带化，但是使用其中任何一种路由都需要权衡取舍。 

根据是要优先考虑数据库的读操作性能还是写操作性能，考虑两种附加多个磁盘的不同方法：

* **Oracle ASM 本身**与使用磁盘阵列的方法相比，可能会导致更好的写操作性能，但会导致更差的读操作 IOPS。 下图在逻辑上描绘了此安排。  
    ![](./media/virtual-machines-windows-classic-mysql-2008r2/image2.png)

> [AZURE.IMPORTANT]
> 逐条评估写性能和读性能之间的得失。 实际结果可能有所不同 - 请执行正确的测试。 ASM 倾向于写入操作，操作系统磁盘条带化倾向于读取操作。
> 

### <a name="clustering-rac-is-not-supported"></a>不支持群集功能 (RAC)
Oracle Real 应用程序群集 (RAC) 用于减少本地多节点群集配置中单一节点的故障。  它依赖于网络多播和共享磁盘这两项本地技术，而这两项技术不适用于 Azure 等超大规模公有云环境。 若要构建 Oracle DB 的异地冗余多节点配置，需要通过 Oracle DataGuard 实现数据复制。

### <a name="high-availability-and-disaster-recovery-considerations"></a>高可用性和灾难恢复注意事项
在 Azure 虚拟机中使用 Oracle 数据库时，你负责实现高可用性和灾难恢复解决方案，以避免任何停机。 你还负责备份自己的数据和应用程序。

可以在 Azure 上使用[数据防护、活动数据防护](http://www.oracle.com/technetwork/articles/oem/dataguardoverview-083155.html)或 [Oracle Golden Gate](http://www.oracle.com/technetwork/middleware/goldengate) 实现 Oracle Database Enterprise Edition（不带 RAC）的高可用性和灾难恢复，其中两个数据库位于两个单独的虚拟机中。 这两个虚拟机应都位于相同的[虚拟网络](/documentation/services/networking/)，以确保它们能够通过专用持久性 IP 地址相互访问。  另外，我们建议将虚拟机放置在同一[可用性集](/documentation/articles/virtual-machines-windows-manage-availability/)中，以便 Azure 将它们置于单独的容错域和升级域中。 每台虚拟机必须至少具有 2 GB 内存和 5 GB 磁盘空间。

有了 Oracle 数据防护，可以通过一个虚拟机中的主数据库、另一个虚拟机中的辅助（备用）数据库和它们之间的单向复制设置实现高可用性。 结果是对数据库副本的读取访问权限。 使用 Oracle GoldenGate，可以配置两个数据库之间的双向复制。 若要了解如何使用这些工具为数据库设置高可用性解决方案，请参阅 Oracle 网站上的[活动数据防护](http://www.oracle.com/technetwork/database/features/availability/data-guard-documentation-152848.html)和 [GoldenGate](http://docs.oracle.com/goldengate/1212/gg-winux/index.html) 文档。 如果需要对数据库副本的读写访问权限，可以使用 [Oracle 活动数据防护](http://www.oracle.com/uk/products/database/options/active-data-guard/overview/index.html)。

## <a name="oracle-weblogic-server-virtual-machine-images"></a>Oracle WebLogic Server 虚拟机映像
* **群集仅在 Enterprise Edition 上受支持。** 仅在使用 WebLogic Server 的 Enterprise Edition 时许可使用 WebLogic 群集。 不要将群集用于 WebLogic Server 标准版。
* **连接超时：**如果应用程序依赖于与另一个 Azure 云服务（例如，数据库层服务）的公共终结点的连接，Azure 可能会在处于非活动状态四分钟后关闭这些打开的连接。 这可能会影响依赖于连接池的功能和应用程序，因于处于非活动状态的时间超过该限制的连接可能不再保持有效。 如果这会影响你的应用程序，请考虑对连接池启用“保持活动状态”逻辑。

    如果终结点位于 Azure 云服务部署的*内部*（例如*同一*云服务中作为 WebLogic 虚拟机的独立数据库虚拟机），则连接是直接的，并且不依赖 Azure 负载均衡器，因此不受连接超时影响。
* **不支持 UDP 多播。** Azure 支持 UDP 单播，但不支持多播和广播。 WebLogic Server 能够依赖于 Azure 的 UDP 单播功能。 为了获得依赖于 UDP 单播的最佳结果，建议将 WebLogic 群集大小保留静态，或者使群集中的托管服务器不超过 10 个。
* **WebLogic Server 预期 T3 访问（例如，使用企业 JavaBean 时）的公用端口和专用端口是相同的。** 请考虑多层方案，其中服务层 (EJB) 应用程序运行在 WebLogic Server 群集中，该群集包含名为 **SLWLS** 的云服务中的两个或更多个托管服务器。 客户端层位于不同云服务中，该云服务运行尝试调用服务层中的 EJB 的简单 Java 程序。 由于有必要对服务层进行负载均衡，因此需要为 WebLogic Server 群集中的虚拟机创建公共负载均衡终结点。 如果为该终结点指定的专用端口不同于公用端口（例如，7006:7008），则会发生下面这样的错误：

        [java] javax.naming.CommunicationException [Root exception is java.net.ConnectException: t3://example.chinacloudapp.cn:7006:

        Bootstrap to: example.chinacloudapp.cn/138.91.142.178:7006' over: 't3' got an error or timed out]

    这是因为对于任何远程 T3 访问，WebLogic Server 都要求负载均衡器端口和 WebLogic 托管服务器端口是相同的。 在上面的示例中，客户端访问端口 7006（负载均衡器端口），托管服务器侦听 7008（专用端口）。 此限制仅适用于 T3 访问，而不适用于 HTTP。

    若要避免此问题，请使用下列解决方法之一：

    * 对于专用于 T3 访问的负载均衡终结点，可使用相同的专用和公用端口号。
    * 启动 WebLogic Server 时，请包括以下 JVM 参数：

            -Dweblogic.rjvm.enableprotocolswitch=true

    如需相关信息，请参阅 <http://support.oracle.com> 上的知识库文章 **860340.1**。

* **动态群集和负载均衡限制。** 假设你要在 WebLogic Server 中使用动态群集并通过 Azure 中单个公共负载均衡终结点公开它。 只要你对每个托管服务器使用固定的端口号（不是从范围中动态分配的），并且启动的托管服务器数不超过管理员正在跟踪的计算机数（即，每个虚拟机不能有一个以上托管服务器），就可以完成此操作。 如果配置导致启动的 WebLogic Server 数多于存在的虚拟机数（即，多个 WebLogic Server 实例共享同一个虚拟机），则多个 WebLogic Server 服务器实例不能绑定到给定端口号 - 该虚拟机上的其他此类实例将失败。

    另一方面，如果你将管理服务器配置为自动向其托管服务器分配唯一端口号，则负载均衡将不可能，因为 Azure 不支持从单个公用端口映射到多个专用端口，而这是此配置所必需的。
* **虚拟机上的 Weblogic Server 的多个实例。** 如果虚拟机足够大，则可考虑按部署要求在同一个虚拟机上运行多个 WebLogic Server 实例。 例如，在包含两个内核的中等大小虚拟机上，可以选择运行两个 WebLogic Server 实例。 但请注意，仍建议你避免在你的体系结构中引入单点故障，如果你只使用了一个运行多个 WebLogic Server 实例的虚拟机，就会出现这种情况。 至少使用两个虚拟机可能是更好的方法，然后其中每个虚拟机可以运行多个 WebLogic Server 实例。 其中每个 WebLogic Server 实例仍可以是同一个群集的一部分。 但是，请注意，目前不能使用 Azure 对同一个虚拟机中此类 WebLogic Server 部署公开的终结点进行负载均衡，因为 Azure 负载均衡器需要将负载均衡服务器分布到唯一的虚拟机中。

## <a name="oracle-jdk-virtual-machine-images"></a>Oracle JDK 虚拟机映像
* **JDK 6 和 7 最新更新。** 尽管建议使用最新公开支持的 Java 版本（当前为 Java 8），但 Azure 还提供 JDK 6 和 7 的映像。 这适用于尚未准备好升级到 JDK 8 的旧版应用程序。 虽然对旧版 JDK 映像的更新不再提供给公众，但是鉴于 Azure.cn 与 Oracle 的合作关系，由 Azure 提供的 JDK 6 和 7 的映像将包含最近的非公开更新，Oracle 通常只向一组自己支持的选定客户提供该更新。 随着时间推移，将使用 JDK 6 和 7 的更新版本提供 JDK 映像的新版本。

    在此 JDK 6 和 7 的映像中提供的 JDK 以及从中派生的虚拟机和映像只能在 Azure 中使用。
* **64 位 JDK。** Azure 提供的 Oracle WebLogic Server 虚拟机映像和 Oracle JDK 虚拟机映像包含 64 位版本的 Windows Server 和 JDK。