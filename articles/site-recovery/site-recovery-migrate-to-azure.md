<properties
    pageTitle="使用 Site Recovery 迁移到 Azure | Azure"
    description="本文概述如何使用 Azure Site Recovery 将 VM 和物理服务器迁移到 Azure"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="" />
<tags
    ms.assetid="c413efcd-d750-4b22-b34b-15bcaa03934a"
    ms.service="site-recovery"
    ms.workload="backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="01/04/2017"
    wacn.date="02/10/2017"
    ms.author="raynew" />  


# 使用 Site Recovery 迁移到 Azure

本文概述如何使用 Azure Site Recovery 服务迁移虚拟机和物理服务器。

组织需要制定 BCDR 策略来确定应用、工作负荷和数据如何在计划和非计划的停机期间保持运行和可用，并尽快恢复正常运行情况。BCDR 策略应保持业务数据的安全性和可恢复性，并确保在发生灾难时工作负荷持续可用。

Site Recovery 是一项 Azure 服务，可以通过协调从本地物理服务器和虚拟机到云 (Azure) 或辅助数据中心的的复制，来为 BCDR 策略提供辅助。当主要位置发生故障时，可以故障转移到辅助位置，使应用和工作负荷保持可用。当主要位置恢复正常时，你可以故障回复到主要位置。有关详细信息，请参阅[什么是 Site Recovery？](/documentation/articles/site-recovery-overview/)
使用 [Azure 经典管理门户](https://manage.windowsazure.cn/)可以维护现有的 Site Recovery 保管库，但无法创建新保管库。



## 迁移的意思是什么？

可以部署 Site Recovery，将本地 VM 和物理服务器完整复制到 Azure 或辅助站点。可以复制计算机，在发生服务中断时从主站点将它们故障转移，然后在主站点恢复服务时将它们故障回复到主站点。除了完整复制以外，还可以使用 Site Recovery 将 VM 和物理服务器迁移到 Azure，以便用户可以从 Azure VM 访问计算机工作负荷。迁移涉及到复制，以及从主站点故障转移到 Azure。但是，与完整复制不同，它不包含故障回复机制。

## Site Recovery 可以迁移哪些工作负荷？

可执行以下操作：

- 迁移本地 Hyper-V VM 和物理服务器上运行的工作负荷，使其在 Azure VM 上运行。在此方案中，还可以执行完整复制和故障回复。
- 在 Azure 区域之间迁移 [Azure IaaS VM](/documentation/articles/site-recovery-migrate-azure-to-azure/)。目前只有此方案才支持迁移，这意味着，故障回复不受支持。
- 将 [AWS Windows 实例](/documentation/articles/site-recovery-migrate-aws-to-azure/)迁移到 Azure IaaS VM。目前只有此方案才支持迁移，这意味着，故障回复不受支持。

## 迁移本地 VM 和物理服务器

若要迁移本地 Hyper-V VM 和物理服务器，遵循的步骤大致与普通复制相同。此过程包括设置恢复服务保管库，配置所需的管理服务器（具体取决于想要迁移哪些工作负荷），将这些服务器添加到保管库，然后指定复制设置。为想要迁移的计算机启用复制，并运行快速测试故障转移，确保一切按预期运行。

确认复制环境正常工作后，根据方案[支持的操作](/documentation/articles/site-recovery-failover/#failover-and-failback)使用计划内或计划外故障转移。迁移时，不需要提交故障转移或删除任何内容。对于想要迁移的每台计算机，可以选择“完成迁移”选项。“完成迁移”操作会完成迁移过程，删除计算机复制设置，使计算机不再产生 Site Recovery 费用。

## 在 Azure 区域之间迁移

可以使用 Site Recovery 在区域之间迁移 Azure VM。此方案仅支持迁移。换而言之，可以复制 Azure VM 并将其故障转移到另一区域，但无法故障回复。在此方案中，需要设置一个恢复服务保管库，部署本地配置服务器用于管理复制，将该服务器添加到保管库，然后指定复制设置。为想要迁移的计算机启用复制，运行快速测试故障转移。然后，使用“完成迁移”选项运行计划外故障转移。

## 将 AWS 迁移到 Azure

可将 AWS 实例迁移到 Azure VM。此方案仅支持迁移。换而言之，可以复制 AWS 实例并将其故障转移到 Azure，但无法故障回复。在迁移过程中，AWS 实例的处理方式与物理服务器相同。需要设置一个恢复服务保管库，部署本地配置服务器用于管理复制，将该服务器添加到保管库，然后指定复制设置。为想要迁移的计算机启用复制，运行快速测试故障转移。然后，使用“完成迁移”选项运行计划外故障转移。




## 后续步骤

- [将 VMM 云中的 Hyper-V VM 迁移到 Azure](/documentation/articles/site-recovery-vmm-to-azure/)
- [将不包含 VMM 的 Hyper-V VM 迁移到 Azure](/documentation/articles/site-recovery-hyper-v-site-to-azure/)
- [在 Azure 区域之间迁移 Azure VM](/documentation/articles/site-recovery-migrate-azure-to-azure/)
- [将 AWS 实例迁移到 Azure](/documentation/articles/site-recovery-migrate-aws-to-azure/)

<!---HONumber=Mooncake_0206_2017-->