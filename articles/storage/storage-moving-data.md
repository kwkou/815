<properties
    pageTitle="将数据移入和移出 Azure 存储 | Azure"
    description="本文概述了将数据移入和移出 Azure 存储的各种方法。"
    services="storage"
    documentationcenter=""
    author="micurd"
    manager="jahogg"
    editor="tysonn" />  

<tags
    ms.assetid="5e3947a9-d99b-4108-9d57-3eb67c03e7ba"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/18/2016"
    wacn.date="12/05/2016"
    ms.author="micurd" />  


# 将数据移入和移出 Azure 存储
若要将本地数据移动到 Azure 存储（或反向移动），可通过多种方式执行此操作。最适合的方法因具体情况而异。本文将提供不同方案以及针对每个方案的适当产品/服务的快速概述。

## 构建应用程序
若要构建应用程序，可针对 REST API 或我们众多客户端库之一进行开发，将数据移入和移出 Azure 存储。

Azure 存储为 .NET、iOS、Java、Android、通用 Windows 平台 (UWP)、Xamarin、C++、Node.JS、PHP、Ruby 和 Python 提供了丰富的客户端库。客户端库提供高级功能，如重试逻辑、日志记录和并行上传。也可以直接针对 REST API（可发出 HTTP/HTTPS 请求的任何语言都可调用它）进行开发。

请参阅[开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)了解详细信息。

此外，我们还提供了 [Azure 存储数据移动库](https://www.nuget.org/packages/Microsoft.Azure.Storage.DataMovement)，该库专为高性能设计，用于将数据复制到 Azure 和从 Azure 复制数据。请参阅数据移动库[文档](https://github.com/Azure/azure-storage-net-data-movement)，以了解详细信息。

## 快速查看数据/与数据进行交互
如果希望轻松查看 Azure 存储数据，同时能够上传和下载数据，请考虑使用 Azure 存储资源管理器。

请查看 [Azure 存储资源管理器](/documentation/articles/storage-explorers/)列表，了解详细信息。

## 系统管理
如果需要或更愿意使用命令行实用程序（例如系统管理员），可考虑以下选项：

### AzCopy
AzCopy 是一个 Windows 命令行实用程序，用于将数据高性能复制到 Azure 存储（或从中进行复制）。还可在存储帐户内或在存储帐户间复制数据。

请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)，了解详细信息。

### Azure PowerShell
Azure PowerShell 是一个模块，它提供用于管理 Azure 上的服务的 cmdlet。这是一种基于任务的命令行 Shell 和脚本语言，专为系统管理而设计。

请参阅[通过 Azure 存储使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full/)，了解详细信息。

### Azure CLI
Azure CLI 提供了一组开源的跨平台命令，可以用于 Azure 服务。Azure CLI 在 Windows、OSX 和 Linux 上可用。

请参阅[通过 Azure 存储使用 Azure CLI](/documentation/articles/storage-azure-cli/)，了解详细信息。



## 备份数据

如果只需将数据备份到 Azure 存储，请使用 Azure 备份。它是用于备份本地数据和 Azure VM 的强大解决方案。

请参阅 [Azure 备份](/documentation/articles/backup-introduction-to-azure-backup/)，了解详细信息。

## 访问本地和云中的数据
如果需要用于访问本地和云中的数据的解决方案，则应考虑使用 Azure 的混合云存储解决方案 StorSimple。此解决方案包含一个物理 StorSimple 设备，该设备智能地将频繁使用的数据存储在 SSD 上，将偶尔使用的数据存储在 HDD 上，并将非活动/备份/存档数据存储在 Azure 存储上。

## 恢复数据
拥有本地工作负荷和应用程序时，需要让业务能够在发生灾难时继续运行的解决方案。Azure Site Recovery 可以处理虚拟机和物理服务器的复制、故障转移与恢复。复制的数据存储在 Azure 存储中，因此不再需要辅助现场数据中心。

请参阅 [Azure Site Recovery](/documentation/articles/site-recovery-overview/) 了解详细信息。

<!---HONumber=Mooncake_1128_2016-->