<properties
    pageTitle="配合使用 VMAccess 扩展和 Azure CLI 2.0 来重置访问权限 | Azure"
    description="如何使用 VMAccess 扩展和 Azure CLI 2.0 在 Linux VM 上管理用户和重置访问权限"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="261a9646-1f93-407e-951e-0be7226b3064"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/16/2017"
    wacn.date="04/24/2017"
    ms.author="v-livech"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="fd9f74c9312517f63af44ab87d4abb3b0295a023"
    ms.lasthandoff="04/14/2017" />

# <a name="manage-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli-20"></a>配合使用 VMAccess 扩展和 Azure CLI 2.0 管理用户、SSH，并检查或修复 Linux VM 上的磁盘
Linux VM 上的磁盘显示错误。 不知道怎样重置 Linux VM 的根密码，或者不小心删除了 SSH 私钥。 如果在数据中心时代发生这种情况，则需要开车到那里，然后打开 KVM 访问服务器控制台。 请将 Azure VMAccess 扩展想像成该 KVM 交换机，它允许你访问控制台以重置 Linux 访问或执行磁盘级维护。

本文说明如何使用 Azure VMAcesss 扩展检查或修复磁盘、重置用户访问权限、管理用户帐户，或重置 Linux 上的 SSHD 配置。 也可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-using-vmaccess-extension-nodejs/) 执行这些步骤。

## <a name="ways-to-use-the-vmaccess-extension"></a>使用 VMAccess 扩展的方法
可通过两种方法在 Linux VM 上使用 VMAccess 扩展：

* 使用 Azure CLI 2.0 和所需参数。
* [使用 VMAccess 要处理和操作的原始 JSON 文件](#use-json-files-and-the-vmaccess-extension)。

以下示例使用 [az vm access](https://docs.microsoft.com/zh-cn/cli/azure/vm/access) 及相应参数。 若要执行这些步骤，需要安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="reset-ssh-key"></a>重置 SSH 密钥
以下示例重置名为 `myVM` 的 VM 上用户 `azureuser` 的 SSH 密钥：

    az vm access set-linux-user \
      --resource-group myResourceGroup \
      --name myVM \
      --username azureuser \
      --ssh-key-value ~/.ssh/id_rsa.pub

## <a name="reset-password"></a>重置密码
以下示例重置名为 `myVM` 的 VM 上用户 `azureuser` 的密码：

    az vm access set-linux-user \
      --resource-group myResourceGroup \
      --name myVM \
      --username azureuser \
      --password myNewPassword

## <a name="reset-sshd"></a>重置 SSHD
以下示例重置名为 `myVM`的 VM 上的 SSHD 配置：

    az vm access reset-linux-ssh \
      --resource-group myResourceGroup \
      --name myVM

## <a name="create-a-user"></a>创建用户
以下示例使用用于身份验证的 SSH 密钥在名为 `myVM` 的 VM 上创建名为 `myNewUser` 的用户：

    az vm access set-linux-user \
      --resource-group myResourceGroup \
      --name myVM \
      --username myNewUser \
      --ssh-key-value ~/.ssh/id_rsa.pub

## <a name="deletes-a-user"></a>删除用户
以下示例删除名为 `myVM` 的 VM 上名为 `myNewUser` 的用户：

    az vm access delete-linux-user \
      --resource-group myResourceGroup \
      --name myVM \
      --username myNewUser

## <a name="use-json-files-and-the-vmaccess-extension"></a>使用 JSON 文件和 VMAccess 扩展
以下示例使用原始 JSON 文件。 然后使用 [az vm extension set](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) 调用 JSON 文件。 也可以从 Azure 模板调用这些 JSON 文件。 

### <a name="reset-user-access"></a>重置用户访问权限
如果已失去 Linux VM 的 root 访问权限，可以启动 VMAccess 脚本来重置用户密码。

若要重置用户的 SSH 密钥，请创建名为 `reset_ssh_key.json` 的文件并添加以下内容：

    {
      "username":"azureuser",
      "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings reset_ssh_key.json

若要重置用户密码，请创建名为 `reset_user_password.json` 的文件并添加以下内容：

    {
      "username":"azureuser",
      "password":"myNewPassword" 
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings reset_user_password.json

### <a name="reset-ssh"></a>重置 SSH
如果你更改了 Linux VM SSHD 配置，并在验证更改之前关闭了 SSH 连接，则可能无法恢复 SSH 操作。  使用 VMAccess 可将 SSHD 配置重置回已知正常的配置，而无需通过 SSH 登录。

若要重置 SSHD 配置，请创建名为 `reset_sshd.json` 的文件并添加以下内容：

    {
      "reset_ssh": true
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings reset_sshd.json

### <a name="manage-users"></a>管理用户
VMAccess 是一种 Python 脚本，可用于管理 Linux VM 上的用户，而不需要登录和使用 sudo 或根帐户。

若要创建用户，请创建名为 `create_new_user.json` 的文件并添加以下内容：

    {
      "username":"myNewUser",
      "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
      "password":"myNewUserPassword"
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings create_new_user.json

若要删除用户，请创建名为 `delete_user.json` 的文件并添加以下内容：

    {
      "remove_user":"myDeleteUser"
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings delete_user.json

### <a name="check-or-repair-the-disk"></a>检查或修复磁盘
使用 VMAccess 可以运行在 Linux VM 磁盘上运行的 fsck。 还可以使用 VMAccess 执行磁盘检查和磁盘修复。

若要使用此 VMAccess 脚本检查和修复磁盘，请创建名为 `disk_check_repair.json` 的文件并添加以下内容：

    {
      "check_disk": "true",
      "repair_disk": "true, user-disk-name"
    }

结合以下参数执行 VMAccess 脚本：

    az vm extension set \
      --resource-group myResourceGroup \
      --vm-name myVM \
      --name VMAccessForLinux \
      --publisher Microsoft.OSTCExtensions \
      --version 1.4 \
      --protected-settings disk_check_repair.json

## <a name="next-steps"></a>后续步骤
使用 Azure VMAccess 扩展更新 Linux 是对运行中 Linux VM 进行更改的一种方法。 也可以使用 cloud-init 等工具和 Azure Resource Manager 在 Linux VM 启动时对其进行修改。

[关于虚拟机扩展和功能](/documentation/articles/virtual-machines-linux-extensions-features/)

[使用 Linux VM 扩展创作 Azure Resource Manager 模板](/documentation/articles/virtual-machines-linux-extensions-authoring-templates/)

[在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)
<!--Update_Description: wording update-->