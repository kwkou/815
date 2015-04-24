<properties 
	pageTitle="了解快照如何产生费用" 
	description="帮助你了解 Azure 存储 Blob 快照计费方式的指南" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date=""
    wacn.date="04/15/2015"
    />










# 了解快照如何产生费用

创建快照（它是 Blob 的只读副本）会导致你的帐户产生额外的数据存储费用。在设计应用程序时，你有必要了解在哪些情况下会产生这些费用，以便能最大程度地减少不必要的费用。

## 重要计费注意事项

以下列表包含创建快照时要考虑的要点。

- 唯一的块或页面会产生费用，无论它们是在 Blob 还是快照中。在更新快照所基于的 Blob 之前，你的帐户将不会就与 Blob 关联的快照产生额外费用。在更新基本 Blob 后，它将偏离其快照，并且将就每个 Blob 或快照中的唯一块或页向你收取费用。

- 在替换块 Blob 中的某个块后，会将该块作为唯一块进行收费。即使该块具有的块 ID 和数据与它在快照中所具有的 ID 和数据相同也是如此。重新提交块后，它将偏离它在任何快照中的对应部分，并且你将就其数据支付费用。对于使用相同的数据更新的页 Blob 中的页面，也是如此。

- 通过调用 UploadFile、UploadText、UploadStream 或 UploadByteArray 方法替换块 Blob 可替换该 Blob 中的所有块。如果你具有与该 Blob 关联的快照，则基本 Blob 和快照中的所有块现在将发生偏离，并且你需为这两个 Blob 中的所有块支付费用。即使基本 Blob 和快照中的数据保持相同也是如此。

- Azure BLOB 服务无法确定这两个块是否包含相同的数据。每个上载和提交的块均被视为唯一的快，即使它具有相同的数据和块 ID 也是如此。由于唯一的块会产生费用，因此考虑到更新具有快照的 Blob 将导致产生其他唯一块和额外费用这一点很重要。

> [AZURE.NOTE] 最佳实践要求你仔细管理快照以避免额外费用。建议你通过以下方式管理快照：

> - 除非你的应用程序设计需要保留与 Blob 关联的快照，否则请在更新 Blob 时删除并重新创建这些快照，即使你使用相同的数据进行更新也是如此。通过删除并重新创建 Blob 的快照，可以确保 Blob 和快照不会发生偏离。

> - 如果要保留 Blob 的快照，请避免调用 UploadFile、UploadText、UploadStream 或 UploadByteArray 来更新 Blob，因为这些方法将替换 Blob 中的所有块。相反，请使用 PutBlock 和 PutBlockList 方法更新尽可能少的块。


## 快照计费方案


下列方案说明了块 Blob 及其快照将如何产生费用。 

在方案 1 中，基本 Blob 自拍摄快照后未进行更新，因此仅唯一块 1、2、3 会产生费用。

![Azure Storage Resources](./media/storage-blob-snapshots-billing/storage-blob-snapshots-billing-scenario-1.png)

在方案 2 中，已更新基本 Blob，但未更新快照。已更新块 3，即使它包含相同的数据和 ID，它也与快照中的块 3 不同。因此，帐户需要为四个块支付费用。

![Azure Storage Resources](./media/storage-blob-snapshots-billing/storage-blob-snapshots-billing-scenario-2.png)

在方案 3 中，已更新基本 Blob，但未更新快照。块 3 已替换为基础 Blob 中的块 4，但快照仍反映块 3。因此，帐户需要为四个块支付费用。
 
![Azure Storage Resources](./media/storage-blob-snapshots-billing/storage-blob-snapshots-billing-scenario-3.png)

在方案 4 中，已完全更新基本 Blob，并且其中不包含任何原始块。因此，帐户需要为所有八个唯一块支付费用。如果你使用如 UploadFile、UploadText、UploadFromStream 或 UploadByteArray 之类的更新方法，则会出现此情况，因为这些方法将替换 Blob 的所有内容。

![Azure Storage Resources](./media/storage-blob-snapshots-billing/storage-blob-snapshots-billing-scenario-4.png)

<!--HONumber=50-->