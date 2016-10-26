<properties
	pageTitle="在 Linux 和 Mac 上创建 SSH 密钥 | Azure"
	description="在 Linux 和 Mac 上为 Azure 上的资源管理器和经典部署模型生成和使用 SSH 密钥。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="vlivech"
	manager="timlt"
	editor=""
	tags="" />  


<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/08/2016"
	wacn.date="10/24/2016"
	ms.author="v-livech"/>  


# 在 Linux 和 Mac 上为 Azure 中的 Linux VM 创建 SSH 密钥

使用 SSH 密钥对，可以在 Azure 上创建默认为使用 SSH 密钥进行身份验证的虚拟机，从而无需密码就可登录。密码可能被猜到，将 VM 向不间断的强力尝试开放，用于猜测密码。使用 Azure 模板或 `azure-cli` 创建的 VM 可以在部署过程中提供 SSH 公钥，并删除部署后配置。如果要从 Windows 连接到 Linux VM，请参阅[此文档](/documentation/articles/virtual-machines-linux-ssh-from-windows/)。

## 快速命令列表

在以下命令示例中，请将 &lt; 与 &gt; 之间的值替换为你自己环境中的值。

	ssh-keygen -t rsa -b 2048 -C "<your_user@yourdomain.com>"

输入要在 `~/.ssh/` 目录中保存的文件的名称：

	<azure_fedora_id_rsa>

输入 azure\_fedora\_id\_rsa 的通行短语：

	<correct horse battery staple>

将新建的密钥添加到 Linux 和 Mac 上的 `ssh-agent`（同时添加到 OSX Keychain）：

	eval "$(ssh-agent -s)"
	ssh-add ~/.ssh/azure_fedora_id_rsa

将 SSH 公钥复制到你的 Linux 服务器：

	ssh-copy-id -i ~/.ssh/azure_fedora_id_rsa.pub <youruser@yourserver.com>

使用密钥而不是密码测试登录：

	ssh -o PreferredAuthentications=publickey -o PubkeyAuthentication=yes -i ~/.ssh/azure_fedora_id_rsa <youruser@yourserver.com>
	Last login: Tue April 12 07:07:09 2016 from 66.215.22.201
	$

## 介绍

使用 SSH 公钥和私钥是登录到 Linux 服务器的最简单方法。[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)提供了一种比密码安全得多的方法，用于登录到 Azure 中的 Linux 或 BSD VM，密码更容易受到暴力攻击。公钥可与任何人共享；但只有你（或本地安全基础结构）才拥有你的私钥。SSH 私钥应使用[非常安全的密码](https://www.xkcd.com/936/)（源：[xkcd.com](https://xkcd.com)）来保护它。此密码只用于访问 SSH 私钥，**不是**用户帐户密码。向 SSH 密钥添加密码时，它会对私钥进行加密，因此在没有密码解锁的情况下，私钥不可用。如果攻击者窃取了私钥，并且该密钥没有密码，那么他们就能使用该私钥登录到具有相应公钥的任何服务器。如果私钥受密码保护，攻击者就无法使用，它会为 Azure 基础结构提供一个额外的安全层。



本文将创建 *ssh-rsa* 格式的密钥文件，它们是 Resource Manager 上的部署建议使用的文件。在[门户](https://portal.azure.cn)上进行经典部署和 Resource Manager 部署时需要 *ssh-rsa* 密钥。


## 创建 SSH 密钥

Azure 需要至少 2048 位采用 ssh-rsa 格式的公钥和私钥。为了创建密钥，将使用 `ssh-keygen`，它会询问一系列问题，然后编写私钥和匹配的公钥。创建 Azure VM 时，会将公钥复制到 `~/.ssh/authorized_keys`。`~/.ssh/authorized_keys` 中的 SSH 密钥用于在 SSH 登录连接时质询客户端以匹配相应的私钥。


## 使用 ssh-keygen

此命令使用 2048 位 RSA 创建密码保护的（加密）SSH 密钥对，并为其加上注释以方便识别。

	ssh-keygen -t rsa -b 2048 -C "ahmet@fedoraVMAzure"

_命令解释_

`ssh-keygen` = 用于创建密钥的程序

`-t rsa` = 要创建的密钥类型，即 [RSA 格式](https://en.wikipedia.org/wiki/RSA_(cryptosystem)

`-b 2048` = 密钥的位数

`-C "ahmet@fedoraVMAzure"` = 追加到公钥文件末尾以便于识别的注释。通常以电子邮件作为注释，但也可以使用任何最适合基础结构的事物。

### 使用 PEM 密钥

如果使用的是经典部署模型（Azure 经典管理门户或 Azure 服务管理 CLI `asm`），可能需要使用 PEM 格式的 SSH 密钥来访问 Linux VM。下面介绍如何使用现有的 SSH 公钥和现有的 x509 证书创建 PEM 密钥。

使用现有的 SSH 公钥创建 PEM 格式的密钥：

```bash
ssh-keygen -f id_rsa.pub -m 'PEM' -e > id_rsa.pem
```

## ssh-keygen 演练

每个步骤的详解。首先运行 `ssh-keygen`。

	ssh-keygen -t rsa -b 2048 -C "ahmet@fedoraVMAzure"
	Generating public/private rsa key pair.
	Enter file in which to save the key (/home/ahmet/.ssh/id_rsa): azure_fedora_id_rsa
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in azure_fedora_id_rsa.
	Your public key has been saved in azure_fedora_id_rsa.pub.
	The key fingerprint is:
	14:a3:cb:3e:78:ad:25:cc:55:e9:0c:08:e5:d1:a9:08 ahmet@fedoraVMAzure
	The key's randomart image is:
	+--[ RSA 2048]----+
	|        o o. .   |
	|      E. = .o    |
	|      ..o...     |
	|     . o....     |
	|      o S =      |
	|     . + O       |
	|      + = =      |
	|       o +       |
	|        .        |
	+-----------------+

保存的密钥文件：

`Enter file in which to save the key (/home/ahmet/.ssh/id_rsa): azure_fedora_id_rsa`  


本文中的密钥对名称。系统默认提供名为 **id\_rsa** 的密钥对，有些工具可能要求私钥文件名为 **id\_rsa**，因此最好使用此密钥对。目录 `~/.ssh/` 是 SSH 密钥对和 SSH 配置文件的默认位置。

	ahmet@fedora$ ls -al ~/.ssh
	-rw------- 1 ahmet staff  1675 Aug 25 18:04 azure_fedora_id_rsa
	-rw-r--r-- 1 ahmet staff   410 Aug 25 18:04 azure_fedora_id_rsa.pub
`~/.ssh` 目录的列表。`ssh-keygen` 创建 `~/.ssh` 目录（如果不存在），并设置相应的所有权和文件模式。

密钥密码：

`Enter passphrase (empty for no passphrase):`  


`ssh-keygen` 将密码称为“通行短语”。 *强烈*建议在密钥对中添加一个密码。如果不使用密码来保护密钥对，任何人只要拥有私钥文件，就可以用它登录到具有相应公钥的任何服务器。添加密码可提供更多保护以防有人能够获取私钥文件，可让用户有时间更改用于进行身份验证的密钥。

## 使用 ssh-agent 来存储私钥密码

为了避免在每次 SSH 登录时输入私钥文件密码，可以使用 `ssh-agent` 来缓存私钥文件密码。如果用户使用 Mac，OSX Keychain 将在用户调用 `ssh-agent` 时安全存储私钥密码。

首先请验证 `ssh-agent` 是否正在运行

	eval "$(ssh-agent -s)"

现在，使用命令 `ssh-add` 将私钥添加到 `ssh-agent`。

	ssh-add ~/.ssh/azure_fedora_id_rsa

私钥密码现在存储在 `ssh-agent` 中。

## 创建并配置 SSH 配置文件

建议的最佳做法是，创建并配置 `~/.ssh/config` 文件以加快登录速度并优化 SSH 客户端行为。

以下示例显示了标准配置。

### 创建文件

	touch ~/.ssh/config

### 编辑文件以添加新的 SSH 配置：

	vim ~/.ssh/config

### 示例 `~/.ssh/config` 文件：

	# Azure Keys
	Host fedora22
	  Hostname 102.160.203.241
	  User ahmet
	# ./Azure Keys
	# Default Settings
	Host *
	  PubkeyAuthentication=yes
	  IdentitiesOnly=yes
	  ServerAliveInterval=60
	  ServerAliveCountMax=30
	  ControlMaster auto
	  ControlPath ~/.ssh/SSHConnections/ssh-%r@%h:%p
	  ControlPersist 4h
	  IdentityFile ~/.ssh/id_rsa

此 SSH 配置可以指定每个服务器的部分，以便它们各自获得专用的密钥对。默认设置 (`Host *`) 适用于不匹配配置文件上部列出的任何特定主机的任何主机。


### 配置文件解释

`Host` = 在终端上调用的主机名。`ssh fedora22` 告知 `SSH` 使用标记为 `Host fedora22` 的设置块中的值。注意：这可以是符合用途的任何标签，并不代表任何服务器的实际主机名。

`Hostname 102.160.203.241` = 所访问的服务器的 IP 地址或 DNS 名称。

`User git` = 要使用的远程用户帐户。

`PubKeyAuthentication yes` = 告知 SSH 要使用 SSH 密钥来登录。

`IdentityFile /home/ahmet/.ssh/id_id_rsa` = 要用于身份验证的 SSH 私钥和对应的公钥。


## 在不提供密码的情况下使用 SSH 连接到 Linux

创建 SSH 密钥对并配置 SSH 配置文件后，便可以快速安全地登录到 Linux VM。首次使用 SSH 密钥登录到服务器时，命令将提示用户输入该密钥文件的通行短语。

	ssh fedora22

### 命令解释

执行 `ssh fedora22` 后，SSH 先从 `Host fedora22` 块中找到并加载所有设置，然后从最后一个块 (`Host *`) 中加载所有剩余设置。

## 后续步骤

下一步是使用新 SSH 公钥创建 Azure Linux VM。使用 SSH 公钥作为登录名创建的 Azure VM 受到的保护优于使用默认登录方法（即密码）创建的 VM。使用 SSH 密钥创建的 Azure VM 默认情况下配置为禁用密码，以避免强力猜测尝试。

- [使用 Azure 模板创建安全 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
- [使用 Azure 门户预览创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
- [使用 Azure CLI 创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)

<!---HONumber=Mooncake_1017_2016-->