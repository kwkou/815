<properties
	pageTitle="VM 常见问题解答 | Azure"
	description="回答了通过经典部署模型创建的 Azure 虚拟机的一些常见问题。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="11/16/2015"
	wacn.date="03/21/2016"/>

	
# 使用经典部署模型创建的 Azure 虚拟机的常见问题

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


本文讨论了通过经典部署模型创建的 Azure 虚拟机的一些用户常见问题。

## 我可以在 Azure VM 上运行什么程序？

所有订户都可以在 Azure 虚拟机上运行服务器软件。你可以运行最新版本的 Windows Server，以及各种 Linux 分发版。如需详细的支持信息，请参阅：

• Windows VM -- [Microsoft 服务器软件对 Azure 虚拟机的支持](https://support.microsoft.com/zh-cn/kb/2721672)

• Linux VM -- [Azure 认可分发版中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distributions/)

对于 Windows 客户端映像，某些版本的 Windows 7 和 Windows 8.1 可供 MSDN Azure 权益订户以及 MSDN 开发和测试即用即付订户用于开发和测试任务。有关详细信息（包括说明和限制），请参阅[适用于 MSDN 订户的 Windows 客户端映像](https://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/)。

## 使用虚拟机时，我可以使用多少存储？

每个数据磁盘的容量高达 1 TB。你可以使用的数据磁盘的数目取决于虚拟机的大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs)。

Azure 存储帐户提供可用于操作系统磁盘和任意数据磁盘的存储。每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。如需定价详细信息，请参阅[存储定价详细信息](/home/features/storage/#price)。

## 我可以使用哪些虚拟硬盘类型？

Azure 只支持固定的 VHD 格式的虚拟硬盘。如果你想在 Azure 中使用 VHDX，需先使用 Hyper-V 管理器或 [convert-VHD](https://technet.microsoft.com/zh-cn/library/hh848454.aspx) cmdlet 对其进行转换。然后，请使用 [Add-AzureVHD](https://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx) cmdlet（在“服务管理”模式下）将 VHD 上载到 Azure 中的存储帐户，以便将其用于虚拟机。

- 如需 Linux 说明，请参阅[创建并上载包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-create-upload-vhd)。

- 如需 Windows 说明，请参阅[创建 Windows Server VHD 并将其上载到 Azure](/documentation/articles/virtual-machines-create-upload-vhd-windows-server)。

## 这些虚拟机与 Hyper-V 虚拟机相同吗？

它们在很多方面与“第 1 代”Hyper-V VM 类似，但并非完全相同。两种类型都提供虚拟化的硬件，而 VHD 格式的虚拟硬盘是兼容的。这意味着你可以在 Hyper-V 与 Azure 之间移动它们。存在以下三大区别，这有时会令 Hyper-V 用户感到惊讶：

- Azure 不提供对虚拟机的控制台访问。在 VM 完成启动之前，无法对其进行访问。
- 多数[大小](/documentation/articles/virtual-machines-size-specs)的 Azure VM 只有 1 个虚拟网络适配器，这意味着它们也只能有 1 个外部 IP 地址。
- Azure VM 不支持第 2 代 Hyper-V VM 功能。有关这些功能的详细信息，请参阅[适用于 Hyper-V 的虚拟机规格](http://technet.microsoft.com/zh-cn/library/dn592184.aspx)和[第 2 代虚拟机概述](https://technet.microsoft.com/zh-cn/library/dn282285.aspx)。

## 这些虚拟机能否使用我现有的本地网络基础结构？

对于通过经典部署模型创建的虚拟机，你可以使用 Azure 虚拟网络来扩展现有的基础结构。该方法类似于设置一个分支机构。你可以预配和管理 Azure 中的虚拟专用网络 (VPN)，并将其安全地连接到本地 IT 基础结构。有关详细信息，请参阅[虚拟网络概述](/documentation/articles/virtual-networks-overview)。

在创建虚拟机时，需根据需要指定虚拟机所属的网络。不能将现有虚拟机加入虚拟网络。不过，将虚拟硬盘 (VHD) 与现有虚拟机分离即可解决此问题，然后你就可以使用它来创建具有所需网络配置的新虚拟机。

## 如何访问我的虚拟机？

你需要通过适用于 Windows VM 的远程桌面连接或适用于 Linux VM 的安全外壳 (SSH) 建立登录虚拟机所需的远程连接。有关说明，请参阅：

- [如何登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)。除非将服务器配置为远程桌面服务会话主机，否则最多支持 2 个并发连接。  
- [如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。默认情况下，SSH 允许的并发连接最多为 10 个。通过编辑配置文件，可以增大此数目。

如果你遇到远程桌面或 SSH 方面的问题，请安装和使用 [VMAccess](/documentation/articles/virtual-machines-extensions-features) 扩展来帮助解决问题。

Windows VM 的其他选项包括：

- 在 Azure 管理门户中找到 VM，然后单击命令栏中的“重置远程访问”。
- 查看[解决远程桌面连接到基于 Windows 的 Azure 虚拟机的问题](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。
- 使用 Windows PowerShell 远程处理连接到 VM，或创建其他终结点以方便其他资源连接到 VM。有关详细信息，请参阅[如何设置虚拟机的终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

如果你熟悉 Hyper-V，可以寻找类似于 VMConnect 的工具。Azure 不提供类似的工具，因为不支持通过控制台来访问虚拟机。

## 我是否可以使用临时磁盘（Windows 的 D: 盘或 Linux 的 /dev/sdb1）来存储数据？

不应使用临时磁盘（Windows 默认的 D: 盘或 Linux 的 /dev/sdb1）来存储数据。这些磁盘只是临时存储空间，因此存在丢失数据且数据不能恢复的风险。将虚拟机移到另一主机时，可能会发生这种情况。调整虚拟机大小、更新主机、主机硬件故障等都是需要移动虚拟机的原因。

## 如何更改临时磁盘的驱动器号？

在 Windows 虚拟机中，你可以通过移动页面文件和重新分配驱动器号来更改驱动器号，但需确保按特定顺序执行这些步骤。有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-change-drive-letter)。

## 如何升级来宾操作系统？

“升级”这一术语通常是指在保留原有硬件的情况下，移到更新的操作系统版本。就 Azure VM 来说，移到更新的 Linux 版本和移到更新的 Windows 版本是不同的流程：

- 对于 Linux VM 来说，可使用与分发版相对应的包管理工具和过程。
- 对于 Windows 虚拟机来说，需使用 Windows Server 迁移工具之类的工具来迁移服务器。请勿尝试升级来宾 OS（如果驻留在 Azure 上）。由于存在失去虚拟机访问权限的风险，因此不支持该功能。如果在升级过程中出现问题，你可能无法启动远程桌面会话且无法解决这些问题。 

有关 Windows Server 迁移工具和过程的常规详细信息，请参阅[将角色和功能迁移到 Windows Server](https://technet.microsoft.com/zh-cn/library/jj134039.aspx)。



## 虚拟机上的默认用户名和密码是什么？

由 Azure 提供的映像没有预先配置的用户名和密码。当你使用这些映像之一创建虚拟机时，需提供用户名和密码，你可以使用该用户名和密码登录到虚拟机。

如果你忘记了用户名或密码，并且你已安装了 VM 代理，则可安装并使用 [VMAccess](/documentation/articles/virtual-machines-extensions-features) 扩展来解决该问题。

其他详细信息：


- 对于 Linux 映像，如果你使用 Azure 管理门户，系统会提供“azureuser”作为默认用户名，但你可以更改该用户名，只需使用“从库中”而非“快速创建”作为创建虚拟机的方法即可。使用“从库中”，你可以决定在登录时是使用密码、SSH 密钥还是二者都使用。该用户帐户是一个没有特权的用户，但具有运行特权命令所需的“sudo”访问权限。“root”帐户已禁用。


- 对于 Windows 映像，在创建 VM 时需提供用户名和密码。该帐户添加到管理员组。

## Azure 能否在我的虚拟机上运行防病毒软件？

Azure 针对防病毒解决方案提供了多种选项，但需要用户自行管理。例如，你可能需要另外订阅反恶意软件的软件，并需要自行决定运行扫描和安装更新的时间。你可以在创建 Windows 虚拟机时通过适用于 Microsoft 反恶意软件或 TrendMicro Deep Security Agent 的 VM 扩展来添加防病毒支持，也可以稍后进行。TrendMicro 扩展允许你使用免费的限时试用订阅或使用现有的企业订阅。Microsoft 反恶意软件是免费的。有关详细信息，请参阅：

- [如何在 Azure VM 上安装和配置 Trend Micro Deep Security 即服务](/documentation/articles/virtual-machines-install-trend/)
- [在 Azure 虚拟机上部署反恶意软件解决方案](https://azure.microsoft.com/zh-cn/blog/2014/05/13/deploying-antimalware-solutions-on-azure-virtual-machines/)

## 有哪些选项可用于备份和恢复？

在某些区域，Azure 备份提供预览版。有关详细信息，请参阅[备份 Azure 虚拟机](/documentation/articles/backup-azure-vms)。认证合作伙伴提供了其他解决方案。若要了解目前提供的内容，请搜索 Azure 库。

另一个选项是使用 blob 存储的快照功能。为此，需要在进行任何依赖于 blob 快照的操作之前关闭 VM。这会保存挂起数据写入并保持文件系统的一致状态。

## Azure 如何对 VM 收费？

Azure 根据 VM 的大小和操作系统按小时价格进行计费。对于不足一小时的部分，Azure 将仅根据使用的分钟数计费。如果你使用包含特定的预安装软件的 VM 映像创建 VM，则还会按小时收取额外的软件费用。对于 VM 的操作系统和数据磁盘，Azure 会分开收取存储费用。临时磁盘存储是免费的。

当 VM 状态为“正在运行”或“已停止”时将计费，但是当 VM 状态为“已停止”（“已释放”）时不会计费。若要将 VM 置于“已停止”（“已释放”）状态，请执行以下操作之一：

- 从 Azure 管理门户关闭或删除该 VM。
- 使用 Azure PowerShell 模块中提供的 Stop-AzureVM cmdlet。
- 使用服务管理 REST API 中的关闭角色操作，并为 PostShutdownAction 元素指定 StoppedDeallocated。

如需更多详细信息，请参阅[虚拟机定价](/home/features/virtual-machines/#price)。

## Azure 是否会重新启动我的 VM 进行维护？

在进行 Azure 数据中心定期计划内维护更新时，Azure 有时会重新启动你的 VM。

当 Azure 检测到影响你的 VM 的严重硬件问题时，可能会发生计划外维护事件。对于计划外事件，Azure 会自动将 VM 迁移到正常运行的主机并重新启动 VM。

对于任何独立的 VM（该 VM 不属于可用性集），Azure 将在进行计划内维护之前提前至少 1 周向订阅的服务管理员发送电子邮件通知，因为在更新期间各个 VM 可能会重新启动。在 VM 上运行的应用程序可能会遭遇停机。

因计划内维护而重新启动时，你还可以使用 Azure 管理门户或 Azure PowerShell 查看重新启动日志。有关详细信息，请参阅[查看 VM 重新启动日志](https://azure.microsoft.com/blog/2015/04/01/viewing-vm-reboot-logs/)。

若要提供冗余，请将两个或更多个采用类似配置的 VM 放到同一个可用性集中。这可以确保在计划内或计划外维护期间至少有一个 VM 可用。对于此配置，Azure 可以保证一定级别的 VM 可用性。有关详细信息，请参阅[管理虚拟机的可用性](/documentation/articles/virtual-machines-manage-availability)。

## 其他资源

[关于 Azure 虚拟机](/documentation/articles/virtual-machines-about)

[创建 Linux 虚拟机的不同方式](/documentation/articles/virtual-machines-linux-choices-create-vm)

[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-choices-create-vm)

<!---HONumber=Mooncake_0314_2016-->