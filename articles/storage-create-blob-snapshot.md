<properties 
	pageTitle="创建 Blob 的快照" 
	description="有关创建 Azure 存储 Blob 快照的操作方法指南" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date=""
    wacn.date="04/15/2015"
    />










# 创建 Blob 的快照

你可以创建 Blob 的快照。快照是在某一时间点拍摄的只读版本的 Blob。在创建快照后，可以读取、复制或删除该快照，但无法对其进行修改。利用快照，你可以在某个时间点备份 Blob。

Blob 的快照具有与从中拍摄快照的基本 Blob 相同的名称，并且后面会附加一个 DateTime 值以指示拍摄快照的时间。例如，如果页 Blob URI 为 http://storagesample.core.blob.chinacloudapi.cn/mydrives/myvhd，则快照 URI 将与 http://storagesample.core.blob.chinacloudapi.cn/mydrives/myvhd?snapshot=2011-03-09T01:42:34.9360000Z 类似。此值可用于引用快照，以供将来操作之用。一个 Blob 的多个快照共享其 URI，仅按照此 DateTime 值进行区分。在客户端库代码中，Blob 的 Snapshot 属性会返回一个 DateTime 值，该值唯一标识相对于其基本 Blob 的快照。你可使用此值来对快照执行其他操作。

一个 Blob 可以有任意数目的快照。在显式删除快照前，快照会一直存在。快照的生存期限不能超过其源 Blob。你可以枚举与 Blob 关联的快照，以跟踪当前快照。

## 继承属性

创建 Blob 的快照时，会将系统属性复制到具有相同值的快照。此列表显示 .NET 存储客户端库的复制的系统属性：

- ContentType 

- ContentEncoding 

- ContentLanguage

- Length

- CacheControl

- ContentMd5 


不会将与基本 Blob 关联的租约复制到快照。不能租用快照。

## 复制快照 

涉及 Blob 和快照的复制操作遵循以下规则：

- 可以将快照复制到其基本 Blob 上。通过将快照提升到基本 Blob 的位置，可还原早期版本的 Blob。快照会保留，但会使用可读写的副本覆盖其源。

- 你可将快照复制到具有不同名称的目标 Blob。生成的目标 Blob 是可编写的 Blob，而不是快照。

- 复制源 Blob 时，不会将该源 Blob 的任何快照复制到目标。使用副本覆盖目标 Blob 时，与该目标 Blob 关联的所有快照都根据其名称保持不变。 

- 创建块 Blob 的快照时，也会将该 Blob 的已提交块列表复制到快照。不会复制任何未提交的块。

## 指定访问条件 

你可指定访问条件，以便仅在符合某条件时创建快照。若要指定访问条件，请使用 AccessCondition 属性。如果不符合指定条件，则不会创建快照，并且 Blob 服务会返回状态代码 HTTPStatusCode.PreconditionFailed。

## 删除快照 

不能删除具有快照的 Blob，除非也删除这些快照。可单独删除快照，也可指示存储服务在删除源 Blob 时删除所有快照。如果尝试删除仍包含快照的 Blob，那么你的调用会返回错误。

## 在高级存储中使用快照
在高级存储中使用快照需遵循以下规则：

- 高级存储帐户中每个页 Blob 的快照数限制为 100。如果超出该限制，快照 Blob 操作将返回错误代码 409 (SnapshotCountExceeded)。

- 可以每隔十分钟拍摄高级存储帐户中页 Blob 的快照一次。如果超出该速率，快照 Blob 操作将返回错误代码 409 (SnaphotOperationRateExceeded)。

- 不支持通过"获取 Blob"读取通过高级存储帐户中页 Blob 的快照。对高级存储帐户中的快照调用"获取 Blob"将返回错误代码 400（无效操作）。但是，支持针对快照调用"获取 Blob 属性"和"获取 Blob 元数据"。

- 若要读取快照，你可以使用"复制 Blob"操作将快照复制到帐户中的另一个页 Blob。复制操作的目标 Blob 不能包含任何现有快照。如果目标 Blob 确实包含快照，则"复制 Blob"将返回错误代码 409 (SnapshotsPresent)。

## 构造快照的绝对 URI 

以下代码示例从快照的基本 Blob 对象构造快照的绝对 URI。

C#

	var snapshot = blob.CreateSnapshot();
	var uri = Microsoft.WindowsAzure.StorageClient.Protocol.BlobRequest.Get
    (snapshot.Uri, 
    0, 
    snapshot.SnapshotTime.Value, 
    null).Address.AbsoluteUri;

<!--HONumber=50-->