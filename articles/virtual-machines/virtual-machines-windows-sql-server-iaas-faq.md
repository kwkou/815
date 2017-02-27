<properties
    pageTitle="Azure 虚拟机中的 SQL Server 常见问题 | Azure"
    description="本文提供有关运行 Azure VM 中的 SQL Server 时遇到的常见问题的解答。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="v-shysun"
    manager="felixwu"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="2fa5ee6b-51a6-4237-805f-518e6c57d11b"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="iaas-sql-server"
    ms.date="11/30/2016"
    wacn.date="02/24/2017"
    ms.author="v-shysun" />

# Azure 虚拟机中的 SQL Server 常见问题
本主题提供有关运行 [Azure 虚拟机中的 SQL Server](/home/features/virtual-machines/#virtual-machine-SQLserver) 时出现的一些最常见问题的解答。

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 常见问题
1. **如何创建装有 SQL Server 的 Azure 虚拟机？**
   
    最简单的解决方法是创建包含 SQL Server 的虚拟机。有关注册 Azure 并从门户创建 SQL VM 的教程，请参阅[在 Azure 门户预览中预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)。可以选择按分钟支付 SQL Server 许可费的虚拟机映像，或者使用允许自带 SQL Server 许可证的映像。也可以选择手动在 VM 上安装 SQL Server，并重复使用本地许可证。如果自带许可证，必须提供 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)。
2. **SQL VM 与 SQL 数据库服务之间的差别是什么？**
   
    从概念上讲，在 Azure 虚拟机上运行 SQL Server 与在远程数据中心运行 SQL Server 并没什么不同。相比之下，[SQL 数据库](/documentation/articles/sql-database-technical-overview/)可提供数据库即服务。使用 SQL 数据库时，无法访问托管数据库的计算机。有关完整比较，请参阅[选择云 SQL Server 选项：Azure SQL \(PaaS\) 数据库或 Azure VM 上的 SQL Server \(IaaS\)](/documentation/articles/sql-database-paas-vs-sql-server-iaas/)。
3. **如何将本地 SQL Server 数据库迁转到云中？**
   
    首先，请创建装有 SQL Server 实例的 Azure 虚拟机。然后将本地数据库迁转到该实例。有关数据迁移策略，请参阅[将 SQL Server 数据库迁移到 Azure VM 中的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)。
4. **是否可以更改已安装的功能，或在相同的 VM 上安装另一个 SQL Server 实例？**
   
    可以。SQL Server 安装介质位于 **C** 驱动器上的某个文件夹中。可从该位置运行 **Setup.exe** 以添加新的 SQL Server 实例，或更改计算机上 SQL Server 的其他已安装功能。
5. **如何将 Azure VM 中的 SQL Server 升级到新版本？**
   
    目前，对于在 Azure VM 中运行的 SQL Server，不提供就地升级。因此，请使用所需的 SQL Server 版本创建新的 Azure 虚拟机，然后使用标准[数据迁移技术](/documentation/articles/virtual-machines-windows-migrate-sql/)将数据库迁移到新的服务器。
6. **如何在 Azure VM 上安装 SQL Server 的许可版本？**
   
    可将 SQL Server 安装媒体复制到 Windows Server VM，然后在该 VM 上安装 SQL Server。出于许可原因，必须提供 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)。
7. **如果 VM 是基于一个即用即付库映像创建的，是否可以将它更改为使用我自己的 SQL Server 许可证？**

    不可以。无法从按分钟付费许可证改为使用自己的许可证。请使用一个 [BYOL 映像](/documentation/articles/virtual-machines-windows-migrate-sql/)创建新的 Azure 虚拟机，然后使用标准数据迁移技术将数据库迁移到新服务器。

7. **如果 Azure VM 仅供备用/故障转移，是否必须支持该 VM 上的 SQL Server 许可费？**
   
    对于用作 HA 部署中的被动辅助副本的 SQL Server，如果客户购买了软件保障并使用许可移动性，则不需要支付许可费。
    
8. **如何将更新和服务包应用于 SQL Server VM？**
   
    虚拟机允许你控制主机，包括应用更新的时间与方法。对于操作系统，可以手动应用 Windows 更新，或者启用名为[自动修补](/documentation/articles/virtual-machines-windows-sql-automated-patching/)的计划服务。自动修补将安装任何标记为重要的更新，包括该类别中的 SQL Server 更新。必须手动安装其他可选的 SQL Server 更新。
9. **是否可以设置虚拟机库中未显示的配置（例如 Windows 2008 R2 + SQL Server 2012）？**
   
    不可以。对于包含 SQL Server 的虚拟机库映像，必须选择提供的映像之一。
10. **如何在 Azure VM 上安装 SQL Data Tools？**
    
     从 [Microsoft SQL Server Data Tools - Business Intelligence for Visual Studio 2013 ](https://www.microsoft.com/download/details.aspx?id=42313) 下载并安装 SQL Data Tools。

## 资源
有关 Azure 虚拟机上 SQL Server 的概述，请观看视频 [Azure VM 是 SQL Server 2016 的最佳平台](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/Azure-VM-is-the-best-platform-for-SQL-Server-2016)。也可以在 [Azure 虚拟机中的 SQL Server 概述](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)主题中获取详细介绍。

其他资源包括：

* [在 Azure 门户预览中预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)
* [将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)
* [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)
* [Azure 虚拟机中 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)
* [Azure 虚拟机中 SQL Server 的应用程序模式和开发策略](/documentation/articles/virtual-machines-windows-sql-server-app-patterns-dev-strategies/)

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->