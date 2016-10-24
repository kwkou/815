<properties
    pageTitle="在创建期间使用 cloud-init 自定义 Linux VM | Azure"
    description="在创建期间使用 cloud-init 自定义 Linux VM。"
    services="virtual-machines-linux"
    documentationCenter=""
    authors="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
/>  


<tags
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="08/30/2016"
    wacn.date=""
    ms.author="v-livech"
/>  


# 在创建期间使用 cloud-init 自定义 Linux VM

本文说明如何制作 cloud-init 脚本来设置主机名、更新已安装的包及管理用户帐户。在 VM 创建期间可以从 Azure CLI 调用 cloud-init 脚本。

## 先决条件

先决条件包括：[Azure 帐户](/pricing/1rmb-trial/)、[SSH 公钥与私钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)，并已使用 `azure config mode arm` 将 [Azure CLI](/documentation/articles/xplat-cli-install/) 切换到 Azure Resource Manager 模式。

## 快速命令

创建 cloud-init.txt 脚本，用于设置主机名、更新所有包，并将 sudo 用户添加到 Linux。

	#cloud-config
	hostname: exampleServerName
	apt_upgrade: true
	users:
	  - name: exampleUser
	    groups: sudo
	    shell: /bin/bash
	    sudo: ['ALL=(ALL) NOPASSWD:ALL']
	    ssh-authorized-keys:
	      - ssh-rsa AAAAB3<snip>==exampleuser@slackwarelaptop

使用 cloud-init 创建要在启动过程中进行配置的 Linux VM。

	azure group create cloudinitexample chinanorth

<br/>  


	azure vm create \
	--resource-group cloudinitexample \
	--name cloudinitexample \
	--location chinanorth \
	--os-type Linux \
	--nic-name cloudinitnicexample \
	--vnet-name cloudinitvnetexample \
	--vnet-address-prefix 10.0.0.0/22 \
	--vnet-subnet-name cloudinitvsubnet \
	--vnet-subnet-address-prefix 10.0.0.0/24 \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
	--admin-username ahmet \
	--custom-data cloud-init.txt

## 介绍

启动新 Linux VM 时，将获得一个未经过任何自定义或者不能够现成地满足需求的标准 Linux VM。[Cloud-init](https://cloudinit.readthedocs.org) 是在首次启动 Linux VM 时在其中注入脚本或配置设置的标准方法。

Azure 有三种不同的方法可在部署或启动 Linux VM 时对其进行更改。

- 使用 cloud-init 注入脚本。
- 使用 Azure [VMAccess 扩展](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)注入脚本。
- 使用 cloud-init 的 Azure 模板。
- 使用 [CustomScriptExtention](/documentation/articles/virtual-machines-linux-extensions-customscript/) 的 Azure 模板。

若要在启动后随时注入脚本，请执行以下操作：

- 直接通过 SSH 运行命令
- 以命令方式或在 Azure 模板中使用 Azure [VMAccess 扩展](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)注入脚本
- Ansible、Salt、Chef 和 Puppet 等配置管理工具。

>[AZURE.NOTE]\: VMAccess 扩展以使用 SSH 可以进行的相同方式以 root 身份执行脚本。但是，使用 VM 扩展可以启用 Azure 提供的几项功能，这些功能可以很有用，具体取决于用户的方案。

### Azure VM 上的 Cloud-init 可用性快速创建映像别名：

| 别名 | 发布者 | 产品 | SKU | 版本 | cloud-init |
|:----------|:----------|:-------------|:------------|:--------|:-----------|
| CentOS | OpenLogic | Centos | 7\.2 | 最新 | 否 |
| CoreOS | CoreOS | CoreOS | Stable | 最新 | 是 |
| Debian | credativ | Debian | 8 | 最新 | 否 |
| openSUSE | SUSE | openSUSE | 13\.2 | 最新 | 否 |
| UbuntuLTS | Canonical | UbuntuServer | 14\.04.3-LTS | 最新 | 是 |

Microsoft 正在与合作伙伴协作将 cloud-init 包含在用户向 Azure 提供的映像中并让它在其中正常工作。


## 详细演练

### 将 cloud-init 脚本添加使用 Azure CLI 创建 VM 的操作中

在 Azure 中创建 VM 时，若要启动 cloud-init 脚本，请使用 Azure CLI `--custom-data` 开关来指定 cloud-init 文件。

	azure group create cloudinitexample chinanorth

<br/>  


	azure vm create \
	--resource-group cloudinitexample \
	--name cloudinitexample \
	--location chinanorth \
	--os-type Linux \
	--nic-name cloudinitnicexample \
	--vnet-name cloudinitvnetexample \
	--vnet-address-prefix 10.0.0.0/22 \
	--vnet-subnet-name cloudinitvsubnet \
	--vnet-subnet-address-prefix 10.0.0.0/24 \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
	--admin-username ahmet \
	--custom-data cloud-init.txt

### 创建 cloud-init 脚本以设置 Linux VM 的主机名

对任何 Linux VM 而言，其中一个最简单且最重要的设置就是主机名。使用 cloud-init 和此脚本就可以轻松设置此项。

#### 名为 `cloud_config_hostname.txt` 的示例 cloud-init 脚本。

	#cloud-config
	hostname: exampleServerName

初始启动 VM 期间，此 cloud-init 脚本将主机名设置为 `exampleServerName`。

	azure vm create \
	--resource-group cloudinitexample \
	--name cloudinitexample \
	--location chinanorth \
	--os-type Linux \
	--nic-name cloudinitnicexample \
	--vnet-name cloudinitvnetexample \
	--vnet-address-prefix 10.0.0.0/22 \
	--vnet-subnet-name cloudinitvsubnet \
	--vnet-subnet-address-prefix 10.0.0.0/24 \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
	--admin-username ahmet \
	--custom-data cloud_config_hostname.txt

登录并验证新 VM 的主机名。

	ssh exampleVM
	hostname
	exampleServerName

### 创建 cloud-init 脚本以更新 Linux

为了安全，用户希望 Ubuntu VM 在首次启动时更新。根据所用的 Linux 分发版，我们可以使用 cloud-init 和以下脚本执行此操作。

#### 适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_apt_upgrade.txt`

	#cloud-config
	apt_upgrade: true

启动 Linux 后，所有已安装的包将通过 `apt-get` 进行更新。

	azure vm create \
	--resource-group cloudinitexample \
	--name cloudinitexample \
	--location chinanorth \
	--os-type Linux \
	--nic-name cloudinitnicexample \
	--vnet-name cloudinitvnetexample \
	--vnet-address-prefix 10.0.0.0/22 \
	--vnet-subnet-name cloudinitvsubnet \
	--vnet-subnet-address-prefix 10.0.0.0/24 \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
	--admin-username ahmet \
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

任何新 Linux VM 的首要任务之一，就是为自己添加用户或避免使用 `root`。对于安全性和易用性来说，SSH 密钥是最佳做法，可以使用此 cloud-init 脚本将它们添加到 `~/.ssh/authorized_keys` 文件。

#### 适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_add_users.txt`

	#cloud-config
	users:
	  - name: exampleUser
	    groups: sudo
	    shell: /bin/bash
	    sudo: ['ALL=(ALL) NOPASSWD:ALL']
	    ssh-authorized-keys:
	      - ssh-rsa AAAAB3<snip>==exampleuser@slackwarelaptop

Linux 启动后，所有列出的用户都已创建并添加到 sudo 组。

	azure vm create \
	--resource-group cloudinitexample \
	--name cloudinitexample \
	--location chinanorth \
	--os-type Linux \
	--nic-name cloudinitnicexample \
	--vnet-name cloudinitvnetexample \
	--vnet-address-prefix 10.0.0.0/22 \
	--vnet-subnet-name cloudinitvsubnet \
	--vnet-subnet-address-prefix 10.0.0.0/24 \
	--image-urn canonical:ubuntuserver:14.04.2-LTS:latest \
	--ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
	--admin-username ahmet \
	--custom-data cloud_config_add_users.txt

登录并验证新建的用户。

	cat /etc/group

输出

	root:x:0:
	<snip />
	sudo:x:27:exampleUser
	<snip />
	exampleUser:x:1000:

## 后续步骤

Cloud-init 正在成为一种在 Linux VM 启动时对其进行修改的标准方法。Azure 还提供 VM 扩展，使用这些扩展可以在 LinuxVM 启动或运行时对其进行修改。例如，可以使用 Azure VMAccessExtension 在 VM 运行时重置 SSH 或用户信息。使用 cloud-init，需要重新启动才能重置密码。

[关于虚拟机扩展和功能](/documentation/articles/virtual-machines-linux-extensions-features/)

[管理用户、SSH，并使用 VMAccess 扩展检查或修复 Azure Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)

<!---HONumber=Mooncake_1017_2016-->