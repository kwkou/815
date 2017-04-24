<properties
    pageTitle="如何使用 Azure CLI 2.0 调整 Linux VM 的大小 | Azure"
    description="如何通过更改 VM 大小来增加或减少 Linux 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter="na"
    author="mikewasson"
    manager="timlt"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e163f878-b919-45c5-9f5a-75a64f3b14a0"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/10/2017"
    wacn.date="04/24/2017"
    ms.author="mwasson"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="99351e9e2e4cd692a0becacc61bab22bbdfb92b3"
    ms.lasthandoff="04/14/2017" />

# <a name="resize-a-linux-virtual-machine-using-cli-20"></a>使用 CLI 2.0 调整 Linux 虚拟机的大小

预配虚拟机 (VM) 后，可以通过更改 [VM 大小][vm-sizes]来扩展或缩减 VM。 在某些情况下，必须先解除分配 VM。 如果所需大小在托管 VM 的硬件群集上不可用，则需要解除分配 VM。 本文详细介绍了如何使用 Azure CLI 2.0 来调整 Linux VM 的大小。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-change-vm-size-nodejs/) 执行这些步骤。

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-change-vm-size-nodejs/) - 适用于经典部署模型和资源管理部署模型的 CLI
- Azure CLI 2.0 - 下一代 CLI，适用于资源管理部署模型（详见本文）

若要调整 VM 的大小，需要最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 并已使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

1. 使用 [az vm list-vm-resize-options](https://docs.microsoft.com/zh-cn/cli/azure/vm#list-vm-resize-options) 查看托管 VM 的硬件群集上可用的 VM 大小的列表。 以下示例列出资源组 `myResourceGroup` 区域中名为 `myVM` 的 VM 的 VM 大小：

        az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table

2. 如果列出了所需的 VM 大小，则使用 [az vm resize](https://docs.microsoft.com/zh-cn/cli/azure/vm#resize) 调整 VM 大小。 以下示例将名为 `myVM` 的 VM 调整为 `Standard_DS3_v2` 大小：

        az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2

    在此过程中，VM 将重新启动。 重新启动后，现有 OS 和数据磁盘将重新映射。 临时磁盘上的所有内容将会丢失。

3. 如果所需的 VM 大小未列出，则需先使用 [az vm deallocate](https://docs.microsoft.com/zh-cn/cli/azure/vm#deallocate)解除分配 VM。 此过程允许将 VM 调整为该区域支持的任何可用大小然后将其启动。 以下步骤会将名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM 解除分配、调整大小并启动：

        az vm deallocate --resource-group myResourceGroup --name myVM
        az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
        az vm start --resource-group myResourceGroup --name myVM

    > [AZURE.WARNING]
    > 解除分配 VM 也会释放分配给该 VM 的所有动态 IP 地址。 OS 和数据磁盘不受影响。

## <a name="next-steps"></a>后续步骤
若要提高可伸缩性，请运行多个 VM 实例并进行横向扩展。

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/zh-cn/blog/boot-diagnostics-for-virtual-machines-v2/
[vm-sizes]: /documentation/articles/virtual-machines-linux-sizes/
<!--Update_Description: wording update-->