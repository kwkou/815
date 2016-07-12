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
	ms.date="05/02/2016"
	wacn.date="06/27/2016"/>

# 在 Linux 和 Mac 上为 Azure 中的 Linux VM 创建 SSH 密钥

若要创建受密码保护的 SSH 公钥和私钥，你需要在工作站上打开一个终端。创建 SSH 密钥后，可以默认使用该密钥创建新 VM，或使用 Azure CLI 和 Azure 模板将公钥添加到现有 VM。

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

若要登录到 Linux 服务器，最简单的方法是使用 SSH 公钥和私钥，但比起使用密码登录 Azure 中的 Linux 或 BSD VM，使用[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)安全得多，因为密码非常容易遭到暴力破解。公钥可与任何人共享；但只有你（或本地安全基础结构）才拥有你的私钥。创建的 SSH 私钥将通过[安全密码](https://www.xkcd.com/936/)进行保护，此密码只是用于访问 SSH 私钥，并且**不是**用户帐户密码。任何拥有私钥但没有密码的人都可以使用安装的公钥来访问任何服务器。如果没有密码，就不能使用私钥。


本文将创建 *ssh-rsa* 格式的密钥文件，因为它们是 Resource Manager 上的部署建议使用的文件，并且也是在[门户预览](https://portal.azure.cn)上进行经典部署和资源管理员部署时所要使用的文件。


## 创建 SSH 密钥

Azure 需要至少 2048 位采用 ssh-rsa 格式的公钥和私钥。为了创建密钥对，我们将使用 `ssh-keygen`（询问一系列问题），然后编写私钥和匹配的公钥。在创建 Azure VM 时传递公钥内容，此内容将复制到 Linux VM。当你登录时，此内容将与安全存储在本地的私钥配合使用，以便对你进行身份验证。

## 使用 ssh-keygen

此命令使用 2048 位 RSA 创建密码保护的 SSH 密钥对，并为其加上注释以方便识别。

	ssh-keygen -t rsa -b 2048 -C "ahmet@fedoraVMAzure"

命令解释

`ssh-keygen` = 用于创建密钥的程序

`-t rsa` = 要创建的密钥类型，即 [RSA 格式](https://en.wikipedia.org/wiki/RSA_(cryptosystem))

`-b 2048` = 密钥的位数

`-C "ahmet@fedoraVMAzure"` = 附加到公钥文件末尾以便于识别的注释。通常以电子邮件作为注释，但你也可以使用任何最适合你的基础结构的事物。

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

本文中的密钥对名称。系统默认提供名为 **id\_rsa** 的密钥对，有些工具可能要求私钥文件名为 **id\_rsa**，因此最好使用此密钥对（`~/.ssh/` 是所有 SSH 密钥对和 SSH 配置文件的典型默认位置）。

	ahmet@fedora$ ls -al ~/.ssh
	-rw------- 1 ahmet staff  1675 Aug 25 18:04 azure_fedora_id_rsa
	-rw-r--r-- 1 ahmet staff   410 Aug 25 18:04 azure_fedora_id_rsa.pub

显示新密钥对及其权限。`ssh-keygen` 创建 `~/.ssh` 目录（如果不存在），并设置正确的所有权和文件模式。

密钥密码：

`Enter passphrase (empty for no passphrase):`

强烈建议在密钥对中添加一个密码（`ssh-keygen` 将此密码称为“通行短语”）。如果不使用密码来保护密钥对，任何人只要拥有私钥文件，就可以用它登录到拥有相应公钥的任何服务器，包括你的服务器。添加密码进而提升防护能力以防有人能够获取私钥文件，可让你有时间更改用于对你进行身份验证的密钥。

## 使用 ssh-agent 来存储私钥密码

为了避免在每次 SSH 登录时输入私钥文件密码，可以使用 `ssh-agent` 来缓存私钥文件密码，以便快速安全地登录 Linux VM。如果使用 OSX，Keychain 将在你调用 `ssh-agent` 时安全存储你的私钥密码。

首先请验证 `ssh-agent` 是否正在运行

	eval "$(ssh-agent -s)"

接下来，使用 `ssh-add` 命令将私钥添加到 `ssh-agent`，在 OSX 上，这会启动存储凭据的 Keychain。

	ssh-add ~/.ssh/azure_fedora_id_rsa

现已存储私钥密码，因此你不需要在每次 SSH 登录时键入密钥密码。

## 创建并配置 SSH 配置文件

尽管对于让 Linux VM 启动并运行来说不一定需要，但最好还是创建并配置一个 `~/.ssh/config` 文件，以免不小心使用密码来登录 VM、自动为不同的 Azure VM 使用不同的密钥对，以及设置其他程序（例如 **git**）来锁定多个服务器。

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
	  PubkeyAuthentication yes
	  IdentityFile /home/ahmet/.ssh/azure_fedora_id_rsa
	# ./Azure Keys
	# Default Settings
	Host *
	  PubkeyAuthentication=no
	  IdentitiesOnly=yes
	  ServerAliveInterval=60
	  ServerAliveCountMax=30
	  ControlMaster auto
	  ControlPath /home/ahmet/.ssh/Connections/ssh-%r@%h:%p
	  ControlPersist 4h
	  StrictHostKeyChecking=no
	  IdentityFile /home/ahmet/.ssh/id_rsa
	  UseRoaming=no

此 SSH 配置可以指定每个服务的部分，以便它们各自获得专用的密钥对。默认设置适用于你登录的、但不匹配上述任何组的任何主机。


### 配置文件解释

`Host` = 在终端上调用的主机名。`ssh fedora22` 告知 `SSH` 使用设置块中标记为 `Host fedora22` 的值。注意：这可以是符合用途的任何标签，并不代表任何服务器的实际主机名。

`Hostname 102.160.203.241` = 登录到的服务器的 IP 地址或 DNS 名称。此参数用于路由到服务器，并且可以是映射到内部 IP 的外部 IP。

`User git` = 登录到 Azure VM 时要使用的远程用户帐户。

`PubKeyAuthentication yes` = 告知 SSH 你想要使用 SSH 密钥来登录。

`IdentityFile /home/ahmet/.ssh/azure_fedora_id_rsa` = 告知 SSH 要向服务器提供哪个密钥对来验证登录。


## 在不提供密码的情况下使用 SSH 连接到 Linux

创建 SSH 密钥对并配置 SSH 配置文件后，你可以快速安全地登录到 Linux VM。首次使用 SSH 密钥登录到服务器时，命令将提示你输入该密钥文件的通行短语。

	ssh fedora22

### 命令解释

执行 `ssh fedora22` 后，SSH 先从 `Host fedora22` 块中找到并加载所有设置，然后从最后一个块 (`Host *`) 中加载所有剩余设置。

## 后续步骤

下一步是使用新 SSH 公钥创建 Azure Linux VM。使用 SSH 公钥作为登录名创建的 Azure VM 受到的保护优于使用默认登录方法密码创建的 Azure VM。默认情况下，使用 SSH 密钥作为登录名的 Azure VM 配置为禁用密码登录，以避免暴力破解企图。

- [使用 Azure 模板创建安全 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
- [使用 Azure 门户预览创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
- [使用 Azure CLI 创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)

<!---HONumber=Mooncake_0620_2016-->