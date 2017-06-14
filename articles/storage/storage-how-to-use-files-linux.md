<properties
    pageTitle="如何通过 Linux 使用 Azure 文件 | Azure"
    description="按照此分步教程中的说明，在云中创建 Azure 文件共享。 管理文件共享内容，并从运行 Linux 的 Azure 虚拟机 (VM) 或支持 SMB 3.0 的本地应用程序安装文件共享。"
    services="storage"
    documentationcenter="na"
    author="RenaShahMSFT"
    manager="aungoo"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="6edc37ce-698f-4d50-8fc1-591ad456175d"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="3/8/2017"
    wacn.date="04/24/2017"
    ms.author="renash"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="edc9d41bd38c4fe7359328a34bd0ebc4486d2aa8"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-use-azure-file-storage-with-linux"></a>如何通过 Linux 使用 Azure 文件存储
## <a name="overview"></a>概述
Azure 文件存储使用标准 SMB 协议在云中提供文件共享。 使用 Azure 文件，你可以将依赖于文件服务器的企业应用程序迁移到 Azure。 在 Azure 中运行的应用程序可以轻松地从运行 Linux 的 Azure 虚拟机装载文件共享。 并且使用最新版本的文件存储，你还可以从支持 SMB 3.0 的本地应用程序装载文件共享。

可以使用 [Azure 门户](https://portal.azure.cn)、Azure 存储 PowerShell cmdlet、Azure 存储客户端库或 Azure 存储 REST API 来创建 Azure 文件共享。此外，由于文件共享是 SMB 共享，因此你还可以通过标准的文件系统 API 来访问它们。

文件存储基于与 Blob、表和队列存储相同的技术构建，因此文件存储能够提供 Azure 存储平台内置的现有可用性、持续性、可伸缩性和异地冗余。 有关文件存储性能目标和限制的详细信息，请参阅[Azure 存储的可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

文件存储现已正式推出并同时支持 SMB 2.1 和 SMB 3.0。有关文件存储的更多详细信息，请参阅[文件服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)。

> [AZURE.NOTE]
> 由于 Linux SMB 客户端尚不支持加密，从 Linux 装载文件共享仍需要客户端与文件共享在同一 Azure 区域中。 但是，Linux 的加密支持已经在负责 SMB 功能的 Linux 开发人员的路线图上。 将来支持加密的 Linux 分发也将能够从任何位置装载 Azure 文件共享。
> 
> 



## <a name="choose-a-linux-distribution-to-use"></a>选择要使用的 Linux 分发
在 Azure 中创建 Linux 虚拟机时，可以从 Azure 映像库指定支持 SMB 2.1 或更高版本的 Linux 映像。下面是建议的 Linux 映像的列表：

* Ubuntu Server 14.04+
* RHEL 7+
* CentOS 7+
* Debian 8
* openSUSE 13.2+
* SUSE Linux Enterprise Server 12
* SUSE Linux Enterprise Server 12 (Premium Image)

## <a name="mount-the-file-share"></a>装载文件共享
若要从运行 Linux 的虚拟机装载文件共享，可能需要安装 SMB/CIFS 客户端（如果你使用的分发没有内置的客户端）。 这是 Ubuntu 中的命令，用于安装一个选择 cifs-utils：

    sudo apt-get install cifs-utils

接下来，你需要设置装入点 (mkdir mymountpoint)，然后发出类似于下面所示的装载命令：

     sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename ./mymountpoint -o vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino

你还可以在 /etc/fstab 中添加设置以装载共享。

请注意，此处的 0777 代表目录/文件权限代码，用于向所有用户授予执行/读/写权限。 你可以将其替换为 Linux 文件权限文档后面的其他文件权限代码。

另外要使文件共享在重新启动后装载，可以在 /etc/fstab 中添加如下设置：

    //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username= myaccountname,password= StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino

例如，如果你是使用 Linux 映像 Ubuntu Server 15.04（可从 Azure 映像库中获得）创建的 Azure VM，则可以按如下所示装载文件：

    azureuser@azureconubuntu:~$ sudo apt-get install cifs-utils
    azureuser@azureconubuntu:~$ sudo mkdir /mnt/mountpoint
    azureuser@azureconubuntu:~$ sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino
    azureuser@azureconubuntu:~$ df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

如果你使用 CentOS 7.1，则可以按如下所示装载文件：

    [azureuser@AzureconCent ~]$ sudo yum install samba-client samba-common cifs-utils
    [azureuser@AzureconCent ~]$ sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino
    [azureuser@AzureconCent ~]$ df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

如果你使用 Open SUSE 13.2，则可以按如下所示装载文件：

    azureuser@AzureconSuse:~> sudo zypper install samba*  
    azureuser@AzureconSuse:~> sudo mkdir /mnt/mountpoint
    azureuser@AzureconSuse:~> sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino
    azureuser@AzureconSuse:~> df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

如果使用 RHEL 7.3，则可以按如下所示装载文件：

    azureuser@AzureconRedhat:~> sudo yum install cifs-utils
    azureuser@AzureconRedhat:~> sudo mkdir /mnt/mountpoint
    azureuser@AzureconRedhat:~> sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777,serverino
    azureuser@AzureconRedhat:~> df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

## <a name="manage-the-file-share"></a>管理文件共享
[Azure 门户](https://portal.azure.cn)提供用于管理 Azure 文件存储的用户界面。你可以从 Web 浏览器中执行以下操作：

* 将文件上载到文件共享和从文件共享下载文件。
* 监视每个文件共享的实际使用情况。
* 调整文件共享大小配额。
* 复制 `net use` 命令以用于从 Windows 客户端装载文件共享。

还可以从 Linux 使用 Azure 跨平台命令行界面 (Azure CLI) 来管理文件共享。Azure CLI 提供了一组开放源代码跨平台命令，你可以使用这些命令来处理 Azure 存储（包括文件存储）。它提供 Azure 门户所能提供的很多相同功能，此外还有各种数据访问功能。有关示例，请参阅[将 Azure CLI 用于 Azure 存储](/documentation/articles/storage-azure-cli/)。

## <a name="develop-with-file-storage"></a>使用文件存储进行开发
作为开发人员，你可以通过 [适用于 Java 的 Azure 存储空间客户端库](https://github.com/azure/azure-storage-java)使用文件存储构建应用程序。 有关代码示例，请参阅[如何通过 Java 使用文件存储](/documentation/articles/storage-java-how-to-use-file-storage/)。

你还可以使用 [适用于 Node.js 的 Azure 存储空间客户端库](https://github.com/Azure/azure-storage-node) 针对文件存储进行开发。

## <a name="next-steps"></a>后续步骤
请参阅以下链接以获取有关 Azure 文件存储的更多信息。

### <a name="conceptual-articles-and-videos"></a>概念性文章和视频
* [Azure 文件存储：适用于 Windows 和 Linux 的顺畅的云 SMB 文件系统](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
* [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)

### <a name="tooling-support-for-file-storage"></a>文件存储的工具支持
* [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
* 通过 Azure CLI [创建和管理文件共享](/documentation/articles/storage-azure-cli/#create-and-manage-file-shares)

### <a name="reference"></a>引用
* [文件服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)
* [Azure 文件疑难解答文章](/documentation/articles/storage-troubleshoot-file-connection-problems/)

### <a name="blog-posts"></a>博客文章
* [Azure 文件存储现已正式发布](https://azure.microsoft.com/zh-cn/blog/azure-file-storage-now-generally-available/)
* [Azure 文件存储内部](/home/features/storage) 
* [Azure 文件服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
* [持久连接到 Azure 文件](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)
<!--Update_Description: update "sudo mount" commands, add "serverino" parameter;add anchors to H2 titles-->