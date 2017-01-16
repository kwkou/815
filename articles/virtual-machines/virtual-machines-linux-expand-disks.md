<properties
    pageTitle="在 Azure 中扩展 Linux VM 上的 OS 磁盘 | Azure"
    description="了解如何使用 Azure CLI 和 Resource Manager 部署模型扩展 Linux VM 上的操作系统 (OS) 虚拟磁盘"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ms.service: virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="11/22/2016"
    wacn.date="01/13/2017"
    ms.author="iainfou" />  


# 使用 Azure CLI 扩展 Linux VM 上的 OS 磁盘
在 Azure 中的 Linux 虚拟机 (VM) 上，操作系统 (OS) 的默认虚拟硬盘大小通常为 30 GB。用户可以[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)以提供额外的存储空间，但用户还可能想要扩展 OS 磁盘。本文详细介绍如何使用 Azure CLI 扩展 Linux VM 的 OS 磁盘。


## 先决条件
需要安装[最新的 Azure CLI](/documentation/articles/xplat-cli-install/)，然后按如下所示，使用 Resource Manager 模式登录 [Azure 帐户](/pricing/1rmb-trial/)：

    azure config mode arm

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup` 和 `myVM`。


## 扩展 OS 磁盘

1. VM 正在运行时，无法在虚拟硬盘上执行操作。以下示例停止并解除分配名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM：

        azure vm deallocate --resource-group myResourceGroup --name myVM

> [AZURE.NOTE]
    > `azure vm stop` 不会释放计算资源。若要释放计算资源，请使用 `azure vm deallocate`。若要扩展虚拟硬盘，必须解除分配 VM。

2. 使用 `azure vm set` 命令更新 OS 磁盘的大小。以下示例将名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM 更新为 `50` GB：

        azure vm set --resource-group myResourceGroup --name myVM --new-os-disk-size 50

3. 启动 VM，如下所示：

        azure vm start --resource-group myResourceGroup --name myVM

4. 使用相应的凭据通过 SSH 连接到 VM。若要验证 OS 磁盘是否已调整大小，请使用 `df -h`。以下示例输出显示主分区 (`/dev/sda1`) 现在为 50 GB：

        Filesystem      Size  Used Avail Use% Mounted on
        udev            1.7G     0  1.7G   0% /dev
        tmpfs           344M  5.0M  340M   2% /run
        /dev/sda1        49G  1.3G   48G   3% /

## 后续步骤
如果需要附加存储，还可以[将数据磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)。有关磁盘加密的详细信息，请参阅[使用 Azure CLI 加密 Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-encrypt-disks/)。

<!---HONumber=Mooncake_0109_2017-->