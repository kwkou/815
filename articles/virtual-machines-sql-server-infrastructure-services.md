<properties
	pageTitle="虚拟机上的 SQL Server 概述 | Azure"
	description="本文概述了 Azure 虚拟机上托管的 SQL Server。这包括深度内容的链接。"
	services="virtual-machines"
	documentationCenter=""
	authors="rothja"
	manager="jeffreyg"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="02/03/2016"
	wacn.date=""/>

# Azure 虚拟机中的 SQL Server 概述

## 入门
你可以使用各种配置（从单一数据库服务器到多计算机配置）通过 AlwaysOn 可用性组和 Azure 虚拟网络托管 [Azure 虚拟机中的 SQL Server](/home/features/virtual-machines/sql-server/)。

>[AZURE.NOTE] 在 Azure VM 中运行 SQL Server 是在 Azure 中存储关系数据的一个选项。此外，还可以使用 Azure SQL 数据库服务。有关详细信息，请参阅[了解 Azure SQL 数据库和 Azure VM 中的 SQL Server](/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas)。

若要在 Azure 中创建 SQL Server 虚拟机，你必须首先获取 Azure 平台订阅。你可以在[购买选项](/pricing/overview/)中购买 Azure 订阅。若要免费试用，请访问 [Azure 试用](/pricing/1rmb-trial/)。

### 在单个 VM 上部署 SQL Server 实例

注册订阅后，在 Azure 中部署 SQL Server 虚拟机的最简单方法是[在 Azure 中预配 SQL Server 计算机库映像](/documentation/articles/virtual-machines-provision-sql-server)。这些映像包括 VM 定价中的 SQL Server 许可。

下表提供了虚拟机库中提供的 SQL Server 映像的矩阵。

|SQL Server 版本|操作系统|SQL Server 版本|
|---|---|---|
|SQL Server 2008 R2 SP2|Windows Server 2008 R2|Enterprise、Standard、Web|
|SQL Server 2008 R2 SP3|Windows Server 2008 R2|Enterprise、Standard、Web|
|SQL Server 2012 SP2|Windows Server 2012|Enterprise、Standard、Web|
|SQL Server 2012 SP2|Windows Server 2012 R2|Enterprise、Standard、Web|
|SQL Server 2014|Windows Server 2012 R2|Enterprise、Standard、Web|
|SQL Server 2014 SP1|Windows Server 2012 R2|Enterprise、Standard、Web|
|SQL Server 2016 CTP|Windows Server 2012 R2|计算|

除了这些预配置的映像外，你还可以[创建不预装 SQL Server 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)。可以安装你有许可证的 SQL Server 的任何实例。可使用 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)将许可证迁移到 Azure，以便在 Azure 虚拟机中运行 SQL Server。在这种情况下，你只需为与虚拟机关联的 Azure 计算和存储[成本](/home/features/virtual-machines/#price)付费。

为了确定 SQL Server 映像的最佳虚拟机配置设置，请查看 [Azure 虚拟机中 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-sql-server-performance-best-practices)。对于生产工作负荷，**DS3** 是 SQL Server Enterprise 版建议的最小虚拟机大小，**DS2** 是标准版建议的最小虚拟机大小。

除了查看性能最佳实践外，其他初始任务包括以下项：

- [查看 Azure VM 中 SQL Server 的安全最佳实践](/documentation/articles/virtual-machines-sql-server-security-considerations)
- [设置连接](/documentation/articles/virtual-machines-sql-server-connectivity)

### 迁移数据

在启动并运行 SQL Server 虚拟机后，你可能想要将现有数据库迁移到虚拟机。你可以采用多种方法，但 SQL Server Management Studio 中的部署向导适合大多数方案。有关这些方案的讨论以及向导教程，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-migrate-onpremises-database)。

## 高可用性

如果你需要高可用性，请考虑配置 SQL Server AlwaysOn 可用性组。这涉及虚拟网络中的多个 Azure VM。Azure 管理门户有一个模板为你设置了此配置。有关详细信息，请参阅 [Azure 库中的 SQL Server AlwaysOn 产品](http://blogs.technet.com/b/dataplatforminsider/archive/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery.aspx)。

如果你想要手动配置可用性组和关联的侦听器，请参阅以下文章：

- [在 Azure 中配置 AlwaysOn 可用性组 (GUI)](/documentation/articles/virtual-machines-sql-server-alwayson-availability-groups-gui)
- [在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener)
- [将本地 AlwaysOn 可用性组扩展到 Azure](/documentation/articles/virtual-machines-sql-server-extend-on-premises-alwayson-availability-groups)

有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions)。

## 备份和还原
对于本地数据库，Azure 可以充当用于存储 SQL Server 备份文件的辅助数据中心。有关备份与还原选项的概述，请参阅 [Azure 虚拟机中 SQL Server 的备份和还原](/documentation/articles/virtual-machines-sql-server-backup-and-restore)。

[SQL Server 备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx) 将 Azure 备份文件存储在 Azure Blob 存储中。[SQL Server 托管备份](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)允许你在 Azure 中计划备份和保留期。这些服务可用于本地 SQL Server 实例或 Azure VM 上运行的 SQL Server。Azure VM 还可以利用 SQL Server 的[自动备份](/documentation/articles/virtual-machines-sql-server-automated-backup)和[自动修补](/documentation/articles/virtual-machines-sql-server-automated-patching)。

## SQL Server VM 映像配置详细信息

下表描述了平台提供的 SQL Server 虚拟机映像的配置。

### Windows Server

平台映像中的 Windows Server 安装包含以下配置设置和组件：

|功能|配置
|---|---|
|远程桌面|已为管理员帐户启用|
|Windows 更新|已启用|
|用户帐户|默认情况下，在预配期间指定的用户帐户是本地 Administrators 组的成员。此管理员帐户也是 SQL Server sysadmin 服务器角色的成员|
|工作组|该虚拟机是名为 WORKGROUP 的工作组的成员|
|来宾帐户|已禁用|
|具有高级安全性的 Windows 防火墙|启用|
|.NET framework|版本 4|
|磁盘|所选定的大小会限制你能够配置的数据磁盘个数。请参阅 [Azure 的虚拟机大小](/documentation/articles/virtual-machines-size-specs)|

### SQL Server

平台映像中的 SQL Server 安装包含以下配置设置和组件：

|功能|配置|
|---|---|
|数据库引擎|已安装|
|Analysis Services|已安装|
|Integration Services|已安装|
|Reporting Services|已在本机模式下配置|
|AlwaysOn 可用性组|在 SQL Server 2012 或更高版本中提供。需要[其他配置](/documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions)
|复制|已安装|
|全文和语义提取搜索|已安装（仅限 SQL Server 2012 或更高版本中的语义提取）|
|Data Quality Services|已安装（仅限 SQL Server 2012 或更高版本）|
|Master Data Services|已安装（仅限 SQL Server 2012 或更高版本）。需要[其他配置和组件](https://msdn.microsoft.com/zh-cn/library/ee633752.aspx)
|PowerPivot for SharePoint|可用（仅限 SQL Server 2012 或更高版本）。需要其他配置和组件（包括 SharePoint）|
|分布式重播客户端|可用（仅限 SQL Server 2012 或更高版本），但未安装。请参阅[从平台提供的 SQL Server 映像运行 SQL Server 安装程序](#run-sql-server-setup-from-the-platform-provided-sql-server-image)|
|工具|所有工具，其中包括 SQL Server Management Studio、SQL Server 配置管理器、Business Intelligence Development Studio、SQL Server 安装程序、客户端工具连接、客户端工具 SDK 和 SQL 客户端连接 SDK，以及升级和迁移工具，如数据层应用程序 (DAC)、备份、还原、附加和分离|
|SQL Server 联机丛书|已安装，但需要使用帮助查看器配置|

### 数据库引擎配置

将配置以下数据库引擎设置。有关更多设置，请检查 SQL Server 的实例。

|功能|配置|
|---|---|
|实例|包含 SQL Server 数据库引擎的默认（未命名）实例，该实例仅侦听共享内存协议|
|身份验证|默认情况下，Azure 在 SQL Server 虚拟机安装期间选择 Windows 身份验证。如果你要使用 sa 登录名或要创建新的 SQL Server 帐户，则需要更改身份验证模式。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的安全注意事项](/documentation/articles/virtual-machines-sql-server-security-considerations)。|
|sysadmin|安装了虚拟机的 Azure 用户最初是 SQL Server sysadmin 固定服务器角色的唯一成员|
|内存|数据库引擎内存将设置为动态内存配置|
|已包含数据库身份验证|关闭|
|默认语言|英语|
|跨数据库所有权链接|关闭|

### 客户体验改善计划 (CEIP)

[客户体验改善计划 (CEIP)](https://technet.microsoft.com/zh-cn/library/cc730757.aspx) 已启用。可以通过使用“SQL Server 错误和使用情况报告”实用工具来禁用 CEIP。若要启动“SQL Server 错误和使用情况报告”实用工具，请在“开始”菜单上依次单击“所有程序”、Microsoft SQL Server 版本、“配置工具”和“SQL Server 错误和使用情况报告”。如果你不想使用已启用 CEIP 的 SQL Server 实例，则还可以考虑将你自己的虚拟机映像部署到 Azure。有关详细信息，请参阅[创建和上载包含 Windows Server 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-create-upload-vhd-windows-server)。

## 从平台提供的 SQL Server 映像运行 SQL Server 安装程序

如果你通过使用平台提供的 SQL Server 映像创建虚拟机，可以在虚拟机上的 **C:\\SqlServer\_SQLMajorVersion.SQLMinorVersion\_Full** 目录中找到保存的 SQL Server 安装介质。你可以从该目录运行安装程序以执行任何安装操作，包括添加或删除功能、添加新实例或修复实例（如果磁盘空间允许）。

>[AZURE.NOTE] Azure 提供了多个版本的 SQL Server 映像。如果平台提供的 SQL Server 映像的版本发布日期是 2014 年 5 月 15 日或以后的日期，则它在默认情况下包含产品密钥。如果你使用在此日期之前发布的平台提供的 SQL Server 映像预配虚拟机，则该 VM 不包含产品密钥。最佳做法是，建议你在预配新 VM 时始终选择最新的映像版本。

## 资源

- [在 Azure 上设置 SQL Server 虚拟机](/documentation/articles/virtual-machines-provision-sql-server)
- [将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-migrate-onpremises-database)
- [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions)
- [Azure 虚拟机中的 SQL Server 的应用程序模式和开发策略](/documentation/articles/virtual-machines-sql-server-application-patterns-and-development-strategies)
- [Azure 虚拟机](/documentation/articles/virtual-machines-about)

<!---HONumber=Mooncake_0321_2016-->