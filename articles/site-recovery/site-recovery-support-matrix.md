<properties
    pageTitle="Azure Site Recovery 支持矩阵 | Azure"
    description="汇总了 Azure Site Recovery 支持的操作系统和组件"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="" />  

<tags
    ms.assetid="1bbcc13c-ea21-4349-9ddf-0d7dfdcdcbfb"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="12/04/2016"
    wacn.date="01/11/2017"
    ms.author="raynew" />

# Azure Site Recovery 支持矩阵

本文汇总了 Azure Site Recovery 支持的操作系统和组件。每篇相应部署文章中均提供有每种可用部署方案支持的组件和先决条件的列表，本文档是对这些内容的汇总。

## 支持 Azure 复制方案

**部署** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
---  | --- | ---
**经典门户** |  仅限维护模式。 | 仅限维护模式。
**PowerShell** | 支持 | 支持


## 支持辅助站点复制方案

**部署** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
**经典门户** | 仅限维护模式。 | 仅限维护模式。
**PowerShell**  | 不可用 | 支持



## 虚拟化服务器操作系统支持

### 主机服务器（复制到 Azure）

**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | ---
Windows Server 2016、Windows Server 2012 R2（含最新更新）<br/><br/> 目前不支持混合使用主机（运行 Windows Server 2016 和 2012 R2）的 Hyper-V 站点。 | Windows Server 2016、Windows Server 2012 R2（含最新更新）<br/><br/> Windows Server 2016 主机应由运行 System Center 2016 的 VMM 托管。<br/><br/> 目前不支持混合使用 Windows Server 2016 和 2012 R2 主机的 VMM 2016 云。

### 主机服务器（复制到辅助站点）

**VMware VM/物理服务器** | **Hyper-V（包含 VMM）**
--- | --- |
vCenter 5.5 版或 6.0 版（仅支持 5.5 版功能）<br/><br/>具有最新更新的 vSphere 6.0 版、5.5 版或 5.1 版 | Windows Server 2016、Windows Server 2012 R2 或 Windows Server 2012（含最新更新）。<br/><br/> Windows Server 2016 主机应由运行 System Center 2016 的 VMM 托管。<br/><br/> 目前不支持混合使用 Windows Server 2016 和更低版主机的 VMM 2016 云。


##<a name="support-for-replicated-machines"></a> 复制的计算机支持

### 计算机（复制到 Azure）

虚拟机必须满足 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。

**要求** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
复制内容 |  任何工作负荷 | 任何工作负荷
主机 OS |  [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS | [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS


### 计算机（复制到辅助站点）

**要求** | **Hyper-V（包含 VMM）**
--- | ---
复制内容 |任何工作负荷
主机 OS |  [Hyper-V 支持的任何来宾 OS](https://technet.microsoft.com/zh-cn/library/mt126277.aspx)


## 提供程序和代理支持

**名称** | **说明** | **支持** | **详细信息**
---|---|---| ---
**Azure Site Recovery 提供程序** | <p>协调本地服务器与 Azure/辅助站点之间的通信 </p><p> 安装在本地 VMM 服务器或 Hyper-V 服务器（如果没有 VMM 服务器）上</p> | [在线提交工单](/support/support-ticket-form/?l=zh-cn) | [最新功能和修复](https://support.microsoft.com/zh-cn/kb/3155002)
**Azure 恢复服务 (MARS) 代理** | 协调 Hyper-V 虚拟机和 Azure 之间的复制<br/><br/>安装在本地 Hyper-V 服务器（具有或不具有 VMM 服务器）上 | |

## 网络支持

### 网络（复制到 Azure）

**主机网络** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- |--- | ---
NIC 组合 |  是 | 是
VLAN |  是 | 是
IPv4 |  是 | 是
IPv6 | 否 | 否

**来宾 VM 网络** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 否 | 否
IPv4 |是 | 是
IPv6 |  否 | 否
静态 IP (Windows) |  是 | 是
静态 IP (Linux) |  否 | 否
多 NIC |  是 | 是

**Azure 网络** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
Express Route |  是 | 是
ILB | 是 | 是
ELB | |
流量管理器 | 是 | 是
多 NIC | 是 | 是
保留 IP | 是 | 是
IPv4 | 是 | 是
保留源 IP | 是 | 是

### 网络（复制到辅助站点）

**主机网络** | **VMware/物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 是 | 是
VLAN | 是 | 是
IPv4 | 是 | 是
IPv6 | 否 | 否

**来宾 VM 网络** |**Hyper-V（包含 VMM）**
--- | ---
NIC 组合 |否
IPv4 | 是
IPv6 | 否
静态 IP (Windows) | 是
静态 IP (Linux) | 是
多 NIC |  是


## 存储支持

### 复制到 Azure

**存储（主机）** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- |  --- | ---
NFS | 不可用 | 不可用
SMB 3.0 | 是 | 是
SAN (ISCSI) |是 | 是
多路径 (MPIO) | 是 | 是

**存储（来宾 VM/物理服务器）** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
VMDK |不可用 | 不可用
VHD/VHDX | 是 | 是
第 2 代 VM | 是 | 是
共享群集磁盘 | 否 | 否
加密磁盘 |否 | 否
EFI/UEFI|  是 | 是
NFS | 否 | 否
SMB 3.0 | 否 | 否
RDM |  不可用 | 不可用
磁盘 > 1 TB |否 | 否
包含条带化磁盘的卷 > 1 TB<br/><br/> LVM | 是 | 是
存储空间 |是 | 是
热添加/移除磁盘 |否 | 否
排除磁盘 |  否 | 否
多路径 (MPIO) |  是 | 是

**Azure 存储** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
LRS | 是 | 是
GRS | 是 | 是
冷存储 |否 | 否
热存储| 否 | 否
静态加密 | 是 | 是
高级存储 | 否 | 否
导入/导出服务 | 否 | 否


### 存储（复制到辅助站点）

**存储（主机）** |  **Hyper-V（包含 VMM）**
--- | ---
NFS |不可用
SMB 3.0 | 是
SAN (ISCSI) | 是
多路径 (MPIO) | 是

**存储（来宾 VM/物理服务器）** | **Hyper-V（包含 VMM）**
--- | ---
VMDK | 不可用
VHD/VHDX | 是（最多 64 个磁盘）
第 2 代 VM | 是
共享群集磁盘 | 否
加密磁盘 |  否
UEFI| 不可用
NFS | 否
SMB 3.0 | 否
RDM | 不可用
磁盘 > 1 TB |是
包含条带化磁盘的卷 > 1 TB<br/><br/> LVM | 是
存储空间 |是
热添加/移除磁盘 | 否
排除磁盘 |  否
多路径 (MPIO) | 是

## 恢复服务保管库操作支持

### 保管库（复制到 Azure）

**操作** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
跨资源组移动保管库<br/><br/> 订阅内和跨订阅移动 |否 | 否
跨资源组移动存储、网络和 Azure VM<br/><br/> 订阅内和跨订阅移动 | 否 | 否

### 保管库（复制到辅助站点）

**操作** | **VMware/物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
跨资源组移动保管库<br/><br/> 订阅内和跨订阅移动 | 否 | 否
跨资源组移动存储、网络和 Azure VM<br/><br/> 订阅内和跨订阅移动 | 否 | 否


## Azure 计算支持（复制到 Azure）

**计算功能** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
共享磁盘来宾群集 | 否 | 否
可用性集 |  否 | 否
HUB | 是 | 是

## 支持 Azure VM

可以部署 Site Recovery 以复制运行受 Azure 支持的任何操作系统的虚拟机和物理服务器。这包括大多数的 Windows 和 Linux 版本。要复制的本地 VM 必须符合 Azure 要求。

**功能** | **要求** | **详细信息**
--- | --- | ---
**Hyper-V 主机** | 应运行 Windows Server 2012 R2 或更高版本 | 如果操作系统不受支持，先决条件检查将会失败
**来宾操作系统** | 对于从 Hyper-V 到 Azure 的复制，Site Recovery 支持 [Azure 支持](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的所有操作系统。 | 如果不支持，先决条件检查将会失败。
**来宾操作系统体系结构** | 64 位 | 如果不支持，先决条件检查将会失败
**操作系统磁盘大小** | 最大 1023 GB | 如果不支持，先决条件检查将会失败
**操作系统磁盘计数** | 1 | 如果不支持，先决条件检查将会失败。
**数据磁盘计数** | 16 或更少（最大值取决于所创建的虚拟机大小。16 = XL） | 如果不支持，先决条件检查将会失败
**数据磁盘 VHD 大小** | 最大 1023 GB | 如果不支持，先决条件检查将会失败
**网络适配器** | 支持多个适配器 |
**静态 IP 地址** | 支持 | 如果主虚拟机使用的是静态 IP 地址，则可为要在 Azure 中创建的虚拟机指定静态 IP 地址。<br/><br/> **在 Hyper-V 上运行的 Linux VM** 不支持静态 IP 地址。
**iSCSI 磁盘** | 不支持 | 如果不支持，先决条件检查将会失败
**共享 VHD** | 不支持 | 如果不支持，先决条件检查将会失败
**FC 磁盘** | 不支持 | 如果不支持，先决条件检查将会失败
**硬盘格式** | VHD <br/><br/> VHDX | 尽管 Azure 目前不支持 VHDX，但当你故障转移到 Azure 时，Site Recovery 会自动将 VHDX 转换为 VHD。当你故障回复到本地时，虚拟机将继续使用 VHDX 格式。
**Bitlocker** | 不支持 | 保护虚拟机之前，必须先禁用 Bitlocker。
**VM 名称** | 介于 1 和 63 个字符之间。限制为字母、数字和连字符。应以字母或数字开头和结尾 | 在 Site Recovery 中更新虚拟机属性中的值
**VM 类型** | 第 1 代<br/><br/> 第 2 代 - Windows | OS 磁盘类型为“基本”的第 2 代 VM，其中包括一个或两个格式化为 VHDX 的数据卷，并且支持的大小小于 300 GB。<br/><br/>.不支持 Linux 第 2 代 VM。[了解详细信息](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/) |








## 后续步骤

[准备部署](/documentation/articles/site-recovery-best-practices/)

<!---HONumber=Mooncake_1226_2016-->