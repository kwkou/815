<properties
    pageTitle="复制到 Azure 时的 Azure Site Recovery 支持矩阵 | Azure"
    description="汇总了 Azure Site Recovery 支持的操作系统和组件"
    services="site-recovery"
    documentationcenter=""
    author="Rajani-Janaki-Ram"
    manager="rochakm"
    editor="" />
<tags
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="01/25/2017"
    wacn.date="03/03/2017"
    ms.author="rajanaki" />  


# 复制到 Azure 时的 Azure Site Recovery 支持矩阵
>[AZURE.SELECTOR]
- [复制到 Azure](/documentation/articles/site-recovery-support-matrix-to-azure/)
- [复制到客户所有的辅助站点](/documentation/articles/site-recovery-support-matrix-to-sec-site/)

本文汇总了复制和恢复到 Azure 时 Azure Site Recovery 支持的配置和组件。有关 Azure Site Recovery 先决条件的详细信息，请参阅 [Site Recovery 最佳做法](/documentation/articles/site-recovery-best-practices/)。


## 部署选项支持

**部署** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
**经典管理门户** | 仅限维护模式。无法创建新的保管库。 | 仅限维护模式。 | 仅限维护模式。
**PowerShell** | 目前不支持。 | 支持 | 支持


## 数据中心管理服务器支持

### 虚拟化管理实体

**部署** | **支持**
--- | ---
**VMware VM/物理服务器** | vSphere 6.0、5.5 或具有最新更新的 5.1
**Hyper V（有 Virtual Machine Manager）** | System Center Virtual Machine Manager 2016 和 System Center Virtual Machine Manager 2012 R2

  >[AZURE.NOTE]目前不支持混合使用 Windows Server 2016 和 2012 R2 主机的 System Center Virtual Machine Manager 2016 云。

### 主机服务器

**部署** | **支持**
--- | ---
**VMware VM/物理服务器** | vCenter 5.5 或 6.0（仅支持 5.5 功能）
**Hyper V（无 Virtual Machine Manager）** | Windows Server 2016、具有最新更新的 Windows Server 2012 R2
**Hyper V（有 Virtual Machine Manager）** | Windows Server 2016、具有最新更新的 Windows Server 2012 R2。<br/><br/> Windows Server 2016 主机应由 System Center Virtual Machine Manager 2016 管理。

  >[AZURE.NOTE]目前不支持混合使用主机（运行 Windows Server 2016 和 2012 R2）的 Hyper-V 站点。

## 复制计算机 OS 版本支持

复制到 Azure 时，受保护的虚拟机必须符合 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。下表总结了使用 Azure Site Recovery 时的各种部署方案中的复制操作系统支持。此支持适用于在所述的 OS 上运行的任何工作负荷。

 物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | ---
<p>64 位 Windows Server 2012 R2、Windows Server 2012、至少具有 SP1 的 Windows Server 2008 R2</p><p>Red Hat Enterprise Linux 6.7、6.8、7.1、7.2</p><p>CentOS 6.5、6.6、6.7、6.8、7.0、7.1、7.2</p><p>运行 Red Hat 兼容内核或 Unbreakable Enterprise Kernel Release 3 (UEK3) 的 Oracle Enterprise Linux 6.4、6.5</p><p>SUSE Linux Enterprise Server 11 SP3 </p>| [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS | [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868.aspx)任何来宾 OS


>[AZURE.NOTE]
存储支持包括 Linux 版本文件系统（EXT3、ETX4、ReiserFS、XFS）、多路径软件设备映射器、卷管理器 (LVM2)，但使用 HP CCISS 控制器存储的物理服务器*不*受支持。ReiserFS 文件系统仅在 SUSE Linux Enterprise Server 11 SP3 上受支持。

## 网络配置支持
下表总结了使用 Azure Site Recovery 复制到 Azure 时的各种部署方案中的网络配置支持。

### 主机网络配置

**配置** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
NIC 组合 | <p>是</p><p>在物理机中不受支持</p>| 是 | 是
VLAN | 是 | 是 | 是
IPv4 | 是 | 是 | 是
IPv6 | 否 | 否 | 否

### 来宾 VM 网络配置

**配置** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
NIC 组合 | 否 | 否 | 否
IPv4 | 是 | 是 | 是
IPv6 | 否 | 否 | 否
静态 IP (Windows) | 是 | 是 | 是
静态 IP (Linux) | 否 | 否 | 否
多 NIC | 是 | 是 | 是

### 故障转移 Azure VM 网络配置

**Azure 网络** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
Express Route | 是 | 是 | 是
ILB | 是 | 是 | 是
ELB | 是 | 是 | 是
流量管理器 | 是 | 是 | 是
多 NIC | 是 | 是 | 是
保留 IP | 是 | 是 | 是
IPv4 | 是 | 是 | 是
保留源 IP | 是 | 是 | 是


## 存储支持
下表总结了使用 Azure Site Recovery 复制到 Azure 时的各种部署方案中的存储配置支持。

### 主机存储配置

**配置** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
NFS | 在物理服务器上不支持 | 不适用 | 不适用
SMB 3.0 | 不适用 | 是 | 是
SAN (ISCSI) | 是 | 是 | 是
多路径 (MPIO)<br></br>已使用 Microsoft DSM、EMC PowerPath 5.7 SP4、EMC PowerPath DSM for CLARiiON 进行测试 | 是 | 是 | 是

### 来宾或物理服务器存储配置

**配置** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
VMDK | 是 | 不适用 | 不适用
VHD/VHDX | 不适用 | 是 | 是
第 2 代 VM | 不适用 | 是 | 是
EFI/UEFI| 否 | 是 | 是
共享群集磁盘 | 在物理服务器上不适用 | 否 | 否
加密磁盘 | 否 | 否 | 否
NFS | 否 | 不适用 | 不适用
SMB 3.0 | 否 | 否 | 否
RDM | 在物理服务器上不适用 | 不适用 | 不适用
磁盘 > 1 TB | 否 | 否 | 否
具有 > 1 TB 的带区磁盘的卷<br/><br/>LVM 逻辑卷管理 | 是 | 是 | 是
存储空间 | 否 | 是 | 是
热添加/移除磁盘 | 否 | 否 | 否
排除磁盘 | 是 | 是 | 是
多路径 (MPIO) | 不适用 | 是 | 是

**Azure 存储** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
LRS | 是 | 是 | 是 
GRS | 是 | 是 | 是
冷存储 | 否 | 否 | 否
热存储| 否 | 否 | 否
静态加密 (SSE)| 是 | 是 | 是
高级存储 | 是 | 否 | 否
导入/导出服务 | 否 | 否 | 否


## Azure 计算配置支持

**计算功能** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
可用性集 | 否 | 否 | 否
HUB | 是 | 是 | 是

## 故障转移 Azure VM 要求

可以部署 Site Recovery 以复制运行受 Azure 支持的任何操作系统的虚拟机和物理服务器。这包括大多数的 Windows 和 Linux 版本。复制到 Azure 时，想要复制的本地 VM 必须符合以下 Azure 要求。

**实体** | **要求** | **详细信息**
--- | --- | ---
**来宾操作系统** | <p>对于从 Hyper-V 到 Azure 的复制，Site Recovery 支持 [Azure 支持](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的所有操作系统。</p><p> 对于物理服务器复制：请检查 Windows 和 Linux [先决条件](/documentation/articles/site-recovery-vmware-to-azure-classic/#before-you-start-deployment) | 如果不支持，先决条件检查将会失败。
**来宾操作系统体系结构** | 64 位 | 如果不支持，先决条件检查将会失败
**操作系统磁盘大小** | 最大 1023 GB | 如果不支持，先决条件检查将会失败
**操作系统磁盘计数** | 1 | 如果不支持，先决条件检查将会失败。
**数据磁盘计数** | 如果要**将 Hyper-V VM 复制到 Azure**，则 16 个或更少 | 如果不支持，先决条件检查将会失败
**数据磁盘 VHD 大小** | 最大 1023 GB | 如果不支持，先决条件检查将会失败
**网络适配器** | 支持多个适配器 |
**共享 VHD** | 不支持 | 如果不支持，先决条件检查将会失败
**FC 磁盘** | 不支持 | 如果不支持，先决条件检查将会失败
**硬盘格式** | <p>VHD </p><p> VHDX </p>| 尽管 Azure 目前不支持 VHDX，但当你故障转移到 Azure 时，Site Recovery 会自动将 VHDX 转换为 VHD。当你故障回复到本地时，虚拟机将继续使用 VHDX 格式。
**Bitlocker** | 不支持 | 保护虚拟机之前，必须先禁用 Bitlocker。
**VM 名称** | 介于 1 和 63 个字符之间。限制为字母、数字和连字符。VM 名称必须以字母或数字开头和结尾。 | 在 Site Recovery 中更新虚拟机属性中的值。
**VM 类型** | 第 1 代<br/><br/>第 2 代 - Windows |<p> OS 磁盘类型为“基本”的第 2 代 VM（其中包括一个或两个格式化为 VHDX 的数据卷），并且支持的磁盘大小小于 300 GB。</p><p>Linux 第 2 代 VM 不受支持。[了解详细信息](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)</p>|

## 恢复服务保管库操作支持

**操作** | **物理服务器** | **Hyper V（无 Virtual Machine Manager）** | **Hyper V（有 Virtual Machine Manager）**
--- | --- | --- | ---
<p>跨资源组移动保管库</p><p> 订阅内和跨订阅移动</p> | 否 | 否 | 否
<p>跨资源组移动存储、网络和 Azure VM</p><p> 订阅内和跨订阅移动 </p>| 否 | 否 | 否


## 提供程序和代理支持

**名称** | **说明** | **最新版本** | **详细信息**
--- | --- | --- | --- | ---
**Azure Site Recovery 提供程序** | <p>协调本地服务器与 Azure 之间的通信</p><p>安装在本地 Virtual Machine Manager 服务器或 Hyper-V 服务器（如果没有 Virtual Machine Manager 服务器）上</p> | 5\.1.19（[可从门户获取](http://aka.ms/downloaddra)） | [最新功能和修复](https://support.microsoft.com/zh-cn/kb/3155002)
**Azure 恢复服务 (MARS) 代理** | <p>协调 Hyper-V VM 与 Azure 之间的复制</p><p>安装在本地 Hyper-V 服务器（具有或不具有 Virtual Machine Manager 服务器）上</p> | 最新代理（[可从门户获取](http://aka.ms/latestmarsagent)） |






## 后续步骤
[准备部署](/documentation/articles/site-recovery-best-practices/)

<!---HONumber=Mooncake_0227_2017-->