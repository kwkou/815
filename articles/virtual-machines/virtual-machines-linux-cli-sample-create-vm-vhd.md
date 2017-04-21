<properties
    pageTitle="Azure CLI 脚本示例 - 使用 VHD 创建 VM | Azure"
    description="Azure CLI 脚本示例 - 使用虚拟硬盘创建 VM。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="allclark"
    manager="douge"
    editor="tysonn"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/09/2017"
    wacn.date="04/17/2017"
    ms.author="allclark"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="6e0cfe019c966c50481c7a832dd656389a7a914f"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-vm-with-a-virtual-hard-disk"></a>使用虚拟硬盘创建 VM

本示例使用 VHD 创建虚拟机。
本示例创建资源组、存储帐户、容器，然后通过将 VHD 上载到容器来创建 VM。
本示例将 ssh 公钥替换为用户的公钥，因此用户可以访问 VM。

用户需要可引导 VHD。
可以从 https://azclisamples.blob.core.windows.net/vhds/sample.vhd 下载我们所使用的 VHD，也可以使用自己的 VHD。 脚本会查找 `~/sample.vhd`。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Create a resource group
    az group create -n myResourceGroup -l chinanorth

    # Create the storage account to upload the vhd
    az storage account create -g myResourceGroup -n mystorageaccount -l chinanorth --sku PREMIUM_LRS

    # Get a storage key for the storage account
    STORAGE_KEY=$(az storage account keys list -g myResourceGroup -n mystorageaccount --query "[?keyName=='key1'] | [0].value" -o tsv)

    # Create the container for the vhd
    az storage container create -n vhds --account-name mystorageaccount --account-key ${STORAGE_KEY}

    # Upload the vhd to a blob
    az storage blob upload -c vhds -f ~/sample.vhd -n sample.vhd --account-name mystorageaccount --account-key ${STORAGE_KEY}

    # Create the vm from the vhd
    az vm create -g myResourceGroup -n myVM --image "https://myStorageAccount.blob.core.chinacloudapi.cn/vhds/sample.vhd" \
            --os-type linux --admin-username deploy --generate-ssh-keys --use-unmanaged-disk

    # Update the deploy user with your ssh key
    az vm user update --resource-group myResourceGroup -n custom-vm -u deploy --ssh-key-value "$(< ~/.ssh/id_rsa.pub)"

    # Get public IP address for the VM
    IP_ADDRESS=$(az vm list-ip-addresses -g az-cli-vhd -n custom-vm \
        --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

    echo "You can now connect using 'ssh deploy@${IP_ADDRESS}'"

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete -n az-cli-vhd

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机、可用性集、负载均衡器和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az storage account list](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#list) | 列出存储帐户 |
| [az storage account check-name](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#check-name) | 检查存储帐户名称是否有效且目前还不存在 |
| [az storage account keys list](https://docs.microsoft.com/zh-cn/cli/azure/storage/account/keys#list) | 列出存储帐户的密钥 |
| [az storage blob exists](https://docs.microsoft.com/zh-cn/cli/azure/storage/blob#exists) | 检查 Blob 是否存在 |
| [az storage container create](https://docs.microsoft.com/zh-cn/cli/azure/storage/container#create) | 在存储帐户中创建一个容器。 |
| [az storage blob upload](https://docs.microsoft.com/zh-cn/cli/azure/storage/blob#upload) | 通过上载 VHD，在容器中创建一个 Blob。 |
| [az vm list](https://docs.microsoft.com/zh-cn/cli/azure/vm#list) | 与 `--query` 一起使用，用于检查 VM 名称是否已使用。 | 
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm/availability-set#create) | 创建虚拟机。 |
| [az vm access set-linux-user](https://docs.microsoft.com/zh-cn/cli/azure/vm/access#set-linux-user) | 重置 SSH 密钥，以便当前用户能够访问 VM。 |
| [az vm list-ip-addresses](https://docs.microsoft.com/zh-cn/cli/azure/vm#list-ip-addresses) | 获取已创建虚拟机的 IP 地址。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Linux VM 文档](/documentation/articles/virtual-machines-linux-cli-samples/)中找到其他虚拟机 CLI 脚本示例。