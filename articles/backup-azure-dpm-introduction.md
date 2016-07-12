<properties
	pageTitle="Azure DPM 备份简介 | Azure"
	description="使用 Azure 备份服务备份 DPM 服务器的简介"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""
	keywords="System Center Data Protection Manager, Data Protection Manager, dpm 备份"/>

<tags
	ms.service="backup" 
	ms.date="05/10/2016"
	wacn.date="06/06/2016"/>

# 使用 DPM 准备将工作负荷备份到 Azure

> [AZURE.SELECTOR]
- [SCDPM](/documentation/articles/backup-azure-dpm-introduction/)

本文介绍如何使用 Azure 备份来保护 System Center Data Protection Manager (DPM) 服务器和工作负载。通过阅读本文，你将会了解：

- Azure DPM 服务器备份的工作原理
- 实现顺畅备份体验的先决条件
- 遇到的典型错误以及如何处理它们
- 支持的方案

> [AZURE.NOTE] Azure 有两种用于创建和使用资源的部署模型：[Resource Manager 和经典部署模型](/documentation/articles/resource-manager-deployment-model/)。本文提供有关还原使用 Resource Manager 模型部署的 VM 的信息和过程。

System Center DPM 备份文件和应用程序数据。备份到 DPM 的数据可以存储在磁带、磁盘上，或者使用 Microsoft Azure Backup 备份到 Azure。DPM 可与 Azure 备份交互，如下所述：

- **部署为物理服务器或本地虚拟机的 DPM** — 如果 DPM 部署为物理服务器或本地 Hyper-V 虚拟机，则你除了磁盘和磁带备份外，还可以将数据备份到恢复服务保管库。
- **部署为 Azure 虚拟机的 DPM** — 通过 System Center 2012 R2 Update 3，可以将 DPM 部署为 Azure 虚拟机。如果 DPM 部署为 Azure 虚拟机部署，则你可以将数据备份到附加到 DPM Azure 虚拟机的 Azure 磁盘，也可以通过将数据备份到恢复服务保管库来卸载数据存储。

## 为何要备份 DPM 服务器？

使用 Azure 备份来备份 DPM 服务器所带来的业务好处包括：

- 对于本地 DPM 部署，你可以使用 Azure 来取代长期的磁带部署。
- 对于 Azure 中的 DPM 部署，Azure 备份可让你卸载 Azure 磁盘中的存储，从而通过将较旧的数据存储在恢复服务保管库中，并将较新的数据存储在磁盘上来实现扩展。

## DPM 服务器备份的工作原理
为了提供基于磁盘的数据保护，DPM 服务器将创建并维护受保护服务器上的数据的副本。副本存储在存储池中，存储池由 DPM 服务器或自定义卷上的一系列磁盘组成。不管你要保护文件数据还是应用程序数据，都需要首先创建数据源的副本。副本将会根据配置的设置定期同步或更新。
如果你使用基于磁盘的短期保护或者在云中长期保护，DPM 可以将数据从副本卷备份到恢复服务保管库，以免对受保护的计算机造成影响。

## 先决条件
按如下所述让 Azure 备份做好备份 DPM 数据的准备：

1. **创建备份保管库** — 在 Azure 备份控制台中创建一个保管库。
2. **下载保管库凭据** — 在 Azure 备份中，将你创建的管理证书上载到保管库。
3. **安装 Azure 备份代理并注册服务器** — 通过 Azure 备份，在每个 DPM 服务器上安装代理，并在备份保管库中注册 DPM 服务器。

[AZURE.INCLUDE [backup-create-vault](../includes/backup-create-vault.md)]

[AZURE.INCLUDE [backup-download-credentials](../includes/backup-download-credentials.md)]

[AZURE.INCLUDE [backup-install-agent](../includes/backup-install-agent.md)]


## 要求和限制

- DPM 可作为物理服务器或安装在 System Center 2012 SP1 或 System Center 2012 R2 上的 Hyper-V 虚拟机运行。它也可以作为 System Center 2012 R2（至少包含 DPM 2012 R2 Update Rollup 3）上运行的 Azure 虚拟机，或 System Center 2012 R2（至少包含 Update Rollup 5）上运行的 VMWare 中的 Windows 虚拟机运行。
- 如果你要运行 System Center 2012 SP1 中的 DPM，则应安装 System Center Data Protection Manager SP1 的 Update Rollup 2。只有满足此要求，你才能安装 Azure 备份代理。
- DPM 服务器上应已安装 Windows PowerShell 和 .NET Framework 4.5。
- DPM 可将大多数工作负载备份到 Azure 备份。有关所支持内容的完整列表，请参阅下面的 Azure 备份支持项。
- 使用“复制到磁带”选项无法恢复存储在 Azure 备份中的数据。
- 你需要一个启用了 Azure 备份功能的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。阅读 [Azure 备份定价](http://www.windowsazure.cn/home/features/back-up/pricing/)的相关信息。
- 若要使用 Azure 备份，应在要备份的服务器上安装 Azure 备份代理。每台服务器上的可用本地存储空间必须至少为要备份的数据大小的 10%。例如，如果要备份 100 GB 的数据，则暂存位置至少需要 10 GB 的可用空间。尽管最低要求为 10%，但我们建议为缓存位置腾出 15% 的本地可用存储空间。
- 数据将存储在 Azure 保管库存储中。你可以备份到 Azure 备份保管库的数据量没有限制，但数据源（例如虚拟机或数据库）的大小不应超过 54400 GB。

支持将以下文件类型备份到 Azure：

- 加密（仅限完整备份）
- 压缩（支持增量备份）
- 稀疏（支持增量备份）
- 压缩和稀疏（视为稀疏）

以下内容不受支持：

- 不支持区分大小写的文件系统上的服务器。
- 硬链接（跳过）
- 重分析点（跳过）
- 加密和压缩（跳过）
- 加密和稀疏（跳过）
- 压缩流
- 稀疏流

>[AZURE.NOTE] 从 System Center 2012 DPM SP1 开始，你可以使用 Microsoft Azure 备份将 DPM 保护的工作负载备份到 Azure。

<!---HONumber=Mooncake_0530_2016-->