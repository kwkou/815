<properties
		pageTitle="将用户添加到 Azure 上的 Linux VM | Azure"
		description="将用户添加到 Azure 上的 Linux VM。"
		services="virtual-machines-linux"
		documentationCenter=""
		authors="vlivech"
		manager="timlt"
		editor=""
		tags="azure-resource-manager"
/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="03/04/2016"
	wacn.date="06/02/2016"/>

# 将用户添加到 Azure VM

对于新启动的 Linux VM，首要任务之一就是创建新用户。在本文中，我们将逐步创建 sudo 用户帐户、设置密码、添加 SSH 公钥，最后使用 `visudo` 来允许不带密码的 sudo。

先决条件：[Azure 帐户](/pricing/1rmb-trial/)、[SSH 公钥与私钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

## 快速命令

	# Add a new user on RedHat family distros
	bill@slackware$ sudo useradd -G wheel exampleUser
	
	# Add a new user on Debian family distros
	bill@slackware$ sudo useradd -G sudo exampleUser
	
	# Set a password
	bill@slackware$ sudo passwd exampleUser
	Enter new UNIX password:
	Retype new UNIX password:
	passwd: password updated successfully
	
	# Copy the SSH Public Key to the new user
	bill@slackware$ ssh-copy-id -i ~/.ssh/id_rsa exampleuser@exampleserver
	
	# Change sudoers to allow no password
	# Execute visudo as root to edit the /etc/sudoers file
	bill@slackware$ visudo
	
	# On RedHat family distros uncomment this line:
	## Same thing without a password
	# %wheel        ALL=(ALL)       NOPASSWD: ALL
	
	# to this
	## Same thing without a password
	%wheel        ALL=(ALL)       NOPASSWD: ALL
	
	# On Debian family distros change this line:
	# Allow members of group sudo to execute any command
	%sudo   ALL=(ALL:ALL) ALL
	
	# to this
	%sudo   ALL=(ALL) NOPASSWD:ALL
	
	# Verify everything
	# Verify the SSH keys & User account
	bill@slackware$ ssh -i ~/.ssh/id_rsa exampleuser@exampleserver
	
	# Verify sudo access
	exampleuser@exampleserver$ sudo top

## 详细演练

### 介绍

新服务器的最常见首要任务之一是添加用户帐户。Root 登录应该始终禁用，root 帐户本身甚至不应与 Linux 服务器一起使用，只有 sudo 可以。创建新的用户，再使用 sudo 提升新用户的 root 权限，才是管理和使用 Linux 的正确方式。

我们要使用的命令是 `useradd`，它将修改 `/etc/passwd`、`/etc/shadow`、`/etc/group` 和 `/etc/gshadow` 以创建新的 Linux 用户。我们将在 `useradd` 命令中添加一个命令行标志，以便同时将新用户添加到正确的 Linux sudo 组。尽管 `useradd` 在 `/etc/passwd` 中创建项目，但不会为新用户帐户提供密码。我们将使用非常简单的 `passwd` 命令为新用户创建初始密码。最后一个步骤是修改 sudo 规则，以允许用户以 sudo 权限运行命令，而无需在执行每个命令时都要输入密码。由于该用户已使用公钥与私钥对登录，我们可以推断该用户帐户并非想要图谋不轨，因此允许该用户在不使用密码的情况下进行 sudo 访问。

### 将单个 sudo 用户添加到 Azure VM

使用 SSH 密钥登录到 Azure VM。

`useradd` 命令将完成以下任务：

- 创建新用户帐户
- 以相同的名称创建新用户组
- 在 `/etc/passwd` 中添加空白条目
- 在 `/etc/gpasswd` 中添加空白条目

`-G` 命令行标志将新用户帐户添加到正确的 Linux 组，并为新用户帐户分配 root 提升权限。

#### 添加用户

	# On RedHat family distros
	bill@slackware$ sudo useradd -G wheel exampleUser
	
	# On Debian family distros
	bill@slackware$ sudo useradd -G sudo exampleUser

#### 设置密码

`useradd` 命令会创建新用户，并将一个条目添加到 `/etc/passwd` 和 `/etc/gpasswd` 中，但不真正设置密码。我们现在将使用 `passwd` 命令来设置密码。

	bill@slackware$ sudo passwd exampleUser
	Enter new UNIX password:
	Retype new UNIX password:
	passwd: password updated successfully

现在，服务器上已有一个具有 sudo 权限的用户。

### 将 SSH 公钥添加到新用户帐户

在你的计算机上，结合刚刚设置的新密码使用 `ssh-copy-id` 命令。

	bill@slackware$ ssh-copy-id -i ~/.ssh/id_rsa exampleuser@exampleserver

### 通过 visudo 允许使用不带密码的 sudo

使用 `visudo` 编辑 `/etc/sudoers` 文件可提供多层保护，避免不当地修改这个重要文件。执行 `visudo` 时将锁定 `/etc/sudoers` 文件，以确保没有任何其他人能够在编辑文件时对文件进行更改。当你尝试保存或退出时，`visudo` 还会检查 `/etc/sudoers` 文件中是否有错误。这可确保不会在系统上留下损坏的 sudoers 文件。

我们已将用户添加到正确的默认组中以进行 sudo 访问。现在我们要让这些组可以在没有密码的情况下使用 sudo。

	# Execute visudo as root to edit the /etc/sudoers file
	bill@slackware$ visudo
	
	# On RedHat family distros uncomment this line:
	## Same thing without a password
	# %wheel        ALL=(ALL)       NOPASSWD: ALL
	
	# to this
	## Same thing without a password
	%wheel        ALL=(ALL)       NOPASSWD: ALL
	
	# On Debian family distros change this line:
	# Allow members of group sudo to execute any command
	%sudo   ALL=(ALL:ALL) ALL
	
	# to this
	%sudo   ALL=(ALL) NOPASSWD:ALL

### 验证用户、ssh 密钥和 sudo

	# Verify the SSH keys & User account
	bill@slackware$ ssh -i ~/.ssh/id_rsa exampleuser@exampleserver
	
	# Verify sudo access
	exampleuser@exampleserver$ sudo top

<!---HONumber=Mooncake_0503_2016-->