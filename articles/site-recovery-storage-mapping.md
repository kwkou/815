<properties
	pageTitle="映射 Azure Site Recovery 中的存储，以便在本地数据中心之间进行 Hyper-V 虚拟机复制 | Azure"
	description="准备存储映射，以便通过 Azure Site Recovery 在两个本地数据中心之间进行 Hyper-V 虚拟机复制。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"  
	ms.date="02/22/2016"
	wacn.date="04/05/2016"/>


# 准备存储映射，以便通过 Azure Site Recovery 在两个本地数据中心之间进行 Hyper-V 虚拟机复制


Azure Site Recovery 有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。本文介绍存储映射。当你使用 Site Recovery 在两个本地 VMM 数据中心之间复制 Hyper-V 虚拟机时，可以通过存储映射来充分利用存储空间。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## 概述

如下所示，仅当你需要使用 Hyper-V 副本或 SAN 复制将 VMM 云中的 Hyper-V 虚拟机从主数据中心复制到辅助数据中心时，才需要使用存储映射：


- **使用 Hyper-V 副本进行本地到本地复制** - 设置存储映射时，你可以在源和目标 VMM 服务器上映射存储分类，以实现以下功能：

	- **标识副本虚拟机的目标存储** - 虚拟机将复制到选择的存储目标（SMB 共享或群集共享卷 (CSV)）。
	- **放置副本虚拟机** - 使用存储映射能以最佳方式在 Hyper-V 主机服务器上放置副本虚拟机。副本虚拟机将放置在可访问映射存储分类的主机上。
	- **无存储映射** - 如果未配置存储映射，虚拟机将复制到与副本虚拟机关联的 Hyper-V 主机服务器上指定的默认存储位置。

- **使用 SAN 进行本地到本地复制** - 设置存储映射时，你可以在源和目标 VMM 服务器上映射存储阵列池。
	- **指定池** - 指定哪个辅助存储池要从主池接收复制数据。
	- **标识目标存储池** - 确保将源复制组中的 LUN 复制到所选的映射目标存储池。

## 针对 Hyper-V 复制设置存储分类

当你使用 Hyper-V 副本通过 Site Recovery 进行复制时，可以在源和目标 VMM 服务器上的存储分类之间映射；如果两个站点由同一个 VMM 服务器管理，则可以在单个 VMM 服务器上映射。请注意：

- 在正确配置映射并启用复制后，位于主位置的虚拟机的虚拟硬盘将复制到位于映射目标位置的存储。
- 存储分类必须可供源和目标云中的主机组使用。
- 分类不需要具有相同类型的存储。例如，你可以将包含 SMB 共享的源分类映射到包含 CSV 的目标分类。
- 有关详细信息，请阅读[如何在 VMM 中创建存储分类](https://technet.microsoft.com/zh-cn/library/gg610685.aspx)。

## 示例

如果已在 VMM 中正确配置分类，则当你在执行存储映射期间选择源和目标 VMM 服务器时，将显示源和目标分类。下面是在北京和上海具有两个位置的组织的存储文件共享与分类示例。

**位置** | **VMM 服务器** | **文件共享（源）** | **分类（源）** | **映射到** | **文件共享（目标）**
---|---|--- |---|---|---
北京 | VMM_Source| SourceShare1 | GOLD | GOLD_TARGET | TargetShare1
 | | SourceShare2 | SILVER | SILVER_TARGET | TargetShare2
 | | SourceShare3 | BRONZE | BRONZE_TARGET | TargetShare3
上海 | VMM_Target | | GOLD_TARGET | 未映射 |
| | | SILVER_TARGET | 未映射 |
 | | | BRONZE_TARGET | 未映射

可以在 Site Recovery 门户的“资源”页中的“服务器存储”选项卡上配置这些设置。

![配置存储映射](./media/site-recovery-storage-mapping/storage-mapping1.png)

在此示例中：
- 为 GOLD 存储 (SourceShare1) 上的任何虚拟机创建副本虚拟机时，它将会复制到 GOLD_TARGET 存储 (TargetShare1)。
- 为 SILVER 存储 (SourceShare2) 上的任何虚拟机创建副本虚拟机时，它将会复制到 SILVER_TARGET (TargetShare2) 存储。依此类推。

下一屏幕截图显示了 VMM 中的实际文件共享及其分配的分类。

![VMM 中的存储分类](./media/site-recovery-storage-mapping/storage-mapping2.png)

## 多个存储位置

如果将目标分类分配到了多个 SMB 共享或 CSV，则在保护虚拟机时，将自动选择最佳存储位置。如果指定的分类没有合适的目标存储，则使用 Hyper-V 主机上指定的默认存储来放置副本虚拟硬盘。

下表显示了本示例中的存储分类和群集共享卷是如何设置的。

**位置** | **分类** | **关联的存储**
---|---|---
北京 | GOLD | <p>C:\ClusterStorage\SourceVolume1</p><p>\FileServer\SourceShare1</p>
 | SILVER | <p>C:\ClusterStorage\SourceVolume2</p><p>\FileServer\SourceShare2</p>
上海 | GOLD_TARGET | <p>C:\ClusterStorage\TargetVolume1</p><p>\FileServer\TargetShare1</p>
 | SILVER_TARGET| <p>C:\ClusterStorage\TargetVolume2</p><p>\FileServer\TargetShare2</p>

下表汇总了当你在此示例环境中为虚拟机 (VM1 - VM5) 启用保护时的行为。

**虚拟机** | **源存储** | **源分类** | **映射的目标存储**
---|---|---|---
VM1 | C:\ClusterStorage\SourceVolume1 | GOLD | <p>C:\ClusterStorage\SourceVolume1</p><p>\\FileServer\SourceShare1</p><p>两个 GOLD_TARGET</p>
VM2 | \FileServer\SourceShare1 | GOLD | <p>C:\ClusterStorage\SourceVolume1</p><p>\FileServer\SourceShare1</p> <p>两个 GOLD_TARGET</p>
VM3 | C:\ClusterStorage\SourceVolume2 | SILVER | <p>C:\ClusterStorage\SourceVolume2</p><p>\FileServer\SourceShare2</p>
VM4 | \FileServer\SourceShare2 | SILVER |<p>C:\ClusterStorage\SourceVolume2</p><p>\FileServer\SourceShare2</p><p>两个 SILVER_TARGET</p>
VM5 | C:\ClusterStorage\SourceVolume3 | 不适用 | 无映射，因此将使用 Hyper-V 主机的默认存储位置

## 后续步骤

现在，你已经对存储映射有了更好的了解，因此可以[准备部署 Azure Site Recovery](/documentation/articles/site-recovery-best-practices/) 了。

<!---HONumber=Mooncake_0328_2016-->