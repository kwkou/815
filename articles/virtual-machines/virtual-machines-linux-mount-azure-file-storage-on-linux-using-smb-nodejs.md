<properties
    pageTitle="通过 Azure CLI 1.0 使用 SMB 在 Linux VM 上装载 Azure 文件存储 | Azure"
    description="如何使用 SMB 在 Linux VM 上装载 Azure 文件存储。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines-linux"
    author="vlivech"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="12/07/2016"
    wacn.date="04/06/2017"
    ms.author="v-livech" />  


# 通过 Azure CLI 1.0 使用 SMB 在 Linux VM 上装载 Azure 文件存储

本文说明如何使用 SMB 装载利用 Linux VM 上的 Azure 文件存储服务。Azure 文件存储使用标准 SMB 协议在云中提供文件共享。要求包括：

- [一个 Azure 帐户](/pricing/1rmb-trial/)

- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#quick-commands)：用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0（预览版）](/documentation/articles/virtual-machines-linux-mount-azure-file-storage-on-linux-using-smb/)：用于资源管理部署模型的下一代 CLI

## <a name="quick-commands"></a> 快速命令

如果需要快速完成任务，以下部分详细介绍所需的命令。本文档的余下部分（[从此处开始](/documentation/articles/virtual-machines-linux-mount-azure-file-storage-on-linux-using-smb/#detailed-walkthrough)）提供了每个步骤的更详细信息和上下文。

先决条件：资源组、VNet、将 SSH 入站的 NSG、子网、Azure 存储帐户、Azure 存储帐户密钥、Azure 文件存储共享和 Linux VM。将任何示例替换为你自己的设置。

为本地装载创建目录

    mkdir -p /mnt/mymountpoint

将 Azure 文件存储 SMB 共享装载到装入点

    sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint -o vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

若要在重新启动后保留装载，请向 `/etc/fstab` 添加一行

    //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

## <a name="detailed-walkthrough"></a> 详细演练

Azure 文件存储使用标准 SMB 协议在云中提供文件共享。并且使用最新版本的文件存储，还可以从支持 SMB 3.0 的任何 OS 装载文件共享。在 Linux 上使用 SMB 装载可轻松备份到 SLA 支持的可靠、永久的存档存储位置。

将 VM 中的文件移到 Azure 文件存储上托管的 SMB 装载是调试日志的好办法，因为相同的 SMB 共享可本地装载到 Mac、Linux 或 Windows 工作站。SMB 不会是实时流式处理 Linux 或应用程序日志的最佳解决方案，因为 SMB 协议并非为那样重的日志记录职责而构建。专用统一的日志记录层工具（如 Fluentd）会是 SMB 之上的更好选择，可收集 Linux 和应用程序日志记录输出。

在此详细的演练中，我们创建所需的先决条件，即先创建 Azure 文件存储共享，然后通过 SMB 将其装载到 Linux VM 上。

## 创建 Azure 存储帐户

    azure storage account create myStorageAccount \
    --sku-name lrs \
    --kind storage \
    -l chinanorth \
    -g myResourceGroup

## 显示存储帐户密钥

创建存储帐户时，将成对创建 Azure 存储帐户密钥。之所以成对创建存储帐户密钥是为了不中断任何服务就可轮换密钥。将密钥轮换到密钥对中的第二个密钥后，将创建新的密钥对。新的存储帐户密钥始终成对创建，可确保始终至少有一个未使用的存储密钥可以轮换到。

    azure storage account keys list myStorageAccount \
    --resource-group myResourceGroup

## 创建 Azure 文件存储共享

创建包含 SMB 共享的文件存储共享。配额始终以千兆字节 (GB) 为单位。

    azure storage share create mystorageshare \
    --quota 10 \
    --account-name myStorageAccount \
    --account-key nPOgPR<--snip-->4Q==

## 创建装入点目录

在 Linux 文件系统上需要有要将 SMB 共享装载到的本地目录。写入到本地装载目录或从本地装载目录读取的任何内容都将转发到 Azure 文件存储上托管的 SMB 共享。

    sudo mkdir -p /mnt/mymountdirectory

## 装载 SMB 共享

    sudo mount -t cifs //myStorageAccount.file.core.chinacloudapi.cn/mystorageshare /mnt/mymountdirectory -o vers=3.0,username=myStorageAccount,password=myStorageAccountkey,dir_mode=0777,file_mode=0777

## 重新启动后保留 SMB 装载

重新启动 Linux VM 后，已装载的 SMB 共享会在关闭过程中卸载。若要在启动时重新装载 SMB 共享，必须向 Linux `/etc/fstab` 添加一行。Linux 使用 `fstab` 文件列出它需要在启动期间装载哪些文件系统。添加 SMB 共享可确保 Azure 文件存储共享是 Linux VM 的永久装载文件系统。使用 `cloud-init` 可以将 Azure 文件存储 SMB 共享添加到新的 VM。

    //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

## 后续步骤

- [在创建期间使用 cloud-init 自定义 Linux VM](/documentation/articles/virtual-machines-linux-using-cloud-init/)
- [将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)
- [使用 Azure CLI 加密 Linux VM 上的磁盘](/documentation/articles/virtual-machines-linux-encrypt-disks/)

<!---HONumber=Mooncake_0313_2017-->