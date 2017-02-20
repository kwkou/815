<properties
    pageTitle="Azure FreeBSD 简介 | Azure"
    description="了解如何在 Azure 上使用 FreeBSD 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="KylieLiang"
    manager="timlt"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="32b87a5f-d024-4da0-8bf0-77e233d1422b"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure-services"
    ms.date="01/09/2017"
    wacn.date="02/20/2017"
    ms.author="kyliel" />  


# Azure FreeBSD 简介
本主题概述如何在 Azure 中运行 FreeBSD 虚拟机。

## 概述
FreeBSD 是一种高级的计算机操作系统，现代服务器、台式机和嵌入式平台。

Microsoft 公司在 Azure 上提供预先配置了 [Azure 虚拟机代理](https://github.com/Azure/WALinuxAgent/)的 FreeBSD 镜像。目前，Microsoft 提供以下 FreeBSD 版本的镜像：

- FreeBSD 10.3-RELEASE
- FreeBSD 11.0-RELEASE

Azure 虚拟机代理负责 FreeBSD 虚拟机 和 Azure 结构控制器之间的通信，比如首次使用时预配虚拟机（用户名、密码或 SSH 密钥、主机名等）并启用虚拟机扩展的选择功能。

至于未来的版本，策略是跟踪最新版本，确保在 FreeBSD 工程团队发布了最新版本后在 Azure 尽快上线该版本。

## 部署 FreeBSD 虚拟机
在使用 Azure 应用商店中的镜像时，部署 FreeBSD 虚拟机是一个非常简单的过程：

- [Azure 应用商店中的 FreeBSD 10.3](https://portal.azure.cn/#create/Microsoft.FreeBSD103-ARM)
- [Azure 应用商店中的 FreeBSD 11.0](https://portal.azure.cn/#create/Microsoft.FreeBSD110-ARM)

## 适用于 FreeBSD 的虚拟机扩展
以下是 FreeBSD 中支持的虚拟机扩展。

### VMAccess
[VMAccess](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) 扩展可以：

* 重置原始的 sudo 用户的密码。
* 使用指定的密码创建新的 sudo 用户。
* 使用给定的密钥设置公共主机密钥。
* 重置在虚拟机预配期间提供的公共主机密钥（如果未提供主机密钥）。
* 打开 SSH 端口 \(22\) 并还原 sshd\_config（如果 reset\_ssh 设置为 true）。
* 删除现有用户。
* 检查磁盘。
* 修复添加的磁盘。

### CustomScript
[CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) 扩展可以：

* 从 Azure 存储或外部公共存储（例如 GitHub）下载自定义的脚本（如果已提供）。
* 运行入口点脚本。
* 支持内联命令。
* 在 shell 和 Python 脚本中自动进行 Windows 样式的换行符转换。
* 自动删除 shell 和 Python 脚本中的 BOM。
* 保护 CommandToExecute 中的敏感数据。

[AZURE.NOTE]FreeBSD VM 目前仅支持 CustomScript 1.x 版。

## 身份验证：用户名、密码和 SSH 密钥
使用 Azure 门户预览创建 FreeBSD 虚拟机时，必须提供用户名、密码或 SSH 公钥。在 Azure 上部署 FreeBSD 虚拟机时，用户名必须与已经存在于虚拟机中的系统帐户 \(UID \<100\) 的名称（例如“root”）相符。目前仅支持 RSA SSH 密钥。多行 SSH 密钥必须以 `---- BEGIN SSH2 PUBLIC KEY ----` 开头，以 `---- END SSH2 PUBLIC KEY ----` 结尾。

## 获取超级用户特权
在 Azure 上部署虚拟机实例的过程中指定的用户帐户是特权帐户。sudo 包安装在已发布的 FreeBSD 镜像中。通过此用户帐户登录后，即可使用命令语法以 root 用户身份运行命令。

    $ sudo <COMMAND>

可以选择使用 `sudo -s` 获取 root shell。

## 已知问题
1. 目前在基于 Hyper-V（以及 Azure）的 FreeBSD 11.0 上存在一个尚待解决的问题，如果操作系统使用 freebsd-update 进行更新，该问题可能导致虚拟机无法启动。在 Azure 应用商店的 FreeBSD 镜像中提供了[建议的修正](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=212721)；但是，FreeBSD 团队尚未在社区接受该修正，因此运行 freebsd-update 会将内核替换为未修正的版本。建议 Azure 用户在 FreeBSD 11.0 发布该修正的 ERRATA 之前不要进行更新。

2. [Azure 虚拟机代理](https://github.com/Azure/WALinuxAgent/)版本 2.2.2 有一个[已知问题](https://github.com/Azure/WALinuxAgent/pull/517)会导致 FreeBSD 在 Azure 上预配的失败。建议 Azure 用户使用 2.2.1 以及之前的版本。这个修正将会在后续的 2.2.3 版本。

## 后续步骤
* 转到 [Azure 应用商店](https://portal.azure.cn/#create/Microsoft.FreeBSD103-ARM)创建 FreeBSD 虚拟机。
* 若要将自己的 FreeBSD 上载到 Azure，请参阅[创建 FreeBSD VHD 并将其上载到 Azure](/documentation/articles/virtual-machines-linux-classic-freebsd-create-upload-vhd/)。

<!---HONumber=Mooncake_0213_2017-->