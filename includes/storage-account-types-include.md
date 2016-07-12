有两种类型的存储帐户：

### 通用存储帐户

通用存储帐户有权使用单个帐户访问诸如表、队列、文件、Blob 和 Azure 虚拟机磁盘等 Azure 存储空间服务。此类型存储帐户具有两个性能层：

- 标准存储性能层，允许存储表、队列、文件、Blob 和 Azure 虚拟机磁盘。
- 高级存储性能层，当前仅支持 Azure 虚拟机磁盘。有关高级存储的详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。

### Blob 存储帐户

Blob 存储帐户是将非结构化数据作为 Blob（对象）存储在 Azure 存储空间的专用存储帐户。Blob 存储帐户类似于现有的通用存储帐户，并且具有你现在使用的所有卓越的耐用性、可用性、可伸缩性和性能功能，包括用于块 blob 和追加 blob 的 100% API 一致性。对于仅需要块 blob 或追加 blob 存储的应用程序，我们建议使用 Blob 存储帐户。

> [AZURE.NOTE] Blob 存储帐户仅支持块 blob 和追加 blob，不支持页 blob。

Blob 存储帐户公开**访问层**属性，该属性可在帐户创建过程中指定，并稍后根据需要进行修改。



你必须具有 Azure 订阅（这是允许你访问各种 Azure 服务的计划），然后才能创建存储帐户。你可以使用 [1rmb 帐户](/pricing/1rmb-trial/)开始使用 Azure。一旦决定购买某个订阅计划，你可以从各种[购买选项](/pricing/purchase-options/)中进行选择。有关批量定价的信息，请参阅 [Azure Storage Pricing（Azure 存储空间定价）](/home/features/storage/pricing/)。

若要了解如何创建存储帐户，请参阅[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)以获取详细信息。通过单个订阅，你最多可以创建 100 个唯一的命名存储帐户。有关存储帐户限制的详细信息，请参阅 [Azure Storage Scalability and Performance Targets（Azure 存储空间可伸缩性和性能目标）](/documentation/articles/storage-scalability-targets/)。

<!---HONumber=Mooncake_0530_2016-->