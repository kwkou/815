<properties
    pageTitle="使用 SMB 在 Linux VM 上装载 Azure 文件存储 | Azure"
    description="如何通过 Azure CLI 2.0 使用 SMB 在 Linux VM 上装载 Azure 文件存储"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines-linux"
    author="vlivech"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/13/2017"
    wacn.date="04/24/2017"
    ms.author="v-livech"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="89f524ae51fdaaf1226c63129fa200f515ebd735"
    ms.lasthandoff="04/14/2017" />

# <a name="mount-azure-file-storage-on-linux-vms-using-smb"></a>使用 SMB 在 Linux VM 上装载 Azure 文件存储

本文说明如何通过 Azure CLI 2.0 使用 SMB 装载利用 Linux VM 上的 Azure 文件存储服务。 Azure 文件存储使用标准 SMB 协议在云中提供文件共享。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-mount-azure-file-storage-on-linux-using-smb-nodejs/) 执行这些步骤。 要求如下：

- [一个 Azure 帐户](/pricing/1rmb-trial/)
- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## <a name="quick-commands"></a>快速命令

* 资源组
* Azure 虚拟网络
* 使用 SSH 入站类型的网络安全组
* 一个子网
* 一个 Azure 存储帐户
* Azure 存储帐户密钥
* 一个 Azure 文件存储共享
* 一个 Linux VM

将任何示例替换为你自己的设置。

### <a name="create-a-directory-for-the-local-mount"></a>为本地装载创建目录

    mkdir -p /mnt/mymountpoint

### <a name="mount-the-file-storage-smb-share-to-the-mount-point"></a>将文件存储 SMB 共享装载到装入点

    sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint -o vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

### <a name="persist-the-mount-after-a-reboot"></a>在重新启动后保留装载
为此，请将以下代码行添加到 `/etc/fstab`：

    //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

## <a name="detailed-walkthrough"></a>详细演练

文件存储使用标准 SMB 协议在云中提供文件共享。 使用最新版本的文件存储，还可以从支持 SMB 3.0 的任何 OS 装载文件共享。 在 Linux 上使用 SMB 装载时，可轻松备份到 SLA 支持的可靠、永久的存档存储位置。

将文件从 VM 移至托管在文件存储上的 SMB 装载，可以很轻松地调试日志。 这是因为同一 SMB 共享可以通过本地方式装载到 Mac、Linux 或 Windows 工作站。 SMB 不会是实时流式处理 Linux 或应用程序日志的最佳解决方案，因为 SMB 协议并非为处理那样重的日志记录任务而构建。 专用统一的日志记录层工具（如 Fluentd）会是 SMB 之上的更好选择，可收集 Linux 和应用程序日志记录输出。

在此详细的演练中，我们创建所需的先决条件，即先创建文件存储共享，然后通过 SMB 将其装载到 Linux VM 上。

1. 使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建资源组，以便保存文件共享。

    若要在“中国北部”位置创建名为 `myResourceGroup` 的资源组，请使用以下示例：

        az group create --name myResourceGroup --location chinanorth

2. 使用 [az storage account create](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#create) 创建 Azure 存储帐户，用于存储实际文件。

    若要通过 Standard_LRS 存储 SKU 创建名为 mystorageaccount 的存储帐户，请使用以下示例：

        az storage account create --resource-group myResourceGroup \
            --name mystorageaccount \
            --location chinanorth \
            --sku Standard_LRS

3. 显示存储帐户密钥。

    创建存储帐户时，帐户密钥是成对创建的，这样是为了不中断任何服务就可轮换密钥。 轮换到密钥对中的第二个密钥后，将创建新的密钥对。 新的存储帐户密钥始终成对创建，可确保始终至少有一个未使用的存储帐户密钥可以轮换到。

    使用 [az storage account keys list](https://docs.microsoft.com/zh-cn/cli/azure/storage/account/keys#list) 查看存储帐户密钥。 名为 `mystorageaccount` 的存储帐户的密钥列在以下示例中：

        az storage account keys list --resource-group myResourceGroup \
            --account-name mystorageaccount

    若要提取单个密钥，请使用 `--query` 标志。 以下示例提取了第一个密钥 (`[0]`)：

        az storage account keys list --resource-group myResourceGroup \
            --account-name mystorageaccount \
            --query '[0].{Key:value}' --output tsv

4. 创建文件存储共享。

    使用 [az storage share create](https://docs.microsoft.com/zh-cn/cli/azure/storage/share#create) 创建的文件存储共享包含 SMB 共享。 配额始终以千兆字节 (GB) 表示。 从前面的 `az storage account keys list` 命令传入其中一个密钥。 使用以下示例创建名为 mystorageshare 的共享，配额为 10 GB：

        az storage share create --name mystorageshare \
            --quota 10 \
            --account-name mystorageaccount \
            --account-key nPOgPR<--snip-->4Q==

5. 创建一个装入点目录。

    在 Linux 文件系统中创建将 SMB 共享装载到其中的本地目录。 写入到本地装载目录或从本地装载目录读取的任何内容都将转发到文件存储上托管的 SMB 共享。 若要在 /mnt/mymountdirectory 中创建本地目录，请使用以下示例：

        sudo mkdir -p /mnt/mymountdirectory

6. 将 SMB 共享装载到本地目录。

    提供自己的装载凭据的存储帐户用户名和存储帐户密钥，如下所示：

        sudo mount -t cifs //myStorageAccount.file.core.chinacloudapi.cn/mystorageshare /mnt/mymountdirectory -o vers=3.0,username=mystorageaccount,password=mystorageaccountkey,dir_mode=0777,file_mode=0777

7. 重新启动后保留 SMB 装载。

    重新启动 Linux VM 后，已装载的 SMB 共享会在关闭过程中卸载。 若要在启动时重新装载 SMB 共享，请向 Linux /etc/fstab 添加一行。 Linux 使用 fstab 文件列出在启动过程中需要装载的文件系统。 添加 SMB 共享可确保文件存储共享是 Linux VM 的永久装载文件系统。 使用 cloud-init 可以将文件存储 SMB 共享添加到新的 VM。

        //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

## <a name="next-steps"></a>后续步骤

- [在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)
- [将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)
- [使用 Azure CLI 加密 Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-encrypt-disks/)
<!--Update_Description: wording update-->