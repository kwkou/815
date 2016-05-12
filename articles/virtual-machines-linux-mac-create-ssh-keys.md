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
	ms.date="04/12/2016"
	wacn.date="05/12/2016"/>

# 在 Linux 和 Mac 上为 Azure 中的 Linux VM 创建 SSH 密钥

若要创建受密码保护的 SSH 公钥和私钥，你需要在工作站上打开一个终端。创建 SSH 密钥后，可以默认使用该密钥创建新 VM，或使用 Azure CLI 和 Azure 模板将公钥添加到现有 VM。

## 快速命令列表

在以下命令示例中，请将 &lt; 与 &gt; 之间的值替换为你自己环境中的值。

	[ahmet@fedora ~]$ ssh-keygen -t rsa -b 2048 -C "<your_user@yourdomain.com>"
	
	#Enter the name of the file that will be saved in the `~/.ssh/` directory.
	<azure_fedora_id_rsa>
	
	#Enter passphrase for azure_fedora_id_rsa:
	<correct horse battery staple>
	
	#Add the newly created key to `ssh-agent` on Linux and Mac (also added to OSX Keychain).
	[ahmet@fedora ~]$ eval "$(ssh-agent -s)"
	[ahmet@fedora ~]$ ssh-add ~/.ssh/azure_fedora_id_rsa
	
	#Copy the SSH public key to your Linux Server.
	[ahmet@fedora ~]$ ssh-copy-id -i ~/.ssh/azure_fedora_id_rsa.pub <youruser@yourserver.com>
	
	#Test the login using keys instead of a password.
	[ahmet@fedora ~]$ ssh -o PreferredAuthentications=publickey -o PubkeyAuthentication=yes -i ~/.ssh/azure_fedora_id_rsa <youruser@yourserver.com>
	
	Last login: Tue April 12 07:07:09 2016 from 66.215.22.201
	[ahmet@fedora ~]$

## 介绍

若要登录到 Linux 服务器，最简单的方法是使用 SSH 公钥和私钥，但比起使用密码登录 Azure 中的 Linux 或 BSD VM，使用[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)安全得多，因为密码非常容易遭到暴力破解。公钥可与任何人共享；但只有你（或本地安全基础结构）才拥有你的私钥。创建的 SSH 私钥将通过[安全密码](https://www.xkcd.com/936/)进行保护，此密码只是用于访问 SSH 私钥，并且**不是**用户帐户密码。任何拥有私钥但没有密码的人都可以使用安装的公钥来访问任何服务器。如果没有密码，就不能使用私钥。

## 创建 SSH 密钥

Azure 需要至少 2048 位采用 ssh-rsa 格式的公钥和私钥。为了创建密钥对，我们将使用 `ssh-keygen`（询问一系列问题），然后编写私钥和匹配的公钥。在创建 Azure VM 时传递公钥内容，此内容将复制到 Linux VM。当你登录时，此内容将与安全存储在本地的私钥配合使用，以便对你进行身份验证。

### 使用 `ssh-keygen`

此命令使用 2048 位 RSA 创建密码保护的 SSH 密钥对，并为其加上注释以方便识别。

	ahmet@fedora$ ssh-keygen -t rsa -b 2048 -C "ahmet@fedoraVMAzure"

##### 命令解释

`ssh-keygen` = 用于创建密钥的程序

`-t rsa` = 要创建的密钥类型，即 [RSA 格式](https://en.wikipedia.org/wiki/RSA_(cryptosystem))

`-b 2048` = 密钥的位数

`-C "ahmet@fedoraVMAzure"` = 附加到公钥文件末尾以便于识别的注释。通常以电子邮件作为注释，但你也可以使用任何最适合你的基础结构的事物。

#### `ssh-keygen` 演练

	ahmet@fedora$ ssh-keygen -t rsa -b 2048 -C "ahmet@fedoraVMAzure"
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
	
	ahmet@fedora$ ls -al ~/.ssh
	-rw------- 1 ahmet staff  1675 Aug 25 18:04 azure_fedora_id_rsa
	-rw-r--r-- 1 ahmet staff   410 Aug 25 18:04 azure_fedora_id_rsa.pub

`Enter file in which to save the key (/home/ahmet/.ssh/id_rsa): azure_fedora_id_rsa`
本文中的密钥对名称。系统默认提供名为 **id\_rsa** 的密钥对，有些工具可能要求私钥文件名为 **id\_rsa**，因此最好使用此密钥对（`~/.ssh/` 是所有 SSH 密钥对和 SSH 配置文件的典型默认位置）。

`Enter passphrase (empty for no passphrase):` 
强烈建议在密钥对中添加一个密码（`ssh-keygen` 将此密码称为“通行短语”）。如果不使用密码来保护密钥对，任何人只要拥有私钥文件，就可以用它登录到拥有相应公钥的任何服务器，包括你的服务器。添加密码进而提升防护能力以防有人能够获取私钥文件，可让你有时间更改用于对你进行身份验证的密钥。

`ahmet@fedora$ ls -al ~/.ssh` 
显示新密钥对及其权限。`ssh-keygen` 创建 `~/.ssh` 目录（如果不存在），并设置正确的所有权和文件模式。

## 使用 ssh-agent 来存储私钥密码

为了避免在每次 SSH 登录时输入私钥文件密码，可以使用 `ssh-agent` 来缓存私钥文件密码，以便快速安全地登录 Linux VM。如果使用 OSX，Keychain 将在你调用 `ssh-agent` 时安全存储你的私钥密码。

首先请验证 `ssh-agent` 是否正在运行

`[ahmet@fedora ~]$ eval "$(ssh-agent -s)"`

接下来，使用 `ssh-add` 命令将私钥添加到 `ssh-agent`，在 OSX 上，这会启动存储凭据的 Keychain。

`[ahmet@fedora ~]$ ssh-add ~/.ssh/azure_fedora_id_rsa`

现已存储私钥密码，因此你不需要在每次 SSH 登录时键入密钥密码。

## 创建并配置 SSH 配置文件

尽管对于让 Linux VM 启动并运行来说不一定需要，但最好还是创建并配置一个 `~/.ssh/config` 文件，以免不小心使用密码来登录 VM、自动为不同的 Azure VM 使用不同的密钥对，以及设置其他程序（例如 **git**）来锁定多个服务器。

以下示例显示了标准配置。

### 创建文件

	ahmet@fedora$ touch ~/.ssh/config

### 编辑文件以添加新的 SSH 配置

	ahmet@fedora$ vim ~/.ssh/config
	
	#Azure Keys
	Host fedora22
	  Hostname 102.160.203.241
	  User ahmet
	  PubkeyAuthentication yes
	  IdentityFile /home/ahmet/.ssh/azure_fedora_id_rsa
	# ./Azure Keys
	# GitHub keys
	Host github.com
	  Hostname github.com
	  User git
	  PubKeyAuthentication yes
	  IdentityFile /home/ahmet/.ssh/azure_fedora_id_rsa
	Host github.private
	  Hostname github.com
	  User git
	  PubKeyAuthentication yes
	  IdentityFile /home/ahmet/.ssh/private_repo_azure_fedora_id_rsa
	# ./Github Keys
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

此 SSH 配置可以指定每个服务的部分，以便它们各自获得专用的密钥对。默认设置适用于你登录的、但不匹配上述任何组的任何主机。SSH 配置还可让你获得两个不同的 [GitHub](https://github.com) 登录名，一个用于公共工作，另一个只用于工作时常用的专用存储库。


##### 配置文件解释

`Host` = 在终端上调用的主机名。`ssh fedora22` 告知 `SSH` 使用设置块中标记为 `Host fedora22` 的值。注意：这可以是符合用途的任何标签，并不代表任何服务器的实际主机名。

`Hostname 102.160.203.241` = 登录到的服务器的 IP 地址或 DNS 名称。此参数用于路由到服务器，并且可以是映射到内部 IP 的外部 IP。

`User git` = 登录到 Azure VM 时要使用的远程用户帐户。

`PubKeyAuthentication yes` = 告知 SSH 你想要使用 SSH 密钥来登录。

`IdentityFile /home/ahmet/.ssh/azure_fedora_id_rsa` = 告知 SSH 要向服务器提供哪个密钥对来验证登录。


## 在不提供密码的情况下使用 SSH 登录到 Linux VM

创建 SSH 密钥对并配置 SSH 配置文件后，你可以快速安全地登录到 Linux VM。首次使用 SSH 密钥登录到服务器时，命令将提示你输入该密钥文件的通行短语。

`ahmet@fedora$ ssh fedora22`

##### 命令解释

执行 `ahmet@fedora$ ssh fedora22` 后，SSH 先从 `Host fedora22` 块中找到并加载所有设置，然后从最后一个块 (`Host *`) 中加载所有剩余设置。

## 后续步骤

现在，你可以使用 SSH 密钥文件来执行以下操作：

- [使用 Azure 模板创建安全 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template)
- [使用 Azure 门户创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal)
- [使用 Azure CLI 创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli)

<!---HONumber=Mooncake_0503_2016-->