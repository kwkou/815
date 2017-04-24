<properties
    pageTitle="创建和使用适用于 Azure 中 Linux VM 的 SSH 密钥对 | Azure"
    description="如何创建和使用适用于 Azure 中 Linux VM 的 SSH 公钥和私钥对，提高身份验证过程的安全性。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    experimental="true"
    experiment_id="rasquill-ssh-20170308"
    translationtype="Human Translation" />
<tags
    ms.assetid="34ae9482-da3e-4b2d-9d0d-9d672aa42498"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/07/2017"
    wacn.date="04/24/2017"
    ms.author="iainfou"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d5c11f2eb0fae1cf081d15d06fc80c47e95aeb07"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-create-and-use-an-ssh-public-and-private-key-pair-for-linux-vms-in-azure"></a>如何创建和使用适用于 Azure 中 Linux VM 的 SSH 公钥和私钥对
使用安全外壳 (SSH) 密钥对，可以在 Azure 上创建使用 SSH 密钥进行身份验证的虚拟机 (VM)，从而无需密码即可登录。 本文介绍如何快速生成和使用适用于 Linux VM 的 SSH 协议版本 2 RSA 公钥和私钥文件对。 如需更详细的步骤和其他示例，例如适用于经典管理门户的步骤和示例，请参阅[创建 SSH 密钥对和证书的详细步骤](/documentation/articles/virtual-machines-linux-create-ssh-keys-detailed/)。

## <a name="create-an-ssh-key-pair"></a>创建 SSH 密钥对
请使用 `ssh-keygen` 命令创建 SSH 公钥和私钥文件，这些文件默认在 `~/.ssh` 目录中创建，但是你可以在系统提示时指定其他位置和其他通行短语（用于访问私钥文件的密码）。 请通过 Bash 外壳程序运行以下命令，在出现提示时使用你自己的信息进行回应。

    ssh-keygen -t rsa -b 2048 

## <a name="use-the-ssh-key-pair"></a>使用 SSH 密钥对
放置在 Azure 中 Linux VM 上的公钥默认存储在 `~/.ssh/id_rsa.pub` 中，除非你在创建该公钥时更改了位置。 如果使用 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure) 创建 VM，请在将 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 与 `--ssh-key-path` 选项结合使用时指定该公钥的位置。 如果复制和粘贴要在 Azure 门户预览或 Resource Manager 模板中使用的公钥文件的内容，请确保不复制额外的空格。 例如，如果使用 OS X，则可将公钥文件（默认为 **~/.ssh/id_rsa.pub**）通过管道传送到 **pbcopy**，以便复制内容（也可通过其他 Linux 程序（例如 `xclip`）执行此类操作）。 

如果不熟悉 SSH 公钥，则可通过运行 `cat` 来查看公钥（如下所示），注意需将 `~/.ssh/id_rsa.pub` 替换为你自己的公钥文件位置：

    cat ~/.ssh/id_rsa.pub

请使用 Azure VM 上的公钥，通过 SSH 使用 VM 的 IP 地址或 DNS 名称连接到 VM（记住将下面的 `azureuser` 和 `myvm.chinanorth.chinacloudapp.cn` 替换为管理员用户名和完全限定域名或 IP 地址）：

    ssh azureuser@myvm.chinanorth.chinacloudapp.cn

如果你在创建密钥对时提供的是通行短语，则在登录过程中遇到提示时，请输入该通行短语。 （服务器添加到 `~/.ssh/known_hosts` 文件夹。系统不会要求你再次进行连接，除非更改了 Azure VM 上的公钥，或者从 `~/.ssh/known_hosts` 中删除了服务器名称。）

## <a name="next-steps"></a>后续步骤

使用 SSH 密钥创建的 VM 默认配置为禁用密码，使得强力猜测尝试代价相当高昂，因此也更为困难。 本主题介绍如何创建一个简单的、可以快速使用的 SSH 密钥对。 如果你在创建 SSH 密钥对方面需要更多帮助，或者需要其他的证书（例如用于经典管理门户的证书），请参阅[创建 SSH 密钥对和证书的详细步骤](/documentation/articles/virtual-machines-linux-create-ssh-keys-detailed/)。

可以通过 Azure 门户预览、CLI 和模板创建使用 SSH 密钥对的 VM：

* [使用 Azure 门户预览创建安全 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
* [使用 Azure CLI 2.0 创建安全的 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)
* [使用 Azure 模板创建安全 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
<!--Update_Description: wording update-->