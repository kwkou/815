<!-- ARM: tested -->

<properties
    pageTitle="在创建期间使用 cloud-init 自定义 Linux VM | Azure"
    description="在创建期间使用 cloud-init 自定义 Linux VM。"
    services="virtual-machines-linux"
    documentationCenter=""
    authors="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/29/2016"
	wacn.date="06/20/2016"/>

# 在创建期间使用 cloud-init 自定义 Linux VM

本文说明如何制作 cloud-init 脚本来设置主机名、更新已安装的包及管理用户帐户。然后，可以在 VM 创建期间从 [Azure CLI](/documentation/articles/xplat-cli-install/) 使用这些 cloud-init 脚本。

## 先决条件

先决条件：[Azure 帐户](/pricing/1rmb-trial/)、[SSH 公钥与私钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)、在其中启动 Linux VM 的 Azure 资源组、已安装 Azure CLI，并已使用 `azure config mode arm` 切换到 ARM 模式。

## 介绍

启动新 Linux VM 时，你将获得一个未经过任何自定义或者不能够现成地满足需求的标准 Linux VM。[Cloud-init](https://cloudinit.readthedocs.org) 是在首次启动 Linux VM 时在其中注入脚本或配置设置的标准方法。

Azure 有三种不同的方法可在 Linux VM 启动时进行更改。

- 可以使用 cloud-init 注入脚本。
- 可以使用 Azure [CustomScriptExtention](/documentation/articles/virtual-machines-linux-extensions-customscript/) 注入脚本。
- 可以在 Azure 模板中指定自定义设置，并且用它来启动及自定义 Linux VM，此模板中包括 cloud-init 支持以及 CustomScript VM 扩展和许多其他功能。

若要随时注入脚本，可以：

- 使用 SSH 直接运行命令，可以命令方式或在 Azure 模板中使用 Azure [CustomScriptExtention](/documentation/articles/virtual-machines-linux-extensions-customscript/)，或者使用 Ansible、Salt、Chef 及 Puppet 等通用配置管理工具，这些工具在 VM 启动完成后在 SSH 上工作。

注意：尽管 [CustomScriptExtention](/documentation/articles/virtual-machines-linux-extensions-customscript/) 和使用 SSH 时一样只将脚本作为 root 运行，但是使用 VM 扩展可让 Azure 提供的几项功能发挥功用，具体取决于你的方案。

## 快速命令

创建主机名 cloud-init 脚本

	#cloud-config
	hostname: exampleServerName

适用于 Debian 系列的第一次启动创建更新 Linux cloud-init 脚本

	#cloud-config
	apt_upgrade: true

创建和添加用户 cloud-init 脚本

	#cloud-config
	users:
	  - name: exampleUser
	    groups: sudo
	    shell: /bin/bash
	    sudo: ['ALL=(ALL) NOPASSWD:ALL']
	    ssh-authorized-keys:
	      - ssh-rsa AAAAB3<snip>==exampleuser@slackwarelaptop

## 详细演练

### 将 cloud-init 脚本添加使用 Azure CLI 创建 VM 的操作中

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

在 Azure 中创建 VM 时，若要启动 cloud-init 脚本，请使用 Azure CLI `--custom-data` 开关来指定 cloud-init 文件。

注意：本文介绍内容尽管是使用 `--custom-data` 开关处理 cloud-init 文件，但也可以使用此参数来发送任意代码或文件。如果 Linux VM 已知道要用此类文件做什么，它们将自动执行。

	azure vm create \
	--resource-group exampleRG \
	--name exampleVM \
	--location chinanorth \
	--admin-username exampleAdminUserName \
	--os-type Linux \
	--nic-name exampleNIC \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--custom-data cloud_init_script.txt

### 创建 cloud-init 脚本以设置 Linux VM 的主机名

对任何 Linux VM 而言，其中一个最简单且最重要的设置就是主机名。使用 cloud-init 和此脚本就可以轻松设置此项。

#### 名为 `cloud_config_hostname.txt` 的示例 cloud-init 脚本。

	#cloud-config
	hostname: exampleServerName

在 VM 首次启动期间，此 cloud-init 脚本将主机名设置为 `exampleServerName`。

	azure vm create \
	--resource-group exampleRG \
	--name exampleVM \
	--location chinanorth \
	--admin-username exampleAdminUserName \
	--os-type Linux \
	--nic-name exampleNIC \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--custom-data cloud_config_hostname.txt

登录并验证新 VM 的主机名。

	ssh exampleVM
	hostname
	exampleServerName

### 创建 cloud-init 脚本以更新 Linux

为确保安全，你希望 Ubuntu VM 能在首次启动时更新。根据所用的 Linux 分发版，我们可以使用 cloud-init 和以下脚本执行此操作。

#### 适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_apt_upgrade.txt`

	#cloud-config
	apt_upgrade: true

新 Linux VM 启动之后，它通过 `apt-get` 立即更新所有已安装的包。

	azure vm create \
	--resource-group exampleRG \
	--name exampleVM \
	--location chinanorth \
	--admin-username exampleAdminUserName \
	--os-type Linux \
	--nic-name exampleNIC \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--custom-data cloud_config_apt_upgrade.txt

登录并验证所有包是否都已更新。

	ssh exampleVM
	sudo apt-get upgrade
	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	Calculating upgrade... Done
	The following packages have been kept back:
	  linux-generic linux-headers-generic linux-image-generic
	0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

### 创建 cloud-init 脚本以将用户添加到 Linux

任何新 Linux VM 的首批任务之一，就是将自己添加为用户或避免使用 `root`。将 SSH 公钥添加该用户的 `~/.ssh/authorized_keys` 文件中后，进行密码较少且安全的 SSH 登录，对安全和使用性而言都是绝佳的做法。

#### 适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_add_users.txt`

	#cloud-config
	users:
	  - name: exampleUser
	    groups: sudo
	    shell: /bin/bash
	    sudo: ['ALL=(ALL) NOPASSWD:ALL']
	    ssh-authorized-keys:
	      - ssh-rsa AAAAB3<snip>==exampleuser@slackwarelaptop

新的 Linux VM 启动后，创建新用户并将其添加到 sudo 组中。

	azure vm create \
	--resource-group exampleRG \
	--name exampleVM \
	--location chinanorth \
	--admin-username exampleAdminUserName \
	--os-type Linux \
	--nic-name exampleNIC \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--custom-data cloud_config_add_users.txt

登录并验证新建的用户。

	cat /etc/group

输出

	root:x:0:
	<snip />
	sudo:x:27:exampleUser
	<snip />
	exampleUser:x:1000:


<!---HONumber=Mooncake_0613_2016-->