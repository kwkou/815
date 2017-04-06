<properties
    pageTitle="如何使用 Azure CLI 2.0（预览版）调整 Linux VM 的大小 | Azure"
    description="如何通过更改 VM 大小来增加或减少 Linux 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter="na"
    author="mikewasson"
    manager="timlt"
    editor=""
    tags="" />
<tags 
    ms.assetid="e163f878-b919-45c5-9f5a-75a64f3b14a0"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/10/2017"
    wacn.date="03/24/2017"
    ms.author="mwasson" />

# 如何调整 Linux VM 的大小
预配虚拟机 (VM) 后，可以通过更改 [VM 大小][vm-sizes]来扩展或缩减 VM。在某些情况下，必须先解除分配 VM。如果所需大小在托管 VM 的硬件群集上不可用，则需解除分配 VM。本文详述了如何使用 Azure CLI 2.0（预览版）调整 Linux VM 的大小。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-change-vm-size-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](#resize-a-linux-vm)：用于资源管理部署模型（本文）的下一代 CLI

## <a name="resize-a-linux-vm"></a> 调整 Linux VM 的大小
若要调整 VM 的大小，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

1. 通过 [az vm list-vm-resize-options](https://docs.microsoft.com/cli/azure/vm#list-vm-resize-options) 查看可以在托管 VM 的硬件群集上使用的 VM 大小的列表。以下示例列出资源组 `myResourceGroup` 区域中名为 `myVM` 的 VM 的大小：

        az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table

2. 如果列出了所需的 VM 大小，则使用 [az vm resize](https://docs.microsoft.com/cli/azure/vm#resize) 调整 VM 的大小。以下示例将名为 `myVM` 的 VM 的大小调整为 `Standard_DS3_v2` 大小：

        az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2

    VM 在此过程中重新启动。重新启动后，现有 OS 和数据磁盘将重新映射。临时磁盘上的所有内容将会丢失。

3. 如果所需的 VM 大小未列出，则需先使用 [az vm deallocate](https://docs.microsoft.com/cli/azure/vm#deallocate) 解除分配 VM。然后，可以通过此过程将 VM 的大小调整为区域支持的任何可用大小，再启动该 VM。以下步骤在名为 `myResourceGroup` 的资源组中对名为 `myVM` 的 VM 执行解除分配、调整大小和启动操作：

        az vm deallocate --resource-group myResourceGroup --name myVM
        az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
        az vm start --resource-group myResourceGroup --name myVM

    > [AZURE.WARNING]
    解除分配 VM 也会释放分配给该 VM 的所有动态 IP 地址。OS 和数据磁盘不受影响。

## 后续步骤
若要提高可伸缩性，请运行多个 VM 实例并进行横向扩展。

<!-- links -->

[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[vm-sizes]: /documentation/articles/virtual-machines-linux-sizes/

<!---HONumber=Mooncake_0320_2017-->