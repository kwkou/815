<properties 
	pageTitle="了解 Azure SQL Database 和 Azure VM 中的 SQL Server" 
	description="了解 Azure SQL Database 和 Azure 虚拟机中的 SQL Server。探讨常见的业务动机，以判断哪种 SQL 技术最适合你的应用程序。" 
	services="sql-database, virtual-machines" 
	documentationCenter="" 
	authors="Selcin" 
	manager="jeffreyg" 
	editor="tysonn"/>

<tags 
	ms.service="sql-database" ms.date="07/15/2015" wacn.date=""/>

# 了解 Azure SQL Database 和 Azure VM 中的 SQL Server

Microsoft Azure 提供了用于托管 SQL Server 的两个选项：**Azure SQL Database** 和 **Azure 虚拟机中的 SQL Server**。本文首先介绍每个选项如何适应 Microsoft 数据平台的大体形势，然后根据业务要求进行更深入的探讨，以帮助你选择正确的选项。无论你是以节省成本为优先考虑，还是将精简管理视为第一要素，本文都会根据每种方法所提供的与你最重视的业务要求进行比较，以帮助你确定最适合的方法。

- [Microsoft 的数据平台](#platform)
- [Azure SQL Database 和 Azure VM 中的 SQL Server 详述](#close)	
- [选择 Azure SQL Database 或 Azure VM 中的 SQL Server 时的业务动机](#business)	
	- [成本](#cost)
		- [计费和许可基础概念](#billing)	
		- [计算应用程序总成本](#appcost)	
	- [管理](#admin)	
	- [服务级别协议 (SLA)](#sla)	
	- [面市时间](#market)	
- [摘要](#summary)	
- [致谢](#ack)	
- [其他资源](#resources)	


##<a name="platform"></a>Microsoft 的数据平台

在 Azure 与本地 SQL Server 数据库的任何介绍中，要了解的要点之一是你可以同时使用两者。Microsoft 数据平台利用 SQL Server 技术，使其可在跨本地物理机、私有云环境、第三方托管的私有云环境和公有云中使用。这样，你就可以通过本地和云托管部署的组合来满足独特的多样化业务需求，并同时在这些环境中使用相同的服务器产品、开发工具和专业知识组合。

   ![][1]

如图所示，每个产品可根据你在基础结构（X 轴）中所拥有的管理级别，以及数据库级别合并与自动化（Y 轴）所达到的成本效益程度等特征进行分类。

设计应用程序时，可以使用四个基本选项来托管属于应用程序一部分的 SQL Server：

- 非虚拟化物理机上的 SQL Server 
- 本地虚拟机中的 SQL Server（私有云）
- Azure 虚拟机中的 SQL Server（公有云）
- Azure SQL Database（公有云）

在以下部分中，我们将了解最后两个基本选项：Azure SQL Database 和 Azure VM 中的 SQL Server。此外，我们将探讨常见的业务动机，以判断哪一个选项最适合你的应用程序。

##<a name="close"></a>Azure SQL Database 和 Azure VM 中的 SQL Server 详述

**Azure SQL Database (Azure SQL Database)** 是关系数据库即服务，属于*平台即服务 (PaaS)* 行业类别。Azure SQL Database 构建在 Microsoft 所拥有、托管及维护的标准化硬件和软件基础之上。使用 SQL Database，你可以使用内置的特性和功能在服务上直接进行开发。使用 SQL Database 时，你可以即用即付，并使用向上或向外缩放选项获得更强大的功能。

**Azure 虚拟机 (VM) 中的 SQL Server** 属于*基础结构即服务 (IaaS)* 行业类别，可让你在云中的虚拟机上运行 SQL Server。与 Azure SQL Database 一样，它构建在 Microsoft 所拥有、托管及维护的标准化硬件基础之上。使用 VM 中的 SQL Server 时，你可以在 Azure 中使用自己的 SQL Server 许可证，或使用 Azure 门户中某个预先配置的 SQL Server 映像。

通常，这两个 SQL 选项已针对不同的用途进行优化：

- **Azure SQL Database** 经过优化，可将设置和管理许多数据库的整体成本降到最低。由于你无需管理任何虚拟机、操作系统或数据库软件（包括升级、高可用性和备份），因此可将进行中的管理成本降到最低。一般而言，SQL Database 可以大幅增加由单个 IT 或开发资源管理的数据库数目。
- **Azure VM 中运行的 SQL Server** 经过优化，可在混合方案中将现有本地 SQL Server 应用程序扩展到 Azure，或者在迁移方案或开发/测试方案中将现有应用程序部署到 Azure。混合方案的一个示例是通过 [Azure 虚拟网络](http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)在 Azure 中保留辅助数据库副本。有了 Azure VM 中的 SQL server，即拥有了专用 SQL Server 实例和基于云的 VM 的完全管理权限。当组织拥有可用来维护虚拟机的 IT 资源时，此选项是最佳选择。使用 VM 中的 SQL Server，你可以构建高度定制的系统，以解决应用程序的特定性能和可用性要求。

下表汇总了 Azure SQL Database 和 Azure VM 中 SQL Server 的主要特征：

<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle"></th>
   <th align="left" valign="middle">Azure SQL Database</th>
   <th align="left" valign="middle">Azure VM 中的 SQL Server</th>
   
</tr>
<tr>
   <td valign="middle"><p><b>最适合</b></p></td>
   <td valign="middle">
          <ul>
          <li type=round>开发与营销阶段有时间限制的新云式设计应用程序。
          <li type=round>需要内置自动高可用性、灾难恢复解决方案和升级机制的应用程序。
          <li type=round>你有数百甚至数千个数据库，但不想要管理基础操作系统、硬件和配置设置。
         <li type=round>使用向外缩放模式的应用程序。
         <li type=round>大小高达 500 GB 的数据库。
         <li type=round>构建软件即服务应用程序。
         
  </ul>
</td>
   <td valign="middle">
      <ul>
      <li type=round>需要几乎无需进行任何更改即可快速迁移到云的现有应用程序。
      <li type=round>需要通过安全隧道从 Azure 访问本地资源的 SQL Server 应用程序（例如 Active Directory）。
      <li type=round>你需要一个具有完全管理权限的定制 IT 环境。
      <li type=round>你想要快速完成开发和测试方案，但又不想购买本地 SQL Server 非生产硬件。
      <li type=round>使用 <a href="http://msdn.microsoft.com/zh-cn/library/jj919148.aspx">Azure 存储空间上的备份</a>或 <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx">Azure VM 中的 AlwaysOn 副本</a>的本地 SQL Server 应用程序实现灾难恢复。
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
   <td valign="middle"><ul><li type=round>除了内置的容错基础结构功能以外，Azure SQL Database 还提供可提高业务连续性的功能，例如时间点还原、地域还原和异地复制。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx">Azure SQL Database 业务连续性</a>。</ul></td>
   <td valign="middle"><ul><li type=round>Azure VM 中的 SQL Server 可让你设置高可用性和灾难恢复解决方案，以满足数据库的具体需求。因此，可以构建针对应用程序高度优化的系统。你可以视需要自我测试并运行故障转移。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx">Azure 虚拟机中 SQL Server 的高可用性和灾难恢复</a>。</ul></td>
   
</tr>
<tr>
   <td valign="middle"><p><b>混合云</b></p></td>
   <td valign="middle"><ul><li type=round>本地应用程序可以访问 Azure SQL Database 中的数据。</ul></td>
   <td valign="middle"><ul>
      <li type=round>使用 Azure VN 中的 SQL Server，应用程序可以一部分在云中运行，一部分在本地运行。例如，可以通过 <a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433091.aspx">Azure 网络服务</a>，将本地网络和 Active Directory 网络扩展到云中。此外，可以使用 <a href="http://msdn.microsoft.com/zh-cn/library/dn385720.aspx">Azure 中的 SQL Server 数据文件功能</a>，将本地数据文件存储在 Azure 存储空间中。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/dn606154.aspx">SQL Server 2014 混合云简介</a>。
      <li type=round>使用 <a href="http://msdn.microsoft.com/zh-cn/library/jj919148.aspx">Azure 存储空间上的备份</a>或 <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx">Azure VM 中的 AlwaysOn 副本</a>的本地 SQL Server 应用程序支持灾难恢复。
      </ul></td>
   
</tr>
</table>

##<a name="business"></a>选择 Azure SQL Database 或 Azure VM 中的 SQL Server 时的业务动机

###<a name="cost"></a>成本

无论你是现金不足的新公司，或是在预算有限的情况下运作的已成立公司的小组，有限资金经常是决定数据库托管方式的主要考虑因素。在本部分中，我们首先了解 Azure 中有关以下两个关系数据库选项的计费和许可基本概念：Azure SQL Database 和 Azure VM 中的 SQL Server。然后，我们将了解应该如何计算应用程序总成本。

####<a name="billing"></a>计费和许可基础概念

**Azure SQL Database** 以服务（不含许可证）的形式销售给客户，而 Azure VM 中的 SQL Server 则需要传统的 SQL Server 许可。

目前，你可以在多个服务层中使用 **Azure SQL Database**。对于基本、标准和高级服务层，我们将根据你选择的服务层和性能级别，以固定费率向你收取每小时费用。基本、标准和高级服务层旨在以多个性能级别提供可预测的性能，以满足应用程序的高峰要求。你可以在服务层和性能级别之间进行更改，以满足应用程序的不同吞吐量需求。有关目前支持的服务层的最新信息，请参阅 [Azure SQL Database 服务层（版本）](http://msdn.microsoft.com/zh-cn/library/azure/dn741340.aspx)。

使用 **Azure SQL Database**，世界各地数据中心内的 Microsoft Azure 将自动配置、修补和升级数据库软件。因此，你就能降低管理成本。此外，它的[内置备份](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)功能可帮助你大幅降低成本，尤其是当你拥有大量的数据库时。使用 Azure SQL Database 时，你不必支付针对 Azure SQL Database 运行的单个查询或传入/传出 Internet 流量的费用。如果你的数据库具有高事务量且必须支持许多并发用户，我们建议你使用高级（而不是基本或标准）服务层。

有了 **Azure VM 中的 SQL Server**，你可以利用传统的 SQL Server 许可。你可以使用平台提供的 SQL Server 映像，或在 Azure 中使用你的 SQL Server 许可证。使用 SQL Server 平台提供的映像时，成本取决于选择的 VM 大小以及 SQL Server 版本。简单而言，你需要支付 SQL Server 的每分钟许可成本、Windows Server 每分钟许可和 Azure 存储空间成本。每分钟计费选项可让你随时使用 SQL Server，而无需购买完整的 SQL Server 许可证。如果在 Azure 中使用自己的 SQL Server 许可证，则只需支付 Azure 计算和存储成本。

####<a name="appcost"></a>计算应用程序总成本

当你开始使用云平台时，运行应用程序的成本主要包括开发和管理成本，以及公有云平台所需的服务成本。

以下是针对在 Azure SQL Database 和 Azure VM 中的 SQL Server 上运行的应用程序的详细成本计算方法：

**使用 Azure SQL Database 时：**

*应用程序总成本 = 大幅降低的管理成本 + 软件开发成本 + Azure SQL Database 服务成本*

**使用 Azure VM 中的 SQL Server 时：**

*应用程序总成本 = 降到最低的软件开发/修改成本 + 管理成本 + SQL Server 与 Windows Server 许可成本 + Azure 存储空间成本*

**重要说明：**目前，Azure SQL Database 并非支持 SQL Server 的所有功能。有关详细的比较信息，请参阅 [Azure SQL Database 指导原则和限制](http://msdn.microsoft.com/zh-cn/library/azure/ff394102.aspx)。当你需要将现有数据库移到 Azure SQL Database 时请留意这条说明，因为在数据库重新设计上你可能需要一些额外的预算。Azure SQL Database 是 Microsoft 平台即服务产品。当你将现有的本地 SQL Server 应用程序迁移到 Azure SQL Database 时，建议你更新应用程序以充分利用平台即服务产品的所有好处。例如，开始在应用程序层上使用 [Azure 网站](http://www.windowsazure.cn/documentation/services/web-sites/)或 [Azure 云服务](http://www.windowsazure.cn/home/features/cloud-services/)，以提高成本效益。此外，请针对不同的 Azure SQL Database 服务层验证你的应用程序，并检查哪个层最符合应用程序的需求。此过程可帮助你达到更好的性能效果并将成本降到最低。有关详细信息，请参阅 [Azure SQL Database 服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)。

要进行详细的成本估算，请使用 [Azure 价格计算器](http://www.windowsazure.cn/pricing/calculator)。

有关定价的详细信息，请参阅以下资源：

- [Azure SQL Database 定价详细信息](http://www.windowsazure.cn/home/features/sql-database/#price) 
- [虚拟机定价详细信息](http://www.windowsazure.cn/home/features/virtual-machines/)
- [Azure VM 中的 SQL Server - 定价详细信息](http://www.windowsazure.cn/home/features/virtual-machines/#SQL)
- [Azure VM 中的 Windows Server - 定价详细信息](http://www.windowsazure.cn/home/features/virtual-machines/#windows) 

###<a name="admin"></a>管理

如果你手头已有许多任务，或许你并不期望采用服务器和数据库管理。对许多企业来说，决定使用云服务的关键在于降低管理复杂性的能力。使用 **Azure SQL Database**，Microsoft 可以管理物理硬件（例如硬盘、服务器和存储）；自动复制所有数据以提供高可用性；配置及升级数据库软件；管理负载平衡；在发生服务器故障时执行透明的故障转移。你可以继续管理 Azure SQL Database 实例，但无需控制基础 SQL Server 实例和 Azure 平台的物理资源。例如，你可以管理数据库和登录、执行索引调整以及优化查询，但无法管理系统表和文件组管理。有关详细信息，请参阅 [Azure SQL Database 指导原则和限制](http://msdn.microsoft.com/zh-cn/library/ff394102.aspx)。

在另一方面，你可能有内部专业人员，并想要保持只由计算机本身控制数据库位置。使用 **Azure VM 中运行的 SQL Server**，你可以完全掌控操作系统和 SQL Server 实例配置。使用 VM，你可以决定何时更新/升级操作系统与数据库软件，以及何时安装任何其他软件（例如防病毒和备份工具）。此外，你还可以控制 VM 的大小、磁盘数目及其存储配置。例如，Azure 允许你视需要更改正在运行的 VM 的大小。有关信息，请参阅 [Azure 的虚拟机和云服务大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)。

###<a name="sla"></a>服务级别协议 (SLA)

对于某些人而言，达到服务级别协议 (SLA) 规定的正常运行时间义务是首要任务。在本部分中，我们将了解 SLA 对每个数据库托管选项代表的含义。

对于 **Azure SQL Database**，假设使用基本、标准和高级服务层，则 Microsoft 将提供 99.99% 的可用性 SLA。请注意，可用性 SLA 提供连接到数据库的能力。换句话说，它是个数据库级别 SLA。有关 SLA 的最新信息，请参阅[服务级别协议](http://www.windowsazure.cn/support/legal/sla)。有关 Azure SQL Database 服务层（版本）和支持的业务连续性计划的最新信息，请参阅 [Azure SQL Database 服务层](http://msdn.microsoft.com/zh-cnlibrary/dn741340.aspx)。

对于 **Azure 中托管的虚拟机**，Microsoft 提供 99.95% 的可用性 SLA，这个可用性适用于 VM，而不适用于 VM 内部运行的进程（例如 SQL Server）。VM SLA 要求你在可用性集中至少托管两个 VM。使用此类配置，Azure 保证至少一个 VM 将有 99.95% 的时间可用。为了在 VM 中实现数据库高可用性 (HA)，你应在 SQL Server 中配置一个受支持的高可用性选项，例如 AlwaysOn 可用性组。请注意，在 Azure 中设置 AlwaysOn 需要一些手动配置和管理，并且你需要额外支付所操作的每个辅助数据库。


###<a name="market"></a>面市时间

当开发人员生产力和快速面市为关键要素时，**Azure SQL Database** 是云式应用程序的理想解决方案。此选项提供类似于编程 DBA 的功能，非常适合云架构师和开发员，因为它能降低管理基础操作系统和数据库的需求。它可以帮助开发人员了解与配置数据库相关任务。例如，可以使用 [REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx) 和 [PowerShell cmdlet](http://msdn.microsoft.com/zh-cn/library/azure/dn546726.aspx) 来自动化和管理数千个数据库的管理操作。通过云中的[弹性缩放](/documentation/articles/sql-database-elastic-pool)，你可以轻松地将重点放在应用程序层，更快地将应用程序推向市场。

如果现有和新的应用程序需要访问与控制 SQL Server 实例的所有功能，并想要依现状将现有的本地应用程序和数据库迁移到云，**Azure VM 中运行的 SQL Server** 是理想选择。由于你无需更改呈现、应用程序和数据层，你在重新架构现有解决方案时节省时间和预算。相反地，你可以将重点放在将所有解决方案包迁移到 VM，并执行 Azure 平台所需的某些性能优化。有关信息，请参阅 [Azure 虚拟机中 SQL Server 的性能最佳实践](http://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)。

##<a name="summary"></a>摘要

在本文中，我们探讨了：Azure SQL Database 和 Azure VM 中的 SQL Server。此外，我们还讨论了哪些常见业务动机会影响到如何做出取舍。

以下是一些建议摘要，供你在选择适当的选项时考虑：

对于以下情况，请选择 **Azure SQL Database**：

- 你打算构建全新的基于云的应用程序，或者想要将现有 SQL Server 数据库迁移到 Azure，而且你的数据库未使用 Azure SQL Database 中不支持的功能。有关详细信息，请参阅 [Azure SQL Database Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。此方法提供全面管理云服务的优点，并可确保产品快速面市。

- 你想要让 Microsoft 在数据库上运行常见管理操作，因而数据库需要更高的可用性 SLA。这种方法可以减少管理成本，同时为数据库提供有保证的可用性。

对于以下情况，请选择 **Azure VM 中的 SQL Server**：

- 你有现有的本地应用程序，并且想要停止维护自己的硬件或考虑混合解决方案。这种方法可让你更快地享用高数据库容量，并通过安全隧道将本地应用程序连接到云。

- 你有现有的 IT 资源，需要 SQL Server 的完全管理权限，并需要与本地 SQL server 完全兼容（例如，Azure SQL Database 不提供某些功能）。这种方法可让你灵活运行大多数应用程序，将开发或修改现有应用程序的成本降到最低。此外，它还提供对 VM、操作系统和数据库配置的完全控制。




##<a name="ack"></a>致谢

本文是在 Microsoft 社区中众多人士的大力帮助下，由 Microsoft 云与企业内容服务组编写的。

**作者：**Selcin Turkarslan

**技术供稿人：**Conor Cunningham

**技术审校：**Joanne Marone（Hodgins）、Karthika Raman、Lindsey Allen、Lori Clark、Luis Carlos Vargas Herring、Nosheen Syed Wajahatulla Hussain、Pravin Mittal、Shawn Bice、Silvano Coriani、Tony Petrossian、Tracy Daugherty。

**编辑审校：**Heidi Steen、Maggie Sparkman。

感谢各位让这篇文章得已出刊！

##<a name="resources"></a>其他资源 

<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle">资源</th>
   <th align="left" valign="middle">说明</th>
</tr>
<tr>
   <td valign="middle"><p><a href="http://msdn.microsoft.com/zh-cn/library/azure/ee336279.aspx">MSDN：Azure SQL Database</a></p>
<p><a href="http://msdn.microsoft.com/library/azure/jj823132.aspx">MSDN：Azure 虚拟机中的 SQL Server</a></p>

<p><a href="http://www.windowsazure.cn/home/features/sql-database/">Azure.com：Azure SQL Database</a></p></td>
   <td valign="middle">库文档的链接。</td>   
</tr>
<tr>
   <td valign="middle"><p><a href="http://msdn.microsoft.com/zh-cn/library/dn574746.aspx">Azure 虚拟机中的 SQL Server 的应用程序模式和开发策略</p></td>
   <td valign="middle">此文章介绍了适用于 Azure VM 中 SQL Server 的最常见应用程序模式，以及包括 Azure SQL Database 的混合方案。</td>   
</tr>
<tr>
   <td valign="middle"><p><a href="http://msdn.microsoft.com/zh-cn/library/hh680934(v=PandP.50).aspx">Microsoft Enterprise Library 暂时性故障处理应用程序块</p></td>
   <td valign="middle">此库可让开发人员通过添加稳健的暂时性故障处理逻辑，以更弹性的方式在 Azure SQL Database 上运行其应用程序。暂时性故障是指因某些临时情况（如网络连接问题或服务不可用）而引发的错误。由于 Azure SQL Database 是多租户服务，因此必须处理此类错误，以将任何应用程序停机时间降到最低。</td>   
</tr>
</table>

<!--Image references-->
[1]: ./media/data-management-azure-sql-database-and-sql-server-iaas/SQLIAAS_SQL_Server_Cloud_Continuum.png
 

<!---HONumber=66-->