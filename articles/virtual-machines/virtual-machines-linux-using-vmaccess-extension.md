<properties
    pageTitle="使用 VMAccess 扩展重置 Azure Linux VM 上的访问权限 | Azure"
    description="使用 VMAccess 扩展重置 Azure Linux VM 上的访问权限。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="261a9646-1f93-407e-951e-0be7226b3064"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/25/2016"
    wacn.date="12/20/2016"
    ms.author="v-livech" />

# 管理用户、SSH，并使用 VMAccess 扩展检查或修复 Azure Linux VM 上的磁盘
本文说明如何使用 Azure VMAcesss 扩展检查或修复磁盘、重置用户访问权限、管理用户帐户，或重置 Linux 上的 SSHD 配置。本文需要以下条件：

* 一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）
* 已使用 `azure login -e AzureChinaCloud` 登录 [Azure CLI](/documentation/articles/xplat-cli-install/)。
* Azure CLI *必须处于* Azure Resource Manager 模式 `azure config mode arm`。

## 快速命令
有两种方法可在 Linux VM 上使用 VMAccess：

* 使用 Azure CLI 以及所需的参数。
* 使用 VMAccess 处理和操作的原始 JSON 文件。

在快速命令部分，我们将使用 Azure CLI `azure vm reset-access` 方法。在以下命令示例中，请将包含“example”的值替换为自己环境中的值。

## 创建资源组和 Linux VM

    azure group create myResourceGroup chinanorth

## 创建 Debian VM

    azure vm quick-create \
      -M ~/.ssh/id_rsa.pub \
      -u myAdminUser \
      -g myResourceGroup \
      -l chinanorth \
      -y Linux \
      -n myVM \
      -Q Debian

## 重置根密码
重置根密码：

    azure vm reset-access \
      -g myResourceGroup \
      -n myVM \
      -u root \
      -p myNewPassword

## SSH 密钥重置
重置非根用户的 SSH 密钥：

    azure vm reset-access \
      -g myResourceGroup \
      -n myVM \
      -u myAdminUser \
      -M ~/.ssh/id_rsa.pub

## 创建用户
创建用户：

    azure vm reset-access \
      -g myResourceGroup \
      -n myVM \
      -u myAdminUser \
      -p myAdminUserPassword

## 删除用户

    azure vm reset-access \
      -g myResourceGroup \
      -n myVM \
      -R myRemovedUser

## 重置 SSHD
重置 SSHD 配置：

    azure vm reset-access \
      -g myResourceGroup \
      -n myVM
      -r

## 详细演练
### 定义的 VMAccess：
Linux VM 上的磁盘显示错误。不知道怎样重置 Linux VM 的根密码，或者不小心删除了 SSH 私钥。如果在数据中心时代发生这种情况，则需要开车到那里，然后打开 KVM 访问服务器控制台。请将 Azure VMAccess 扩展想像成该 KVM 交换机，它允许你访问控制台以重置 Linux 访问或执行磁盘级维护。

在详细演练中，将采用使用原始 JSON 文件的 VMAccess 的长格式。从 Azure 模板也可以调用这些 VMAccess JSON 文件。

### 使用 VMAccess 检查或修复 Linux VM 的磁盘
使用 VMAccess 可以运行在 Linux VM 磁盘上运行的 fsck。还可以使用 VMAccess 执行磁盘检查和磁盘修复。

若要检查并修复磁盘，请使用此 VMAccess 脚本：

`disk_check_repair.json`  


    {
      "check_disk": "true",
      "repair_disk": "true, user-disk-name"
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path disk_check_repair.json

### 使用 VMAccess 重置 Linux 的用户访问权限
如果已失去 Linux VM 的根访问权限，可以启动 VMAccess 脚本重置根密码。

若要重置根密码，请使用此 VMAccess 脚本：

`reset_root_password.json`  


    {
      "username":"root",
      "password":"myNewPassword",   
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path reset_root_password.json

若要重置非根用户的 SSH 密钥，请使用此 VMAccess 脚本：

`reset_ssh_key.json`  


    {
      "username":"myAdminUser",
      "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myAdminUser@myVM",   
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path reset_ssh_key.json


### 使用 VMAccess 管理 Linux 上的用户帐户
VMAccess 是一种 Python 脚本，可用于管理 Linux VM 上的用户，而不需要登录和使用 sudo 或根帐户。

若要创建用户，请使用此 VMAccess 脚本：

`create_new_user.json`  


    {
    "username":"myNewUser",
    "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
    "password":"myNewUserPassword",
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path create_new_user.json

若要删除用户，请使用此 VMAccess 脚本：

`remove_user.json`  


    {
    "remove_user":"myDeletedUser",
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path remove_user.json

### 使用 VMAccess 重置 SSHD 配置
如果你更改了 Linux VM SSHD 配置，并在验证更改之前关闭了 SSH 连接，则可能无法恢复 SSH 操作。使用 VMAccess 可将 SSHD 配置重置回已知正常的配置，而无需通过 SSH 登录。

使用此 VMAccess 脚本重置 SSHD 配置：

`reset_sshd.json`  


    {
      "reset_ssh": true
    }

结合以下参数执行 VMAccess 脚本：

    azure vm extension set \
      myResourceGroup \
      myVM \
      VMAccessForLinux \
      Microsoft.OSTCExtensions * \
      --private-config-path reset_sshd.json

## 后续步骤
使用 Azure VMAccess 扩展更新 Linux 是一种对正在运行的 Linux VM 进行更改的方法。还可以使用 cloud-init 和 Azure 模板之类的工具在 Linux VM 启动时对其进行修改。

[关于虚拟机扩展和功能](/documentation/articles/virtual-machines-linux-extensions-features/)

[使用 Linux VM 扩展创作 Azure Resource Manager 模板](/documentation/articles/virtual-machines-linux-extensions-authoring-templates/)

[在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)

<!---HONumber=Mooncake_1212_2016-->