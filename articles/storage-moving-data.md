<properties
	pageTitle="将数据移动到和移出 Azure 存储空间 | Azure"
	description="本文提供了有关将数据移到和移出 Azure 存储空间的不同方法的概述。"
	services="storage"
	documentationCenter=""
	authors="micurd"
	manager="jahogg"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.date="03/18/2016"
	wacn.date="04/11/2016"/>

# 将数据移动到和移出 Azure 存储空间

如果你想将本地数据移动到 Azure 存储空间（或执行相反的操作），有许多种方式可以执行此操作。最适合你的方法将取决于你的方案。本文将提供不同方案以及针对每个方案的适当产品/服务的快速概述。

## 构建应用程序

如果你正在构建应用程序，针对 REST API 或我们的许多客户端库之一进行开发是将数据移动到和移出 Azure 存储空间的一个好办法。

Azure 存储空间为 .NET、iOS、Java、Android、通用 Windows 平台 (UWP)、Xamarin、C++、Node.JS、PHP、Ruby 和 Python 提供了丰富的客户端库。客户端库提供高级功能，如重试逻辑、日志记录和并行上传。你也可以直接针对可以由发出 HTTP/HTTPS 请求的任何语言调用的 REST API 进行开发。

请参阅[开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)以了解详细信息。

## 快速查看数据/与数据进行交互

如果你想要轻松地查看你的 Azure 存储空间数据，同时也能上传和下载你的数据，那么请考虑使用 Azure 存储空间资源管理器。

请查看我们的 [Azure 存储资源管理器](/documentation/articles/storage-explorers/)列表以了解详细信息。

## 系统管理

如果你需要或更熟悉命令行实用程序（例如系统管理员），以下是供你考虑的一些选项：

### AzCopy

AzCopy 是一个 Windows 命令行实用程序，旨在实现高性能地将数据复制到 Azure 存储空间和从 Azure 存储空间中复制。你还可以在存储帐户内或者在不同的存储帐户之间复制数据。

请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)以了解详细信息。

### Azure PowerShell

Azure PowerShell 是一个模块，它提供用于管理 Azure 上的服务的 cmdlet。这是一种基于任务的命令行外壳和脚本语言，专为系统管理而设计。

请参阅[通过 Azure 存储空间使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full/)以了解详细信息。

### Azure CLI

Azure CLI 提供了一组开源且跨平台的命令，这些命令可以用于 Azure 服务。Azure CLI 在 Windows、OSX 和 Linux 上可用。

请参阅[通过 Azure 存储空间使用 Azure CLI](/documentation/articles/storage-azure-cli/) 以了解详细信息。



## 备份你的数据

如果你只需将你的数据备份至 Azure 存储空间，Azure 备份是一个选择。这是一个用于备份本地数据和 Azure VM 的强大解决方案。

请参阅 [Azure 备份](/documentation/articles/backup-introduction-to-azure-backup/)以了解详细信息。



## 恢复你的数据

当你拥有本地工作负荷和应用程序时，你将需要一个解决方案，允许你的业务在发生灾难时继续运行。Azure Site Recovery 可以处理虚拟机和物理服务器的复制、故障转移与恢复。复制的数据存储在 Azure 存储空间中，使你不再需要辅助现场数据中心。

请参阅 [Azure Site Recovery](/documentation/articles/site-recovery-overview/) 以了解详细信息。

<!---HONumber=Mooncake_0405_2016-->