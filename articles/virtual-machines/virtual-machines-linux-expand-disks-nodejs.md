<properties
    pageTitle="使用 Azure CLI 1.0 扩展 Linux VM 上的 OS 磁盘 | Azure"
    description="了解如何使用 Azure CLI 1.0 和 Resource Manager 部署模型扩展 Linux VM 上的操作系统 (OS) 虚拟磁盘"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/10/2017"
    wacn.date="04/17/2017"
    ms.author="iainfou"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="ac837ea672be97fe89ce7fc30833f3dad8e006e1"
    ms.lasthandoff="04/06/2017" />

# <a name="expand-os-disk-on-a-linux-vm-using-the-azure-cli-with-the-azure-cli-10"></a>结合使用 Azure CLI 和 Azure CLI 1.0 扩展 Linux VM 上的 OS 磁盘
在 Azure 的 Linux 虚拟机 (VM) 上，操作系统 (OS) 的默认虚拟硬盘大小通常为 30 GB。 可以通过[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)来扩充存储空间，也可扩展 OS 磁盘。 本文详述如何使用 Azure CLI 1.0 为使用非托管磁盘的 Linux VM 扩展 OS 磁盘

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#prerequisites) - 适用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0](/documentation/articles/virtual-machines-linux-expand-disks/) - 适用于资源管理部署模型的下一代 CLI

## <a name="prerequisites"></a>先决条件
需要安装[最新的 Azure CLI 1.0](/documentation/articles/cli-install-nodejs/)，然后按如下所示，使用 Resource Manager 模式登录 [Azure 帐户](/pricing/1rmb-trial/)：

    azure config mode arm

在以下示例中，请将示例参数名称替换为你自己的值。 示例参数名称包括 `myResourceGroup` 和 `myVM`。

## <a name="expand-os-disk"></a>扩展 OS 磁盘

1. VM 正在运行时，无法在虚拟硬盘上执行操作。 以下示例在名为 `myResourceGroup` 的资源组中停止和释放名为 `myVM` 的 VM：

        azure vm deallocate --resource-group myResourceGroup --name myVM

    > [AZURE.NOTE]
    > `azure vm stop` 不释放计算资源。 若要释放计算资源，请使用 `azure vm deallocate`。 只有释放 VM 才能扩展虚拟硬盘。

2. 使用 `azure vm set` 命令更新 OS 非托管磁盘的大小。 以下示例将名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM 更新为 `50` GB：

        azure vm set --resource-group myResourceGroup --name myVM --new-os-disk-size 50

3. 启动 VM，如下所示：

        azure vm start --resource-group myResourceGroup --name myVM

4. 使用相应的凭据通过 SSH 连接到 VM。 若要验证是否已调整 OS 磁盘的大小，请使用 `df -h`。 以下示例输出显示主分区 (`/dev/sda1`) 现在为 50 GB：

        Filesystem      Size  Used Avail Use% Mounted on
        udev            1.7G     0  1.7G   0% /dev
        tmpfs           344M  5.0M  340M   2% /run
        /dev/sda1        49G  1.3G   48G   3% /

## <a name="next-steps"></a>后续步骤
如需更多存储，也可[向 Linux VM 添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。 有关磁盘加密的详细信息，请参阅[使用 Azure CLI 加密 Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-encrypt-disks/)。