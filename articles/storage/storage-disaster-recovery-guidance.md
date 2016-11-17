<properties
	pageTitle="Azure 存储空间中断时怎么办 | Azure"
	description="Azure 存储空间中断时怎么办"
	services="storage"
	documentationCenter=".net"
	authors="robinsh"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="08/03/2016"
	wacn.date="11/16/2016"
	ms.author="jutang;robinsh"/>


# 在 Azure 存储空间中断时该怎么办

Azure 一直努力确保所提供的服务始终可用。但有时候，各种不可控因素会导致一个或多个区域出现计划外服务中断，对我们造成影响。为了帮助你应对这些偶发事件，我们提供了下述针对 Azure 存储服务的概述性指导。

## 如何准备 

每个客户都应准备好自己的灾难恢复计划，这很重要。从存储中断进行恢复时，通常需要操作人员和自动化过程的参与，目的是在正常运行状态下重新激活你的应用程序。制定你自己的灾难恢复计划时，请参阅以下 Azure 文档：

-   [Azure Site Recovery 服务](/home/features/site-recovery/)

-   [Azure 存储空间复制](/documentation/articles/storage-redundancy/)

-   [Azure 备份服务](/home/features/back-up/)

## 如何检测 

若要确定 Azure 服务状态，建议你订阅 [Azure 服务运行状况仪表板](/support/service-dashboard/)。

## 在存储空间中断时该怎么办

如果一个或多个区域的一个或多个存储服务临时不可用，你可以考虑两种选项。如果你需要立即访问数据，请考虑“选项 2”。

### 选项 1：等待恢复

在此情况下，你不需要采取任何操作。我们正在努力还原 Azure 服务的可用性。你可以在 [Azure 服务运行状况仪表板](/support/service-dashboard/)上监视服务状态。

### 选项 2：从辅助数据库复制数据

如果你为存储帐户选择[读取访问异地冗余存储 (RA-GRS)](/documentation/articles/storage-redundancy/#read-access-geo-redundant-storage)（推荐），你就可以从次要区域访问数据。你可以使用 [AzCopy](/documentation/articles/storage-use-azcopy/)、[Azure PowerShell](/documentation/articles/storage-powershell-guide-full/) 和 [Azure 数据移动库](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)之类的工具将数据从次要区域复制到不受影响区域的其他存储帐户中，然后将应用程序指向该存储帐户，以确保读取和写入可用性。

## 进行存储空间故障转移时会发生什么情况

如果你选择了[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage) 或 [读取访问地域冗余存储 (RA-GRS)](/documentation/articles/storage-redundancy/#read-access-geo-redundant-storage)（推荐），Azure 存储空间会将你的数据持久保存在两个区域（主要区域和次要区域）。在这两个区域，Azure 存储空间始终维护你数据的多个副本。

当区域灾难影响你的主要区域时，我们会首先尝试还原该区域的服务。在很少的情况下，我们可能无法还原主要区域，具体取决于灾难的性质及其影响。在那种情况下，我们会进行异地故障转移。跨区域数据复制是一个可能有延迟的异步过程，因此，可能会丢失尚未复制到次要区域的更改。你可以通过查询[存储帐户的“上次同步时间”](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/)，获取有关复制状态的详细信息。

有关存储空间异地故障转移体验的一些观点：

-   只能通过 Azure 存储团队触发存储空间异地故障转移 – 不需客户操作。

-   针对 Blob、表、队列和文件的现有存储服务终结点在故障转移后保持不变；需要更新 DNS 条目才能从主要区域切换到次要区域。

-   在异地故障转移之前和过程中，由于灾难的影响，你将无法对存储帐户进行写入访问，但如果你的存储帐户已配置为 RA-GRS，你仍然可以从次要区域进行读取。

-   完成异地故障转移且传播 DNS 更改以后，将恢复你对存储帐户的读取和写入访问权限。你可以查询[存储帐户的“上次异地故障转移时间”](https://msdn.microsoft.com/zh-cn/library/azure/ee460802.aspx)以获取更多详细信息。

-   在故障转移之后，你的存储帐户完全可以正常使用，但处于“已降级”状态，因为实际上它是托管在独立区域中，不可能进行异地复制。为了缓解此风险，我们需要还原原始的主要区域，然后通过异地故障回复还原原始状态。如果原始的主要区域不可恢复，我们会分配其他次要区域。
有关 Azure 存储异地复制基础结构的更多详细信息，请参阅存储团队博客中有关[冗余选项和 RA-GRS](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/) 的文章。

##数据保护最佳实践

可以通过一些推荐的方法定期备份你的存储数据。

-   VM 磁盘 – 使用 [Azure 备份服务](/home/features/back-up/)备份 Azure虚拟机使用的 VM 磁盘。

-   块 Blob – 使用 [AzCopy](/documentation/articles/storage-use-azcopy/)、[Azure PowerShell](/documentation/articles/storage-powershell-guide-full/) 或 [Azure 数据移动库](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)创建每个块 Blob 的[快照](https://msdn.microsoft.com/zh-cn/library/azure/hh488361.aspx)，或者将 Blob 复制到其他区域的其他存储帐户。

-   表 – 使用 [AzCopy](/documentation/articles/storage-use-azcopy/) 将表数据导出到其他区域的其他存储帐户中。

-   文件 – 使用 [AzCopy](/documentation/articles/storage-use-azcopy/) 或 [Azure PowerShell](/documentation/articles/storage-powershell-guide-full/) 将文件复制到其他区域的其他存储帐户。

<!---HONumber=Mooncake_0829_2016-->