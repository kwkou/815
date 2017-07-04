<properties
    pageTitle="使用 cloud-init 自定义 Linux VM | Azure"
    description="如何通过 Azure CLI 2.0 预览版使用 cloud-init 在创建期间自定义 Linux VM"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="195c22cd-4629-4582-9ee3-9749493f1d72"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/10/2017"
    wacn.date="04/24/2017"
    ms.author="iainfou"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="b08255187104d87ae59dbb6974f895ea4c8cfb9e"
    ms.lasthandoff="04/14/2017" />

# <a name="use-cloud-init-to-customize-a-linux-vm-during-creation"></a>在创建期间使用 cloud-init 自定义 Linux VM
本文说明如何使用 Azure CLI 2.0 制作 cloud-init 脚本来设置主机名、更新已安装的包及管理用户帐户。  可以在创建 VM 时从 Azure CLI 调用 cloud-init 脚本。  还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-using-cloud-init-nodejs/) 执行这些步骤。

## <a name="quick-commands"></a>快速命令
创建 cloud-init.txt 脚本，用于设置主机名、更新所有包，并将 sudo 用户添加到 Linux。

    #cloud-config
    hostname: myVMhostname
    apt_upgrade: true
    users:
      - name: myNewAdminUser
        groups: sudo
        shell: /bin/bash
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
        ssh-authorized-keys:
          - ssh-rsa AAAAB3<snip>==myAdminUser@myVM

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建一个要在其中启动 VM 的资源组。 以下示例创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

使用 cloud-init 通过 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要在启动过程中进行配置的 Linux VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --custom-data cloud-init.txt \
	--use-unmanaged-disk

## <a name="detailed-walkthrough"></a>详细演练
### <a name="introduction"></a>介绍
启动新 Linux VM 时，将获得一个未经过任何自定义或者不能够现成地满足需求的标准 Linux VM。 [Cloud-init](https://cloudinit.readthedocs.org) 是在首次启动 Linux VM 时在其中注入脚本或配置设置的标准方法。

Azure 有三种不同的方法可在部署或启动 Linux VM 时对其进行更改。

* 使用 cloud-init 注入脚本。
* 使用 Azure [VMAccess 扩展](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)注入脚本。
* 使用 cloud-init 的 Azure 模板。
* 使用 [CustomScriptExtention](/documentation/articles/virtual-machines-linux-extensions-customscript/) 的 Azure 模板。

若要在启动后随时注入脚本，请执行以下操作：

* 直接通过 SSH 运行命令
* 以命令方式或在 Azure 模板中使用 Azure [VMAccess 扩展](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)注入脚本
* Ansible、Salt、Chef 和 Puppet 等配置管理工具。

> [AZURE.NOTE]
> VMAccess 扩展以使用 SSH 可以进行的相同方式以根用户身份执行脚本。  但是，使用 VM 扩展可以启用 Azure 提供的几项功能，这些功能可以很有用，具体取决于用户的方案。
> 
> 

## <a name="cloud-init-availability-on-azure-vm-quick-create-image-aliases"></a>Azure VM 上的 Cloud-init 可用性快速创建映像别名：
| 别名 | 发布者 | 产品 | SKU | 版本 | Cloud-init |
|:--- |:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |Centos |7.2 |最新 |否 |
| CoreOS |CoreOS |CoreOS |Stable |最新 |是 |
| Debian |credativ |Debian |8 |最新 |否 |
| openSUSE |SUSE |openSUSE |13.2 |最新 |否 |
| UbuntuLTS |Canonical |UbuntuServer |14.04.3-LTS |最新 |是 |

Microsoft 正在与合作伙伴协作将 cloud-init 包含在用户向 Azure 提供的映像中并让它在其中正常工作。

## <a name="add-a-cloud-init-script-to-the-vm-creation-with-the-azure-cli"></a>将 cloud-init 脚本添加使用 Azure CLI 创建 VM 的操作中
在 Azure 中创建 VM 时，若要启动 cloud-init 脚本，请使用 Azure CLI `--custom-data` 开关来指定 cloud-init 文件。

创建要在其中启动 VM 的资源组。

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建一个要在其中启动 VM 的资源组。 以下示例创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

使用 cloud-init 通过 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要在启动过程中进行配置的 Linux VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --custom-data cloud-init.txt \
	--use-unmanaged-disk

## <a name="create-a-cloud-init-script-to-set-the-hostname-of-a-linux-vm"></a>创建 cloud-init 脚本以设置 Linux VM 的主机名
对任何 Linux VM 而言，其中一个最简单且最重要的设置就是主机名。 使用 cloud-init 和此脚本就可以轻松设置此项。  

### <a name="example-cloud-init-script-named-cloudconfighostnametxt"></a>名为 `cloud_config_hostname.txt`的示例 cloud-init 脚本。

    #cloud-config
    hostname: myservername

初始启动 VM 期间，此 cloud-init 脚本将主机名设置为 `myservername`。 使用 cloud-init 通过 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要在启动过程中进行配置的 Linux VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --custom-data cloud-init.txt \
	--use-unmanaged-disk

登录并验证新 VM 的主机名。

    ssh myVM
    hostname
    myservername

## <a name="create-a-cloud-init-script-to-update-linux"></a>创建 cloud-init 脚本以更新 Linux
为了安全，用户希望 Ubuntu VM 在首次启动时更新。  根据所用的 Linux 分发，我们可以使用 cloud-init 和以下脚本执行此操作。

### <a name="example-cloud-init-script-cloudconfigaptupgradetxt-for-the-debian-family"></a>适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_apt_upgrade.txt`

    #cloud-config
    apt_upgrade: true

启动 Linux 后，所有已安装的包将通过 `apt-get` 进行更新。 使用 cloud-init 通过 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要在启动过程中进行配置的 Linux VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --custom-data cloud_config_apt_upgrade.txt \
	--use-unmanaged-disk

登录并验证所有包是否都已更新。

    ssh myUbuntuVM
    sudo apt-get upgrade
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    Calculating upgrade... Done
    The following packages have been kept back:
      linux-generic linux-headers-generic linux-image-generic
    0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

## <a name="create-a-cloud-init-script-to-add-a-user-to-linux"></a>创建 cloud-init 脚本以将用户添加到 Linux
任何新 Linux VM 的首要任务之一，就是为自己添加用户或避免使用 `root`。 对于安全性和易用性来说，SSH 密钥是最佳做法，可以使用此 cloud-init 脚本将它们添加到 `~/.ssh/authorized_keys` 文件。

### <a name="example-cloud-init-script-cloudconfigadduserstxt-for-debian-family"></a>适用于 Debian 系列的示例 cloud-init 脚本 `cloud_config_add_users.txt`

    #cloud-config
    users:
      - name: myCloudInitAddedAdminUser
        groups: sudo
        shell: /bin/bash
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
        ssh-authorized-keys:
          - ssh-rsa AAAAB3<snip>==myAdminUser@myUbuntuVM

Linux 启动后，所有列出的用户都已创建并添加到 sudo 组。 使用 cloud-init 通过 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要在启动过程中进行配置的 Linux VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --custom-data cloud_config_add_users.txt \
	--use-unmanaged-disk

登录并验证新建的用户。

    ssh myVM
    cat /etc/group

输出

    root:x:0:
    <snip />
    sudo:x:27:myCloudInitAddedAdminUser
    <snip />
    myCloudInitAddedAdminUser:x:1000:

## <a name="next-steps"></a>后续步骤
Cloud-init 正在成为一种在 Linux VM 启动时对其进行修改的标准方法。 Azure 还提供 VM 扩展，使用这些扩展可以在 LinuxVM 启动或运行时对其进行修改。 例如，可以使用 Azure VMAccessExtension 在 VM 运行时重置 SSH 或用户信息。 使用 cloud-init，需要重新启动才能重置密码。

[关于虚拟机扩展和功能](/documentation/articles/virtual-machines-linux-extensions-features/)

[管理用户、SSH，并使用 VMAccess 扩展检查或修复 Azure Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->