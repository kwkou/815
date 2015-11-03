<properties
	pageTitle="Azure 备份简介 | Windows Azure"
	description="本文概述了可让客户将数据备份到 Azure 以及备份 Azure 中数据的 Azure 备份服务"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.date="08/18/2015"
	wacn.date="11/02/2015"/>

# Azure 备份简介
本文全面概述了可让客户备份其本地数据或 Azure 中数据的 Microsoft 云集成备份解决方案。

## 什么是 Azure 备份？
Azure 备份是一个多租户 Azure 服务，可让你备份任意位置（本地或 Azure 中）的数据。它取代了现有的本地或异地备份解决方案，并且是可靠、安全、高性价比的基于云的产品。它还提供了保护云中运行的资产的弹性。Azure 备份构建在一流的基础结构之上，具有可缩放、持久且高度可用的优点。使用此解决方案，你可以从 System Center Data Protection Manager (SCDPM) 服务器、Windows 服务器、Windows 客户端计算机或 Azure IaaS 虚拟机备份数据与应用程序。Azure 备份和 SCDPM 是构成 Microsoft 云集成备份解决方案的基本技术。

## 云设计点
传统的备份解决方案已演变成将云视为类似于磁盘或磁带的终结点。尽管这种方法简单、易于部署且提供一致的体验，但它的用途有限，并且不能充分利用基础平台。这对于最终客户而言，就是一个效率低下而昂贵的解决方案。如果将 Azure 看作"无非就是一个存储终结点"，那么，备份解决方案将无法发挥功能丰富性和公共云平台的强大功能。在另一方面，Azure 备份提供的真正服务使用云结构来提供强大且经济实惠的备份解决方案。它与在本地备份解决方案 (SCDPM) 集成，以提供端到端混合解决方案。

这种方法的优点是：

+ 高效的云存储体系结构提供低成本弹性数据存储

+ 无干扰自动缩放的服务提供高可用性保证

+ 针对本地、混合和 IaaS 部署提供一致的备份方式

此解决方案的重要特性包括：

1. **可靠的服务**：如果采用 Azure 备份，你将获得高度可用的备份服务。该服务是多租户的，你无需担心如何管理基础计算或存储资源。

2. **可靠的存储**：Azure 备份构建在可靠的云存储之上，有很高的 SLA 作为后盾。你无需维护存储所产生的资本性开支或运营开支。此外，你还可以选择备份到 LRS（本地冗余存储）或 GRS（地域复制存储）。同时，LRS 在同一地域启用 3 个数据副本，可以防范本地硬件故障；GRS 在配对的数据中心提供 3 个附加副本（总共 6 个副本）。这可以确保你的备份数据高度可用。即使发生 Azure 站点级的灾难，备份数据也能安全保存。

3. **安全**：Azure 备份数据在传输过程中会在源头加密，在 Azure 中存储时同样也会加密。加密密钥存储在源中，永远不会传输或存储到 Azure。还原任何数据都要使用加密密钥，只有客户才对服务中的数据拥有完全访问权限。

4. **场外保护**：备份到 Azure 后，客户无需支付场外磁带备份解决方案的费用，Azure 以极低的价格提供类似于磁带语义的有吸引力解决方案。

5. **简单**：Azure 备份提供熟悉的界面，其容量可以缩放以保护任何大小的部署。随着服务的发展，你可以使用中心管理等功能，从单个位置管理你的备份基础结构。

6. **经济高效**：Azure 备份的价格包括每个实例的备份管理费用，以及在 Azure 上使用的存储的费用（块 Blob 价格）。与其他基于云的备份产品不同，Azure 备份不向客户收取任何还原操作的费用。此外，在还原操作过程中，客户无需支付任何传出（出站）数据传输费用。

7. **云中备份**：Azure 备份为运行中的 Azure IaaS 虚拟机提供基于 VSS 的应用程序一致性备份，并且备份时无需关闭虚拟机。它还可以备份 Azure 中的 Linux 虚拟机，并保证文件系统一致性。


## 应用程序和工作负荷

| 工作负载 | 源计算机 | Azure 备份解决方案 |
| --- | --- |---|
| 文件和文件夹 | Windows Server、Windows 客户端 | Azure 备份代理 |
| 文件和文件夹 | Windows Server、Windows 客户端 | System Center DPM |
| Hyper-V 虚拟机 (Windows) | Windows Server | System Center DPM |
| Hyper-V 虚拟机 (Linux) | Windows Server | System Center DPM |
| Microsoft SQL Server | Windows Server | System Center DPM |
| Microsoft SharePoint | Windows Server | System Center DPM |
| Microsoft Exchange | Windows Server | System Center DPM |
| Azure IaaS VMs (Windows)| - | Azure 备份 | | Azure IaaS VMs (Linux) | - | Azure 备份 |

## 后续步骤
- [尝试 Azure 备份](/documentation/articles/backup-try-azure-backup-in-10-mins)
- [此处](/documentation/articles/backup-azure-backup-faq)列出了有关 Azure 备份服务的常见问题。
- 访问 [Azure 备份论坛](https://social.msdn.microsoft.com/forums/azure/zh-cn/home?forum=windowsazureonlinebackup)。

<!---HONumber=56-->