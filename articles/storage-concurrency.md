<properties 
	pageTitle="在 Azure 存储中管理并发" 
	description="如何管理 Blob、队列、表和文件服务的并发" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="02/20/2016"
	wacn.date="04/11/2016"/>



#在 Azure 存储中管理并发

## 概述 

通常情况下，基于 Internet 的新型应用程序允许多名用户同时查看和更新数据。这就要求应用程序开发人员仔细考虑如何为他们的最终用户提供可预测的体验，尤其是在多名用户可以更新相同数据的情况下。开发人员通常将考虑下面三个主要数据并发策略：


1.	乐观并发 – 执行更新的应用程序在更新过程中要验证数据是否自该应用程序上次读取该数据起已发生更改。例如，如果两名查看 wiki 页的用户对同一页进行更新，则 wiki 平台必须确保第二次更新不会覆盖第一次更新，并且两名用户都了解他们的更新成功与否。此策略最常用于 Web 应用程序中。
2.	悲观并发 – 要执行更新的应用程序将对对象获取一个锁，以防其他用户在该锁释放前更新数据。例如，在主/从数据复制情况下，如果只有主对象要执行更新，则该对象通常将在长时间内对数据持有一个独占锁，以确保其他任何用户都不能更新该数据。
3.	上次编写者赢 – 一种方法，即允许任何更新操作继续进行，而不需要验证其他任何应用程序是否自应用程序首次读取数据起已更新该数据。此策略（或缺乏正式策略）通常用于以下情况：以多名用户不可能访问相同数据的方式对数据进行分区。该策略可能还适用于正在处理短期数据流的情况。  

本文概述 Azure 存储平台如何通过为所有这三个并发策略提供一类支持来简化开发。  

## Azure 存储 – 简化云开发
Azure 存储服务支持所有三个策略，尽管它具有独特的功能，可以为乐观并发和悲观并发提供完全支持，因为该服务旨在采用强大的一致性模型，该模型用于保证，当存储服务提交数据插入或更新操作时，所有对该数据的进一步访问都将显示最新更新。存储平台如果使用最终一致性模型，则在一名用户执行写入与其他用户可以查看已更新的数据之间存在着时滞，因而会使客户端应用程序开发复杂化，以防不一致性影响最终用户。

除了选择相应的并发策略，开发人员还应了解存储平台如何隔离更改，尤其是对跨事务的相同对象的更改。Azure 存储服务使用快照隔离允许在单个分区中并发执行读取操作与写入操作。与其他隔离级别不同，快照隔离保证，所有读取操作看到的数据快照是一致的，即使在执行更新时也是如此。事实上，这是通过在处理更新事务时返回上次提交的值实现的。  

## 在 Blob 存储中管理并发
你可以选择使用乐观并发模型或悲观并发模型，来管理对 Blob 服务中的 Blob 和容器的访问。如果你没有明确指定策略，则上次编写者赢是默认策略。

### Blob 和容器的乐观并发  

存储服务会为每个存储对象分配一个标识符。只要对对象执行更新操作，就会更新此标识符。在使用 HTTP 协议中定义的 ETag（实体标记）标头的情况下，该标识符作为 HTTP GET 响应的一部分返回到客户端。对此类对象执行更新的用户，可以连同条件标头一起在原始 ETag 中发送，以确保仅当已满足特定条件时才会进行更新 – 在这种情况下，该条件是“If-Match”标头，该标头要求存储服务确保在更新请求中指定的 ETag 值与存储服务中存储的值相同。

此进程的概述如下：  

1.	从存储服务检索 Blob，响应包括用于在存储服务中标识对象当前版本的 HTTP ETag 标头值。
2.	在更新 Blob 时，应将您在步骤 1 中获得的 ETag 值包括在发送到服务的请求的 **If-Match** 条件标头中。
3.	该服务会将请求中的 ETag 值与 Blob 的当前 ETag 值进行比较。
4.	如果 Blob 的当前 ETag 值与请求的 **If-Match** 条件标头中的 ETag 的版本不同，则该服务将 412 错误返回到客户端。这向客户端表明，其他进程自客户端检索到 Blob 起已更新该 Blob。
5.	如果 Blob 的当前 ETag 值与请求的 **If-Match** 条件标头中的 ETag 的版本相同，则该服务将执行请求的操作，并更新该 Blob 的当前 ETag 值，以说明它已创建新版本。  

以下 C# 代码段（使用客户端存储库 4.2.0）显示一个简单示例，说明如何根据从以前检索到或插入的 Blob 访问的 ETag 值构造 **If-Match AccessCondition**。然后，在更新该 Blob 时使用 **AccessCondition** 对象：**AccessCondition** 对象将 **If-Match** 标头添加到请求中。如果其他进程已更新该 Blob，则 Blob 服务将返回 HTTP 412 (不满足前提条件) 状态消息。可以在这里下载完整的示例：[使用 Azure 存储空间管理并发](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。

	// Retrieve the ETag from the newly created blob
	// Etag is already populated as UploadText should cause a PUT Blob call 
	// to storage blob service which returns the etag in response.
	string orignalETag = blockBlob.Properties.ETag;
	 
	// This code simulates an update by a third party.
	string helloText = "Blob updated by a third party.";
	 
	// No etag, provided so orignal blob is overwritten (thus generating a new etag)
	blockBlob.UploadText(helloText);
	Console.WriteLine("Blob updated. Updated ETag = {0}", 
	blockBlob.Properties.ETag);
	 
	// Now try to update the blob using the orignal ETag provided when the blob was created
	try
	{
	    Console.WriteLine("Trying to update blob using orignal etag to generate if-match access condition");
	    blockBlob.UploadText(helloText,accessCondition:
	    AccessCondition.GenerateIfMatchCondition(orignalETag));
	}
	catch (StorageException ex)
	{
	    if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
	    {
	        Console.WriteLine("Precondition failure as expected. Blob's orignal etag no longer matches");
	        // TODO: client can decide on how it wants to handle the 3rd party updated content.
	    }
	    else
	        throw;
	}  

存储服务还包括支持其他条件标头，例如 **If-Modified-Since**、**If-Unmodified-Since** 和 **If-None-Match**，以及这些标头的组合。有关详细信息，请参阅 MSDN 上的[为 Blob 服务操作指定条件标头](http://msdn.microsoft.com/zh-cn/library/dd179371.aspx)。  

下表概述接受条件标头（例如请求中的 **If-Match**）的容器操作，以及在响应中返回 ETag 值的容器操作。  

操作 |返回容器的 ETag 值|	接受条件标头|
------------|-----------------------|------------------------------------|
创建容器|	是|	否|
获取容器属性|	是|	否|
获取容器元数据|	是|	否|
设置容器元数据|	是|	是|
获取容器 ACL|	是|	否|
设置容器 ACL|	是|	是 (*)|
删除容器|	否|	是|
租赁容器|	是|	是|
列出 Blob|	否|	否  

(*) 将缓存由 SetContainerACL 定义的权限，传播这些权限的更新需要 30 秒的时间，在此期间，不保证更新是一致的。  

下表概述接受条件标头（例如请求中的 **If-Match**）的 Blob 操作，以及在响应中返回 ETag 值的 Blob 操作。  

操作	|返回 ETag 值	|接受条件标头|
-----------|-------------------|----------------------------|
放置 Blob|	是|	是|
获取 Blob|	是|	是|
获取 Blob 属性|	是|	是|
设置 Blob 属性|	是|	是|
获取 Blob 元数据|	是|	是|
设置 Blob 元数据|	是|	是|
租赁 Blob (*)| 是| 是|
制作 Blob 快照| 是| 是|
复制 Blob| 是| 是（用于源 Blob 和目标 Blob）|
中止复制 Blob| 否| 否|
删除 Blob| 否| 是|
放置块| 否| 否|
放置块列表| 是| 是|
获取块列表| 是| 否|
放置页面| 是| 是|
获取页面范围| 是| 是

(*) 租赁 Blob 不更改 Blob 中的 ETag。  

### Blob 的悲观并发
若要锁定 Blob 以供独占使用，您可以对该 Blob 获得[租约](http://msdn.microsoft.com/zh-cn/library/azure/ee691972.aspx)。在获取租约时，可以指定需要该租约的时长：该值可以介于 15 到 60 秒之间，也可以是无限期（相当于独占锁）。你可以续订有限期租约来延展该租约，也可以在租约完成后将其释放。Blob 服务在有限期租约到期时会自动释放这些租约。

租约允许支持各种同步策略，包括独占写入/共享读取、独占写入/独占读取和共享写入/独占读取。如果租约存在，则存储服务会强制执行独占写入（放置、设置和删除操作），但是，若要确保读取操作的独占性，开发人员需要确保所有客户端应用程序都使用一个租约 ID，并且一次只有一个客户端具有有效的租约 ID。不包括租约 ID 的读取操作会导致共享读取。  

以下 C# 代码段显示一个示例，说明如何在 30 秒内对 Blob 获取独占租约，更新 Blob 的内容，然后释放该租约。当你尝试获取新租约时，如果 Blob 中已经存在有效租约，则 Blob 服务将返回“HTTP (409) 冲突”状态结果。在发出请求以在存储服务中更新 Blob 时，下面的代码段使用 **AccessCondition** 对象封装租约信息。可以在这里下载完整的示例：[使用 Azure 存储空间管理并发](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。

	// Acquire lease for 15 seconds
	string lease = blockBlob.AcquireLease(TimeSpan.FromSeconds(15), null);
	Console.WriteLine("Blob lease acquired. Lease = {0}", lease);
	 
	// Update blob using lease. This operation will succeed
	const string helloText = "Blob updated";
	var accessCondition = AccessCondition.GenerateLeaseCondition(lease);
	blockBlob.UploadText(helloText, accessCondition: accessCondition);
	Console.WriteLine("Blob updated using an exclusive lease");
	 
	//Simulate third party update to blob without lease
	try
	{
	    // Below operation will fail as no valid lease provided
	    Console.WriteLine("Trying to update blob without valid lease");
	    blockBlob.UploadText("Update without lease, will fail");
	}
	catch (StorageException ex)
	{
	    if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
	        Console.WriteLine("Precondition failure as expected. Blob's lease does not match");
	    else
	        throw;
	}  

如果你尝试对租赁的 Blob 执行写入操作，而不传递租约 ID，则请求将失败，显示 412 错误。请注意，如果该租约在调用 **UploadText** 方法前到期，但您仍传递租约 ID，则请求也将失败，显示 **412** 错误。有关如何管理租约到期时间和租约 ID 的详细信息，请参阅[租赁 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691972.aspx) REST 说明文档。

以下 Blob 操作可以使用租约来管理悲观并发：  


-   放置 Blob
-	获取 Blob
-	获取 Blob 属性
-	设置 Blob 属性
-	获取 Blob 元数据
-	设置 Blob 元数据
-	删除 Blob
-	放置块
-	放置块列表
-	获取阻止列表
-	放置页面
-	获取页面范围
-	制作 Blob 快照 - 可选租约 ID（如果租约存在）
-	复制 Blob - 如果目标 Blob 中存在租约，则需要提供租约 ID
-	中止复制 Blob - 如果目标 Blob 中存在无限期租约，则需要提供租约 ID
-	租赁 Blob  

### 容器的悲观并发
同 Blob 中的租约一样，容器中的租约可以实现对同步策略（独占写入/共享读取、独占写入/独占读取和共享写入/独占读取）的支持，但是，与 Blob 不同，存储服务仅对删除操作强制实施独占性。若要删除具有活动租约的容器，客户端必须将活动租约 ID 包括在删除请求中。在不包括租约 ID 的情况下，对租赁容器的其他所有容器操作都将成功，在这种情况下，这些操作是共享操作。如果需要更新（放置或设置）或读取操作的独占性，则开发人员应确保所有客户端都使用一个租约 ID，而且一次只有一个客户端具有有效的租约 ID。

以下容器操作可以使用租约来管理悲观并发：  

-	删除容器
-	获取容器属性
-	获取容器元数据
-	设置容器元数据
-	获取容器 ACL
-	设置容器 ACL
-	租赁容器  

有关详细信息，请参阅：  

- [为 Blob 服务操作指定条件标头](http://msdn.microsoft.com/zh-cn/library/azure/dd179371.aspx)
- [租赁容器](http://msdn.microsoft.com/zh-cn/library/azure/jj159103.aspx)
- [租赁 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691972.aspx) 

## 在表服务中管理并发
在处理实体时，表服务使用乐观并发检查作为默认行为，而 Blob 服务不同，你必须明确选择执行乐观并发检查。表服务与 Blob 服务之间的另一个区别在于，使用表服务，你只能管理实体的并发行为，而使用 Blob 服务，你既可以管理容器的并发，又可以管理 Blob 的并发。

若要使用乐观并发，并检查其他进程是否自你从表存储服务检索到实体起修改了该实体，你可以使用在表服务返回实体时获得的 ETag 值。此进程的概述如下：  

1.	从表存储服务检索实体，响应包括用于标识与存储服务中的该实体关联的当前标识符的 ETag 值。
2.	在更新实体时，应将您在步骤 1 中获得的 ETag 值包括在发送到服务的请求的必需 **If-Match** 标头中。
3.	该服务会将请求中的 ETag 值与实体的当前 ETag 值进行比较。
4.	如果实体的当前 ETag 值与请求的必需 **If-Match** 标头中的 Etag 不同，则该服务将 412 错误返回到客户端。这向客户端表明，其他进程自该客户端检索到实体起已更新该实体。
5.	如果实体的当前 ETag 值与请求中的必需 **If-Match** 标头内的 ETag 相同，或者 **If-Match** 标头包含通配符 (*)，则服务执行请求的操作并更新实体的当前 ETag 值以指出已更新该值。

请注意，与 Blob 服务不同，表服务要求客户端将 **If-Match** 标头包括在更新请求中。但是，如果客户端在请求中将 **If-Match** 标头设置为通配符 (*)，则可以强制执行非条件更新（“以最后写入者为准”策略）并绕过并发检查。

以下 C# 代码段显示以前创建或检索到的客户实体是如何更新其电子邮件地址的。初始插入或检索操作将 ETag 值存储在客户对象中，因为示例在执行替换操作时使用相同的对象实例，所以将 ETag 值自动发送回表服务，从而使该服务可以检查是否存在并发违规情况。如果其他进程已更新表存储中的实体，则该服务将返回 HTTP 412 (不满足前提条件) 状态消息。可以在这里下载完整的示例：[使用 Azure 存储空间管理并发](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。

	try
	{
	    customer.Email = "updatedEmail@contoso.org";
	    TableOperation replaceCustomer = TableOperation.Replace(customer);
	    customerTable.Execute(replaceCustomer);
	    Console.WriteLine("Replace operation succeeded.");
	}
	catch (StorageException ex)
	{
	    if (ex.RequestInformation.HttpStatusCode == 412)
	        Console.WriteLine("Optimistic concurrency violation – entity has changed since it was retrieved.");
	    else
	        throw; 
	}  

若要明确禁用并发检查，您应该在执行替换操作前，将 **employee** 对象的 **ETag** 属性设置为“*”。

customer.ETag = "*"；

下表概述表实体操作是如何使用 ETag 值的：  

操作	|返回 ETag 值	|需要 If-Match 请求标头|
------------|-------------------|--------------------------------|
查询实体|	是|	否|
插入实体|	是|	否|
更新实体|	是|	是|
合并实体|	是|	是|
删除实体|	否|	是|
插入或替换实体|	是|	否|
插入或合并实体|	是|	否 

请注意，**插入或替换实体**和**插入或合并实体**操作不可以执行任何并发检查，因为这些操作不会将 ETag 值发送到表服务。

一般情况下，使用表的开发人员应依靠乐观并发来开发可伸缩的应用程序。如果需要悲观锁定，则开发人员在访问表时可采用的一种方法是，为每个表分配指定的 Blob，并在对该表执行操作前获取 Blob 中的租约。这种方法要求应用程序确保在对表执行操作前，所有数据访问路径都获得租约。你还应注意，最短租赁时间为 15 秒，这要求你慎重考虑可伸缩性。  

有关详细信息，请参阅：  

- [对实体的操作](http://msdn.microsoft.com/zh-cn/library/azure/dd179375.aspx)  

## 在队列服务中管理并发
在队列服务中考虑并发的一种情况是，多个客户端正从一个队列检索消息。在从队列检索消息时，响应包括消息和 pop 接收方值，这是删除该消息所必需的。该消息不会从队列中自动删除，但是在检索到后，该消息在 visibilitytimeout 参数指定的时间间隔内不显示给其他客户端。检索消息的客户端应在消息处理后、响应的 TimeNextVisible 元素指定的时间前删除该消息，该时间是根据 visibilitytimeout 参数的值计算的。若要确定 TimeNextVisible 的值，可将 visibilitytimeout 的值添加到消息的检索时间。

队列服务不支持乐观并发或悲观并发，因此，处理从队列检索到的消息的客户端应确保以幂等方式处理消息。上次编写者赢策略用于更新操作，例如 SetQueueServiceProperties、SetQueueMetaData、SetQueueACL 和 UpdateMessage。  

有关详细信息，请参阅：  

- [队列服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179363.aspx)
- [获取消息](http://msdn.microsoft.com/zh-cn/library/azure/dd179474.aspx)  

## 在文件服务中管理并发
文件服务可以使用下面两个不同的协议终结点访问：SMB 和 REST。REST 服务不支持乐观锁定或悲观锁定，并且所有更新将遵循上次编写者赢策略。SMB 客户端如果装载文件共享，则可以充分利用文件系统锁定机制来管理对共享文件的访问，包括可以执行悲观锁定。在打开文件后，SMB 客户端会同时指定文件访问权限和共享模式。如果将“文件访问权限”选项设置为“读取”或“读取/写入”，同时将“文件共享”模式设置为“无”，则将导致文件在关闭前被 SMB 客户端锁定。如果尝试对 SMB 客户端已锁定的文件执行 REST 操作，则 REST 服务将返回状态代码 409 (冲突)，以及错误代码 SharingViolation。

当 SMB 客户端打开文件以进行删除时，会将该文件标记为待删除，直到该文件上的其他所有 SMB 客户端打开句柄关闭为止。当文件标记为待删除时，对该文件执行的任何 REST 操作将返回状态代码 409 (冲突)，以及错误代码 SMBDeletePending。不返回状态代码 404 (找不到)，因为 SMB 客户端在关闭文件前，可能无法删除待删除标志。换而言之，仅当删除文件后，才应返回状态代码 404 (找不到)。请注意，当文件处于 SMB 待删除状态时，“列出文件”结果中将不包含该文件。另请注意，“REST 删除文件”和“REST 删除目录”操作以原子方式提交，并且不会导致待删除状态。

有关详细信息，请参阅：  

- [管理文件锁](http://msdn.microsoft.com/zh-cn/library/azure/dn194265.aspx)  

## 摘要和后续步骤
经过精心设计，Azure 存储服务可以满足最复杂的在线应用程序需要，而不会迫使开发人员对主要设计假设做出妥协或重新思考，例如他们视为理所当然的并发和数据一致性。

对于本博客中引用的完整示例应用程序：  

- [使用 Azure 存储管理并发 - 示例应用程序](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)  

有关 Azure 存储的详细信息，请参阅：  

- [Azure 存储主页](/home/features/storage)
- [Azure 存储简介](/documentation/articles/storage-introduction/)
- [Blob](/documentation/articles/storage-dotnet-how-to-use-blobs/)、[表](/documentation/articles/storage-dotnet-how-to-use-tables/)和[队列](/documentation/articles/storage-dotnet-how-to-use-queues/)的存储使用入门
- 存储体系结构 - [Azure 存储：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

 
<!---HONumber=Mooncake_0405_2016-->