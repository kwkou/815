<properties
    pageTitle="如何使用 Azure CLI 1.0 调整 Linux VM 的大小 | Azure"
    description="如何通过更改 VM 大小来增加或减少 Linux 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter="na"
    author="mikewasson"
    manager="timlt"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/16/2016"
    wacn.date="04/24/2017"
    ms.author="mwasson"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="70dfb9339bc1b4d5657069c1561865d2f515cbed"
    ms.lasthandoff="04/14/2017" />

# <a name="resize-a-linux-vm-with-azure-cli-10"></a>使用 Azure CLI 1.0 调整 Linux VM 的大小

## <a name="overview"></a>概述

预配虚拟机 (VM) 后，可以通过更改 [VM 大小][vm-sizes]来扩展或缩减 VM。 在某些情况下，必须先解除分配 VM。 如果新大小在托管 VM 的硬件群集上不可用，则可能会出现这种情况。

本文介绍了如何使用 [Azure CLI][azure-cli]来调整 Linux VM 的大小。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#resize-a-linux-vm) - 适用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0](/documentation/articles/virtual-machines-linux-change-vm-size/) - 适用于资源管理部署模型的下一代 CLI

## <a name="resize-a-linux-vm"></a> 调整 Linux VM 的大小
若要调整 VM 的大小，请执行以下步骤。

1. 运行以下 CLI 命令。 此命令将列出托管 VM 的硬件群集上的可用 VM 大小。

        azure vm sizes -g myResourceGroup --vm-name myVM

2. 如果列出了所需大小，请运行以下命令来调整 VM 的大小。

        azure vm set -g myResourceGroup --vm-size <new-vm-size> -n myVM  \
            --enable-boot-diagnostics
            --boot-diagnostics-storage-uri https://mystorageaccount.blob.core.chinacloudapi.cn/ 

    在此过程中，VM 将重新启动。 重新启动后，现有 OS 和数据磁盘将重新映射。 临时磁盘上的所有内容将会丢失。

    使用 `--enable-boot-diagnostics` 选项启用 [启动诊断][boot-diagnostics]，以记录所有与启动相关的错误。
3. 如果未列出所需大小，请运行以下命令来解除分配 VM、调整其大小，然后将它重新启动。

        azure vm deallocate -g myResourceGroup myVM
        azure vm set -g myResourceGroup --vm-size <new-vm-size> -n myVM \
            --enable-boot-diagnostics --boot-diagnostics-storage-uri \
            https://mystorageaccount.blob.core.chinacloudapi.cn/ 
        azure vm start -g myResourceGroup myVM

    > [AZURE.WARNING]
    > 解除分配 VM 也会释放分配给该 VM 的所有动态 IP 地址。 OS 和数据磁盘不受影响。
    > 
    > 

## <a name="next-steps"></a>后续步骤
若要提高可伸缩性，请运行多个 VM 实例并进行横向扩展。

<!-- links -->

[azure-cli]: /documentation/articles/cli-install-nodejs/
[boot-diagnostics]: https://azure.microsoft.com/zh-cn/blog/boot-diagnostics-for-virtual-machines-v2/
[vm-sizes]: /documentation/articles/virtual-machines-linux-sizes/
<!--Update_Description: wording update-->