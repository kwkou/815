<properties
	pageTitle="Azure 虚拟机中的 SQL Server 常见问题 | Azure"
	description="本文提供有关运行 Azure VM 中的 SQL Server 时遇到的常见问题的解答。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="v-shysun"
	manager="msmets"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/27/2016"
	wacn.date="06/13/2016"/>

# Azure 虚拟机中的 SQL Server 常见问题

本主题提供有关运行 [Azure 虚拟机中的 SQL Server](/home/features/virtual-machines#virtual-machine-SQLserver) 时出现的一些最常见问题的解答。

[AZURE.INCLUDE [support-disclaimer](../includes/support-disclaimer.md)]

## 常见问题

1. **如何创建装有 SQL Server 的 Azure 虚拟机？**

	可通过两种方式来执行此操作。最简单的解决方法是创建包含 SQL Server 的虚拟机。有关注册 Azure 并创建 SQL VM 的教程，请参阅 [使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。也可以选择手动在 VM 上安装 SQL Server，并重复使用 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)提供的本地许可证。

1. **SQL VM 与 SQL 数据库服务之间的差别是什么？**

	从概念上讲，在 Azure 虚拟机上运行 SQL Server 与在远程数据中心运行 SQL Server 并没什么不同。相比之下，[SQL 数据库](/documentation/articles/sql-database-technical-overview/)可提供数据库即服务。使用 SQL 数据库时，无法访问托管数据库的计算机。有关完整比较，请参阅 [Choose a cloud SQL Server option: Azure SQL (PaaS) Database or SQL Server on Azure VMs (IaaS)（选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)）](/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas/)。

1. **如何将本地 SQL Server 数据库迁转到云中？**

	首先，请创建装有 SQL Server 实例的 Azure 虚拟机。然后将本地数据库迁转到该实例。有关数据迁移策略，请参阅 [Migrate a SQL Server database to SQL Server in an Azure VM（将 SQL Server 数据库迁移到 Azure VM 中的 SQL Server）](/documentation/articles/virtual-machines-windows-migrate-sql/)。

2. **是否可以更改已安装的功能，或在相同的 VM 上安装另一个 SQL Server 实例？**

	是的。SQL Server 安装媒体位于 **C** 驱动器上的某个文件夹中。可从该位置运行 **Setup.exe** 以添加新的 SQL Server 实例，或更改计算机上 SQL Server 的其他已安装功能。

3. **如何将 Azure VM 中的 SQL Server 升级到新版本？**

	目前，在 Azure VM 中运行的 SQL Server 不提供任何就地升级。因此，请使用所需的 SQL Server 版本创建新的 Azure 虚拟机，然后使用标准[数据迁移技术](/documentation/articles/virtual-machines-windows-migrate-sql/)，将数据库迁移到新的服务器。

4. **如何在 Azure VM 上安装 SQL Server 的许可版本？**

	将 SQL Server 安装媒体复制到 Windows Server VM 上，然后在 VM 上安装 SQL Server。出于许可原因，必须提供 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)。

5. **如果 VM 只用于备用/故障转移，是否仍必须支付 VM 的 SQL 费用？**

	如果要通过库创建 SQL VM，则必须有独立的备用 SQL VM 许可证，这种许可证的价格是相同的。如果要使用[许可移动性](/pricing/license-mobility/)在虚拟机上手动安装 SQL Server，可以选择使用一个免费的被动 SQL 实例来进行故障转移。请查看相关限制和要求。

6. **如何将更新和服务包应用到 SQL Server VM？**

	虚拟机允许你控制主机，包括应用更新的时间与方法。对于操作系统，可以手动应用 Windows 更新，或者启用名为[自动修补](/documentation/articles/virtual-machines-windows-classic-sql-automated-patching/)的计划服务。自动修补将安装任何标记为重要的更新，包括该类别中的 SQL Server 更新。必须手动安装其他可选的 SQL Server 更新。

7. **是否可以设置虚拟机库中未显示的配置（例如 Windows 2008 R2 + SQL Server 2012）？**

	不可以。对于包含 SQL Server 的虚拟机库映像，必须选择提供的的映像之一。

9. **如何在 Azure VM 上安装 SQL Data Tools？**

	从 [Microsoft SQL Server Data Tools - Business Intelligence for Visual Studio 2013 ](https://www.microsoft.com/zh-cn/download/details.aspx?id=42313) 下载并安装 SQL Data Tools。

## 资源

有关 Azure 虚拟机上的 SQL Server 概述，请观看视频 [Azure VM is the best platform for SQL Server 2016（Azure VM 是 SQL Server 2016 的最佳平台）](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/Azure-VM-is-the-best-platform-for-SQL-Server-2016)。也可以在 [SQL Server on Azure Virtual Machines overview（Azure 虚拟机中的 SQL Server 概述）](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)主题中获取详细介绍。

其他资源包括：

- [使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)
- [将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)
- [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)
- [SQL Server 在 Azure 虚拟机中的性能最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)
- [Azure 虚拟机中的 SQL Server 的应用程序模式和开发策略](/documentation/articles/virtual-machines-windows-sql-server-app-patterns-dev-strategies/)

<!---HONumber=Mooncake_0606_2016-->