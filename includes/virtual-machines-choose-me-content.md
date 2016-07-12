
<a name="tellmevm"></a>
## 告诉我有关虚拟机的信息

你可以使用 Azure 虚拟机在云中创建和使用虚拟机。虚拟机提供所谓的*基础结构即服务 (IaaS)*，是一项可广泛使用的技术。下面是一些示例：

- **用于开发和测试的虚拟机 (VM)。** 开发组经常会使用 VM，因为这样可以快速轻松地创建具有特定配置的计算机来满足编程和应用程序测试的需要。Azure 虚拟机提供一种简单且经济的方法来创建、使用 VM，以及在不再需要时删除它们。
- **在云中运行应用程序。** 在公有云中运行某些应用程序可以带来经济效益。需求会急剧上升的应用程序就是一个例子。尽管你可以在自己的数据中心装配足够的硬件来处理高峰需求，但这些硬件在大部分时间可能利用不足。如果在 Azure 上运行此应用程序，只有在需要额外 VM 时才支付额外 VM 的费用，在不需要的时候则可以将它们关闭。或者，假设你是一家初创公司，需要在不做出任何承诺的情况下快速地按需计算资源。同样，Azure 也是合适的选择。
- **将你自己的数据中心扩展到公有云。** 利用 Azure 虚拟网络，你的组织可以创建一个虚拟网络 (VNET) 作为自有本地网络的扩展，然后将 VM 添加到该 VNET。这样，便可以在 Azure VM 上运行 [SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/) 和其他应用程序。与在自有数据中心的 VM 上运行这些应用程序相比，这种做法可能更易于部署，或者可以降低成本。   
- **灾难恢复。** 基于 IaaS 灾难恢复使您可以只在真正需要计算资源时才为所需的计算资源付费，而不用不停地为很少使用的备份数据中心付费。例如，如果您的主数据中心出现故障，您可以创建在 Azure 上运行的 VM 来运行至关重要的应用程序，然后在不再需要时关闭它们。

与其他虚拟机一样，Azure 中的 VM 具有操作系统、存储和网络功能，并可以运行各种应用程序。你可以使用 Azure 或其合作伙伴之一提供的映像，或使用自己的映像。示例包含以下产品的各个版本和配置：
 
-	Windows Server 
-	Linux 服务器，例如 Suse、Ubuntu 和 CentOS
-	SQL Server
-	SharePoint Server

虚拟机使用虚拟硬盘 (VHD) 来存储其操作系统 (OS) 和数据。VHD 还可用于存储映像，你可以选择某个映像来安装 OS。下图显示了这项特性，以及用于创建和管理 VM 的两个工具。

<a name="fig_createvms"></a> ![vm\_diagram](./media/virtual-machines-choose-me-content/diagram.png)

**图：Azure 虚拟机提供“基础结构即服务”。**

可以使用基于浏览器的经典管理门户、支持脚本的命令行工具或直接通过 REST API 管理 VM。Microsoft 合作伙伴（如 RightScale 和 ScaleXtreme）也提供依赖 REST API 的管理服务。

除了操作系统以外，可对 VM 使用的其他配置选项包括：

- 大小，决定了可附加的磁盘数和处理能力等因素。Azure 提供各种大小来支持多种类型的用途。有关详细信息，请参阅 [Windows](/documentation/articles/virtual-machines-windows-sizes/) 或者 [Linux](/documentation/articles/virtual-machines-Linux-sizes/) 虚拟机大小。  
- 托管新 VM 的 Azure 区域，例如中国东部或中国北部。 
- VM 扩展，为虚拟机提供更多的功能，例如，运行防病毒软件，或使用 Windows PowerShell 的所需状态配置功能。

VM 带来的其他好处包括：

**即付即用** - Azure 根据 VM 的大小和操作系统每小时计费。对于不足一小时的部分，Azure 将仅根据使用的分钟数计费。存储将另行定价和收费。有关详细信息，请参阅[虚拟机定价](/home/features/virtual-machines/pricing/)。

**复原** - Azure 将会监视每个正在运行的 VM 所在的物理硬件。如果运行 VM 的物理服务器出现故障，Azure 会及时发现，并将该 VM 转移到新硬件，然后重新启动该 VM。此过程有时称为服务修复。Azure 还会通过在 Blob 存储中保留 VHD 的冗余副本，来保护虚拟机的数据。

<!---HONumber=70-->