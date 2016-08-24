<properties
    pageTitle="Blob 的 Azure 冷存储 | Azure"
    description="Azure Blob 存储的存储层基于访问模式为对象数据提供经济高效的存储。冷存储层适用于不常访问的数据。"
    services="storage"
    documentationCenter=""
    authors="sribhat-msft"
    manager=""
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.date="07/05/2016"
    wacn.date="08/01/2016"/>


# Azure Blob 存储：“热”和“冷”存储层

## 概述

Azure 存储空间现在为 Blob 存储（对象存储）提供两个存储层，以便你可以根据你使用数据的方式最经济高效地存储数据。Azure **热存储层**为存储经常访问的数据进行了优化。Azure **冷存储层**为存储不常访问且长期留存的数据进行了优化。冷存储层中的数据可容许略低的可用性，但仍需要像热数据一样的高持续性和类似的访问时间以及吞吐量特征。对于冷数据，略低的可用性 SLA 和较高的访问成本对于更低的存储成本而言是可接受的折衷。

如今，存储在云中的数据以指数速度增长。若要为扩展的存储需求管理成本，根据属性（如访问频率和计划保留期）整理数据将很有帮助。存储在云中的数据在其生成方式、处理方式以及在生存期内的访问方式等方面会大不相同。某些数据在其整个生存期中都会受到积极的访问和修改。某些数据则在生存期早期会受到很频繁地访问，随着数据变旧，访问会极大地减少。某些数据在云中保持空闲状态，并且在存储后很少（如果有）被访问。

上面所述的这些数据访问方案的每一个都受益于针对特定访问模式进行了优化的不同存储层。随着热和冷存储层的引入，Azure Blob 存储现在使用单独的定价模型来满足不同存储层的需要。

## Blob 存储帐户

**Blob 存储帐户**是将非结构化数据作为 Blob（对象）存储在 Azure 存储空间的专用存储帐户。使用 Blob 存储帐户，你现在可以在热和冷存储层之间进行选择，以便以较低的存储成本存储你不经常访问的冷数据，并且以较低的访问成本存储更频繁访问的热数据。Blob 存储帐户类似于你现有的通用存储帐户，并且具有你现在使用的所有卓越的持续性、可用性、可伸缩性和性能功能，包括用于块 blob 和追加 blob 的 100% API 一致性。

> [AZURE.NOTE] Blob 存储帐户仅支持块 blob。

Blob 存储帐户公开**访问层**属性，它允许你将存储层指定为**热**或**冷**，具体取决于存储在帐户中的数据。如果你的数据的使用模式有所更改，你也可以随时在这些存储层之间切换。

> [AZURE.NOTE] 更改存储层可能会产生额外费用。有关更多详细信息，请参阅[定价和计费](/documentation/articles/storage-blob-storage-tiers/#pricing-and-billing)部分。

热存储层的示例使用方案包括：

- 处于活跃使用状态或预期会频繁访问（读取和写入）的数据。
- 分阶段进行处理，并最终迁移至冷存储层的数据。

冷存储层的示例使用方案包括：

- 备份、存档和灾难恢复数据集。
- 不再经常查看、但访问时应立即可用的较旧的媒体内容。
- 在收集更多数据以便将来处理时需要经济高效地存储的大型数据集。（例如，长期存储的科学数据、来自制造设施的原始遥测数据）
- 必须保留的原始数据，即使它已被处理为最终可用的形式。（例如，转码为其他格式后的原始媒体文件）
- 需要长时间存储并且几乎不访问的合规性和存档数据。（例如，安全相机数据片段、面向医疗保健机构旧 X-Rays/MRI、面向金融服务的客户呼叫录音和记录）

有关存储帐户的详细信息，请参阅[关于 Azure 存储空间帐户](/documentation/articles/storage-create-storage-account/)。

对于仅需要块 blob 或追加 blob 存储的应用程序，我们建议使用 Blob 存储帐户，以利用分层存储的不同定价模型。但是，我们知道这在某些情况（可以选择使用通用存储帐户）下是不可能的，如：

- 你需要使用表、队列或文件，并且想要 Blob 存储在同一个存储帐户中。请注意，除了具有相同的共享密钥以外，将这些存储在同一帐户中没有技术优势。
- 你仍需要使用经典部署模型。Blob 存储帐户只有通过 Azure Resource Manager 部署模型才可用。
- 你需要使用页 blob。Blob 存储帐户不支持页 blob。我们通常建议使用块 blob，除非你对页 blob 有特定需求。
- 使用早于 2014-02-14 的[存储服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd894041.aspx)的版本或使用版本低于 4.x 的客户端库，并且无法升级你的应用程序。

> [AZURE.NOTE] Blob 存储帐户当前在大多数 Azure 区域受支持，后续会有更多。

## 存储层之间的比较

下表突出显示了两种存储层之间的比较：

<table border="1" cellspacing="0" cellpadding="0" style="border: 1px solid #000000;">
<col width="250">
<col width="250">
<col width="250">
<tbody>
<tr>
    <td><strong><center></center></strong></td>
    <td><strong><center>热存储层</center></strong></td>
    <td><strong><center>冷存储层</center></strong></td>
</tr>
<tr>
    <td><strong><center>可用性</center></strong></td>
    <td><center>99.9%</center></td>
    <td><center>99%</center></td>
</tr>
<tr>
    <td><strong><center>可用性<br>（RA-GRS 读取）</center></strong></td>
    <td><center>99.99%</center></td>
    <td><center>99.9%</center></td>
</tr>
<tr>
    <td><strong><center>使用费</center></strong></td>
    <td><center>存储成本较高<br>访问和事务成本较低</center></td>
    <td><center>存储成本较低<br>访问和事务成本较高</center></td>
</tr>
<tr>
    <td><strong><center>最小对象大小<center></strong></td>
    <td colspan="2"><center>不适用</center></td>
</tr>
<tr>
    <td><strong><center>最小存储持续时间<center></strong></td>
    <td colspan="2"><center>不适用</center></td>
</tr>
<tr>
    <td><strong><center>延迟<br>（距第一字节时间）<center></strong></td>
    <td colspan="2"><center>毫秒</center></td>
</tr>
<tr>
    <td><strong><center>可伸缩性和性能目标<center></strong></td>
    <td colspan="2"><center>与通用存储帐户相同</center></td>
</tr>
</tbody>
</table>

> [AZURE.NOTE] Blob 存储帐户支持与通用存储帐户相同的性能和可伸缩性目标。有关详细信息，请参阅 [Azure 存储空间的可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

##<a id="pricing-and-billing"></a> 定价和计费

Blob 存储帐户使用基于存储层的 Blob 存储的新定价模型。使用 Blob 存储帐户时，请注意以下计费方式：

- **存储成本**：除了存储的数据量，存储数据的成本因存储层而异。冷存储层每 GB 的成本比热存储层每 GB 的成本要低一些。
- **数据访问成本**：对于冷存储层中的数据，你需要为读取和写入支付每 GB 的数据访问费用。
- **事务成本**：两个层都存在每个事务费用。但是，冷存储层的每个事务成本比热存储层的每个事务成本要高一些。
- **异地复制数据传输成本**：这仅适用于配置了异地复制的帐户，包括 GRS 和 RA-GRS。异地复制数据传输会产生每 GB 费用。
- **出站数据传输成本**：出站数据传输（传出 Azure 区域的数据）会按每 GB 产生带宽使用费，与通用存储帐户一致。
- **更改存储层**：将存储层从“冷”更改为“热”将会产生费用，此费用等于读取每个转换的存储帐户中存在的所有数据的费用。另一方面，将存储层从“热”更改为“冷”则免费。

> [AZURE.NOTE] 为了使用户能够试用新的存储层并在上市后验证功能，在 2016 年 6 月 30 日之前，将免收将存储层从“冷”更改为“热”的费用。从 2016 年 7 月 1 日 开始，将对所有从“冷”到“热”的转换收取费用。有关 Blob 存储帐户的定价模型的详细信息，请参阅 [Azure 存储空间定价](/pricing/details/storage/)页。有关出站数据传输费用的详细信息，请参阅[数据传输定价详细信息](/pricing/details/data-transfer/)页。

## 快速启动

在本部分，我们将使用 Azure 门户预览演示以下方案：

- 如何创建 Blob 存储帐户。
- 如何管理 Blob 存储帐户。

### 使用 Azure 门户预览

#### 使用 Azure 门户预览创建 Blob 存储帐户

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。

2. 在“中心”菜单上，选择“新建”->“数据 + 存储”->“存储帐户”。

3. 输入你的存储帐户的名称。

	该名称必须全局唯一；在访问存储帐户中的对象时，该名称用作所需 URL 的一部分。

4. 选择 **Resource Manager** 作为部署模型。

	分层存储只能用于 Resource Manager 存储帐户；建议对新资源使用该部署模型。有关详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

5. 在“帐户类型”下拉列表中，选择“Blob 存储”。

	你可以在这里选择存储帐户的类型。分层存储不适用于通用存储，只适用于 Blob 存储类型帐户。

	请注意，当你选择此项时，性能层将设置为“标准”。分层存储不适用于“高级”性能层。

6. 选择存储帐户的复制选项：“LRS”、“GRS”或“RA-GRS”。默认值为“RA-GRS”。

	LRS = 本地冗余存储；GRS = 异地冗余存储（2 个区域）；RA-GRS 是可以进行读取访问的异地冗余存储（2 个区域，可以对第二个区域进行读取访问）。

	有关 Azure 存储空间复制选项的详细信息，请参阅 [Azure 存储空间复制](/documentation/articles/storage-redundancy/)。

7. 根据需要选择正确的存储层：将“访问层”设置为“冷”或“热”。默认值为“热”。

8. 选择想在其中创建新存储帐户的订阅。

9. 指定新资源组或选择现有资源组。有关资源组的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles//resource-group-overview)。

10. 选择存储帐户的区域。

11. 单击“创建”以创建存储帐户。

#### 使用 Azure 门户预览更改 Blob 存储帐户的存储层

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。

2. 若要导航到你的存储帐户，请先选择“所有资源”，然后选择你的存储帐户。

3. 在“设置”边栏选项卡中，单击“配置”以查看和/或更改帐户配置。

4. 根据需要选择正确的存储层：将“访问层”设置为“冷”或“热”。

5. 单击边栏选项卡顶部的“保存”。

> [AZURE.NOTE] 更改存储层可能会产生额外费用。有关更多详细信息，请参阅[定价和计费](/documentation/articles/storage-blob-storage-tiers/#pricing-and-billing)部分。

## 评估 Blob 存储帐户和迁移到 Blob 存储帐户

本部分旨在帮助用户顺利转换成使用 Blob 存储帐户。有两个用户方案：

- 你已经有了一个通用存储帐户，想要使用适当的存储层来评估对 Blob 存储帐户所做的更改。
- 你已经决定使用 Blob 存储帐户，或者已经有了一个这种帐户，想要评估一下是应使用热存储层还是冷存储层。

在这两种情况下，首要任务是评估一下对存储在 Blob 存储帐户中的数据进行存储和访问操作所需的成本，并将该成功与当前成本进行比较。

### 评估 Blob 存储帐户层

若要评估对存储在 Blob 存储帐户中的数据进行存储和访问操作所需的成本，你需要评估现有的使用模式，或对预期的使用模式进行一个大致的估计。一般情况下，你将想要了解：

- 存储消耗 - 正在存储的数据量以及每月如何变化？
- 存储访问模式 - 从帐户读取以及写入到帐户的数据量（包括新数据）？ 使用了多少事务来进行数据访问？这些事务是什么类型的事务？

#### 监视现有存储帐户

若要监视现有存储帐户并收集该数据，你可以利用 Azure 存储分析进行日志记录，并为存储帐户提供度量数据。
存储分析可以存储的度量值包括有关 Blob 存储服务请求的聚合事务统计信息和容量数据，该存储服务适用于通用存储帐户和 Blob 存储帐户。
该数据存储在同一存储帐户中的已知表中。

如需更多详细信息，请参阅[关于存储分析指标](https://msdn.microsoft.com/zh-cn/library/azure/hh343258.aspx)和[存储分析指标表架构](https://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx)

> [AZURE.NOTE] Blob 存储帐户公开表服务终结点的目的只是为了存储和访问该帐户的度量值数据。

若要监视 Blob 存储服务的存储消耗情况，需启用容量度量值。
启用此功能后，将会每天为存储帐户的 Blob 服务记录容量数据，并将容量数据记录为表条目写入到同一存储帐户中的 $MetricsCapacityBlob 表。

若要监视 Blob 存储服务的数据访问模式，需在 API 级别启用每小时事务度量值。
启用此功能后，将会每小时聚合按 API 进行的事务，并将这些事务记录为表条目写入到同一存储帐户中的 $MetricsHourPrimaryTransactionsBlob 表。在使用 RA-GRS 存储帐户时，$MetricsHourSecondaryTransactionsBlob 表会将事务记录到辅助终结点。

> [AZURE.NOTE] 如果你有一个通用存储帐户，并且在其中存储了页 Blob、虚拟机磁盘以及块 Blob 数据和追加 Blob 数据，则不适合使用此估算过程。这是因为，你将无法根据 Blob 的类型来区分块 Blob 和追加 Blob（均可迁移到 Blob 存储帐户）的容量和事务度量值。

若要对数据使用和访问模式进行准确的估算，建议你为度量值选择一个可代表你日常使用情况的保留期，然后进行推断。
一种选择是让度量值数据保留 7 天，每周收集一次数据，然后在月底进行分析。
另一种选择是让度量值数据保留 30 天，在 30 天到期以后再收集和分析数据。

如需详细了解如何启用、收集和查看度量值数据，请参阅[启用 Azure 存储空间度量值并查看度量值数据](/documentation/articles/storage-enable-and-view-metrics/)。

> [AZURE.NOTE] 存储、访问和下载分析数据也会收费，就像使用常规用户数据一样。

#### 通过使用情况度量值来估算费用

##### 存储费用

容量度量值表 $MetricsCapacityBlob 中行键为“data”的最新条目显示了用户数据所占用的存储容量。
容量度量值表 $MetricsCapacityBlob 中行键为“analytics”的最新条目显示了分析日志所占用的存储容量。

用户数据和分析日志（如果已启用）所占用的这个总容量就可以用来估算在存储帐户中存储数据的费用。
也可以使用相同方法来估算在通用存储帐户中存储块 Blob 和追加 Blob 的存储费用。

##### 事务费用

事务度量值表中某个 API 的所有条目的“TotalBillableRequests”计得之和表示该特定 API 的事务总数。例如，给定期间的“GetBlob”事务的总数可以通过将行键为“user;GetBlob”的所有条目的可计费请求数求和来计算。

若要估算 Blob 存储帐户的事务费用，需将事务细分成三组，因为这些事务价格不一样。

- 写入事务，例如“PutBlob”、“PutBlock”、“PutBlockList”、“AppendBlock”、“ListBlobs”、“ListContainers”、“CreateContainer”、“SnapshotBlob”和“CopyBlob”。
- 删除事务，例如“DeleteBlob”和“DeleteContainer”。
- 所有其他事务。

若要估算通用存储帐户的事务费用，需聚合所有事务而不考虑操作/API。

##### 数据访问和异地复制数据传输费用

虽然存储分析不提供从存储帐户读取以及写入到存储帐户的数据量，但该数据量可以通过查看事务度量值表来大致进行估算。
事务度量值表中某个 API 的所有条目的 'TotalIngress' 计得之和表示该特定 API 的传入数据的总量。
与此类似，'TotalEgress' 计得之和表示传出数据的总量，以字节为单位。

若要估算 Blob 存储帐户的数据访问费用，需将事务细分成两组。

- 从存储帐户检索的数据量可以通过查看主要为 'GetBlob' 和 'CopyBlob' 操作的 'TotalEgress' 计得之和来估算。
- 写入到存储帐户的数据量可以通过查看主要为 'PutBlob'、'PutBlock'、'CopyBlob' 和 'AppendBlock' 操作的 'TotalIngress' 计得之和来估算。

在使用 GRS 或 RA-GRS 存储帐户时，也可以通过所写入数据量的估算值来计算 Blob 存储帐户的异地复制数据传输费用。

> [AZURE.NOTE] 如需更详细的示例来了解如何计算热存储层或冷存储层的使用费用，请参阅 [Azure 存储空间定价](/pricing/details/storage/)页中名为“什么是‘热’和‘冷’访问层以及如何确定应使用哪一个？”的常见问题。

### 迁移现有数据

Blob 存储帐户专用于仅存储块 blob 和追加 blob。现有的通用存储帐户（使你可以存储表、队列、文件和磁盘以及 Blob）无法转换为 Blob 存储帐户。若要使用存储层，将需要创建新的 Blob 存储帐户并将你现有的数据迁移到新创建的帐户。
可以使用以下方法从本地存储设备、第三方云存储提供程序或 Azure 中现有的通用存储帐户将现有数据迁移到 Blob 存储帐户：

#### AzCopy

AzCopy 是一个 Windows 命令行实用程序，旨在实现高性能地将数据复制到 Azure 存储空间和从 Azure 存储空间中复制。可以使用 AzCopy 从现有的通用存储帐户将数据复制到你的 Blob 存储帐户，或从你的本地存储设备将数据上传到 Blob 存储帐户。

有关更多详细信息，请参阅[使用 AzCopy 命令行实用工具传输数据](/documentation/articles/storage-use-azcopy/)。

#### 数据移动库

适用于 .NET 的 Azure 存储空间数据移动库基于为 AzCopy 提供技术支持的核心数据移动框架。库旨在实现类似于 AzCopy 的高性能、可靠且简单的数据传输操作。这样使你能够以本机模式利用应用程序中的 AzCopy 提供的功能的全部好处，而无需处理运行和监视 AzCopy 的外部实例。

有关更多详细信息，请参阅[适用于 .Net 的 Azure 存储空间数据移动库](https://github.com/Azure/azure-storage-net-data-movement)

#### REST API 或客户端库

你可以创建自定义应用程序以使用其中一个 Azure 客户端库或 Azure 存储空间服务 REST API 将数据迁移到 Blob 存储帐户。Azure 存储空间对多种语言和平台（如 .NET、Java、C++、Node.JS、PHP、Ruby 和 Python）提供了内容丰富的客户端库。客户端库提供高级功能，如重试逻辑、日志记录和并行上传。你也可以直接针对可以由发出 HTTP/HTTPS 请求的任何语言调用的 REST API 进行开发。

有关更多详细信息，请参阅[开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)。

> [AZURE.NOTE] 使用客户端加密进行加密的 blob 将存储与 blob 一起存储的加密相关元数据。任何复制机制应确保保留 blob 元数据，尤其是与加密相关的元数据，这一点非常重要。如果复制不包含此元数据的 blob，则 blob 内容将不再可进行检索。有关加密相关元数据的详细信息，请参阅 [Azure 存储空间客户端加密](/documentation/articles/storage-client-side-encryption/)。

## 常见问题

1. **现有存储帐户是否仍然可用？**

    是，现有存储帐户仍然可用，并且定价或功能不变。它们不能选择存储层，并且在将来也不会有分层功能。

2. **为什么以及何时应开始使用 Blob 存储帐户？**

    Blob 存储帐户专用于存储 blob，并且使我们能够引入新的以 blob 为中心的功能。今后，Blob 存储帐户将是用于存储 blob 的推荐方法，因为未来的功能（如分级存储和分层）将基于此帐户类型引入。但是，想要何时基于你的业务需求进行迁移，这由你决定。

3. **是否可以将现有存储帐户转换成 Blob 存储帐户？**

    不能。Blob 存储帐户是一种不同的存储帐户，你需要新建它并如上文所述迁移数据。

4. **是否可以将对象存储在同一帐户中的两个存储层中？**

    用于指示存储层的“访问层”属性在帐户级别设置，并且适用于该帐户中的所有对象。不能在对象级别设置访问层属性。

5. **是否可以更改我的 Blob 存储帐户的存储层？**

    是的。你可以通过设置存储帐户上的“访问层”属性来更改存储层。更改存储层将适用于存储在帐户中的所有对象。将存储层从“热”更改为“冷”将不会产生任何费用，但是从“冷”更改为“热”将产生每 GB 用于读取该帐户中所有数据的费用。

6. **我可以多频繁地更改我的 Blob 存储帐户的存储层？**

    虽然我们不会强制限制可以更改存储层的频率，但请注意，将存储层从“冷”更改为“热”将会产生大量费用。我们不建议频繁地更改存储层。

7. **冷存储层中的 Blob 的行为方式是否与热存储层中的不同？**

    热存储层中的 Blob 具有与通用存储帐户中的 Blob 相同的延迟。冷存储层中的 Blob 具有与通用存储帐户中的 Blob 类似的延迟（以毫秒为单位）。

    冷存储层中的 Blob 将具有的可用性服务级别 (SLA) 比存储在热存储层中的 Blob 略低。有关更多详细信息，请参阅[存储的 SLA](/support/legal/sla/storage/)。

8. **是否可以将页 blob 和虚拟机磁盘存储在 Blob 存储帐户中？**

    Blob 存储帐户仅支持块 blob 和追加 blob，不支持页 blob。Azure 虚拟机磁盘由页 blob 支持，因此不能使用 Blob 存储帐户来存储虚拟机磁盘。但是，可以在 Blob 存储帐户中将虚拟机磁盘的备份存储为块 blob。

9. **是否需要更改现有应用程序以使用 Blob 存储帐户？**

    Blob 存储帐户与用于存储块 blob 和追加 blob 的通用存储帐户 100% API 一致。只要你的应用程序使用的是块 Blob 或追加 Blob，并且使用[存储服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd894041.aspx) 的 2014-02-14 版或更高版本，则你的应用程序应该可以正常工作。如果使用协议的较旧版本，则将需要更新你的应用程序以使用新版本，以便与存储帐户的两种类型完美的配合工作。一般情况下，无论所使用的存储帐户类型，我们通常均建议使用最新版本。

10. **用户体验是否会有变化？**

    Blob 存储帐户非常类似于用于存储块 blob 和追加 blob 的通用存储帐户，并支持 Azure 存储空间的所有主要功能，包括高持续性和可用性、可伸缩性、性能和安全。除特定于 Blob 存储帐户及其以上已列出的存储层的功能和限制以外，其余全部内容将保持不变。

## 后续步骤

### 评估 Blob 存储帐户



[通过启用 Azure 存储空间度量值来评估当前存储帐户的使用情况](/documentation/articles/storage-enable-and-view-metrics/)

[按区域检查 Blob 存储定价](/pricing/details/storage/)

[检查数据传输定价](/pricing/details/data-transfer/)

### 开始使用 Blob 存储帐户

[开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)

[将数据移动到和移出 Azure 存储空间](/documentation/articles/storage-moving-data/)

[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)

[浏览和了解你的存储帐户](http://storageexplorer.com/)

<!---HONumber=Mooncake_0725_2016-->