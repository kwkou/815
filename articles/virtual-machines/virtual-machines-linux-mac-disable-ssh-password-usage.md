<properties
	pageTitle="通过配置 SSHD 禁用 Linux VM 上的 SSH 密码 | Azure"
	description="通过禁用 SSH 的密码登录来保护 Azure 上的 Linux VM。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="vlivech"
	manager="timlt"
	editor=""
	tags="" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="01/29/2016"
	wacn.date="06/07/2016"/>

# 通过配置 SSHD 禁用 Linux VM 上的 SSH 密码

本文重点介绍如何锁定 Linux VM 的登录安全性。对外开放 SSH 端口 22 后，bot 将开始通过猜测密码来尝试登录。在本中，我们将禁用通过 SSH 的密码登录。通过完全去除使用密码的能力，我们可以避免 Linux VM 遭受这种暴力密码破解攻击。这种做法还有额外的优点，那就是可以将 Linux SSHD 配置为只允许通过 SSH 公钥和私钥登录，而这是目前最安全的 Linux 登录方式。由于可能的私钥组合千变万化，若要在这种情况下猜测，即使使用 bot 来尝试暴力破解 SSH 密钥也无济于事。


## 目标

- 将 SSHD 为配置禁止：
  - 密码登录
  - Root 用户登录
  - 质询-响应身份验证
- 将 SSHD 配置为允许：
  - 仅限 SSH 密钥登录
- 在保持登录的情况下重新启动 SSHD
- 测试新的 SSHD 配置

## 介绍

[SSH 定义](https://zh.wikipedia.org/wiki/Secure_Shell)

SSHD 是在 Linux VM 上运行的 SSH 服务器。SSH 是从 MacBook 或 Linux 工作站上的 shell 运行的客户端。SSH 也是用于保护和加密工作站与 Linux VM 的通信的协议。

在本文中，必须让用户在 Linux VM 中保持登录，使整个演练得以顺畅进行。出于此原因，我们将打开两个终端，并从它们打开 Linux VM 的 SSH。我们将使用第一个终端来更改 SSHD 配置文件，然后重新启动 SSHD 服务。重新启动服务后，我们将使用第二个终端来测试这些更改。由于我们要禁用 SSH 密码并严重依赖于 SSH 密钥，因此如果 SSH 密钥不正确而关闭 VM 连接，VM 将永久被锁定，并且没有人能够登录。这样，只能将它删除后再重新创建。

## 先决条件

- 在 Linux 和 Mac 上为 Azure 中的 Linux VM 创建 SSH 密钥
- Azure 帐户
  - [试用版注册](/pricing/1rmb-trial/)
  - [Azure 门户预览](http://portal.azure.cn)
- 在 Azure 上运行的 Linux VM
- `~/.ssh/` 中的 SSH 公钥和私钥对
- Linux VM 上的 `~/.ssh/authorized_keys` 中的 SSH 公钥
- VM 上的 Sudo 权限
- 端口 22 已打开

## 快速命令

只需要 TLDR 版本的资深 Linux 管理员请从此处开始。其他需要详细说明和演练的用户请跳过本部分。

	username@macbook$ sudo vim /etc/ssh/sshd_config
	
	# Change PasswordAuthentication to this:
	PasswordAuthentication no
	
	# Change PubkeyAuthentication to this:
	PubkeyAuthentication yes
	
	# Change PermitRootLogin to this:
	PermitRootLogin no
	
	# Change ChallengeResponseAuthentication to this:
	ChallengeResponseAuthentication no
	
	# On the Debian Family
	username@macbook$ sudo service ssh restart
	
	# On the RedHat Family
	username@macbook$ sudo service sshd restart

## 详细演练

在终端 1 (T1) 上登录到 Linux VM。在终端 2 (T2) 上登录到 Linux VM。

我们将在 T2 上编辑 SSHD 配置文件。

	username@macbook$ sudo vim /etc/ssh/sshd_config

在这里，我们只需编辑设置来禁用密码，并启用 SSH 密钥登录。此文件中有许多需要研究和更改的设置，它们能让 Linux 和 SSH 的安全性符合要求。

#### 禁用密码登录

	username@macbook$ sudo vim /etc/ssh/sshd_config
	
	# Change PasswordAuthentication to this:
	PasswordAuthentication no

#### 启用公钥身份验证

	username@macbook$ sudo vim /etc/ssh/sshd_config
	
	# Change PubkeyAuthentication to this:
	PubkeyAuthentication yes

#### 禁用 Root 登录

	username@macbook$ sudo vim /etc/ssh/sshd_config
	
	# Change PermitRootLogin to this:
	PermitRootLogin no

#### 禁用质询-响应身份验证

	# Change ChallengeResponseAuthentication to this:
	ChallengeResponseAuthentication no

### 重新启动 SSHD

从 T1 shell 验证你是否仍保持登录。此步骤非常重要，因为我们已禁用密码，这样做能够避免当 SSH 密钥不正确时 VM 被锁定。如果 Linux VM 上有任何不正确的设置，你可以使用 T1 来修复 sshd\_config，因为你仍保持登录状态，并且 SSH 在 SSHD 服务重新启动期间会保持连接的活动状态。

在 T2 上运行：

##### 在 Debian 系列上

	username@macbook$ sudo service ssh restart

##### 在 RedHat 系列上

	username@macbook$ sudo service sshd restart

VM 上的密码现已禁用，可以防止有人尝试进行避免暴力破解密码登录。由于只允许 SSH 密钥，你可以使用更快速、更安全的方式登录。

<!---HONumber=Mooncake_0503_2016-->