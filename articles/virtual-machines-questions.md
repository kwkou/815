<properties
	pageTitle="有关 Azure 虚拟机的常见问题"
	description="解答有关 Azure 虚拟机的一些最常见的问题"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags 
	ms.service="virtual-machines"
	ms.date="07/17/2015"
	wacn.date="08/29/2015"/>

# Azure 虚拟机 FAQ

本文根据 Azure VM 支持团队以及论坛、新闻组和其他文章中的评论所提供的信息，解答了用户关于 Azure 虚拟机的一些常见问题。若要了解基本信息，请从[关于虚拟机](/documentation/articles/virtual-machines-about)开始。

## 我可以在 Azure VM 上运行什么？

所有订户都可以在 Azure 虚拟机上运行服务器软件。此外，MSDN 订户还可以访问由 Azure 提供的某些 Windows 客户端映像。

对于服务器软件，你可以运行 Windows Server 的最新版本以及各种 Linux 分发版，并在其上托管各种服务器工作负荷和服务。有关支持详细信息，请参阅：

• 对于 Windows VM — [Azure 虚拟机的 Microsoft 服务器软件支持](https://support.microsoft.com/zh-cn/kb/2721672)

• 对于 Linux VM — [Azure 上的 Linux — 认可的分发版](http://www.windowsazure.cn/documentation/articles/virtual-machines-linux-endorsed-distributions/)

对于 Windows 客户端映像，可为 MSDN Azure 权益订户以及 MSDN 即用即付开发和测试订户提供 Windows 7 和 Windows 8.1 的特定版本，用于各种开发和测试任务。有关详细信息，包括说明和限制，请参阅[面向 MSDN 订户的 Windows 客户端映像](http://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/)。

## 一个虚拟机可以与多少存储配合使用？

每个数据磁盘可高达 1 TB。可使用的数据磁盘数取决于虚拟机的大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs)。

Azure 存储空间帐户为操作系统磁盘和任何数据磁盘提供存储。每个磁盘就是一个存储为页 blob 的 .vhd 文件。有关定价详细信息，请参阅[存储定价详细信息](http://www.windowsazure.cn/home/features/storage/#price)。

## 可以使用哪些虚拟硬盘类型？

Azure 支持固定的、VHD 格式的虚拟硬盘。如果想在 Azure 中使用 VHDX 格式的磁盘，可通过使用 Hyper-V 管理器或 [convert-VHD](https://technet.microsoft.com/zh-cn/library/hh848454.aspx?f=255&MSPPError=-2147217396) cmdlet 对其进行转换。之后，使用 [Add-AzureVHD](https://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx) cmdlet（在服务管理模式下）将 VHD 上载到 Azure 中的存储空间帐户，以便将其与虚拟机配合使用。该 cmdlet 会将动态 VHD 转换成固定 VHD，但不会将 VHDX 转换成 VHD。

- 有关 Linux 说明，请参阅[创建和上载包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-create-upload-vhd)。

- 有关 Windows 说明，请参阅[创建 Windows Server VHD 并将其上载到 Azure](/documentation/articles/virtual-machines-create-upload-vhd-windows-server)。

有关上载数据磁盘的说明，请参阅 Linux 或 Windows 文章并从指导如何连接到 Azure 的步骤开始。

## 这些虚拟机与 Hyper-V 虚拟机相同吗？

它们在许多方面与“第 1 代”Hyper-V VM 类似，但并不完全相同。这两种类型都提供虚拟化硬件，并且 VHD 格式的虚拟硬盘可兼容。这意味着你可以在 Hyper-V 与 Azure 之间迁移它们。Azure 有三个可能会让 Hyper-V 用户意想不到的主要区别：

- Azure 不提供到虚拟机的控制台访问。
- 大多数[大小](/documentation/articles/virtual-machines-size-specs)的 Azure VM 只有 1 个虚拟网络适配器，这意味着它们也只能有 1 个外部 IP 地址。（在少数情况下，A8 和 A9 大小的 VM 使用另一个网络适配器用于实例之间的应用程序通信。）
- Azure VM 不支持第 2 代 Hyper-V VM 的功能。有关这些功能的详细信息，请参阅 [Hyper-V 的虚拟机规范](http://technet.microsoft.com/zh-cn/library/dn592184.aspx)。

## 这些虚拟机可以使用我现有的本地网络基础结构吗？

对于在服务管理中创建的虚拟机，可以使用 Azure 虚拟网络来扩展现有基础结构。该方法类似于设置一个分支机构。你可以预配和管理 Azure 中的虚拟专用网络 (VPN)，并将其安全连接到本地 IT 基础结构。有关详细信息，请参阅[虚拟网络概述](https://msdn.microsoft.com/zh-cn/library/jj156007.aspx)。

创建虚拟机时，你需要指定希望该虚拟机属于哪个网络。举例来说，这意味着不能将现有的虚拟机加入虚拟网络。但是，可以通过以下方法解决此问题：将虚拟硬盘 (VHD) 从现有虚拟机分离，然后用它来创建具有所需网络配置的新虚拟机。

## 如何访问我的虚拟机？

需要建立远程连接以登录到虚拟机：对于 Windows VM，使用远程桌面连接；对于 Linux VM，使用安全外壳 (SSH)。有关说明，请参阅：

- [如何登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)。最多支持 2 个并发连接，除非将服务器配置为远程桌面服务会话主机。  
- [如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。默认情况下，SSH 最多允许 10 个并发连接。可以通过编辑配置文件来增加此数量。

如果遇到与远程桌面或 SSH 相关的问题，请安装并使用 [VMAccess](https://msdn.microsoft.com/zh-cn/library/azure/dn606308.aspx) 扩展来帮助修复此问题。对于 Windows VM，附加选项包括：

- 在 Azure 预览门户中，找到 VM，然后从命令栏中单击“重置远程访问”。
- 查看[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。
- 使用 Windows PowerShell 远程处理连接到 VM，或者为其他资源创建附加终结点以便连接到 VM。有关详细信息，请参阅[如何设置虚拟机的终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

如果熟悉 Hyper-V，可能会要寻找一个与虚拟机连接类似的工具。Azure 不提供类似的工具，因为它不支持到虚拟机的控制台访问。

## 可以使用 D: 盘 (Windows) 或 /dev/sdb1 (Linux) 吗？

不能使用 D: 盘 (Windows) 或 /dev/sdb1 (Linux)。它们仅提供临时存储，因此你会面临丢失数据后无法恢复的风险。一种常见操作可能会导致该情况，即，将虚拟机迁移到其他主机。导致虚拟机可能要迁移的原因有很多，其中包括调整虚拟机大小、更新主机或主机上的硬件故障。

## 如何更改临时磁盘的驱动器号？

在 Windows 虚拟机上，可以通过移动页面文件和重新分配驱动器号来更改驱动器号，但是，需要确保按特定顺序执行这些步骤。有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-change-drive-letter)。

## 如何升级来宾操作系统？

升级一词通常意味着，在同一个硬件上迁移到更高版本的操作系统。对于 Linux 和 Windows，将 Azure VM 迁移到更高版本的过程有所不同：

- 对于 Linux VM，使用适合于分发版的包管理工具和过程。
- 对于 Windows 虚拟机，使用 Windows Server 迁移工具。当来宾 OS 位于 Azure 上时，不要尝试对其升级。不支持升级是因为这样做会有丢失对虚拟机的访问权限的风险。如果在升级过程中出现问题，则可能无法启动远程桌面会话并且无法排除这些问题。有关工具和过程的一般详细信息，请参阅[将角色和功能迁移到 Windows Server](https://technet.microsoft.com/zh-cn/library/jj134039.aspx)。有关升级到 Windows Server 2012 R2 的详细信息，请参阅[适用于 Windows Server 2012 R2 的升级选项](https://technet.microsoft.com/zh-cn/library/dn303416.aspx)。

## 虚拟机上的默认用户名和密码是什么？

Azure 提供的映像没有预先配置的用户名和密码。在使用其中某个映像创建虚拟机时，需要提供用户名和密码，你将使用它们登录到虚拟机。

如果你忘记了用户名或密码，但安装了 VM 代理，则可以安装并使用 VMAccess 扩展来解决该问题。

其他详细信息：

- 对于 Linux 映像，如果你使用管理门户，系统将提供“azureuser”作为默认用户名，但你可以通过使用“从库中”而不是以“快速创建”的方式创建虚拟机来更改此用户名。通过使用“从库中”，你还可以决定是使用密码、SSH 密钥还是同时使用这两项来进行登录。该用户帐户是具有“sudo”访问权限来运行特权命令的非特权用户。“根”帐户被禁用。
- 对于 Windows 映像，需要在创建 VM 时提供用户名和密码。该帐户将被添加到 Administrators 组中。

## Azure 可以在我的虚拟机上运行防病毒软件吗？

Azure 针对防病毒解决方案提供多个选项，但由你进行管理。例如，对于反恶意软件，你可能需要单独订阅，并且你需要决定何时运行扫描和安装更新。在创建 Windows 虚拟机时或在其后某个时间，可以通过使用适用于 Microsoft Antimalware、Symantec Endpoint Protection 或 TrendMicro Deep Security Agent 的 VM 扩展，来添加防病毒支持。Symantec 和 TrendMicro 扩展支持使用限时免费试用版订阅或现有的企业订阅。Microsoft Antimalware 是免费的。有关详细信息，请参阅：

- [如何在 Azure VM 上安装和配置 Symantec Endpoint Protection](/documentation/articles/virtual-machines-install-symantec)
- [如何在 Azure VM 上安装和配置 Trend Micro Deep Security 即服务](/documentation/articles/virtual-machines-install-trend)
- [在 Azure 虚拟机上部署反恶意软件解决方案](http://azure.microsoft.com/blog/2014/05/13/deploying-antimalware-solutions-on-azure-virtual-machines/)

## 我的备份和恢复选项有哪些？

某些地区提供 Azure 备份的预览版。有关详细信息，请参阅[备份 Azure 虚拟机](/documentation/articles/backup-azure-vms)。认证合作伙伴提供了其他解决方案。若要了解当前提供的选项，请搜索 Azure 应用商店。

一个附加选项是使用 blob 存储的快照功能。为此，你需要在执行任何依赖于 blob 快照的操作之前关闭 VM。此选项将保存挂起的数据写入并使文件系统处于一致状态。

## Azure 对我的 VM 如何收费？

Azure 根据 VM 的大小和操作系统每小时计费。对于不足一小时的部分，Azure 将仅根据使用的分钟数计费。如果使用包含某个预装软件的 VM 映像创建 VM，则可能对该软件按小时额外计费。Azure 对用于 VM 操作系统和数据磁盘的存储单独计费。临时磁盘存储免费。

VM 的状态为“正在运行”或“已停止”时计费，VM 的状态为“已停止(已解除分配)”时不计费。若要使 VM 处于“已停止(已解除分配)”状态，请执行下列操作之一：

- 从管理门户中关闭或删除 VM。
- 使用 Azure PowerShell 模块中提供的 Stop-AzureVM cmdlet。
- 在服务管理 REST API 中使用“关闭角色”操作并为 PostShutdownAction 元素指定 StoppedDeallocated。

有关更多详细信息，请参阅[虚拟机定价](http://www.windowsazure.cn/home/features/virtual-machines/#price)。

## Azure 在执行维护时会重新启动我的 VM 吗？

通常情况下，你可以随时启动、停止或重新启动你的 VM。（有关详细信息，请参阅[关于启动、停止和重新启动 Azure VM](https://msdn.microsoft.com/zh-cn/library/azure/dn763934.aspx)）。Azure 在 Azure 数据中心执行定期的计划内维护更新时，有时会重新启动你的 VM。如果 Azure 检测到会影响你的 VM 的严重硬件问题，则可能发生计划外维护事件。对于计划外事件，Azure 会自动将 VM 迁移到运行状况良好的主机并重新启动 VM。

对于任何独立的 VM（意味着该 VM 不是可用性集的一部分），由于可能在更新期间重新启动 VM，因此，Azure 会在执行计划内维护的至少一个星期前以电子邮件的形式通知订阅的服务管理员。VM 上运行的应用程序可能会出现故障时间。

在因计划内维护发生重启时，你还可以使用 Azure 门户或 Azure PowerShell 查看重启日志。有关详细信息，请参阅[查看 VM 重启日志](http://azure.microsoft.com/blog/2015/04/01/viewing-vm-reboot-logs/)。

若要提供冗余，请将两个或更多配置相似的 VM 放在同一个可用性集中。这有助于确保在计划内或计划外维护期间至少有一个 VM 可用。Azure 对此配置可保证一定级别的 VM 可用性。有关详细信息，请参阅[管理虚拟机的可用性](/documentation/articles/virtual-machines-manage-availability)。

## 其他资源

[关于 Azure 虚拟机](/documentation/articles/virtual-machines-about)

[创建 Linux 虚拟机的不同方式](/documentation/articles/virtual-machines-linux-choices-create-vm)

[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-choices-create-vm)

<!---HONumber=67-->