<properties
	pageTitle="如何通过 Linux 使用 Azure 文件存储 | Azure"
        description="按照此分步教程中的说明，在云中创建 Azure 文件共享。管理文件共享内容，并从运行 Linux 的 Azure 虚拟机 (VM) 或支持 SMB 3.0 的本地应用程序安装文件共享。"
        services="storage"
        documentationCenter="na"
        authors="jasontang501"
        manager="jahogg"
        editor="" />

<tags ms.service="storage"
      ms.date="02/29/2016"
      wacn.date="04/11/2016" />


# 如何通过 Linux 使用 Azure 文件存储 

## 概述

Azure 文件存储使用标准 SMB 协议在云中提供文件共享。使用 Azure 文件，你可以将依赖于文件服务器的企业应用程序迁移到 Azure。在 Azure 中运行的应用程序可以轻松地从运行 Linux 的 Azure 虚拟机装载文件共享。并且使用最新版本的文件存储，你还可以从支持 SMB 3.0 的本地应用程序装载文件共享。

你可以使用[管理门户](https://manage.windowsazure.cn)、Azure 存储空间 PowerShell cmdlet、Azure 存储空间客户端库或 Azure 存储空间 REST API 来创建 Azure 文件共享。此外，由于文件共享是 SMB 共享，因此你还可以通过标准的文件系统 API 来访问它们。

文件存储基于与 Blob、表和队列存储相同的技术构建，因此文件存储能够提供 Azure 存储平台内置的现有可用性、持续性、可伸缩性和异地冗余。有关存文件存储性能目标和限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

文件存储现已正式推出并同时支持 SMB 2.1 和 SMB 3.0。有关文件存储的更多详细信息，请参阅[文件服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)。

>[AZURE.NOTE] 由于 Linux SMB 客户端尚不支持加密，从 Linux 装载文件共享仍需要客户端与文件共享在同一 Azure 区域中。但是，Linux 的加密支持已经在负责 SMB 功能的 Linux 开发人员的路线图上。将来支持加密的 Linux 分发也将能够从任何位置装载 Azure 文件共享。

## 选择要使用的 Linux 分发 ##

在 Azure 中创建 Linux 虚拟机时，可以从 Azure 映像库指定支持 SMB 2.1 或更高版本的 Linux 映像。下面是建议的 Linux 映像的列表：

- Ubuntu Server 14.04	
- Ubuntu Server 15.04	
- CentOS 7.1	
- Open SUSE 13.2	
- SUSE Linux Enterprise Server 12
- SUSE Linux Enterprise Server 12 (Premium Image)

## 装载文件共享 ##

若要从运行 Linux 的虚拟机装载文件共享，可能需要安装 SMB/CIFS 客户端（如果你使用的分发没有内置的客户端）。这是 Ubuntu 中的命令，用于安装一个选择 cifs-utils：

    sudo apt-get install cifs-utils

接下来，你需要设置装入点 (mkdir mymountpoint)，然后发出类似于下面所示的装载命令：

     sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename ./mymountpoint -o vers=3.0,username=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

你还可以在 /etc/fstab 中添加设置以装载共享。

请注意，此处的 0777 代表目录/文件权限代码，用于向所有用户授予执行/读/写权限。你可以将其替换为 Linux 文件权限文档后面的其他文件权限代码。
 
另外要使文件共享在重新启动后装载，可以在 /etc/fstab 中添加如下设置：

    //myaccountname.file.core.chinacloudapi.cn/mysharename /mymountpoint cifs vers=3.0,username= myaccountname,password= StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777

例如，如果你是使用 Linux 映像 Ubuntu Server 15.04（可从 Azure 映像库中获得）创建的 Azure VM，则可以按如下所示装载文件：

    azureuser@azureconubuntu:~$ sudo apt-get install cifs-utils
    azureuser@azureconubuntu:~$ sudo mkdir /mnt/mountpoint
    azureuser@azureconubuntu:~$ sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
    azureuser@azureconubuntu:~$ df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

如果你使用 CentOS 7.1，则可以按如下所示装载文件：

    [azureuser@AzureconCent ~]$ sudo yum install samba-client samba-common cifs-utils
    [azureuser@AzureconCent ~]$ sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
    [azureuser@AzureconCent ~]$ df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

如果你使用 Open SUSE 13.2，则可以按如下所示装载文件：

    azureuser@AzureconSuse:~> sudo zypper install samba*  
    azureuser@AzureconSuse:~> sudo mkdir /mnt/mountpoint
    azureuser@AzureconSuse:~> sudo mount -t cifs //myaccountname.file.core.chinacloudapi.cn/mysharename /mnt/mountpoint -o vers=3.0,user=myaccountname,password=StorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
    azureuser@AzureconSuse:~> df -h /mnt/mountpoint
    Filesystem  Size  Used Avail Use% Mounted on
    //myaccountname.file.core.chinacloudapi.cn/mysharename  5.0T   64K  5.0T   1% /mnt/mountpoint

## 管理文件共享 ##

[Azure 管理门户](https://manage.windowsazure.cn/)提供用于管理 Azure 文件存储的用户界面。你可以从 Web 浏览器中执行以下操作：

- 将文件上载到文件共享和从文件共享下载文件。
- 监视每个文件共享的实际使用情况。
- 调整文件共享大小配额。
- 复制 `net use` 命令以用于从 Windows 客户端装载文件共享。 

还可以从 Linux 使用 Azure 跨平台命令行界面 (Azure CLI) 来管理文件共享。Azure CLI 提供了一组开放源代码跨平台命令，你可以使用这些命令来处理 Azure 存储（包括文件存储）。它提供 Azure 管理门户所能提供的相同功能，此外还有各种数据访问功能。有关示例，请参阅[将 Azure CLI 用于 Azure 存储空间](/documentation/articles/storage-azure-cli/)。

## 使用文件存储进行开发 ##

作为开发人员，你可以通过[适用于 Java 的 Azure 存储空间客户端库](https://github.com/azure/azure-storage-java)使用文件存储构建应用程序。有关代码示例，请参阅[如何通过 Java 使用文件存储](/documentation/articles/storage-java-how-to-use-file-storage/)。

你还可以使用[适用于 Node.js 的 Azure 存储空间客户端库](https://github.com/Azure/azure-storage-node)针对文件存储进行开发。

## 反馈和详细信息 ##

Linux 用户，我们希望倾听你的意见！

针对 Linux 用户组的 Azure 文件存储提供了一个论坛，当你在 Linux 上评估和采用文件存储时，可以在该论坛上共享反馈。向 [Azure 文件存储 Linux 用户](mailto:azurefileslinuxusers@microsoft.com)发送电子邮件可加入该用户组。

## 后续步骤

请参阅以下链接以获取有关 Azure 文件存储的更多信息。

### 概念性文章

- [Azure 文件存储：适用于 Windows 和 Linux 的顺畅的云 SMB 文件系统](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
- [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)

### 文件存储的工具支持

- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
- 通过 Azure CLI [创建和管理文件共享](/documentation/articles/storage-azure-cli/#create-and-manage-file-shares)

### 引用

- [文件服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)

### 博客文章

- [Azure 文件存储现已正式发布](/zh-cn/blog/)
- [Azure 文件存储内部](/home/features/storage) 
- [Azure 文件服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
- [将连接保存到 Azure 文件中](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)

<!---HONumber=Mooncake_0405_2016-->