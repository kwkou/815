<properties
	pageTitle="SQL (PaaS) 数据库与VM 上的云中 SQL Server (IaaS) | Azure"
	description="了解哪个云 SQL Server 选项适合你的应用程序：Azure SQL (PaaS) 数据库或 Azure 虚拟机上的云中 SQL Server。"
	services="sql-database, virtual-machines"
	keywords="SQL Server 云, 云中 SQL Server, PaaS 数据库, 云 SQL Server, DBaaS"
	documentationCenter=""
	authors="carlrabeler"
	manager="jhubbard"
	editor="cjgronlund"/>

<tags
	ms.service="sql-database"
	ms.date="03/25/2016"
	wacn.date="05/16/2016"/>

# 选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)

Azure 提供两个选项用于在云中托管 SQL Server 工作负荷：

* [Azure SQL 数据库](/home/features/sql-database)：云的本机 SQL 数据库，也称为平台即服务 (PaaS) 数据库或数据库即服务 (DBaaS)，它已针对软件即服务 (SaaS) 应用开发进行优化。Azure SQL 数据库与大多数 SQL Server 功能兼容。
* [Azure 虚拟机上的 SQL Server](/home/features/virtual-machines#SQL)：在 Azure 上运行的云中 Azure 虚拟机 (VM) 上安装并托管的 SQL Server，也称为基础结构即服务 (IaaS)。

了解每个选项如何配合 Microsoft 数据平台一起运行，并在匹配适合业务要求的选项时获得帮助。无论你是以节省成本为优先考虑，还是将精简管理视为第一要素，本文都会帮助你确定哪种方法能够满足你最重视的业务要求。


## Microsoft 的数据平台

在 Azure 与本地 SQL Server 数据库的任何介绍中，要了解的要点之一是你可以同时使用两者。Microsoft 数据平台利用 SQL Server 技术，使其可在跨本地物理机、私有云环境、第三方托管的私有云环境和公有云中使用。这样，你就可以通过本地和云托管部署的组合来满足独特的多样化业务需求，并同时在这些环境中使用相同的服务器产品、开发工具和专业知识组合。

   ![云 SQL Server 选项：IaaS 上的 SQL Server，或云中的 SaaS SQL 数据库。](./media/data-management-azure-sql-database-and-sql-server-iaas/SQLIAAS_SQL_Server_Cloud_Continuum.png)

如图所示，每个产品可根据你在基础结构（X 轴）中所拥有的管理级别，以及数据库级别合并与自动化（Y 轴）所达到的成本效益程度等特征进行分类。

设计应用程序时，可以使用四个基本选项来托管属于应用程序一部分的 SQL Server：

- 非虚拟化物理机上的 SQL Server
- 本地虚拟机中的 SQL Server（私有云）
- Azure 虚拟机中的 SQL Server（公有云）
- Azure SQL 数据库（公有云）

在以下部分中，我们将了解公有云中的 SQL Server：Azure SQL 数据库和 Azure VM 上的 SQL Server。此外，我们将探讨常见的业务动机，以判断哪一个选项最适合你的应用程序。

## Azure SQL 数据库和 Azure VM 中的 SQL Server 详述

**Azure SQL 数据库**是托管在 Azure 云中的关系数据库即服务 (DBaaS)，属于软件即服务 (SaaS) 和平台即服务 (PaaS) 行业类别。SQL 数据库构建在 Microsoft 所拥有、托管及维护的标准化硬件和软件基础之上。使用 SQL 数据库，你可以使用内置的特性和功能在服务上直接进行开发。使用 SQL 数据库时，你可以即用即付，并使用向上或向外缩放选项获得更强大的功能且不会中断服务。

**Azure 虚拟机 (VM) 上的 SQL Server** 属于基础结构即服务 (IaaS) 行业类别，可让你在云中的虚拟机上运行 SQL Server。与 SQL 数据库一样，它构建在 Microsoft 所拥有、托管及维护的标准化硬件基础之上。使用 VM 中的 SQL Server 时，你可以使用自己的 SQL Server 许可证，或使用 Azure 门户中某个预先授权的 SQL Server 映像。

通常，这两个 SQL 选项已针对不同的用途进行优化：

- **SQL 数据库**经过优化，可将预配和管理许多数据库的整体成本降到最低。由于你无需管理任何虚拟机、操作系统或数据库软件，因此可以持续降低管理成本。这些事务包括升级、高可用性和备份。一般而言，Azure SQL 数据库可以大幅增加由单个 IT 或开发资源管理的数据库数目。
- **Azure VM 中运行的 SQL Server** 经过优化，可在混合方案中将现有本地 SQL Server 应用程序扩展到云，或者在迁移或开发/测试方案中将现有应用程序部署到 Azure。混合方案的一个示例是通过使用 [Azure 虚拟网络](/documentation/articles/virtual-networks-overview/)在 Azure 中保留辅助数据库副本。有了 Azure VM 上的 SQL server，即拥有了专用 SQL Server 实例和基于云的 VM 的完全管理权限。当组织拥有可用来维护虚拟机的 IT 资源时，此选项是最佳选择。使用 VM 上的 SQL Server，你可以构建高度定制的系统，以解决应用程序的特定性能和可用性要求。

下表汇总了 SQL 数据库和 Azure VM 中 SQL Server 的主要特征：

<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle"></th>
   <th align="left" valign="middle">SQL 数据库</th>
   <th align="left" valign="middle">Azure VM 中的 SQL Server</th>

</tr>
<tr>
   <td valign="middle"><p><b>最适合</b></p></td>
   <td valign="middle">
          <ul>
          <li type=round>开发与营销阶段有时间限制的新云式设计应用程序。
          <li type=round>需要内置高可用性、灾难恢复和升级机制的应用程序。
          <li type=round>不想要管理基础操作系统和配置设置的团队。
         <li type=round>使用向外缩放模式的应用程序。
         <li type=round>大小高达 1 TB 的数据库。
         <li type=round>构建软件即服务 (SaaS) 应用程序。
  </ul>
</td>
   <td valign="middle">
      <ul>
      <li type=round>需要几乎无需进行任何更改即可快速迁移到云的现有应用程序。
      <li type=round>需要通过安全隧道从 Azure 访问本地资源的 SQL Server 应用程序（例如 Active Directory）。
      <li type=round>你需要一个具有完全管理权限的定制 IT 环境。
      <li type=round>你想要快速完成开发和测试方案，但又不想购买本地 SQL Server 非生产硬件。
      <li type=round>使用 [备份到 Azure 存储空间](http://msdn.microsoft.com/zh-cn/library/jj919148.aspx) 或 Azure VM 中的 AlwaysOn 副本实现本地 SQL Server 应用程序的灾难恢复。
      <li type=round>大于 1 TB 的大型数据库。
      </ul></td>
</tr>
<tr>
   <td valign="middle"><p><b>资源</b></p></td>
   <td valign="middle">
       <ul>
       <li type=round>你不想要使用 IT 资源来支持和维护底层基础结构。
       <li type=round>你想要将重心放在应用程序层上。
       </ul></td>
   <td valign="middle"><ul><li type=round>你拥有用于支持和维护的 IT 资源。</ul></td>

</tr>
<tr>
   <td valign="middle"><p><b>总拥有成本</b></p></td>
   <td valign="middle"><ul><li type=round>消除硬件成本。降低管理成本。</ul></td>
   <td valign="middle"><ul><li type=round>消除硬件成本。</ul></td>

</tr>
<tr>
   <td valign="middle"><p><b>业务连续性</b></p></td>
   <td valign="middle"><ul><li type=round>除了内置的容错基础结构功能以外，Azure SQL 数据库还提供可提高业务连续性的功能，例如时间点还原、地域还原和异地复制。有关详细信息，请参阅 [Azure SQL 数据库业务连续性概述](/documentation/articles/sql-database-business-continuity/)。</ul></td>
   <td valign="middle"><ul><li type=round>Azure VM 上的 SQL Server 可让你设置高可用性和灾难恢复解决方案，以满足数据库的具体需求。因此，可以构建针对应用程序高度优化的系统。你可以视需要自我测试并运行故障转移。有关详细信息，请参阅 [Azure 虚拟机上 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)。</ul></td>

</tr>
<tr>
   <td valign="middle"><p><b>混合云</b></p></td>
   <td valign="middle"><ul><li type=round>本地应用程序可以访问 Azure SQL 数据库中的数据。</ul></td>
   <td valign="middle"><ul>
      <li type=round>使用 Azure VN 上的 SQL Server，应用程序可以一部分在云中运行，一部分在本地运行。例如，可以通过 [Azure 虚拟网络](/documentation/articles/virtual-networks-overview/)，将本地网络和 Active Directory 域扩展到云中。此外，可以使用 [Azure 中的 SQL Server 数据文件](http://msdn.microsoft.com/zh-cn/library/dn385720.aspx)，将本地数据文件存储在 Azure 存储空间中。有关详细信息，请参阅 [SQL Server 2014 混合云简介](http://msdn.microsoft.com/zh-cn/library/dn606154.aspx)。
      <li type=round>支持通过 [使用 Azure Blob 存储执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj919148.aspx) 或 [Azure VM 中的 AlwaysOn 副本](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/) 实现本地 SQL Server 应用程序的灾难恢复。
      </ul></td>

</tr>
</table>

## 选择 Azure SQL 数据库或 Azure VM 上的 SQL Server 时的业务动机

### 成本

无论你是现金不足的新公司，或是在预算有限的情况下运作的已成立公司的小组，有限资金经常是决定数据库托管方式的主要考虑因素。在本部分中，我们首先了解 Azure 中有关以下两个关系数据库选项的计费和许可基本概念：SQL 数据库和 Azure VM 中的 SQL Server。然后，我们将了解应该如何计算应用程序总成本。

#### 计费和许可基础概念

**SQL 数据库**以服务（不含许可证）的形式销售给客户，而 Azure VM 中的 SQL Server 则需要传统的 SQL Server 许可。

目前，我们在多个服务层中提供 **SQL 数据库**，并根据你选择的服务层和性能级别，以固定费率向你收取每小时费用。此外，将根据传出 Internet 流量计费。基本、标准和高级服务层旨在以多个性能级别提供可预测的性能，以满足应用程序的高峰要求。你可以在服务层和性能级别之间进行更改，以满足应用程序的不同吞吐量需求。如果你的数据库具有高事务量且必须支持许多并发用户，我们建议使用高级服务层。有关目前支持的服务层的最新信息，请参阅 [Azure SQL 数据库服务层](/documentation/articles/sql-database-service-tiers/)。

使用 **Azure SQL 数据库**，Microsoft 将自动配置、修补和升级数据库软件，从而可以降低你的管理成本。此外，它的[内置备份](/documentation/articles/sql-database-business-continuity/)功能可帮助你大幅降低成本，尤其是当你拥有大量的数据库时。

有了 **Azure VM 上的 SQL Server**，你可以利用传统的 SQL Server 许可。你可以使用平台提供的 SQL Server 映像（附带许可证），或使用你自己的 SQL Server 许可证。使用 Azure 提供的映像时，营运成本取决于所选的 VM 大小以及 SQL Server 版本。无论 VM 大小或 SQL Server 版本为何，你都需要支付 SQL Server 和 Windows Server 的每分钟许可成本，以及 VM 磁盘的 Azure 存储空间成本。每分钟计费选项可让你随时使用 SQL Server，而无需另外购买 SQL Server 许可证。如果在 Azure 中使用自己的 SQL Server 许可证，则只需支付 Windows Server 和存储成本。有关自带许可证的详细信息，请参阅 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility)。

#### 计算应用程序总成本

当你开始使用云平台时，运行应用程序的成本主要包括开发和管理成本，以及公有云平台所需的服务成本。

以下是针对在 SQL 数据库和 Azure VM 中的 SQL Server 上运行的应用程序的详细成本计算方法：

**使用 Azure SQL 数据库时：**

应用程序总成本 = 大幅降低的管理成本 + 软件开发成本 + SQL 数据库服务成本

**使用 Azure VM 上的 SQL Server 时：**

应用程序总成本 = 降到最低的软件开发/修改成本 + 管理成本 + SQL Server 与 Windows Server 许可成本 + Azure 存储空间成本

有关定价的详细信息，请参阅以下资源：

- [SQL 数据库定价](/home/features/sql-database/pricing/)
- [SQL](/home/features/virtual-machines/#SQL) 和 [Windows](/home/features/virtual-machines/#windows) 的[虚拟机定价](/home/features/virtual-machines/pricing/)
- [Azure 价格计算器](/pricing/calculator)

> [AZURE.NOTE] SQL Server 上有一小部分的功能不适用于或不可用于 SQL 数据库。有关详细信息，请参阅 [SQL 数据库一般性限制和指导原则](/documentation/articles/sql-database-general-limitations/)以及 [SQL 数据库 Transact-SQL 信息](/documentation/articles/sql-database-transact-sql-information/)。如果要将现有的 SQL Server 解决方案迁移到云中，请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。当将现有的本地 SQL Server 应用程序迁移到 SQL 数据库时，建议你更新应用程序以利用云服务提供的功能。例如，可以考虑使用 [Azure Web App Service](/home/features/web-site) 或 [Azure 云服务](/home/features/cloud-services)来托管你的应用层，以提高成本效益。

### 管理

对许多企业来说，决定过渡到到云服务的关键在于降低管理复杂度。使用 **SQL 数据库**，Microsoft 可以管理底层硬件，自动复制所有数据以提供高可用性，配置及升级数据库软件，管理负载平衡，并在发生服务器故障时执行透明的故障转移。你可以继续管理数据库，但不再需要管理数据库引擎、服务器操作系统或硬件。可以继续管理的项目示例包括数据库和登录、索引和查询优化，以及审核和安全性。

在另一方面，你可能有内部专业人员，并希望全面控制数据库位置甚至磁盘位置。使用 **Azure VM 上运行的 SQL Server**，你可以完全掌控操作系统和 SQL Server 实例配置。使用 VM，你可以决定何时更新/升级操作系统与数据库软件，以及何时安装任何其他软件（例如防病毒和备份工具）。此外，你还可以控制 VM 的大小、磁盘数目及其存储配置。例如，Azure 允许你视需要更改 VM 的大小。有关信息，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-linux-sizes/)。

### 服务级别协议 (SLA)

对于许多 IT 部门而言，达到服务级别协议 (SLA) 规定的正常运行时间义务是首要任务。在本部分中，我们将了解 SLA 对每个数据库托管选项代表的含义。

对于 **SQL 数据库**基本、标准和高级服务层，Microsoft 提供 99.99% 的可用性 SLA。有关最新信息，请参阅[服务级别协议](/support/legal/sla)。有关 SQL 数据库服务层和支持的业务连续性计划的最新信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/)。

对于 **Azure VM 上运行的 SQL Server**，Microsoft 提供 99.95% 的可用性 SLA（仅涵盖虚拟机）。此 SLA 不涵盖 VM 上运行的进程（例如 SQL Server），并且要求你在可用性集中托管至少两个 VM 实例。有关最新信息，请参阅 [VM SLA](/support/legal/sla)。为了在 VM 中实现数据库高可用性 (HA)，你应在 SQL Server 中配置一个受支持的高可用性选项，例如 [AlwaysOn 可用性组](http://blogs.technet.com/b/dataplatforminsider/archive/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery.aspx)。

### <a name="market"></a>面市时间

当开发人员生产力和快速面市为关键要素时，**SQL 数据库**是云式应用程序的理想解决方案。此选项提供类似于编程 DBA 的功能，非常适合云架构师和开发员，因为它能降低管理基础操作系统和数据库的需求。例如，可以使用 [REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx) 和 [PowerShell Cmdlet](http://msdn.microsoft.com/zh-cn/library/azure/dn546726.aspx) 来自动化和管理数千个数据库的管理操作。[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)等功能可让你将重点放在应用程序层上，更快地将解决方案推向市场。

如果现有或新的应用程序需要访问与控制 SQL Server 实例的所有功能，**Azure VM 上运行的 SQL Server** 是理想选择。此外，如果你想要依现状将现有的本地应用程序和数据库迁移到 Azure，则它也是一个不错的选择。由于你无需更改呈现、应用程序和数据层，你在重新架构现有解决方案时节省时间和预算。相反地，你可以将重点放在将所有解决方案迁移到 Azure，并执行 Azure 平台可能需要的某些性能优化。有关详细信息，请参阅 [Azure 虚拟机上 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)。

## 摘要

本文探讨了 SQL 数据库和 Azure 虚拟机 (VM) 上的 SQL Server，并讨论了可能会影响决策的常见业务动机。以下是供你考虑的建议摘要：

对于以下情况，请选择 **Azure SQL 数据库**：

- 你打算构建全新的基于云的应用程序，或者想要迁移现有 SQL Server 解决方案，以利用云服务提供的成本节省和性能优化。此方法提供全面管理云服务的优势，有助于加速产品面市，并提供长期的成本效益。

- 你想要让 Microsoft 在数据库上运行常见管理操作，因而数据库需要更高的可用性 SLA。



对于以下情况，请选择 **Azure VM 上的 SQL Server**：

- 你有现有的本地应用程序，并且想要停止维护自己的硬件或考虑混合解决方案。这种方法可让你更快地享用高数据库容量，并通过安全隧道连接本地应用程序。

- 你有现有的 IT 资源，需要 SQL Server 的完全管理权限，并需要与本地 SQL server 完全兼容。这种方法可让你灵活运行大多数应用程序，将开发或修改现有应用程序的成本降到最低。此外，它还提供对 VM、操作系统和数据库配置的完全控制。



> [AZURE.NOTE] 想要试用 SQL Server 2016 CTP2？ 请注册 Azure，然后转到[此处](http://aka.ms/sql2016vm)创建已经装有 SQL Server 2016 CTP2 的虚拟机。

## 后续步骤
- 若要开始使用 SQL 数据库，请参阅 [SQL Database tutorial: Create a SQL database in minutes using the Azure portal（SQL 数据库教程：使用 Azure 门户在几分钟内创建一个 SQL 数据库）](/documentation/articles/sql-database-get-started/)。
- 请参阅 [SQL Database pricing（SQL 数据库定价）](/home/features/sql-database/pricing/)
- 若要开始在 Azure VM 上使用 SQL Server，请参阅 [使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

<!---HONumber=Mooncake_0503_2016-->
