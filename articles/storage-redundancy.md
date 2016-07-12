
<properties 
  pageTitle="Azure 存储复制 | Azure" 
  description="复制 Azure 存储帐户中的数据以实现持久性和高可用性。复制选项包括本地冗余存储 (LRS)、异地冗余存储 (GRS) 和读取访问异地冗余存储 (RA-GRS)。" 
  services="storage" 
  documentationCenter="" 
  authors="tamram" 
  manager="adinah" 
  editor=""/>

<tags 
  ms.service="storage" 
  ms.date="02/17/2016" 
  wacn.date="04/18/2016"/>

# Azure 存储复制

始终复制 Azure 存储帐户中的数据以确保持久性和高可用性，并且即使在遇到临时硬件故障时也符合 [Azure 存储 SLA 要求](/support/sla/storage) 要求。

创建存储帐户时，必须选择以下复制选项之一：

- [本地冗余存储 (LRS)](#locally-redundant-storage)
- [异地冗余存储 (GRS)](#geo-redundant-storage)
- [读取访问异地冗余存储 (RA-GRS)](#read-access-geo-redundant-storage)

下表简要概述了 LRS、GRS 和 RA-GRS 之间的差异，而后续章节将详细介绍每种类型的复制。


|复制策略|LRS|GRS|RA-GRS 
|--------------------|---|---|------
|数据在多个设施之间进行复制。|否|是|是|
|可以从辅助位置和主位置读取数据。|否|否|是
|在单独的节点上维护的数据副本数。|3|6|6


##<a id="locally-redundant-storage"></a> 本地冗余存储

本地冗余存储 (LRS) 在你创建存储帐户所在的区域中复制数据。为最大程度地提高持久性，针对你的存储帐户中的数据发出的每个请求将复制三次。这三个副本每个都驻留在不同的容错域和升级域中。容错域 (FD) 是一组代表出错的物理单元的节点，可将其视为属于同一物理机架的节点。升级域 (UD) 是一组在服务升级（部署）的过程中一起升级的节点。三个副本将分布在 UD 和 FD 上，以确保即使在硬件故障影响单个机架，以及在部署期间升级节点时，数据也可用。请求仅在写入所有三个副本后，才成功返回。

虽然对于大多数应用程序建议使用异地冗余存储 (GRS)，但是本地冗余存储在某些情况下可能需要：

- 比 GRS 相比，LRS 成本更低，并且还提供更高的吞吐量。如果应用程序存储可轻松重构的数据，则可以选择 LRS。

- 由于数据管理需要，某些应用程序被限制为只能在单个区域内复制数据。

- 如果应用程序具有自己的异地复制策略，则可能不需要 GRS。

##<a id="geo-redundant-storage"></a> 异地冗余存储 

异地冗余存储 (GRS) 将数据复制到距主区域数百英里以外的辅助区域。如果你的存储帐户启用了 GRS，则即使遇到区域完全停电或导致主区域不可恢复的灾难，你的数据也能持久保存。

对于启用了 GRS 的存储帐户，更新将首先提交到主区域，并在主区域复制三次。然后，更新将复制到辅助区域（也是复制三次），并且是在不同的容错域和升级域之间复制。


> [AZURE.NOTE] 使用 GRS 时，写入数据请求将异步复制到辅助区域。请务必注意，选择 GRS 不会影响针对主区域发出的请求的延迟。但是，由于异步复制涉及延迟，遇到区域性灾难时，如果无法将数据从主区域中恢复，则尚未复制到辅助区域的更改可能会丢失。

创建存储帐户时，可以为帐户选择主区域。辅助区域是根据主区域确定的且无法更改。下表显示了配对的主要区域和次要区域。

|主要 |辅助
| ---------------   |----------------
|中国北部 |中国东部
|中国东部 |中国北部
 
##<a id="read-access-geo-redundant-storage"></a> 读取访问异地冗余存储

除了 GRS 所提供的在两个区域之间进行复制外，读取访问异地冗余存储 (RA-GRS) 还提供对辅助位置中的数据的只读访问权限，从而最大限度地提高了存储帐户的可用性。当主区域中的数据不可用时，应用程序可以从辅助区域读取数据。

当你启用对辅助区域中的数据的只读访问权限时，除了你的存储帐户的主终结点外，你还可以在辅助终结点上获取数据。辅助终结点与主终结点类似，但会在帐户名称后面追加后缀`-secondary`。例如，如果 Blob 服务的主终结点是  `myaccount.blob.core.chinacloudapi.cn`，则辅助终结点是  `myaccount-secondary.blob.core.chinacloudapi.cn`。存储帐户的访问密钥对于主终结点和辅助终结点是相同的。

## 后续步骤

- [关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)
- [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/) 
- [Azure 存储冗余选项和读取访问异地冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)  
- [使用 RA-GRS 的  Azure 存储模拟器 3.1](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/08/microsoft-azure-storage-emulator-3-1-with-ra-grs.aspx)
- [SOSP 论文 - Azure 存储空间：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)  
 
<!---HONumber=Mooncake_0411_2016-->