<properties
    pageTitle="使用 Azure CLI 2.0（预览版）复制 Linux VM | Azure"
    description="了解如何使用 Azure CLI 2.0（预览版）在 Resource Manager 部署模型中创建 Azure Linux 虚拟机的副本"
    services="virtual-machines-linux"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    tags="azure-resource-manager" />
<tags 
    ms.assetid="770569d2-23c1-4a5b-801e-cddcd1375164"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/24/2017"
    ms.author="cynthn" />

# 使用 Azure CLI 2.0（预览版）创建 Linux 虚拟机的副本
本文说明如何使用 Resource Manager 部署模型创建运行 Linux 的 Azure 虚拟机 (VM) 副本。首先，通过操作系统和数据磁盘复制到新容器，然后设置网络资源并创建虚拟机。

还可以[上载自定义磁盘映像并从中创建 VM](/documentation/articles/virtual-machines-linux-upload-vhd/)。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-copy-vm-nodejs/) - 适用于经典部署模型和资源管理部署模型的 CLI
- Azure CLI 2.0（预览版）- 下一代 CLI，适用于本文介绍的资源管理部署模型。

## 先决条件
- 需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。
- 需要使用一个 Azure VM 作为副本的源。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## 停止 VM
使用 [az vm deallocate](https://docs.microsoft.com/cli/azure/vm#deallocate) 解除分配源 VM。以下示例解除分配名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM：

    az vm deallocate --resource-group myResourceGroup --name myVM

## 复制 VM
若要复制 VM，请创建底层虚拟硬盘的副本。可以使用此过程创建包含与源 VM 相同的配置和设置的专用 VM。Azure 托管磁盘和非托管磁盘的虚拟磁盘复制过程不同。托管磁盘由 Azure 平台处理，无需任何准备或位置来存储它们。托管磁盘属于顶层资源，因此更容易使用 - 可以直接复制磁盘资源。有关 Azure 托管磁盘的详细信息，请参阅 [Azure 托管磁盘概述](/documentation/articles/storage-managed-disks-overview/)。根据源 VM 的存储类型，在下面选择适当的步骤之一：

- [托管磁盘](#managed-disks)
- [非托管磁盘](#unmanaged-disks)

### <a name="managed-disks"></a> 托管磁盘
使用 [az vm list](https://docs.microsoft.com/cli/azure/vm#list) 列出每个 VM 及其 OS 托管磁盘的名称。以下示例列出名为 `myResourceGroup` 的资源组中的所有 VM：

    az vm list -g myTestRG --query '[].{Name:name,DiskName:storageProfile.osDisk.name}' --output table

输出类似于以下示例：

    Name    DiskName
    ------  --------
    myVM    myDisk

通过使用 [az disk create](https://docs.microsoft.com/cli/azure/disk#create) 创建新的托管磁盘来复制磁盘。以下示例基于名为 `myDisk` 的托管磁盘创建名为 `myCopiedDisk` 的磁盘：

    az disk create --resource-group myResourceGroup --name myCopiedDisk --source myDisk

现在请使用 [az disk list](https://docs.microsoft.com/cli/azure/disk#list) 验证资源组中的托管磁盘。以下示例列出名为 `myResourceGroup` 的资源组中的托管磁盘：

    az disk list --resource-group myResourceGroup --output table

转到[创建和查看虚拟网络设置](#set-up-the-virtual-network)。

### <a name="unmanaged-disks"></a> 非托管磁盘
若要创建 VHD 的副本，需要使用 Azure 存储帐户密钥和磁盘 URI。使用 [az storage account keys list](https://docs.microsoft.com/cli/azure/storage/account/keys#list) 查看存储帐户密钥。以下示例列出名为 `myResourceGroup` 的资源组中名为 `mystorageaccount` 的存储帐户的密钥：

    az storage account keys list --resource-group myResourceGroup \
        --name mystorageaccount --output table

输出类似于以下示例：

    KeyName    Permissions    Value
    ---------  -------------  ----------------------------------------------------------------------------------------
    key1       Full           gi7crXhD8PMs8DRWqAM7fAjQEFmENiGlOprNxZGbnpylsw/nq+lQQg0c4jiKoV3Nytr3dAiXZRpL8jflpAr2Ug==
    key2       Full           UMoyQjWuKCBUAVDr1ANRe/IWTE2o0ZdmQH2JSZzXKNmDKq83368Fw9nyTBcTEkSKI9cm5tlTL8J15ArbPMo8eA==

使用 [az vm list](https://docs.microsoft.com/cli/azure/vm#list) 查看 VM 及其 URI 的列表。以下示例列出名为 `myResourceGroup` 的资源组中的 VM：

    az vm list -g myResourceGroup --query '[].{Name:name,URI:storageProfile.osDisk.vhd.uri}' --output table

输出类似于以下示例：

    Name    URI
    ------  -------------------------------------------------------------
    myVM    https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVHD.vhd

使用 [az storage blob copy start](https://docs.microsoft.com/cli/azure/storage/blob/copy#start) 复制 VHD。使用 **az storage account keys list** 和 **az vm list** 返回的信息提供所需的存储帐户密钥和 VHD URI。

    az storage blob copy start \
        --account-name mystorageaccount \
        --account-key gi7crXhD8PMs8DRWqAM7fAjQEFmENiGlOprNxZGbnpylsw/nq+lQQg0c4jiKoV3Nytr3dAiXZRpL8jflpAr2Ug== \
        --source-uri https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVHD.vhd \
        --destination-container vhds --destination-blob myCopiedVHD.vhd

## <a name="set-up-the-virtual-network"></a> 设置虚拟网络
以下步骤可创建新的虚拟网络、子网、公共 IP 地址和虚拟网络接口卡 (NIC)。这些步骤是可选的。在复制 VM 以进行故障排除或执行其他部署时，你可能不想要使用现有虚拟网络中的 VM。如果想要为复制的 VM 创建虚拟网络基础结构，请遵循后续几个步骤，否则请直接跳到[创建 VM](#create-a-vm)。

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create) 创建虚拟网络。以下示例创建名为 `myVnet` 的虚拟网络和名为 `mySubnet` 的子网：

    az network vnet create --resource-group myResourceGroup --location chinanorth --name myVnet \
        --address-prefix 192.168.0.0/16 --subnet-name mySubnet --subnet-prefix 192.168.1.0/24

使用 [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#create) 创建一个公共 IP。以下示例创建名为 `myPublicIP`、DNS 名称为 `mypublicdns` 的公共 IP。（DNS 名称必须唯一，因此，请提供自己的唯一名称。）

    az network public-ip create --resource-group myResourceGroup --location chinanorth \
        --name myPublicIP --dns-name mypublicdns --allocation-method static --idle-timeout 4

使用 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 创建网络接口卡 (NIC)。以下示例创建名为 `myNic` 的、已附加到 `mySubnet` 子网的 NIC：

    az network nic create --resource-group myResourceGroup --location chinanorth --name myNic \
        --vnet-name myVnet --subnet mySubnet

## <a name="create-a-vm"></a> 创建 VM
现在，可以使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。托管磁盘与非托管磁盘的磁盘复制过程稍有不同。请使用以下适当命令之一创建 VM。

### 托管磁盘
使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。将复制的托管磁盘指定为 OS 磁盘 (`--attach-os-disk`)，如下所示：

    az vm create --resource-group myResourceGroup --name myCopiedVM \
        --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic --size Standard_DS1_v2 --os-type Linux \
        --image UbuntuLTS
        --attach-os-disk myCopiedDisk

### 非托管磁盘
使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。指定通过 **az storage blob copy start** (`--image`) 创建复制的磁盘时使用的存储帐户、容器名称和 VHD，如下所示：

    az vm create --resource-group myResourceGroup --name myCopiedVM  \
        --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic --size Standard_DS1_v2 --os-type Linux \
        --image https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myCopiedVHD.vhd \
        --use-unmanaged-disk

## 后续步骤
若要了解如何使用 Azure CLI 来管理新虚拟机，请参阅 [Azure CLI commands for the Azure Resource Manager](/documentation/articles/azure-cli-arm-commands/)（Azure Resource Manager 的 Azure CLI 命令）。

<!---HONumber=Mooncake_0320_2017-->