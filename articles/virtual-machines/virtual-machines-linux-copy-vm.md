<properties
    pageTitle="使用 Azure CLI 2.0 复制 Linux VM | Azure"
    description="了解如何使用 Azure CLI 2.0 和非托管磁盘创建 Azure Linux VM 的副本。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="770569d2-23c1-4a5b-801e-cddcd1375164"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/10/2017"
    wacn.date="04/24/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7c4f6b1ddac26659057bcee15690ae5657dddbcd"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-copy-of-a-linux-vm-by-using-azure-cli-20-and-managed-disks"></a>使用 Azure CLI 2.0 和非托管磁盘创建 Azure Linux VM 的副本

本文说明了如何使用 Azure CLI 2.0 和 Azure Resource Manager 部署模型创建运行 Linux 的 Azure 虚拟机 (VM) 的副本。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-copy-vm-nodejs/) 执行这些步骤。

还可以[上载 VHD 并从中创建 VM](/documentation/articles/virtual-machines-linux-upload-vhd/)。

## <a name="prerequisites"></a>先决条件

-   安装 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)

-   使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到一个 Azure 帐户。

-   使用一个 Azure VM 作为你的副本的来源。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="step-1-stop-the-source-vm"></a>步骤 1：停止源 VM

使用 [az vm deallocate](https://docs.microsoft.com/zh-cn/cli/azure/vm#deallocate) 解除分配源 VM。
以下示例解除分配资源组**myResourceGroup** 中名为 **myVM** 的 VM：

    az vm deallocate --resource-group myResourceGroup --name myVM

## <a name="step-2-copy-the-source-vm"></a>步骤 2：复制源 VM

若要复制 VM，请创建底层虚拟硬盘的副本。可以使用此过程创建包含与源 VM 相同的配置和设置的专用 VM。

- 托管磁盘 - 在 Azure 中国暂时还不适用。
- [非托管磁盘](#unmanaged-disks)

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

## <a name="step-3-set-up-a-virtual-network"></a>步骤 3：设置虚拟网络

以下可选步骤可创建新的虚拟网络、子网、公共 IP 地址和虚拟网络接口卡 (NIC)。

在复制 VM 来完成故障排除或其他部署操作时，用户可能不希望使用现有虚拟网络中的 VM。

如果希望为复制的 VM 创建虚拟网络基础结构，请按后续几个步骤操作。 如果不希望创建虚拟网络，请跳到[步骤 4：创建 VM](#step-4-create-a-vm)。

1.  使用 [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) 创建虚拟网络。 以下示例创建一个名为 **myVnet** 的虚拟网络和一个名为 **mySubnet** 的子网：

        az network vnet create --resource-group myResourceGroup --location chinanorth --name myVnet \
            --address-prefix 192.168.0.0/16 --subnet-name mySubnet --subnet-prefix 192.168.1.0/24

1.  使用 [az network public-ip create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create) 创建公共 IP。 以下示例创建一个名为 **myPublicIP** 的公共 IP，其 DNS 名称为 **mypublicdns**。 （DNS 名称必须唯一，因此请提供唯一名称。）

        az network public-ip create --resource-group myResourceGroup --location chinanorth \
            --name myPublicIP --dns-name mypublicdns --allocation-method static --idle-timeout 4

1.  使用 [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) 创建 NIC。
    以下示例创建一个附加到 **mySubnet** 子网且名为 **myNic** 的 NIC：

        az network nic create --resource-group myResourceGroup --location chinanorth --name myNic \
            --vnet-name myVnet --subnet mySubnet --public-ip-address myPublicIP

## <a name="step-4-create-a-vm"></a>步骤 4：创建 VM

现在可使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。指定通过 **az storage blob copy start** (`--image`) 创建复制的磁盘时使用的存储帐户、容器名称和 VHD，如下所示：

    az vm create --resource-group myResourceGroup --name myCopiedVM \
        --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic --size Standard_DS1_v2 --os-type Linux \
        --image https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myCopiedVHD.vhd \
        --use-unmanaged-disk

## <a name="next-steps"></a>后续步骤

若要了解如何使用 Azure CLI 管理新 VM，请参阅 [Azure Resource Manager 的 Azure CLI 命令](/documentation/articles/azure-cli-arm-commands/)。
<!--Update_Description: wording update-->