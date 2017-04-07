<properties
    pageTitle="使用 Azure Site Recovery 复制到辅助站点时的支持矩阵 | Azure"
    description="汇总了 Azure Site Recovery 支持的操作系统和组件"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="" />
<tags
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="02/08/2017"
    wacn.date="03/03/2017"
    ms.author="raynew" />  


# 使用 Azure Site Recovery 复制到辅助站点时的支持矩阵
>[AZURE.SELECTOR]
- [复制到 Azure](/documentation/articles/site-recovery-support-matrix-to-azure/)
- [复制到本地位置](/documentation/articles/site-recovery-support-matrix-to-sec-site/)

本文总结了使用 Azure Site Recovery 复制到辅助本地站点时受到支持的功能。

## 部署选项

**部署** | **物理服务器** | **Hyper-V（不包含 VMM）** | **Hyper-V（包含 VMM）**
--- | --- | --- | ---
**经典管理门户** | 仅限维护模式。无法创建新的保管库。 | 不支持 | 仅限维护模式
**PowerShell** | 不支持 | 不支持 | 支持

##<a name="on-premises-servers"></a> 本地服务器

### 虚拟化服务器

**部署** | **支持**
--- | ---
**Hyper-V（包含 VMM）** | VMM 2016 和 VMM 2012 R2

  >[AZURE.NOTE]
  目前不支持混合使用 Windows Server 2016 和 2012 R2 主机的 VMM 2016 云。

### 主机服务器

**部署** | **支持**
--- | ---
**Hyper-V（不包含 VMM）** | 不是复制到辅助站点时的受支持配置
**Hyper-V（有 VMM）** |<p> Windows Server 2016 和具有最新更新的 Windows Server 2012 R2。</p><p> Windows Server 2016 主机应由 VMM 2016 管理。</p>

##<a name="support-for-replicated-machine-os-versions"></a> 复制计算机 OS 版本支持
下表总结了使用 Azure Site Recovery 时遇到的各种部署方案中的操作系统支持。此支持适用于在所述的 OS 上运行的任何工作负荷。

**物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
<p>64 位 Windows Server 2012 R2、Windows Server 2012、至少具有 SP1 的 Windows Server 2008 R2</p><p>Red Hat Enterprise Linux 6.7、7.1、7.2</p><p>CentOS 6.5、6.6、6.7、7.0、7.1、7.2</p><p>运行 Red Hat 兼容内核或 Unbreakable Enterprise Kernel Release 3 (UEK3) 的 Oracle Enterprise Linux 6.4 或 6.5</p><p>SUSE Linux Enterprise Server 11 SP3 </p>| [Hyper-V 支持的](https://technet.microsoft.com/zh-cn/library/mt126277.aspx)任何来宾操作系统

>[AZURE.NOTE]
仅可复制具有以下存储的 Linux 计算机：
文件系统（EXT3、ETX4、ReiserFS、XFS）；
多路径软件设备映射器；
卷管理器 (LVM2)。
不支持使用 HP CCISS 控制器存储的物理服务器。
ReiserFS 文件系统仅在 SUSE Linux Enterprise Server 11 SP3 上受支持。

## 网络配置

### 主机

**配置** | **物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 是 | 是
VLAN | 是 | 是
IPv4 | 是 | 是
IPv6 | 否 | 否

### 来宾 VM

**配置** | **物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
NIC 组合 | 否 | 否
IPv4 | 是 | 是
IPv6 | 否 | 否
静态 IP (Windows) | 是 | 是
静态 IP (Linux) | 是 | 是
多 NIC | 是 | 是


## 存储

### 主机存储

**存储（主机）** | **物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
NFS | 是 | 不适用
SMB 3.0 | 不适用 | 是
SAN (ISCSI) | 是 | 是
多路径 (MPIO) | 是 | 是

### 来宾或物理服务器存储

**配置** | **物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
VMDK | 是 | 不适用
VHD/VHDX | 不适用 | 是（最多 16 个磁盘）
第 2 代 VM | 不适用 | 是
共享群集磁盘 | 是 | 否
加密磁盘 | 否 | 否
UEFI| 否 | 不适用
NFS | 否 | 否
SMB 3.0 | 否 | 否
RDM | 是 | 不适用
磁盘 > 1 TB | 否 | 是
<p>包含条带化磁盘的卷 > 1 TB</p><p> LVM </p>| 是 | 是
存储空间 | 否 | 是
热添加/移除磁盘 | 否 | 否
排除磁盘 | 否 | 否
多路径 (MPIO) | 不适用 | 是

## 保管库

**操作** | **物理服务器** | **Hyper-V（包含 VMM）**
--- | --- | ---
跨资源组移动保管库（订阅内或跨订阅） | 否 | 否
跨资源组移动存储、网络、Azure VM（订阅内或跨订阅） | 否 | 否

## 提供程序和代理

**名称** | **说明** | **最新版本** | **详细信息**
--- | --- | --- | --- | ---
**Azure Site Recovery 提供程序** | <p>协调本地服务器与 Azure 之间的通信</p><p>安装在本地 VMM 服务器或 Hyper-V 服务器（如果没有 VMM 服务器）上 </p>| 5\.1.19（[可从门户获取](http://aka.ms/downloaddra)） | [最新功能和修复](https://support.microsoft.com/zh-cn/kb/3155002)
**移动服务** | <p>协调本地物理服务器与辅助站点之间的复制</p><p>在想要复制的物理服务器上安装</p> | 不适用（可从门户获取） | 不适用


## 后续步骤

了解[部署先决条件](/documentation/articles/site-recovery-prereq/)。

<!---HONumber=Mooncake_0227_2017-->