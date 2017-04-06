<properties
    pageTitle="使用 Azure CLI 2.0（预览版）捕获 Linux VM | Azure"
    description="了解如何使用通过 Azure CLI 2.0（预览版）创建的托管磁盘来捕获和通用化基于 Linux 的 Azure 虚拟机 (VM) 的映像。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="e608116f-f478-41be-b787-c2ad91b5a802"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/20/2017"
    ms.author="iainfou" />

# 如何使用 Azure CLI 2.0（预览版）通用化和捕获 Linux 虚拟机
若要重复使用在 Azure 中部署和配置的虚拟机 (VM)，需要捕获 VM 的映像。该过程还涉及从映像部署新 VM 之前，先通用化 VM 以删除个人帐户信息。本文详细介绍如何使用 Azure CLI 2.0（预览版）为使用 Azure 托管磁盘的 VM 捕获 VM 映像。这些磁盘由 Azure 平台处理，无需任何准备或位置来存储它们。有关详细信息，请参阅 [Azure 托管磁盘概述](/documentation/articles/storage-managed-disks-overview/)。

> [AZURE.TIP]
如果想要创建现有 Linux VM 的副本（包括其专用化备份或调试状态），请参阅[创建在 Azure 上运行的 Linux 虚拟机副本](/documentation/articles/virtual-machines-linux-copy-vm/)。如果想要从本地 VM 上载 Linux VHD，请参阅[上载自定义磁盘映像并从其创建 Linux VM](/documentation/articles/virtual-machines-linux-upload-vhd/)。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-capture-image-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）- Azure 托管磁盘](#quick-commands) - 适用于资源管理部署模型的下一代 CLI（本文）

## 开始之前
请确保符合以下先决条件：

* **在 Resource Manager 部署模型中创建的 Azure VM** - 如果尚未创建 Linux VM，可以使用[门户](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[Azure CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 [Resource Manager 模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)。根据需要配置 VM。例如，[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)、应用更新，并安装应用程序。

还需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="quick-commands"></a> 快速命令
如果需要快速完成任务，请参阅以下部分，其中详细说明了在 Azure 中捕获 Linux VM 映像的基本命令。本文档的余下部分（[从此处开始](#detailed-steps)）提供了每个步骤的更详细信息和应用背景。在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`myVM` 和 `myImage`。

1. 取消预配源 VM：

        ssh ops@myvm.chinanorth.chinacloudapp.cn
        sudo waagent -deprovision+user -force
        exit

2. 使用 [az vm deallocate](https://docs.microsoft.com/cli/azure/vm#deallocate) 解除分配 VM：

        az vm deallocate --resource-group myResourceGroup --name myVM

3. 使用 [az vm generalize](https://docs.microsoft.com/cli/azure/vm#generalize) 通用化 VM：

        az vm generalize --resource-group myResourceGroup --name myVM

4. 使用 [az image create](https://docs.microsoft.com/cli/azure/image#create) 从 VM 资源创建映像：

        az image create --resource-group myResourceGroup --name myImage --source myVM

5. 使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 从映像资源创建 VM：

        az vm create --resource-group myResourceGroup --name myVMDeployed --image myImage
            --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub

## <a name="detailed-steps"></a> 详细步骤
通过下列步骤可以取消设置现有 VM，解除分配并通用化 VM 资源，然后创建映像。可以使用此映像，在订阅内的任何资源组中创建 VM。此过程使 [Azure 托管磁盘](/documentation/articles/storage-managed-disks-overview/)比非托管磁盘更具优势。使用非托管磁盘时，创建基础虚拟硬盘 (VHD) 的 Blob 副本之后，便只能在与复制的 VHD Blob 相同的存储帐户中创建 VM。使用托管磁盘时，所创建的映像资源可部署在整个订阅中。

## 步骤 1：删除 Azure Linux 代理
若要使 VM 做好进行通用化的准备，请使用 Azure VM 代理取消预配 VM，以删除文件和数据。在目标 Linux VM 上，使用带 **deprovision** 参数的 **waagent** 命令。有关详细信息，请参阅 [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

1. 使用 SSH 客户端连接到 Linux VM。
2. 在 SSH 窗口中，键入以下命令：

        sudo waagent -deprovision+user

    > [AZURE.NOTE]
    仅在要捕获为映像的 VM 上运行此命令。不保证映像中的所有敏感信息被清除，或者映像适合用于分发。
 
3. 键入 **y** 继续。添加 **-force** 参数即可免除此确认步骤。
4. 完成该命令后，键入 **exit**。此步骤将关闭 SSH 客户端。

## 步骤 2：捕获 VM
使用 Azure CLI 2.0（预览版）通用化和捕获 VM。在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 **myResourceGroup**、**myVnet** 和 **myVM**。

1. 解除分配已使用 [az vm deallocate](https://docs.microsoft.com/cli//azure/vm#deallocate) 取消预配的 VM。以下示例解除分配名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM：

        az vm deallocate --resource-group myResourceGroup --name myVM

2. 使用 [az vm generalize](https://docs.microsoft.com/cli//azure/vm#generalize) 通用化 VM。以下示例在名为 `myResourceGroup` 的资源组中通用化名为 `myVM` 的 VM：

        az vm generalize --resource-group myResourceGroup --name myVM

3. 现在，使用 [az image create](https://docs.microsoft.com/cli//azure/image#create) 创建 VM 资源的映像。以下示例使用名为 `myVM` 的 VM 资源在名为 `myResourceGroup` 的资源组中创建名为 `myImage` 的映像：

        az image create --resource-group myResourceGroup --name myImage --source myVM

    > [AZURE.NOTE]
    该映像在与源 VM 相同的资源组中创建。可以在订阅内的任何资源组中从此映像创建 VM。从管理角度来看，你可能希望为 VM 资源和映像创建特定的资源组。

## 步骤 3：从捕获的映像创建 VM
使用通过 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建的映像创建 VM。以下示例从名为 `myImage` 的映像创建名为 `myVMDeployed` 的 VM：

    az vm create --resource-group myResourceGroup --name myVMDeployed --image myImage
        --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub

使用托管磁盘，可在订阅内的任何资源组中从映像创建 VM。此行为与非托管磁盘有所不同，在非托管磁盘中只能在与源 VHD 相同的存储帐户中创建 VM。若要在与映像不同的资源组中创建 VM，请指定映像的完整资源 ID。使用 [az image list](https://docs.microsoft.com/cli/azure/image#list) 查看映像列表。输出类似于以下示例：

    "id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage",
       "location": "chinanorth",
       "name": "myImage",

以下示例使用 **az vm create**，通过指定映像资源 ID，在与源映像不同的资源组中创建 VM：

    az vm create --resource-group myOtherResourceGroup --name myOtherVMDeployed 
        --image "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage"
        --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub

### 验证部署
现在将 SSH 连接到你创建的虚拟机以验证部署并开始使用新的 VM。若要通过 SSH 连接，请使用 [az vm show](https://docs.microsoft.com/cli/azure/vm#show) 查找 VM 的 IP 地址或 FQDN：

    az vm show --resource-group myResourceGroup --name myVM --show-details

## 后续步骤
可以从源 VM 映像创建多个 VM。如果需要对映像进行更改，请执行以下操作：

- 打开源 VM 资源。
- 进行任何更新或配置更改。
- 再次遵循相应的步骤来取消预配、解除分配、通用化和捕获 VM。

有关使用 CLI 管理 VM 的详细信息，请参阅 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/overview)。

<!---HONumber=Mooncake_0313_2017-->