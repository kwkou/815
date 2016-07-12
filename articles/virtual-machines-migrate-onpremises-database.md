<properties
	pageTitle="将 SQL Server 数据库迁移到 VM 上的 SQL Server | Azure"
	description="了解如何将本地用户数据库迁移到 Azure 虚拟机中的 SQL Server。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="sabotta"
	manager="jhubbard"
	editor=""
	tags="azure-service-management" />
<tags
	ms.service="virtual-machines-windows"
	ms.date="06/06/2016"
	wacn.date="07/11/2016"/>


# 将 SQL Server 数据库迁移到 Azure VM 中的 SQL Server

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。


将本地 SQL Server 用户数据库迁移到 Azure VM 中的 SQL Server 的方法有很多。本文将简要讨论各种方法，针对各种情况推荐最佳方法，并提供一个[教程](#azure-vm-deployment-wizard-tutorial)，指导你使用**将 SQL Server 数据库部署到 Azure VM** 向导。

[教程](#azure-vm-deployment-wizard-tutorial)中介绍的使用**将 SQL Server 数据库部署到 Azure VM** 向导的方法仅适用于经典部署模型。

## 主要迁移方法有哪些？

主要迁移方法包括：

- 使用“将 SQL Server 数据库部署到 Azure VM”向导
- 使用压缩功能执行本地备份并将备份文件手动复制到 Azure 虚拟机中
- 执行“备份到 URL”并从该 URL 还原到 Azure 虚拟机
- 拆离后，将数据和日志文件复制到 Azure blob 存储，然后从 URL 附加到 Azure VM 中的 SQL Server
- 将本地物理计算机转换为 Hyper-V VHD，上载到 Azure Blob 存储，然后使用上载的 VHD 部署为新 VM
- 使用 Windows 导入/导出服务运送硬盘驱动器
- 如果在本地有 AlwaysOn 部署，请使用[添加 Azure 副本向导](/documentation/articles/virtual-machines-windows-classic-sql-onprem-availability/)在 Azure 中创建副本，然后进行故障转移，并将用户指向 Azure 数据库实例
- 使用 SQL Server [事务复制](https://msdn.microsoft.com/zh-cn/library/ms151176.aspx)将 Azure SQL Server 实例配置为订阅服务器，然后禁用复制，并将用户指向 Azure 数据库实例



## 选择迁移方法

若要获得最出色的数据传输性能，使用压缩后的备份文件将数据库文件迁移到 Azure VM 通常是最好的方法。这是[将 SQL Server 数据库部署到 Azure VM 向导](#azure-vm-deployment-wizard-tutorial)所使用的方法。如果要将 SQL Server 2005 或更高版本上运行的本地用户数据库迁移到 SQL Server 2014 或更高版本，并且压缩后的数据库备份文件小于 1 TB，则建议使用此向导。

若要在数据库迁移过程中最大限度地减少停机时间，请使用 AlwaysOn 选项或事务复制选项。

如果不能使用上述方法，请手动迁移你的数据库。使用此方法时，你通常先进行数据库备份，接下来将数据库备份复制到 Azure，然后执行数据库还原。你还可以将数据库文件本身复制到 Azure，然后附加这些文件。你可以通过多种方法完成将数据库迁移到 Azure VM 的这一手动流程。

> [AZURE.NOTE] 从较旧版本的 SQL Server 升级到 SQL Server 2014 或 SQL Server 2016 时，应考虑是否需要做一些更改。建议在迁移时处理好不受新版 SQL Server 支持的功能上的所有依赖项。有关受支持的版本和方案的详细信息，请参阅[升级到 SQL Server](https://msdn.microsoft.com/zh-cn/library/bb677622.aspx)。

下表列出了各种主要迁移方法，并讨论了最适合使用该方法的场合。

| 方法 | 源数据库版本 | 目标数据库版本 | 源数据库备份大小限制 | 说明 |
|---|---|---|---|---|
| [使用“将 SQL Server 数据库部署到 Azure VM”向导](#azure-vm-deployment-wizard-tutorial) | SQL Server 2005 或更高版本 | SQL Server 2014 或更高版本 | 小于 1 TB | 最简单快捷的方法，尽可能使用该方法迁移到 Azure 虚拟机中新的或现有的 SQL Server 实例 | 
| [使用“添加 Azure 副本”向导](/documentation/articles/virtual-machines-windows-classic-sql-onprem-availability/) | SQL Server 2012 或更高版本 | SQL Server 2012 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | 最大限度地减少停机时间，在你有 AlwaysOn 本地部署时使用 |
| [使用 SQL Server 事务复制](https://msdn.microsoft.com/zh-cn/library/ms151176.aspx) | SQL Server 2005 或更高版本 | SQL Server 2005 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | 在需要最大限度地减少停机时间，但没有 AlwaysOn 本地部署时使用 |
| [使用压缩功能执行本地备份并将备份文件手动复制到 Azure 虚拟机中](#backup-to-file-and-copy-to-vm-and-restore) | SQL Server 2005 或更高版本 | SQL Server 2005 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | 仅在无法使用向导时使用，比如目标数据库版本低于 SQL Server 2012 SP1 CU2 或数据库备份大于 1 TB（SQL Server 2016 为 12.8 TB） |
| [执行“备份到 URL”并从该 URL 还原到 Azure 虚拟机](#backup-to-url-and-restore) | SQL Server 2012 SP1 CU2 或更高版本 | SQL Server 2012 SP1 CU2 或更高版本 | 小于 12.8 TB 用于 SQL Server 2016，否则大于 1 TB | 一般来说，使用[备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx) 在性能上与使用向导等效，但操作不如后者简单 |
| [拆离后，将数据和日志文件复制到 Azure blob 存储，然后从 URL 附加到 Azure 虚拟机中的 SQL Server](#detach-and-copy-to-url-and-attach-from-url) | SQL Server 2005 或更高版本 | SQL Server 2014 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | 如果计划[使用 Azure Blob 存储服务存储这些文件](https://msdn.microsoft.com/zh-cn/library/dn385720.aspx)并将它们附加到 Azure VM 中运行的 SQL Server，尤其是对于非常大的数据库，可以使用此方法。 |
| [将本地计算机转换为 Hyper-V VHD，上载到 Azure Blob 存储，然后使用上载的 VHD 部署为新虚拟机](#convert-to-vm-and-upload-to-url-and-deploy-as-new-vm) | SQL Server 2005 或更高版本 | SQL Server 2005 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | 在以下场合使用：[使用你自己的 SQL Server 许可证](/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas/)时；迁移的数据库将在较旧版本的 SQL Server 上运行时；或者将系统数据库和用户数据库一起作为依赖于其他用户数据库和/或系统数据库的数据库的一部分进行迁移时。 |
| 使用 Windows 导入/导出服务运送硬盘驱动器 | SQL Server 2005 或更高版本 | SQL Server 2005 或更高版本 | [Azure VM 存储限制](/documentation/articles/azure-subscription-service-limits/) | Azure 中国当前尚不支持“Windows 导入/导出服务”|

##<a name="azure-vm-deployment-wizard-tutorial"></a> Azure VM 部署向导教程

使用 Microsoft SQL Server Management Studio 中的**将 SQL Server 数据库部署到 Azure VM** 向导可将 SQL Server 2005、SQL Server 2008、SQL Server 2008 R2、SQL Server 2012、SQL Server 2014 或 SQL Server 2016 本地用户数据库（最多 1 TB）迁移到 Azure 虚拟机中的 SQL Server 2014 或 SQL Server 2016。使用此向导可将用户数据库迁移到现有的 Azure 虚拟机或在迁移过程中由向导创建的带有 SQL Server 的 Azure VM。将数据库迁移到较新版本的 SQL Server 时，会在该过程中自动升级数据库。

此方法仅适用于经典部署模型。

### 获取“将 SQL Server 数据库部署到 Azure VM”向导的最新版本

使用最新版本的 Microsoft SQL Server Management Studio for SQL Server，以确保具有**将 SQL Server 数据库部署到 Azure VM** 向导的最新版本。此向导的最新版本包含对 Azure 经典管理门户的最新更新，并且支持库中最新的 Azure VM 映像（向导的较旧版本可能不支持）。若要获取 Microsoft SQL Server Management Studio for SQL Server 的最新版本，请[下载它](http://download.microsoft.com/download/0/6/4/06471151-0668-485E-8CD0-D7D8297EE357/SSMS_BurnInstaller_June2015/SSMS-Web-Setup.exe)并将其安装在与计划迁移的数据库和 Internet 建立了连接的客户端计算机上。

### 配置现有的 Azure 虚拟机和 SQL Server 实例（如果适用）

如果要迁移到现有的 Azure VM，则需执行下列配置步骤：

- 按照[在另一台计算机上从 SSMS 连接到 SQL Server VM 实例](/documentation/articles/virtual-machines-windows-sql-connect/)的步骤进行操作，将 Azure VM 和 SQL Server 实例配置为支持从另一台计算机进行连接。只有使用向导进行迁移时，才支持库中的 SQL Server 2014 和 SQL Server 2016 映像。
- 使用专用端口 11435 为 Azure 网关上的 SQL Server 云适配器服务配置一个打开的终结点。当在 Azure VM 上预配 SQL Server 2014 或 SQL Server 2016 时，将创建此端口。云适配器还创建一项 Windows 防火墙规则，以允许其传入 TCP 连接在默认端口 11435 上通过。此终结点允许向导利用云适配器服务将本地实例中的备份文件复制到 Azure VM。有关详细信息，请参阅[用于 SQL Server 的云适配器](https://msdn.microsoft.com/zh-cn/library/dn169301.aspx)。

	![创建云适配器终结点](./media/virtual-machines-windows-migrate-sql/cloud-adapter-endpoint.png)

### 运行和使用“将 SQL Server 数据库部署到 Azure VM”向导

1. 打开 Microsoft SQL Server Management Studio for Microsoft SQL Server 2016，并连接到包含将迁移到 Azure VM 的用户数据库的 SQL Server 实例。
2. 右键单击要迁移的数据库，指向“任务”，然后单击“部署到 Azure VM”。

	![启动向导](./media/virtual-machines-windows-migrate-sql/start-wizard.png)

3. 在“简介”页上单击“下一步”。
4. 在“源设置”页上，连接到包含要迁移到 Azure VM 的数据库的 SQL Server 实例。
5. 指定备份文件的临时位置。如果连接到远程服务器，则必须指定网络驱动器。

	![源设置](./media/virtual-machines-windows-migrate-sql/source-settings.png)

6. 单击“下一步”。
7. 在“Azure 登录”页上，单击“登录”以登录到你的 Azure 帐户。
8. 选择要使用的订阅并单击“下一步”。

	![Azure 登录](./media/virtual-machines-windows-migrate-sql/azure-signin.png)

9. 在“部署设置”页上，可以指定新的或现有的云服务名称和虚拟机名称：

 - 指定新的云服务名称和虚拟机名称，以便使用 SQL Server 2014 或 SQL Server 2016 库映像创建包含新 Azure 虚拟机的新云服务。

     - 如果指定新的云服务名称，则指定将使用的存储帐户。

     - 如果指定现有的云服务名称，则将检索存储帐户并为你输入该帐户。

 - 指定现有的云服务名称和新的虚拟机名称可在现有云服务中创建新的 Azure 虚拟机。只能指定 SQL Server 2014 或 SQL Server 2016 库映像。
 - 指定现有的云服务名称和虚拟机名称，以便使用现有的 Azure 虚拟机。必须有一个使用 SQL Server 2014 或 SQL Server 2016 库映像生成的映像。

		![Deploymnent Settings](./media/virtual-machines-windows-migrate-sql/deployment-settings.png)

10. 单击“设置”
  - 如果指定了现有的云服务名称和虚拟机名称，系统将提示你提供用户名和密码。

		![Azure machine settings](./media/virtual-machines-windows-migrate-sql/azure-machine-settings.png)

	- 如果指定了新的虚拟机名称，系统将提示你从库映像列表中选择一个映像，并提供以下信息：
	  - 映像 - 只能选择 SQL Server 2014 或 SQL Server 2016
		- 用户名
		- 新密码
		- 确认密码
		- 位置
		- 大小。
 	- 此外，单击接受此新 Azure 虚拟机自行生成的证书，然后单击“确定”。

	![Azure 新计算机设置](./media/virtual-machines-windows-migrate-sql/azure-new-machine-settings.png)

11. 指定目标数据库名称（如果与源数据库名称不同）。如果该目标数据库已存在，系统将自动递增数据库名称，而不是覆盖现有的数据库。
12. 单击“下一步”，然后单击“完成”。

	![结果](./media/virtual-machines-windows-migrate-sql/results.png)

13. 完成向导操作后，连接到你的虚拟机并确保数据库已迁移。
14. 如果创建了新的虚拟机，请按照[在另一台计算机上从 SSMS 连接到 SQL Server VM 实例](/documentation/articles/virtual-machines-windows-sql-connect/)的步骤进行操作，以配置 Azure 虚拟机和 SQL Server 实例。

##<a name="backup-to-file-and-copy-to-vm-and-restore"></a>备份到文件、复制到 VM 并还原

如果因为要迁移到 SQL Server 2014 之前的 SQL Server 版本或者备份文件大于 1 TB 而无法使用“将 SQL Server 数据库部署到 Azure VM”向导，可使用此方法。如果备份文件大于 1 TB，则必须对其进行条带化，因为 VM 磁盘的最大大小是 1 TB。使用此手动方法按照下列常规步骤迁移用户数据库：

1.	执行到本地位置的完整数据库备份。
2.	创建或上载具有所需 SQL Server 版本的虚拟机。
3.	根据你的要求设置连接。请参阅[连接到 Azure (Resource Manager) 上的 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-sql-connect/)。
4.	使用远程桌面、Windows 资源管理器或命令提示符处的复制命令将备份文件复制到 VM。

##<a name="backup-to-url-and-restore"></a>备份到 URL 并还原

如果因为备份文件大于 1 TB 并且要在 SQL Server 2016 中来回迁移而无法使用“将 SQL Server 数据库部署到 Azure VM”向导，可使用[备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx) 方法。对于小于 1 TB 或运行 SQL Server 2016 之前的 SQL Server 版本的数据库，建议使用向导。在 SQL Server 2016 中，条带备份集受到支持，出于性能考虑，建议使用条带备份集，但它们必须超出每个 blob 的大小限制。

##<a name="detach-and-copy-to-url-and-attach-from-url"></a>拆离、复制到 URL 并从 URL 附加

如果计划[使用 Azure Blob 存储服务存储这些文件](https://msdn.microsoft.com/zh-cn/library/dn385720.aspx)并将它们附加到 Azure VM 中运行的 SQL Server，尤其是对于非常大的数据库，可以使用此方法。使用此手动方法按照下列常规步骤迁移用户数据库：

1.	从本地数据库实例拆离数据库文件。
2.	使用 [AZCopy 命令行实用工具](/documentation/articles/storage-use-azcopy/)将拆离的数据库文件复制到 Azure blob 存储。
3.	从 Azure URL 将数据库文件附加到 Azure VM 中的 SQL Server 实例。

##<a name="convert-to-vm-and-upload-to-url-and-deploy-as-new-vm"></a>转换为 VM、上传到 URL 并部署为新的 VM

使用此方法可将本地 SQL Server 实例中的所有系统数据库和用户数据库迁移到 Azure 虚拟机。使用此手动方法按照下列常规步骤迁移整个 SQL Server 实例：

1.	使用 [Microsoft 虚拟机转换器](http://technet.microsoft.com/zh-cn/library/dn873998.aspx)将物理或虚拟机转换为 Hyper-V VHD。
2.	使用 [Add-AzureVHD cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx) 将 VHD 文件上载到 Azure 存储空间。
3.	使用上载的 VHD 部署新的虚拟机。

> [AZURE.NOTE] 若要迁移整个应用程序，请考虑使用 [Azure Site Recovery](/documentation/articles/site-recovery-overview/)。

## 后续步骤

有关在 Azure 虚拟机中运行 SQL Server 的详细信息，请参阅 [Azure 虚拟机中的 SQL Server 概述](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0704_2016-->