<properties 
	pageTitle="Linux 代理用户指南 | Azure" 
	description="了解如何安装和配置 Linux 代理 (waagent) 以管理虚拟机与 Azure 结构控制器的交互。" 
	services="virtual-machines-linux" 
	documentationCenter="" 
	authors="szarkos" 
	manager="timlt" 
	editor=""
	tags="azure-service-management,azure-resource-manager" />  


<tags 
	ms.service="virtual-machines-linux" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="vm-linux" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/17/2016" 
	wacn.date="11/21/2016" 
	ms.author="szark"/>



# Azure Linux 代理用户指南

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## 介绍

Azure Linux 代理 (waagent) 可以管理 Linux 与 FreeBSD 预配，以及 VM 与 Azure 结构控制器之间的交互。它针对 Linux 和 FreeBSD IaaS 部署提供以下功能：

> [AZURE.NOTE] 有关其他详细信息，请参阅 Azure Linux 代理的[自述文件](https://github.com/Azure/WALinuxAgent/blob/master/README.md)。

* **映像设置**
  - 创建用户帐户
  - 配置 SSH 身份验证类型
  - 部署 SSH 公钥和密钥对
  - 设置主机名
  - 将主机名发布到平台 DNS
  - 将 SSH 主机密钥指纹报告给平台
  - 资源磁盘管理
  - 格式化并安装资源磁盘
  - 配置交换空间

* **网络**
  - 管理路由以提高与平台 DHCP 服务器的兼容性
  - 确保网络接口名称的稳定性

* **内核**
  - 配置虚拟 NUMA（版本低于 2.6.37 的内核已禁用）
  - 将 Hyper-V 熵用于 /dev/random
  - 为根设备配置 SCSI 超时（可能通过远程方式）

* **Diagnostics**
  - 控制台重定向到串行端口

* **SCVMM 部署**
    - 当用于 Linux 的 VMM 代理在 System Center Virtual Machine Manager 2012 R2 环境中运行时对其进行检测并启动

* **VM 扩展**
    - 将 Microsoft 和合作伙伴授权的组件注入 Linux VM (IaaS)，以实现软件和配置的自动化
    - [https://github.com/Azure/azure-linux-extensions](https://github.com/Azure/azure-linux-extensions) 上的 VM 扩展引用实现


## 通信
从平台到代理的信息流通过两个通道进行：

* 用于 IaaS 部署的附加了启动时间的 DVD。此 DVD 包含一个与 OVF 兼容的配置文件，该文件包括除实际的 SSH 密钥对之外的所有预配信息。
* 用于获取部署和拓扑配置的一个公开 REST API 的 TCP 终结点。


## 要求
以下系统已经过测试并确认兼容 Azure Linux 代理：

> [AZURE.NOTE] 此列表可能不同于 Azure Platform 所支持系统的官方列表，详见以下网页：[http://support.microsoft.com/kb/2805216](http://support.microsoft.com/kb/2805216)

* CoreOS
* CentOS 6.3+
* Red Hat Enterprise Linux 6.7+
* Debian 7.0+
* Ubuntu 12.04+
* openSUSE 12.3+
* SLES 11 SP3+
* Oracle Linux 6.4+

其他受支持的系统：

* FreeBSD 10+（Azure Linux 代理 v2.0.10+）

正常运行 Linux 代理所依赖的一些系统包：

* Python 2.6+
* OpenSSL 1.0+
* OpenSSH 5.3+
* 文件系统实用程序：sfdisk、fdisk、mkfs、parted
* 密码工具：chpasswd、sudo
* 文本处理工具：sed、grep
* 网络工具：ip-route
* 装载 UDF 文件系统的内核支持。

## 安装
使用分发的包存储库中的 RPM 或 DEB 包进行安装是安装和升级 Azure Linux 代理的首选方法。所有[认可的分发版提供商](/documentation/articles/virtual-machines-linux-endorsed-distros/)会将 Azure Linux 代理包集成到其映像和存储库。

请参阅 [Github 上的 Azure Linux 代理存储库](https://github.com/Azure/WALinuxAgent)中的文档了解高级安装选项，例如从源安装，或者安装到自定义位置或前缀。


## <a name="command-line-options"></a>命令行选项

### 标志

- verbose：增加指定命令的详细程度
- force：跳过某些命令的交互式确认

### 命令

- help：列出支持的命令和标志。

- deprovision：尝试清除系统并使其能够进行重新预配。此操作已删除以下各项：
 * 所有 SSH 主机密钥（如果在配置文件中 Provisioning.RegenerateSshHostKeyPair 为“y”）
 * /etc/resolv.conf 中的 Nameserver 配置
 * /etc/shadow 中的根密码（如果在配置文件中 Provisioning.DeleteRootPassword 为“y”）
 * 缓存的 DHCP 客户端租用
 * 将主机名重置为 localhost.localdomain


> [AZURE.WARNING] 取消预配无法保证清除映像中的所有敏感信息且适用于分发版。


- deprovision+user：执行 -deprovision（上述）下的所有操作，还将删除最后预配的用户帐户（获取自 /var/lib/waagent 中）和关联数据。此参数是取消对以前在 Azure 中预配的映像预配，以捕获并重新使用该映像时的参数。

- version：显示 waagent 的版本

- serialconsole：配置 GRUB 以将 ttyS0（第一个串行端口）标记为启动控制台。这可确保将内核启动日志发送到串行端口，并使其能够用于调试。

- daemon：将 waagent 作为 daemon 运行以管理与平台的交互。在 waagent init 脚本中为 waagent 指定此参数。

- start：将 waagent 作为后台进程运行


## 配置

配置文件 (/etc/waagent.conf) 可控制 waagent 的操作。下面显示了示例配置文件：

	Provisioning.Enabled=y
	Provisioning.DeleteRootPassword=n
	Provisioning.RegenerateSshHostKeyPair=y
	Provisioning.SshHostKeyPairType=rsa
	Provisioning.MonitorHostName=y
	Provisioning.DecodeCustomData=n
	Provisioning.ExecuteCustomData=n
	Provisioning.PasswordCryptId=6
	Provisioning.PasswordCryptSaltLength=10
	ResourceDisk.Format=y
	ResourceDisk.Filesystem=ext4
	ResourceDisk.MountPoint=/mnt/resource
	ResourceDisk.MountOptions=None
	ResourceDisk.EnableSwap=n
	ResourceDisk.SwapSizeMB=0
	LBProbeResponder=y
	Logs.Verbose=n
	OS.RootDeviceScsiTimeout=300
	OS.OpensslPath=None
	HttpProxy.Host=None
	HttpProxy.Port=None

下面详细描述了各种配置选项。配置选项分为三种类型：布尔值、字符串或整数。布尔值配置选项可指定为“y”或“n”。特殊关键字“无”可用于某些字符串类型配置条目，详细信息如下所示。

**Provisioning.Enabled：**类型：布尔值，默认值：y

这允许用户在代理中启用或禁用预配功能。有效值为“y”或“n”。如果禁用预配，则会保留映像中的 SSH 主机和用户密钥，并忽略 Azure 预配 API 中指定的所有配置。

> [AZURE.NOTE] `Provisioning.Enabled` 参数在使用 cloud-init 进行预配的 Ubuntu 云映像上默认为“n”。

  
**Provisioning.DeleteRootPassword：**类型：布尔值，默认值：n

如果设置此参数，则会在预配过程中清除 /etc/shadow 文件中的根密码。


**Provisioning.RegenerateSshHostKeyPair：**类型：布尔值，默认值：y

如果设置此参数，则会在预配过程中从 /etc/ssh/ 中删除所有 SSH 主机密钥对（ecdsa、dsa 和 rsa）。并且会生成一个全新的密钥对。

此全新密钥对的加密类型可通过 Provisioning.SshHostKeyPairType 项进行配置。请注意，在重新启动 SSH 守护程序时（例如，重新启动时），某些分发将为任何缺失的加密类型重新创建 SSH 密钥对。


**Provisioning.SshHostKeyPairType：**类型：字符串，默认值：rsa

可将其设置为虚拟机上的 SSH 守护程序支持的加密算法类型。通常支持的值为“rsa”、“dsa”和“ecdsa”。请注意，Windows 上的“putty.exe”不支持“ecdsa”。因此，若要在 Windows 上使用 putty.exe 连接到 Linux 部署，请使用“rsa”或“dsa”。


**Provisioning.MonitorHostName：**类型：布尔值，默认值：y

如果设置此参数，则 waagent 将监视 Linux 虚拟机的主机名更改情况（由“hostname”命令返回），并自动更新映像中的网络配置以反映此更改。若要将名称更改推送到 DNS 服务器，会重新启用虚拟机中的网络。这将导致 Internet 连接暂时中断。


**Provisioning.DecodeCustomData** 类型：布尔值，默认值：n

如果已设置，waagent 将从 Base64 解码 CustomData。


**Provisioning.ExecuteCustomData** 类型：布尔值，默认值：n

如果已设置，waagent 将在预配后执行 CustomData。


**Provisioning.PasswordCryptId** 类型：字符串，默认值：6

生成密码哈希时加密使用的算法。
1 - MD5 
2a - Blowfish 
5 - SHA-256 
6 - SHA-512


**Provisioning.PasswordCryptSaltLength** 类型：字符串，默认值：10

生成密码哈希时使用的随机 salt 长度。


**ResourceDisk.Format：**类型：布尔值，默认值：y

如果设置此参数，则当“ResourceDisk.Filesystem”中用户请求的 filesystem 类型是“ntfs”之外的任何值时，平台提供的资源磁盘将通过 waagent 进行格式化和装载。将在磁盘上提供类型 Linux (83) 的单个分区。请注意，如果可以成功装载此分区，则不会对其进行格式化。


**ResourceDisk.Filesystem：**类型：字符串，默认值：ext4

这将指定资源磁盘的 filesystem 类型。受支持的值因 Linux 分发而异。如果字符串为 X，则 mkfs.X 应呈现在 Linux 映像上。SLES 11 映像通常应使用“ext3”。FreeBSD 映像在此处应使用“ufs2”。


**ResourceDisk.MountPoint：**类型：字符串，默认值：/mnt/resource

这将指定资源磁盘的装载路径。请注意，资源磁盘是*临时*磁盘，可能在取消预配 VM 时被清空。


**ResourceDisk.MountOptions：**类型：字符串，默认值：无

指定要传递给 mount -o 命令的磁盘装载选项。这是一个逗号分隔值列表，例如“nodev,nosuid”。有关详细信息，请参阅 mount(8)。


**ResourceDisk.EnableSwap：**类型：布尔值，默认值：n

如果设置此参数，则将在资源磁盘上创建交换文件 (/swapfile) 并将该文件添加到系统交换空间。


**ResourceDisk.SwapSizeMB：**类型：整数，默认值：0

交换文件的大小（以兆字节为单位）。


**Logs.Verbose：**类型：布尔值，默认值：n

如果设置此参数，日志的详细程度则会有所提升。Waagent 将日志记录到 /var/log/waagent.log 并使用系统 logrotate 功能来循环日志。


**OS.EnableRDMA** 类型：布尔值，默认值：n

如果已设置，代理将尝试安装然后加载与底层硬件上的固件版本匹配的 RDMA 内核驱动程序。


**OS.RootDeviceScsiTimeout：**类型：整数，默认值：300

这将配置 OS 磁盘和数据驱动器上的 SCSI 超时（以秒为单位）。如果未设置此参数，则使用系统默认值。


**OS.OpensslPath：**类型：字符串，默认值：无

这可用于指定要用于加密操作的 openssl 二进制文件的替代路径。


**HttpProxy.Host、HttpProxy.Port** 类型：字符串，默认值：无

如果已设置，代理将使用此代理服务器访问 Internet。


## Ubuntu 云映像

请注意，Ubuntu 云映像利用 [cloud-init](https://launchpad.net/ubuntu/+source/cloud-init) 执行多种配置任务，这些任务在其他情况下也可以通过 Azure Linux 代理来管理。请注意以下不同：


- **Provisioning.Enabled** 在使用 cloud-init 执行预配任务的 Ubuntu 云映像上默认为“n”。

- 以下配置参数对使用 cloud-init 来管理资源磁盘并交换空间的 Ubuntu 云映像没有影响：

 - **ResourceDisk.Format**
 - **ResourceDisk.Filesystem**
 - **ResourceDisk.MountPoint**
 - **ResourceDisk.EnableSwap**
 - **ResourceDisk.SwapSizeMB**

- 请参阅以下资源来配置资源磁盘装入点，并在预配期间交换 Ubuntu 云映像上的空间：

 - [Ubuntu Wiki：配置交换分区](https://wiki.ubuntu.com/AzureSwapPartitions)
 - [将自定义数据注入到 Azure 虚拟机中](/documentation/articles/virtual-machines-linux-classic-inject-custom-data/)

<!---HONumber=Mooncake_1114_2016-->