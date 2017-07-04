<properties
    pageTitle="在 Azure Linux 虚拟机上配置 SSHD | Azure"
    description="按安全最佳实践配置 SSHD，并将 SSH 锁定到 Azure Linux 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/21/2016"
    wacn.date="01/13/2017"
    ms.author="v-livech" />  


# 在 Azure Linux VM 上配置 SSHD

本文说明如何在 Linux 上锁定 SSH 服务器，以便使用 SSH 密钥而不是密码提供最佳实践安全性并加快 SSH 登录过程。为了进一步锁定 SSHD，我们将禁止根用户登录，通过已批准的组列表限制允许登录的用户，禁用 SSH 协议版本 1，设置最小密钥位，并配置空闲用户的自动注销。本文的必要条件是一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）和 [SSH 公钥与私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

## 快速命令

使用以下设置配置 `/etc/ssh/sshd_config`：

### 禁用密码登录

    PasswordAuthentication no

### 禁止根用户登录

    PermitRootLogin no

### 允许的组列表

    AllowGroups wheel

### 允许的用户列表

    AllowUsers ahmet ralph

### 禁用 SSH 协议版本 1

    Protocol 2

### 最小密钥位

    ServerKeyBits 2048

### 断开空闲用户的连接

    ClientAliveInterval 300
    ClientAliveCountMax 0

## 详细演练

SSHD 是在 Linux VM 上运行的 SSH 服务器。SSH 是从 MacBook、Linux 工作站上的 shell 或 Windows 上的 Bash 运行的客户端。SSH 也是用于保护和加密工作站与 Linux VM 的通信的协议，后者使 SSH 还为 VPN（虚拟专用网络）。

在本文中，必须让用户在 Linux VM 中保持登录，使整个演练得以顺畅进行。建立 SSH 连接后，只要未关闭窗口，它将保持为打开的会话。使一个终端处于已登录状态，可以在进行重大更改时，更改 SSHD 服务而不必锁定该服务。如果由于 SSHD 配置被破坏，用户被挡在 Linux VM 的外面，Azure 提供能够使用 [Azure VM 访问扩展](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)重置被破坏的 SSHD 配置的功能。

出于此原因，我们将打开两个终端，并从它们通过 SSH 登录到 Linux VM。我们将使用第一个终端来更改 SSHD 配置文件，然后重新启动 SSHD 服务。重新启动服务后，我们将使用第二个终端来测试这些更改。由于我们要禁用 SSH 密码并严重依赖于 SSH 密钥，因此如果 SSH 密钥不正确而关闭 VM 连接，VM 将永久被锁定，并且没有人能够登录。这样，只能将它删除后再重新创建。

## 禁用密码登录

保护 Linux VM 的最快捷方法是禁用密码登录。启用密码登录后，在 Web 上爬行的机器人将立即开始尝试使用 SSH 暴力猜测 Linux VM 的密码。完全禁用密码登录可使 SSH 服务器忽略所有密码登录尝试。

    PasswordAuthentication no

## 禁止根用户登录

按照 Linux 最佳做法，`root` 用户应从不通过 SSH 或使用 `sudo su` 登录。需要根级别权限的所有命令始终都应通过 `sudo` 命令运行，后者将记录所有操作以便进一步审核。禁止 `root` 用户通过 SSH 登录是安全最佳实践步骤，可确保只有获得授权的用户可以通过 SSH 登录。

    PermitRootLogin no

## 允许的组列表

SSH 提供了限制用户和组的方法，以允许或禁止用户和组通过 SSH 登录。此功能使用列表来批准或拒绝特定用户和组登录。将 wheel 组设置到 `AllowGroups` 列表可将批准的通过 SSH 登录限制为只是 wheel 组中的用户帐户。

    AllowGroups wheel

## 允许的用户列表

将 SSH 登录限制为只是用户是完成 `AllowGroups` 实现的同一任务的更具体方法。

    AllowUsers ahmet ralph

## 禁用 SSH 协议版本 1

SSH 协议版本 1 不安全，应禁用。SSH 协议版本 2 是当前版本，提供通过 SSH 登录到服务器的安全方法。禁用 SSH 版本 1 将拒绝尝试使用 SSH 版本 1 与 SSH 服务器建立连接的任何 SSH 客户端。仅允许使用 SSH 版本 2 连接与 SSH 服务器协商连接。

    Protocol 2

## 最小密钥位

按照安全最佳实践，将禁用密码 SSH 登录，仅允许使用 SSH 密钥向 SSH 服务器进行身份验证。可以使用不同长度的密钥（以位为单位）来创建这些 SSH 密钥。最佳实践指出长度为 2048 位的密钥是可接受的最小密钥强度。从理论上讲，小于 2048 位的密钥可能被破坏。将 `ServerKeyBits` 设置为 `2048` 允许使用 2048 位或更大密钥的任何连接并拒绝使用小于 2048 位的密钥的连接。

    ServerKeyBits 2048

## 断开空闲用户的连接

SSH 能够断开这样的用户的连接：具有打开的连接且空闲持续时间超过设定的秒段。只保留处于活动状态的这些用户的打开会话可限制 Linux VM 的暴露。

    ClientAliveInterval 300
    ClientAliveCountMax 0

## 重新启动 SSHD

若要启用 `/etc/ssh/sshd_config` 中的设置，请重新启动 SSHD 进程，该进程重新启动 SSH 服务器。用于重新启动 SSH 服务器的终端窗口将保持打开状态而不会丢失打开的 SSH 会话。若要测试新的 SSH 服务器设置，请使用第二个终端窗口或选项卡。使用单独的终端测试 SSH 连接，可返回并在第一个终端中对 `/etc/ssh/sshd_config` 进行其他更改，而不会被重大 SSHD 更改锁定。

### 在 Redhat、Centos 和 Fedora 上

    service sshd restart

### 在 Debian 和 Ubuntu 上

    service ssh restart

## 使用 Azure 重置访问重置 SSHD

如果你由于 SSHD 配置发生重大更改而被锁定，可以使用 Azure VM 访问扩展将 SSHD 配置重置回原始配置。

将任何示例名称替换为你自己的名称。

    azure vm reset-access \
    --resource-group myResourceGroup \
    --name myVM \
    --reset-ssh

## 安装 Fail2ban

强烈建议安装并设置开放源代码应用 Fail2ban，以阻止使用暴力破解通过 SSH 登录到你的 Linux VM 的重复尝试。Fail2ban 记录通过 SSH 登录的重复失败尝试，然后创建防火墙规则以阻止尝试所源自的 IP 地址。

* [Fail2ban 主页](http://www.fail2ban.org/wiki/index.php/Main_Page)

## 后续步骤

现在，已在 Linux VM 上配置并锁定 SSH 服务器，可以按照其他安全最佳实践进行操作了。

* [管理用户、SSH，并使用 VMAccess 扩展检查或修复 Azure Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)

* [使用 Azure CLI 加密 Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-encrypt-disks/)

* [Azure Resource Manager 模板中的访问权限和安全性](/documentation/articles/virtual-machines-linux-dotnet-core-3-access-security/)

<!---HONumber=Mooncake_0109_2017-->