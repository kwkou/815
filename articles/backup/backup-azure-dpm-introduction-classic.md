<properties
    pageTitle="将 DPM 工作负荷备份到 Azure 经典管理门户 | Azure"
    description="使用 Azure 备份服务备份 DPM 服务器的简介"
    services="backup"
    documentationcenter=""
    author="Nkolli1"
    manager="shreeshd"
    editor=""
    keywords="System Center Data Protection Manager, Data Protection Manager, dpm 备份"
    translationtype="Human Translation" />
<tags
    ms.assetid="8f23972b-d167-4231-b331-e198db3b18b4"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/10/2017"
    wacn.date="04/24/2017"
    ms.author="nkolli;giridham;markgal"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="69d39f5c97548d88206f3a22225b87b6313bdbad"
    ms.lasthandoff="04/14/2017" />

# <a name="preparing-to-back-up-workloads-to-azure-with-dpm"></a>使用 DPM 准备将工作负荷备份到 Azure
> [AZURE.SELECTOR]
- [Azure 备份服务器（经典）](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)
- [SCDPM（经典）](/documentation/articles/backup-azure-dpm-introduction-classic/)

本文介绍如何使用 Azure 备份来保护 System Center Data Protection Manager (DPM) 服务器和工作负荷。 通过阅读本文，你将会了解：

- Azure DPM 服务器备份的工作原理
- 实现顺畅备份体验的先决条件
- 遇到的典型错误以及如何处理它们
- 支持的方案

System Center DPM 备份文件和应用程序数据。 备份到 DPM 的数据可以存储在磁带、磁盘上，或者使用 Azure Backup 备份到 Azure。 DPM 可与 Azure 备份交互，如下所述：

- **部署为物理服务器或本地虚拟机的 DPM** — 如果 DPM 部署为物理服务器或本地 Hyper-V 虚拟机，则你除了磁盘和磁带备份外，还可以将数据备份到 Azure 备份保管库。
- **部署为 Azure 虚拟机的 DPM** — 通过 System Center 2012 R2 Update 3，可以将 DPM 部署为 Azure 虚拟机。 如果 DPM 部署为 Azure 虚拟机部署，则你可以将数据备份到附加到 DPM Azure 虚拟机的 Azure 磁盘，也可以通过将数据备份到 Azure 备份保管库来卸载数据存储。

## <a name="why-backup-your-dpm-servers"></a>为何要备份 DPM 服务器？
使用 Azure 备份来备份 DPM 服务器所带来的业务好处包括：

- 对于本地 DPM 部署，你可以使用 Azure 备份来取代长期的磁带部署。
- 对于 Azure 中的 DPM 部署，Azure 备份可让你卸载 Azure 磁盘中的存储，从而通过将较旧的数据存储在 Azure 备份上，并将较新的数据存储在磁盘上来实现扩展。

## <a name="how-does-dpm-server-backup-work"></a>DPM 服务器备份的工作原理
若要备份虚拟机，首先需要获得数据的时间点快照。 Azure 备份服务在计划的时间启动备份作业，并触发要拍摄快照的备份扩展。 备份扩展会与来宾内部的 VSS 服务进行协调以确保一致性，并会在达成一致后调用 Azure 存储服务的 Blob 快照 API。 这样做可以获得虚拟机磁盘的一致性快照，不必关闭虚拟机。

拍摄快照后，数据将由 Azure 备份服务传输到备份保管库中。 该服务负责确定并传输自上次备份以来进行了更改的块，使备份存储和网络更高效。 数据传输完成后，将会删除快照并创建恢复点。 在 Azure 经典管理门户中，可以查看此恢复点。

> [AZURE.NOTE]
> Linux 虚拟机只能使用文件一致性备份。
>
>

## <a name="prerequisites"></a>先决条件
按如下所述让 Azure 备份做好备份 DPM 数据的准备：

1. **创建备份保管库**
2. **下载保管库凭据** — 在 Azure 备份中，将你创建的管理证书上载到保管库。
3. **安装 Azure 备份代理并注册服务器** — 通过 Azure 备份，在每个 DPM 服务器上安装代理，并在备份保管库中注册 DPM 服务器。

[AZURE.INCLUDE [backup-download-credentials](../../includes/backup-download-credentials.md)]

[AZURE.INCLUDE [backup-install-agent](../../includes/backup-install-agent.md)]

## <a name="requirements-and-limitations"></a>要求和限制
- DPM 可作为物理服务器或安装在 System Center 2012 SP1 或 System Center 2012 R2 上的 Hyper-V 虚拟机运行。 它也可以作为 System Center 2012 R2（至少包含 DPM 2012 R2 Update Rollup 3）上运行的 Azure 虚拟机，或 System Center 2012 R2（至少包含 Update Rollup 5）上运行的 VMWare 中的 Windows 虚拟机运行。
- 如果要随 System Center 2012 SP1 一起运行 DPM，则应安装 System Center Data Protection Manager SP1 的 Update Rollup 2。 只有满足此要求，你才能安装 Azure 备份代理。
- DPM 服务器上应已安装 Windows PowerShell 和 .NET Framework 4.5。
- DPM 可将大多数工作负载备份到 Azure 备份。 有关所支持内容的完整列表，请参阅下面的 Azure 备份支持项。
- 使用“复制到磁带”选项无法恢复存储在 Azure 备份中的数据。
- 你需要一个启用了 Azure 备份功能的 Azure 帐户。 如果没有帐户，可以创建一个试用帐户，只需几分钟即可完成。 阅读 [Azure 备份定价](/pricing/details/backup/)的相关信息。
- 若要使用 Azure 备份，应在要备份的服务器上安装 Azure 备份代理。 每台服务器上的可用本地存储空间必须至少为要备份的数据大小的 10%。 例如，如果要备份 100 GB 的数据，则暂存位置至少需要 10 GB 的可用空间。 尽管最低要求为 10%，但我们建议为缓存位置腾出 15% 的本地可用存储空间。
- 数据将存储在 Azure 保管库存储中。 可以备份到 Azure 备份保管库的数据量没有限制，但数据源（例如，虚拟机或数据库）的大小不应超过 54,400 GB。

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

> [AZURE.NOTE]
> 从 System Center 2012 DPM SP1 开始，可以使用 Azure 备份将 DPM 保护的工作负荷备份到 Azure。
>
>

<!---Update_Description: wording update -->