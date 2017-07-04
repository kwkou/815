<properties
    pageTitle="有关在发生 Azure 区域服务中断时进行复原的技术指南 | Azure"
    description="了解和设计可复原、高度可用、容错的应用程序，以及针对灾难恢复进行规划。"
    services=""
    documentationcenter="na"
    author="adamglick"
    manager="saladki"
    editor="" />
<tags
    ms.assetid="f2f750aa-9305-487e-8c3f-1f8fbc06dc47"
    ms.service="resiliency"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="08/18/2016"
    wacn.date="03/24/2017"
    ms.author="aglick" />

# Azure 复原技术指南：在发生区域范围的服务中断后进行恢复

Azure 在物理上和逻辑上划分为称为区域的单位。一个区域由一个或多个邻近的数据中心组成。在撰写本文时，Azure 在中国共有 2 个区域。

只有在极少数情况下，才会出现整个区域的设施都无法访问的情况，例如，网络故障，或自然灾害等原因造成的完全失去联系。本文说明可用于创建在各区域间分布的应用程序的 Azure 功能。这种分布有助于最大程度地减少一个区域的故障影响到其他区域的可能性。

## <a id="cloud-services"></a> 云服务

### 资源管理

要将计算实例分布在多个区域上，可在每个目标区域中创建单独的云服务，然后将部署包发布到每个云服务上。但是，必须由应用程序开发人员或使用流量管理服务在不同区域的云服务间分布流量。

在容量规则方面，事先确定为实现灾难恢复而部署的备用角色实例数目是其中的重要一环。具有规模完整的辅助部署可确保容量在需要时始终可用。但是，这实际上也会使成本加倍。常见的模式是准备小型的辅助部署，规模足以满足运行关键服务即可。建议你建立一个小型辅助部署用于保留容量和测试辅助环境的配置。

> [AZURE.NOTE]
> 订阅配额并非容量保证。配额只是信用限制。若要保证容量，必须在服务模型中定义所需数量的角色，且必须部署这些角色。
> 
> 

### 负载均衡

若要实现各区域流量的负载均衡，需要使用流量管理解决方案。Azure 提供 [Azure 流量管理器](/home/features/traffic-manager/)。你也可以利用提供类似流理管理功能的第三方服务。

### 策略

可以使用多种备用策略在区域间实现分布式计算。这些策略必须根据特定业务要求和应用程序的情况量身定制。大体而言，方法可分为以下几类：

* **发生灾难时重新部署**。如果采用这种方法，则会在灾难发生时从头开始重新部署应用程序。这种重新部署适合不需要保证恢复时间的非关键应用程序。

* **暖备份（主动/被动）**。在备用区域创建次要托管服务，并部署角色以保证最低容量。但是，角色不会接收生产流量。对于未设计为在区域间分布流量的应用程序来说，这种方法很有用。

* **热备份（主动/主动）**。应用程序设计为接收多个区域的生产负载。为每个区域中云服务所配置的容量可能高于灾难恢复用途所需的容量。或者，云服务可在灾难或故障转移时进行扩展。这种方法需要在应用程序设计中进行大量的投资，但是也具有明显的好处。这些好处包括恢复时间更短且更有保证、连续测试所有恢复位置和对容量的有效使用。

关于分布式设计的完整讨论不属于本文档的范畴。有关更多信息，请参阅 [Disaster Recovery and High Availability for Azure Applications（Azure 应用程序的灾难恢复和高可用性）](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)。

## <a id="virtual-machines"></a> 虚拟机

基础结构即服务 (IaaS) 虚拟机 (VM) 的恢复在许多方面与平台即服务 (PaaS) 计算恢复相似。但是，由于 IaaS VM 包含 VM 和 VM 磁盘，因此两者之间也存在一些重要的差异。

* **使用 Azure 备份创建应用程序一致的跨区域备份**。[Azure 备份](/home/features/back-up/)可让客户跨多个 VM 磁盘创建应用程序一致的备份，并支持跨区域的备份复制。若要完成此任务，可在创建时选择异地复制备份保管库。必须在创建时配置备份保管库的复制，而不能在以后设置。如果某个区域发生服务中断，Azure.cn 将向客户提供备份。然后，客户可以还原到配置的任何还原点。

* **将数据磁盘与操作系统磁盘分开**。有关 IaaS VM 的一项重要注意事项就是，如果你不重新创建 VM 就不能更改操作系统磁盘。如果恢复策略是在灾难后部署的，此过程不会造成问题。但是，如果你使用暖备份方法来保留容量，则它可能会成为问题。若要正确实现这一点，必须将正确的操作系统磁盘部署到主要位置和辅助位置，将应用程序数据存储在单独的驱动器上。如果可能，请使用这两个位置均可以提供标准的操作系统配置。在故障转移后，必须将数据驱动器连接到辅助 DC 中的现有 IaaS VM。可以使用 AzCopy 将每个数据磁盘的快照复制到远程站点。

* **注意异地故障转移多个 VM 磁盘后的潜在一致性问题**。每个 VM 磁盘是实现为一个 Azure Blob 存储实例，每个磁盘具有相同的异地复制特征。除非使用 [Azure 备份](/home/features/back-up/)，否则不能保证磁盘间的一致性，因为异地复制是异步进行的，并且是独立复制的。可以保证 VM 磁盘在异地故障转移后处于崩溃一致状态，但不能保证磁盘间的一致性。在某些情况下（例如使用磁盘条带化时）可能会产生问题。

## <a id="storage"></a> 存储

### 使用 Blob、表、队列和 VM 磁盘存储的异地冗余存储进行恢复

在 Azure 中，默认情况下，Blob、表、队列和 VM 磁盘都会进行异地复制。这称为异地冗余存储。异地冗余存储将存储数据复制到几百英里以外特定地理区域中的配对数据中心中。异地冗余存储旨在提供额外的持久性，以防出现重大的数据中心灾难。Azure.cn 控制故障转移发生的时间且故障转移限于重大灾难，也就是原始主要位置被视为无法在合理时间内恢复。在某些情况下，这可能需要几天时间。尽管同步间隔在服务级别协议中并未提及，但是数据通常在几分钟内完成复制。

发生异地故障转移时，不会对访问帐户的方式进行更改（URL 和帐户密钥不会更改）。但是，在故障转移后存储帐户将位于另一个区域。这可能会影响需要与存储帐户保持区域关系的应用程序。即使服务和应用程序不要求存储帐户位于相同的数据中心，跨数据中心的延迟性和带宽费用也使暂时将流量移到故障转移区域成为一种明智之举。整体的灾难恢复策略可以考虑此项因素。

除了异地冗余存储提供的自动故障转移之外，Azure 还推出了一种服务，让你能够对位于辅助存储位置的数据副本进行读取访问。这称为读取访问异地冗余存储。

有关异地冗余存储和读取访问异地冗余存储的详细信息，请参阅 [Azure Storage replication](/documentation/articles/storage-redundancy/)（Azure 存储复制）。

### 异地复制区域映射

必须了解异地复制会将数据复制到哪里，这样才能知道，对于要求与存储保持区域关系的数据，应将其实例部署到哪里。下表显示了配对的主要位置和辅助位置。

[AZURE.INCLUDE [paired-region-list](../../includes/paired-region-list.md)]

### 异地复制定价

与异地冗余存储一样，Azure 存储的当前定价包括异地复制。如果不想对数据进行异地复制，可为帐户禁用异地复制。这称为本地冗余存储，将按照异地冗余存储的折扣价格收费。

### 确定是否发生了异地故障转移

如果发生了异地故障转移，失败状态将发布到 [Azure 服务运行状况仪表板](/support/service-dashboard/)。但是，应用程序可以通过监视其存储帐户的地理区域，来实施自动检测失败的方法。这种自动检测可用于触发其他恢复操作，如激活存储移往的地理区域中的计算资源。可以使用[获取存储帐户属性](https://msdn.microsoft.com/zh-cn/library/ee460802.aspx)通过服务管理 API 针对此事件执行查询。相关的属性包括：

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### VM 磁盘和异地故障转移

如本文的“虚拟机”部分中所述，故障转移后无法保证 VM 磁盘之间的数据一致性。为确保备份正确，应使用 System Center Data Protection Manager 等备份产品来备份和还原应用程序数据。

## 数据库

### <a id="sql-database"></a> Azure SQL 数据库

Azure SQL 数据库提供两种类型的恢复：异地还原和活动异地复制。

#### 异地还原

[异地还原](/documentation/articles/sql-database-recovery-using-backups/#geo-restore)也适用于基本、标准和高级数据库。当数据库由于它所在的区域发生事故而不可用时，异地还原会提供默认的恢复选项。与时间点还原一样，异地还原依赖于异地冗余的 Azure 存储中的数据库备份。它会从异地复制的备份副本中还原，因此可以灵活应对主要区域中的存储中断。有关详细信息，请参阅[还原 Azure SQL 数据库或故障转移到辅助数据库](/documentation/articles/sql-database-disaster-recovery/)。

#### 活动异地复制

[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)适用于所有数据库层。它专为恢复要求超出了异地还原的能力的应用程序设计。使用活动异地复制，最多可以在不同区域中的服务器上创建四个可读辅助数据库。可以启动到任何辅助数据库的故障转移。此外，活动异地复制可用于支持应用程序升级或重定位方案，以及只读工作负荷的负载均衡。有关详细信息，请参阅[配置活动异地复制](/documentation/articles/sql-database-geo-replication-portal/)和[故障转移到辅助数据库](/documentation/articles/sql-database-geo-replication-failover-portal/)。有关如何在无需停机的情况下设计和实现应用程序及应用程序升级的详细信息，请参阅[使用 SQL 数据库中的活动异地复制功能针对云灾难恢复设计应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)和[使用 SQL 数据库活动异地复制功能管理云应用程序的滚动升级](/documentation/articles/sql-database-manage-application-rolling-upgrade/)。

### <a id="sql-server-on-virtual-machines"></a> Azure 虚拟机中的 SQL Server

可以使用各种选项对 Azure 虚拟机部署中运行的 SQL Server 2012（和更高版本）实现恢复和高可用性目的。有关详细信息，请参阅 [High availability and disaster recovery for SQL Server in Azure Virtual Machines](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)（Azure 虚拟机中 SQL Server 的高可用性和灾难恢复）。


## <a id="other-azure-platform-services"></a> 其他 Azure 平台服务

在尝试于多个 Azure 区域运行云服务时，必须考虑每个依赖项的含义。在以下部分中，服务特定的指南假设你在备用 Azure 数据中心中使用相同的 Azure 服务。这同时涉及到配置和数据复制任务。

> [AZURE.NOTE]
> 在某些情况下，这些步骤有助于缓解服务特定的中断，而不是整个数据中心的中断事件。从应用程序的角度而言，服务特定的中断可能只是受到限制，需要临时将服务迁移到备用 Azure 区域。
> 
> 

### Azure 服务总线

Azure 服务总线使用不跨越 Azure 区域的唯一命名空间。因此，首要要求是在备用区域中设置必要的服务总线命名空间。但是，对于排队消息的持久性，也有一些注意事项。有几种在 Azure 区域间复制消息的策略。有关这些复制策略和其他灾难恢复策略的详细信息，请参阅 [Best practices for insulating applications against Service Bus outages and disasters](/documentation/articles/service-bus-outages-disasters/)（使应用程序免受服务总线中断和灾难影响的最佳实践）。有关其他可用性注意事项，请参阅 [Service Bus (Availability)](/documentation/articles/resiliency-technical-guidance-recovery-local-failures/#other-azure-platform-services)（服务总线（可用性））。

### <a id="web-apps"></a> Azure App Service

若要将 Azure App Service 应用程序（例如 Web 应用或移动应用）迁移到次要 Azure 区域，必须拥有该网站可供发布的备份。如果中断不涉及整个 Azure 数据中心，则也许可以使用 FTP 下载站点内容的最新备份。然后，在备用区域创建新应用，除非之前已执行此操作来保留容量。将站点发布到新区域，然后进行必要的配置更改。这些更改可能包括数据库连接字符串或其他区域特定的设置。如有必要，请添加站点的 SSL 证书，并更改 DNS CNAME 记录，以使自定义域名指向重新部署的 Web 应用 URL。

### Azure HDInsight

与 Azure HDInsight 关联的数据默认存储在 Azure Blob 存储中。HDInsight 要求处理 MapReduce 作业的 Hadoop 群集必须与包含所分析数据的存储帐户位于同一区域中。如果使用可用于 Azure 存储的异地复制功能，在主要区域不再可用的情况下，可以访问已复制到次要区域的数据。可以在数据复制到的区域中创建 Hadoop 群集并继续处理这些数据。有关其他可用性注意事项，请参阅 [HDInsight (Availability)](/documentation/articles/resiliency-technical-guidance-recovery-local-failures/#other-azure-platform-services)（HDInsight（可用性））。

### <a id="sql-reporting"></a> SQL 报告

目前，要从 Azure 区域的损失中恢复，需要有多个位于其他 Azure 区域的 SQL 报告实例。这些 SQL 报告实例应当访问相同的数据，且该数据应当有自己的恢复计划用于处理灾难。你还可以为每个报表维护 RDL 文件的外部备份副本。

### <a id="media-services"></a> Azure 媒体服务

Azure 媒体服务对于编码和流有不同的恢复方法。通常，在区域中断期间，流更为重要。若要做好准备，应在两个不同的 Azure 区域中各有一个媒体服务帐户。这两个区域都应有编码的内容。在故障期间，你可以将流式流量重定向到备用区域。编码可在任意 Azure 区域执行。如果编码对时间敏感，如在现场活动处理期间，必须准备好如何在故障期间向备用数据中心提交代码。

### <a id="virtual-network"></a> 虚拟网络

配置文件是在备用 Azure 区域设置虚拟网络的最快速方式。在主要 Azure 区域配置虚拟网络之后，将当前网络的[虚拟网络设置导出](/documentation/articles/virtual-networks-create-vnet-classic-portal/)为网络配置文件。主要区域发生中断时，可从存储的配置文件[还原虚拟网络](/documentation/articles/virtual-networks-create-vnet-classic-portal/)。然后，配置其他云服务、虚拟机或跨部署设置，来使用新的虚拟网络。

## 灾难恢复清单

## 云服务清单

1. 查看本文的“云服务”部分。
2. 创建跨区域的灾难恢复策略。
3. 了解在备用区域保留容量的利弊。
4. 使用流量路由工具，如 Azure 流量管理器。

## 虚拟机清单

1. 查看本文的“虚拟机”部分。
2. 使用 [Azure 备份](/home/features/back-up/)创建应用程序一致的跨区域备份。

## 存储清单

1. 查看本文的“存储”部分。
2. 不要禁用存储资源的异地复制。
3. 了解发生故障转移时用于异地复制的备用区域。
4. 为用户控制的故障转移策略创建自定义备份策略。

## Azure SQL 数据库清单

1. 查看本文的“SQL 数据库”部分。
2. 根据情况使用[异地还原](/documentation/articles/sql-database-recovery-using-backups/#geo-restore)或[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)。

## Azure 虚拟机上的 SQL Server 清单

1. 查看本文的“Azure 虚拟机上的 SQL Server”部分。
2. 使用跨区域 AlwaysOn 可用性组或数据库镜像。
3. 也可以使用备份和还原到 Blob 存储。

## 服务总线清单

1. 查看本文的“Azure 服务总线”部分。
2. 在备用区域中配置服务总线命名空间。
3. 考虑自定义跨区域消息的复制策略。

## 应用服务清单

1. 查看本文的“Azure 应用服务”部分。
2. 在主要区域外部维护站点备份。
3. 如果发生局部中断，可尝试使用 FTP 检索当前站点。
4. 计划将站点部署到备用区域中的新站点或现有站点。
5. 计划对应用程序和 DNS CNAME 记录的配置更改。

## HDInsight 清单

1. 查看本文的“Azure HDInsight”部分。
2. 使用复制的数据在区域中创建 Hadoop 群集。

## SQL 报告清单

1. 查看本文的“SQL 报告”部分。
2. 在另一个区域维护备用 SQL 报告实例。
3. 维护单独的计划以将目标复制到该区域。

## 媒体服务清单

1. 查看本文的“Azure 媒体服务”部分。
2. 在备用区域创建媒体服务帐户。
3. 在两个区域对相同的内容进行编码，以支持流式故障转移。
4. 在发生服务中断时，将编码作业提交到备用区域。

## 虚拟网络清单

1. 查看本文的“虚拟网络”部分。
2. 使用导出的虚拟网络设置，在另一个区域重新创建该虚拟网络。

## 后续步骤

本文是着重介绍 [Azure 复原技术指南](/documentation/articles/resiliency-technical-guidance/)的系列教程的一部分。本系列教程的下一篇文章着重介绍如何[从本地数据中心恢复到 Azure](/documentation/articles/resiliency-technical-guidance-recovery-on-premises-azure/)。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: update meta properties; wording update -->