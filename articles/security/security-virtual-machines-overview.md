<properties
   pageTitle="Azure 虚拟机安全概述 | Microsoft Azure"
   description=" 使用 Azure 虚拟机可以灵活实现虚拟化，无需购买和维护运行虚拟机的物理硬件。本文概述了可用于 Azure 虚拟机的核心 Azure 安全功能。"
   services="security"
   documentationCenter="na"
   authors="lingche"
   manager="shlan"
   editor="lingche"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/16/2016"
   wacn.date="10/31/2016"
   ms.author="terrylan"/>  


# Azure 虚拟机安全概述

使用 Azure 虚拟机可以灵活部署各种计算解决方案。通过对 Microsoft Windows、Linux、Microsoft SQL Server、Oracle、IBM、SAP 和 Azure BizTalk 服务的支持，几乎可以在所有操作系统上部署任何解决方案。

Azure 虚拟机让你能够灵活地进行虚拟化，而无需购买和维护运行虚拟机的物理硬件。可以构建并部署应用程序，保证数据在高度安全的数据中心受到保护。

使用 Azure 可以构建安全增强且符合法规的解决方案：

- 保护虚拟机不受病毒和恶意软件的侵害
- 加密敏感数据
- 安全的网络流量
- 识别和检测威胁
- 满足合规性要求

本文旨在对可用于虚拟机的核心 Azure 安全功能提供概述。此外还提供了文章链接，更详细说明每项功能。

本文介绍的核心 Azure 虚拟机安全功能包括：

- 反恶意软件
- 硬件安全模块
- 虚拟机备份
- Azure Site Recovery
- 虚拟网络
- 合规性

## 反恶意软件

借助 Azure，可以使用安全供应商（例如 Microsoft、Symantec、Trend Micro、McAfee 和 Kaspersky）提供的反恶意软件，保护虚拟机抵御恶意文件、广告软件和其他威胁。请参阅下面的“了解详细信息”部分，找到有关合作伙伴解决方案的文章。

适用于 Azure 云服务和虚拟机的 Microsoft 反恶意软件是提供一种实时保护功能，可帮助识别并移除病毒、间谍软件和其他恶意软件。当已知恶意软件或不需要的软件试图在你的 Azure 系统上安装或运行时，Microsoft 反恶意软件将提供可配置的警报。

Microsoft 反恶意软件是一个针对应用程序和租户环境所提供的单一代理解决方案，可在在后台运行而无需人工干预。你可以根据应用程序工作负荷的需求，选择默认的基本安全性或高级的自定义配置（包括反恶意软件监视）来部署保护。

部署并启用 Microsoft 反恶意软件后，便可以使用以下几项核心功能：

- 实时保护 - 监视云服务和虚拟机上的活动，以检测和阻止恶意软件的执行。
- 计划的扫描 - 定期执行有针对性的扫描，以检测恶意软件，包括主动运行的程序。
- 恶意软件消除 - 自动针对检测到的恶意软件采取措施，例如删除或隔离恶意文件以及清除恶意注册表项。
- 签名更新 - 自动安装最新的保护签名（病毒定义）以确保按预定的频率保持最新保护状态。
- 反恶意软件引擎更新 - 自动更新 Microsoft 反恶意软件引擎。
- 反恶意软件平台更新 – 自动更新 Microsoft 反恶意软件平台。
- 主动保护 - 将检测到的威胁和可疑资源的遥测元数据报告给 Azure，确保做出快速响应，并通过 Microsoft Active Protection System (MAPS) 启用实时同步签名传递。
- 示例报告 - 将示例提供并报告给 Microsoft 反恶意软件服务，帮助改善服务并实现故障排除。
- 排除项 - 允许应用程序和服务管理员配置特定的文件、进程与驱动器，以便出于性能和/或其他原因将其从保护和扫描中排除。
- 恶意软件事件收集 -在操作系统事件日志中记录反恶意软件服务的运行状况、可疑活动及采取的补救措施，并将这些数据收集到客户的 Azure 存储帐户。

了解详细信息：有关使用反恶意软件保护虚拟机的详细信息，请参阅：

- [适用于 Azure 云服务和虚拟机的 Microsoft 反恶意软件](/documentation/articles/azure-security-antimalware/)
- [在 Azure 虚拟机上部署反恶意软件解决方案](https://azure.microsoft.com/blog/deploying-antimalware-solutions-on-azure-virtual-machines/)
- [如何在 Windows VM 上安装和配置 Asiainfo Deep Security 即服务](/documentation/articles/virtual-machines-windows-classic-install-trend/)
- [Azure 映像应用商店中的安全解决方案](https://market.azure.cn/List/Index?sort=Featured&filters=tag:security)

## 硬件安全模块

加密和身份验证无法提高安全性，除非密钥本身也受到保护。通过将关键密码和密钥存储在 Azure 密钥保管库中，可以简化此类密码和密钥的管理和保护。密钥保管库使你可以将密钥存储在 FIPS 140-2 级别 2 标准认证的硬件安全模块 (HSM) 中。用于备份或[透明数据加密](https://msdn.microsoft.com/zh-cn/library/bb934049.aspx)的 SQL Server 加密密钥可以存储在密钥保管库中，此外还可存储应用程序中的任意密钥或机密。对这些受保护项的权限和访问权限通过 [Azure Active Directory](/documentation/services/identity/) 进行管理。

了解详细信息：

- [什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)
- [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)
- [Azure 密钥保管库博客](https://blogs.technet.microsoft.com/kv/)

## 虚拟机备份

Azure 备份是一个可缩放的解决方案，无需资本投资便可保护应用程序数据，从而最大限度降低运营成本。应用程序错误可能会损坏数据，人为错误可能会将 bug 引入应用程序。使用 Azure 备份可以保护运行 Windows 和 Linux 的虚拟机。

了解详细信息：

- [什么是 Azure 备份？](/documentation/articles/backup-introduction-to-azure-backup/)
- [Azure 备份服务 - 常见问题](/documentation/articles/backup-azure-backup-faq/)

## Azure Site Recovery

组织的 BCDR 策略的其中一个重要部分是，找出在发生计划的和非计划的中断时让企业工作负荷和应用保持启动并运行的方法。Azure Site Recovery 可以帮助协调工作负荷和应用的复制、故障转移及恢复，因此能够在主要位置发生故障时通过辅助位置来提供工作负荷和应用。

Site Recovery：

- **简化 BCDR 策略** — 使用 Site Recovery 可从单个位置轻松处理多个企业工作负荷和应用的复制、故障转移及恢复。Site Recovery 会协调复制和故障转移，但不会拦截应用程序数据或拥有任何相关信息。
- **提供灵活的复制** — 使用 Site Recovery，可以复制 Hyper-V 虚拟机、VMware 虚拟机和 Windows/Linux 物理服务器上运行的工作负荷。
- **支持故障转移和恢复** — Site Recovery 提供测试故障转移，既能支持灾难恢复练习，又不会影响生产环境。你也可以运行计划的故障转移，因为是预期中的中断，所以不会丢失任何数据；或是运行非计划的故障转移，以在发生非预期的灾难时将数据损失减到最少（取决于复制频率）。在故障转移之后，可以故障回复到主站点。Site Recovery 提供包含脚本和 Azure 自动化工作簿的恢复计划，以供你自定义多层应用程序的故障转移和恢复。
- **消除辅助数据中心** — 可以复制到辅助本地站点或 Azure。使用 Azure 作为灾难恢复的目标可以消除维护辅助站点所带来的成本和复杂性。复制的数据存储在 Azure 存储中。
- **与现有 BCDR 技术集成** — Site Recovery 能够与其他应用程序的 BCDR 功能搭配使用。例如，可以使用 Site Recovery 来保护企业工作负荷的 SQL Server 后端。这包括本机支持使用 SQL Server AlwaysOn 来管理可用性组的故障转移。

了解详细信息：

- [什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)
- [Azure Site Recovery 的工作原理](/documentation/articles/site-recovery-components/)
- [Azure Site Recovery 保护哪些工作负荷？](/documentation/articles/site-recovery-workload/)

## 虚拟网络

虚拟机需要网络连接。为了满足该要求，Azure 需要虚拟机连接到 Azure 虚拟网络。Azure 虚拟网络是构建在物理 Azure 网络结构基础之上的逻辑构造。每个逻辑 Azure 虚拟网络与其他所有 Azure 虚拟网络隔离。这种隔离有助于确保其他 Azure 客户无法访问部署中的网络流量。

了解详细信息：

- [Azure 网络安全概述](/documentation/articles/security-network-overview/)
- [虚拟网络概述](/documentation/articles/virtual-networks-overview/)


## 合规性

世纪互联运营的 Azure 虚拟机已通过 ISO/IEC 20000/27001、DJCP、Trusted Cloud Service Certification、GB18030 及其他重要合规计划的认证。由于通过了这些认证，你自己的 Azure 应用程序更容易符合法规请求，对于企业而言，可以解决各种国内与国际法规要求。

了解详细信息：

- [信任中心：合规性](https://www.trustcenter.cn/zh-cn/compliance/default.html)

<!---HONumber=Mooncake_1024_2016-->