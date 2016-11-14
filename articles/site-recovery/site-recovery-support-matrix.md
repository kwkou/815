<properties
	pageTitle="Azure Site Recovery 支持矩阵 | Azure"
	description="汇总了 Azure Site Recovery 支持的操作系统和组件"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>  


<tags
	ms.service="site-recovery"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery"
	ms.date="10/05/2016"
	wacn.date="11/14/2016"
	ms.author="raynew"/>

# Azure Site Recovery 支持矩阵

本文汇总了 Azure Site Recovery 部署支持的操作系统和组件。每篇相应部署文章中均提供有每种可用部署方案支持的组件和先决条件的列表，本文档是对这些内容的汇总。

## 虚拟化服务器支持的操作系统


**源** | **目标** | **主机 OS**
---|---|---|--- 
**Hyper-V 主机（不包含 VMM）** | Azure | 带有最新更新的 Windows Server 2012 R2
**Hyper-V 主机（包含 VMM）** | Azure | 带有最新更新的 Windows Server 2012 R2
**Hyper-V 主机（包含 VMM）** | 辅助 VMM 站点 | 至少为装有最新更新的 Windows Server 2012


## 复制计算机的支持要求

**源** | **复制内容** | **目标** | **主机 OS**
---|---|---|--- 
**Hyper-V VM** | 任何工作负荷 | Azure | [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS<br/><br/> VM 必须满足 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)
**Hyper-V VM（包含 VMM）** | 任何工作负荷 | Azure | [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS<br/><br/> VM 必须满足 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)
**Hyper-V VM（包含 VMM）** | 任何工作负荷 | 辅助 VMM 站点 | [Hyper-V 支持的](https://technet.microsoft.com/zh-cn/library/mt126277.aspx)任何来宾 OS
**物理服务器** | 在 Windows 上运行的任何工作负荷 | Azure | 64 位 Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 SP1 及其更高版本
**物理服务器** | 在 Linux 上运行的任何工作负荷 | Azure | Red Hat Enterprise Linux 6.7、7.1、7.2 <br/><br/> Centos 6.5、6.6、6.7、7.0、7.1、7.2 <br/><br/> Oracle Enterprise Linux 6.4、6.5（运行 Red Hat 兼容内核或 Unbreakable Enterprise Kernel Release 3 (UEK3)）<br/><br/> SUSE Linux Enterprise Server 11 SP3 <br/><br/> 所需的存储：文件系统 (EXT3, ETX4, ReiserFS, XFS)；多路径软件设备映射器 (multipath))；卷管理器：(LVM2)。不支持使用 HP CCISS 控制器存储的物理服务器。ReiserFS 文件系统仅在 SUSE Linux Enterprise Server 11 SP3 上受支持。
**物理服务器** | 在 Windows 上运行的任何工作负荷 | 辅助站点 | 64 位 Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 SP1 及其更高版本
**物理服务器** | 在 Linux 上运行的任何工作负荷 | 辅助站点 | Red Hat Enterprise Linux 6.7、7.1、7.2 <br/><br/> Centos 6.5、6.6、6.7、7.0、7.1、7.2 <br/><br/> Oracle Enterprise Linux 6.4、6.5（运行 Red Hat 兼容内核或 Unbreakable Enterprise Kernel Release 3 (UEK3)）<br/><br/> SUSE Linux Enterprise Server 11 SP3 <br/><br/> 所需的存储：文件系统 (EXT3, ETX4, ReiserFS, XFS)；多路径软件设备映射器 (multipath))；卷管理器：(LVM2)。不支持使用 HP CCISS 控制器存储的物理服务器。ReiserFS 文件系统仅在 SUSE Linux Enterprise Server 11 SP3 上受支持。


## 提供程序版本

**名称** | **说明** | **最新版本** | **支持** | **详细信息**
---|---|---|---| ---
**Azure Site Recovery 提供程序** | 协调本地服务器与 Azure/辅助站点之间的通信 <br/><br/> 安装在本地 VMM 服务器或 Hyper-V 服务器（如果没有 VMM 服务器）上 | 5\.1.1700（可从门户获取） | [最新功能和修复](https://support.microsoft.com/zh-cn/kb/3155002)
**移动服务** | 协调本地物理服务器和 Azure/辅助站点之间的复制 | NA（可从门户获取） | 在想要复制的每个物理服务器上安装。**Azure 恢复服务 (MARS) 代理** | 协调 Hyper-V 虚拟机和 Azure 之间的复制<br/><br/>安装在本地 Hyper-V 服务器（具有或不具有 VMM 服务器）上 | 2\.0.8689.0（可从门户获取） | 该代理由 Azure Site Recovery 和 Azure 备份使用）。[了解详细信息](https://support.microsoft.com/zh-cn/kb/2997692)

## 后续步骤

[准备部署](/documentation/articles/site-recovery-best-practices/)

<!---HONumber=Mooncake_1107_2016-->