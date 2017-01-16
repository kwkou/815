<properties
    pageTitle="使用 SCP 将文件移到 Linux VM 和从 Linux VM 移动文件 | Azure"
    description="使用 SCP 和 SSH 密钥对安全地将文件移到 Linux VM 和从 Linux VM 移动文件。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="ms.service: virtual-machines-linux"
    ms.workload="infrastructure"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/14/2016"
    wacn.date="01/13/2017"
    ms.author="v-livech" />  


# 使用 SCP 将文件移到 Linux VM 和从 Linux VM 移动文件

本文说明如何使用安全复制 (SCP) 将文件从工作站向上移到 Azure Linux VM，或从 Azure Linux VM 向下移到工作站。作为示例，我们要将 Azure 配置文件向上移到 Linux VM，并向下拉取日志文件目录，这两项操作都使用 SCP 和 SSH 密钥完成。

本文的必要条件如下：

- [一个 Azure 帐户](/pricing/1rmb-trial/)

- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## 快速命令

将文件向上复制到 Linux VM

    scp file user@host:directory/targetfile

从 Linux VM 向下复制文件

    scp user@host:directory/file targetfile

## 详细演练

在工作站和 Linux VM 之间快速安全地来回移动文件是管理 Azure 基础结构的关键部分。在本文中我们使用 SCP 进行演练，该工具在 SSH 的基础上构建，并包含在 Linux、Mac 和 Windows 的默认 Bash shell 中。

## SSH 密钥对身份验证

SCP 将 SSH 用于传输层。通过使用 SSH 进行传输，SSH 处理目标主机上的身份验证，同时还在 SSH 默认提供的加密隧道中移动文件。对于 SSH 身份验证，可以使用用户名和密码，但作为安全最佳做法，强烈建议使用 SSH 公钥和私钥进行身份验证。SSH 对连接进行身份验证后，SCP 将开始复制文件的过程。使用正确配置的 `~/.ssh/config` 和 SSH 公钥和私钥，无需使用用户名，只使用服务器名称就可建立 SCP 连接。如果只有一个 SSH 密钥，SCP 将在 `~/.ssh/` 目录中查找它，并默认情况下使用它登录到 VM。

有关配置 `~/.ssh/config` 以及 SSH 公钥和私钥的详细信息，请按照此文（[创建 SSH 密钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)）进行操作。

## 通过 SCP 将文件复制到 Linux VM

在第一个示例中，我们将 Azure 凭据文件向上复制到用于部署自动化的 Linux VM。由于此文件包含包括机密在内的 Azure API 凭据，因此安全性很重要，SSH 提供的加密隧道用于保护文件的内容。

    scp ~/.azure/credentials myserver:/home/ahmet/.azure/credentials

## 通过 SCP 从 Linux VM 复制目录

在此示例中，我们会将装满日志文件的目录从 Linux VM 向下复制到工作站。日志文件不一定包含敏感或机密数据，使用 SCP 可确保对日志文件的内容进行加密。使用 SCP 安全地传输文件是将日志目录和文件获取到工作站上同时确保安全的最简单方法。

    scp -r myserver:/home/ahmet/logs/ /tmp/.

`-r` cli 标志指示 SCP 从命令中列出目录的时点起以递归方式复制文件和目录。另请注意，命令行语法类似于 `cp` 复制命令。

## 后续步骤

* [管理用户、SSH，并使用 VMAccess 扩展检查或修复 Azure Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)
* [通过配置 SSHD 禁用 Linux VM 上的 SSH 密码](/documentation/articles/virtual-machines-linux-mac-disable-ssh-password-usage/)

<!---HONumber=Mooncake_0109_2017-->