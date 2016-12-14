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
    ms.date="11/20/2016"
    wacn.date="12/12/2016"
    ms.author="raynew" />

# Azure Site Recovery 支持矩阵

本文汇总了 Azure Site Recovery 支持的操作系统和组件。每篇相应部署文章中均提供有每种可用部署方案支持的组件和先决条件的列表，本文档是对这些内容的汇总。

## 虚拟化服务器操作系统支持


**源** | **目标** | **主机 OS**
---|---|---|--- 
**Hyper-V 主机（不包含 VMM）** | Azure | 带有最新更新的 Windows Server 2012 R2
**Hyper-V 主机（包含 VMM）** | Azure | 带有最新更新的 Windows Server 2012 R2
**Hyper-V 主机（包含 VMM）** | 辅助 VMM 站点 | 至少为装有最新更新的 Windows Server 2012


## 复制的计算机支持

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

**主机网络** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 是 | 是
VLAN | 是 | 是
IPv4 | 是 | 是
IPv6 | 否 | 否

**来宾 VM 网络** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 否 | 否
IPv4 | 是 | 是
IPv6 | 否 | 否
静态 IP (Windows) | 是 | 是
静态 IP (Linux) | 否 | 否
多 NIC | 是 | 是

**Azure 网络** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
Express Route | 是 | 是
ILB |是 | 是
ELB |  |
流量管理器 |是 | 是
多 NIC | 是 | 是
保留 IP | 是 | 是
IPv4 | 是 | 是
保留源 IP | 是 | 是

### 网络（复制到辅助站点）

**主机网络**  | **Hyper-V（包含 VMM）**
--- |---
NIC 组合 | 是
VLAN | 是
IPv4 | 是
IPv6 | 否

**来宾 VM 网络**| **Hyper-V（包含 VMM）**
--- |---
NIC 组合 | 否
IPv4 | 是
IPv6 | 否
静态 IP (Windows) | 是
静态 IP (Linux) | 是
多 NIC | 是


## 存储支持

### 复制到 Azure

**存储（主机）** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- |--- | ---
NFS | 不可用 | 不可用
SMB 3.0 | 是 | 是
SAN (ISCSI) | 是 | 是
多路径 (MPIO) | 是 | 是

**存储（来宾 VM/物理服务器）** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
VMDK |不可用 | 不可用
VHD/VHDX |是 | 是
第 2 代 VM | 是 | 是
共享群集磁盘 | 否 | 否
加密磁盘 |否 | 否
EFI/UEFI|是 | 是
NFS |否 | 否
SMB 3.0 |否 | 否
RDM |不可用 | 不可用
磁盘 > 1 TB |否 | 否
包含条带化磁盘的卷 > 1 TB<br/><br/> LVM |是 | 是
存储空间 |是 | 是
热添加/移除磁盘 |否 | 否
排除磁盘 |否 | 否
多路径 (MPIO) |是 | 是

**Azure 存储** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- |--- | ---
LRS |是 | 是
GRS |是 | 是
冷存储 |否 | 否
热存储|否 | 否
静态加密 |是 | 是
高级存储 |否 | 否
导入/导出服务 |否 | 否


### 存储（复制到辅助站点）

**存储（主机）** |**Hyper-V（包含 VMM）**
--- |---
NFS |不可用
SMB 3.0 |是
SAN (ISCSI) |是
多路径 (MPIO) |是

**存储（来宾 VM/物理服务器）** |**Hyper-V（包含 VMM）**
--- |---
VMDK |不可用
VHD/VHDX |是（最多 64 个磁盘）
第 2 代 VM |是
共享群集磁盘 |否
加密磁盘 |否
UEFI|不可用
NFS |否
SMB 3.0 |否
RDM |不可用
磁盘 > 1 TB |是
包含条带化磁盘的卷 > 1 TB<br/><br/> LVM |是
存储空间 |是
热添加/移除磁盘 |否
排除磁盘 |否
多路径 (MPIO) |是

## 恢复服务保管库操作支持

### 保管库（复制到 Azure）

**操作** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | ---
跨资源组移动保管库<br/><br/> 订阅内和跨订阅移动 | 否 | 否
跨资源组移动存储、网络和 Azure VM<br/><br/> 订阅内和跨订阅移动 |否 | 否

### 保管库（复制到辅助站点）

**操作** |**Hyper-V（包含 VMM）**
--- | ---
跨资源组移动保管库<br/><br/> 订阅内和跨订阅移动 | 否
跨资源组移动存储、网络和 Azure VM<br/><br/> 订阅内和跨订阅移动 | 否


## Azure 计算支持（复制到 Azure）

**计算功能** |**Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- |--- | ---
共享磁盘来宾群集 |否 | 否
可用性集 |否 | 否
HUB | 是 | 是









## 后续步骤

[准备部署](/documentation/articles/site-recovery-best-practices/)

<!---HONumber=Mooncake_1205_2016-->