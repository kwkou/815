<properties
    pageTitle="如何使用队列存储 (C++) | Azure"
    description="了解如何在 Azure 中使用队列存储服务。 示例用 C++ 编写。"
    services="storage"
    documentationcenter=".net"
    author="seguler"
    manager="jahogg"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="c8a36365-29f6-404d-8fd1-858a7f33b50a"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/28/2017"
    wacn.date="04/24/2017"
    ms.author="seguler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="c40d807ab64d70559b7dcef68f033aec7604caa2"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-use-queue-storage-from-c"></a>如何通过 C++ 使用队列存储
[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]


## <a name="overview"></a>概述
本指南将演示如何使用 Azure 队列存储服务执行常见方案。 示例采用 C++ 编写，并使用了[适用于 C++ 的 Azure 存储客户端库](http://github.com/Azure/azure-storage-cpp/blob/master/README.md)。 介绍的方案包括**插入**、**扫视**、**获取**和**删除**队列消息以及**创建和删除队列**。

> [AZURE.NOTE]
> 本指南主要面向适用于 C++ 的 Azure 存储空间客户端库 1.0.0 版及更高版本。 推荐版本：存储客户端库 2.2.0（可通过 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](http://github.com/Azure/azure-storage-cpp/) 获得）。
> 
> 

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>创建 C++ 应用程序
在本指南中，你将使用存储功能，这些功能可以在 C++ 应用程序中运行。

为此，需要安装适用于 C++ 的 Azure 存储客户端库，并在 Azure 订阅中创建 Azure 存储帐户。

若要安装适用于 C++ 的 Azure 存储客户端库，可使用以下方法：

* **Linux：**按照[适用于 C++ 的 Azure 存储客户端库自述文件](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)页中提供的说明操作。
* **Windows：**在 Visual Studio 中，单击“工具”>“NuGet 包管理器”>“程序包管理器控制台”。 在 [NuGet 包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)中，键入以下命令，然后按 **ENTER**。

		Install-Package wastorage

## <a name="configure-your-application-to-access-queue-storage"></a>配置应用程序以访问队列存储
将以下 include 语句添加到要在其中使用 Azure 存储 API 来访问队列 C++ 文件的顶部：  

    #include <was/storage_account.h>
    #include <was/queue.h>

## <a name="set-up-an-azure-storage-connection-string"></a>设置 Azure 存储连接字符串
Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并对 *AccountName* 和 *AccountKey* 值使用 [Azure 门户预览](https://portal.azure.cn)中列出的存储帐户的名称和存储帐户的存储访问密钥。有关存储帐户和访问密钥的信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。此示例演示如何声明一个静态字段以保存连接字符串：

	// Define the connection-string with your values.
	const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key;EndpointSuffix=core.chinacloudapi.cn"));

若要在本地 Windows 计算机中测试应用程序，可以使用随 [Azure SDK](/downloads/) 一起安装的 Azure [存储模拟器](/documentation/articles/storage-use-emulator/)。 存储模拟器是一种用于模拟本地开发计算机上 Azure 中可用的 Blob、队列和表服务的实用程序。 以下示例演示如何声明一个静态字段以将连接字符串保存到你的本地存储模拟器：  

	// Define the connection-string with Azure Storage Emulator.
	const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  

若要启动 Azure 存储模拟器，请选择“开始”按钮或按 **Windows** 键。开始键入“Azure 存储模拟器”，然后从应用程序列表中选择“Azure 存储模拟器”。

下面的示例假定使用了这两个方法之一来获取存储连接字符串。

## <a name="retrieve-your-connection-string"></a>检索你的连接字符串
可使用 **cloud_storage_account** 类来表示存储帐户信息。 若要从存储连接字符串中检索存储帐户信息，可以使用 **parse** 方法。

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

## <a name="how-to-create-a-queue"></a>如何：创建队列
**cloud_queue_client** 对象可用于获取队列的引用对象。 以下代码将创建 **cloud_queue_client** 对象。

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create a queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

使用 **cloud_queue_client** 对象可获取对要使用的队列的引用。 如果队列不存在，你可以创建它。

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Create the queue if it doesn't already exist.
 	queue.create_if_not_exists();  

## <a name="how-to-insert-a-message-into-a-queue"></a>如何：在队列中插入消息
若要将消息插入到现有队列中，请先创建新的 **cloud_queue_message**。 接下来，调用 **add_message** 方法。 可以从字符串或**字节**数组创建 **cloud_queue_message**。 以下代码将创建队列（如果该队列不存在）并插入消息“Hello, World”：

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Create the queue if it doesn't already exist.
	queue.create_if_not_exists();

	// Create a message and add it to the queue.
	azure::storage::cloud_queue_message message1(U("Hello, World"));
	queue.add_message(message1);  

## <a name="how-to-peek-at-the-next-message"></a>如何：扫视下一条消息
通过调用 **peek_message** 方法，可以扫视队列最前面的消息，而不必从该队列中将其删除。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Peek at the next message.
	azure::storage::cloud_queue_message peeked_message = queue.peek_message();

	// Output the message content.
	std::wcout << U("Peeked message content: ") << peeked_message.content_as_string() << std::endl;

## <a name="how-to-change-the-contents-of-a-queued-message"></a>如何：更改已排队消息的内容
可更改队列中现有消息的内容。如果消息表示工作任务，可使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常保留重试计数，当消息重试次数超过 n 时再删除此该消息。这可避免每次处理某条消息时都触发应用程序错误。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_conection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Get the message from the queue and update the message contents.
	// The visibility timeout "0" means make it visible immediately.
	// The visibility timeout "60" means the client can get another minute to continue
	// working on the message.
	azure::storage::cloud_queue_message changed_message = queue.get_message();

	changed_message.set_content(U("Changed message"));
	queue.update_message(changed_message, std::chrono::seconds(60), true);

	// Output the message content.
	std::wcout << U("Changed message content: ") << changed_message.content_as_string() << std::endl;  

## <a name="how-to-de-queue-the-next-message"></a>如何：取消下一条消息的排队
你的代码通过两个步骤来取消对队列中某条消息的排队。 调用 **get_message** 时，你将获取队列中的下一条消息。 从 **get_message** 返回的消息对从此队列读取消息的其他任何代码不可见。 若要完成从队列中删除消息，还必须调用 **delete_message**。 此删除消息的两步过程可确保，如果你的代码因硬件或软件故障而无法处理消息，则你的代码的其他实例可以获取相同消息并重试。 代码在处理消息后会立即调用 **delete_message**。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Get the next message.
	azure::storage::cloud_queue_message dequeued_message = queue.get_message();
	std::wcout << U("Dequeued message: ") << dequeued_message.content_as_string() << std::endl;

	// Delete the message.
	queue.delete_message(dequeued_message);

## <a name="how-to-leverage-additional-options-for-de-queuing-messages"></a>如何：使用其他方法取消对消息的排队
可通过两种方式自定义队列中消息的检索。 首先，可获取一批消息（最多 32 条）。 其次，你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少时间来完全处理每个消息。 以下代码示例使用 **get_messages** 方法来在一次调用中获取 20 条消息。 然后，它会使用 **for** 循环处理每条消息。 它还将每条消息的不可见超时时间设置为 5 分钟。 请注意，将对所有消息同时启动 5 分钟的超时设置，因此调用 **get_messages** 的 5 分钟后，任何尚未删除的消息都将再次可见。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Dequeue some queue messages (maximum 32 at a time) and set their visibility timeout to
	// 5 minutes (300 seconds).
	azure::storage::queue_request_options options;
	azure::storage::operation_context context;

	// Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
	std::vector<azure::storage::cloud_queue_message> messages = queue.get_messages(20, std::chrono::seconds(300), options, context);

	for (auto it = messages.cbegin(); it != messages.cend(); ++it)
	{
		// Display the contents of the message.
		std::wcout << U("Get: ") << it->content_as_string() << std::endl;
	}

## <a name="how-to-get-the-queue-length"></a>如何：获取队列长度
可以获取队列中消息的估计数。 **download_attributes** 方法允许队列服务检索队列属性，包括消息计数。 **approximate_message_count** 方法可获取队列中的大致消息数。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// Fetch the queue attributes.
	queue.download_attributes();

	// Retrieve the cached approximate message count.
	int cachedMessageCount = queue.approximate_message_count();

	// Display number of messages.
	std::wcout << U("Number of messages in queue: ") << cachedMessageCount << std::endl;  

## <a name="how-to-delete-a-queue"></a>如何：删除队列
若要删除队列及其包含的所有消息，请对队列对象调用 **delete_queue_if_exists**。

	// Retrieve storage account from connection-string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the queue client.
	azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

	// Retrieve a reference to a queue.
	azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

	// If the queue exists and delete it.
	queue.delete_queue_if_exists();  

## <a name="next-steps"></a>后续步骤
已了解队列存储的基本知识，可通过以下链接了解有关 Azure 存储的详细信息。

- [如何通过 C++ 使用 Blob 存储](/documentation/articles/storage-c-plus-plus-how-to-use-blobs/)
- [如何通过 C++ 使用表存储](/documentation/articles/storage-c-plus-plus-how-to-use-tables/)
- [使用 C++ 列出 Azure 存储资源](/documentation/articles/storage-c-plus-plus-enumeration/)
- [适用于 C++ 的存储空间客户端库参考](http://azure.github.io/azure-storage-cpp)
- [Azure 存储文档](/documentation/services/storage/)
<!--Update_Description:wording update; add anchors to H2 titles-->
