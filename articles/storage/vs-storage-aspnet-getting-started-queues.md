<properties
    pageTitle="开始使用 Azure 队列存储和 Visual Studio 连接服务 (ASP.NET) | Azure"
    description="在使用 Visual Studio 连接服务连接到存储帐户后，如何开始在 Visual Studio 的 ASP.NET 项目中使用 Azure 队列存储"
    services="storage"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />
<tags
    ms.assetid="94ca3413-5497-433f-abbe-836f83a9de72"
    ms.service="storage"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/02/2016"
    wacn.date="01/06/2017"
    ms.author="tarcher" />  


# 开始使用 Azure 队列存储和 Visual Studio 连接服务 (ASP.NET)

## 概述

Azure 队列存储是一项存储大量非结构化数据的服务，用户可通过 HTTP 或 HTTPS 访问这些数据。一条队列消息的大小最多可为 64 KB，一个队列可以包含无数条消息，直至达到存储帐户的总容量限值。

本文介绍如何以编程方式管理 Azure 队列存储实体，执行常见任务，例如创建 Azure 队列，以及添加、修改、读取和删除队列消息。

> [AZURE.NOTE]
> 
> 本文的代码部分假定用户已使用连接服务连接到 Azure 存储帐户。连接服务的配置方法是：打开 Visual Studio 解决方案资源管理器，右键单击项目，然后从上下文菜单中选择“添加”->“连接服务”选项。在该处按照对话框的说明连接到所需的 Azure 存储帐户。

## 创建队列

以下步骤演示了如何以编程方式创建队列。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示所需队列名称引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要创建的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 如果队列不存在，则调用 **CloudQueue.CreateIfNotExists** 方法来创建队列。

	    queue.CreateIfNotExists();


## 向队列添加消息

以下步骤演示了如何以编程方式向队列添加消息。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示所需队列名称引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要向其添加消息的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 创建表示要添加到队列的消息的 **CloudQueueMessage** 对象。可从字符串（UTF-8 格式）或字节数组创建 **CloudQueueMessage** 对象。（将 *<queue-message>* 更改为要添加的消息。）

		CloudQueueMessage message = new CloudQueueMessage(<queue-message>);

6. 调用向队列添加消息的 **CloudQueue.AddMessage** 方法。

    	queue.AddMessage(message);

## 从队列中读取一条消息，不删除它

以下步骤演示了如何以编程方式查看排队的消息（读取第一条消息，不删除它）。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示队列引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要从其读取消息的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 调用 **CloudQueue.PeekMessage** 方法读取队列前面的消息，不从队列中将其删除。

    	CloudQueueMessage message = queue.PeekMessage();

6. 通过 **CloudQueueMessage.AsBytes** 或 **CloudQueueMessage.AsString** 属性访问 **CloudQueueMessage** 对象的值。

		string messageAsString = message.AsString;
		byte[] messageAsBytes = message.AsBytes;

## 读取和删除队列中的消息

以下步骤演示了如何以编程方式读取排队的消息，然后将其删除。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示队列引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要从其读取消息的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 调用 **CloudQueue.GetMessage** 方法读取队列中的第一条消息。**CloudQueue.GetMessage** 方法可以让消息对任何其他读取消息的代码不可见 30 秒（默认），因此当用户正在处理消息时，其他代码无法修改或删除该消息。若要更改消息不可见的时间，请修改传递给 **CloudQueue.GetMessage** 方法的 **visibilityTimeout** 参数。

		// This message will be invisible to other code for 30 seconds.
		CloudQueueMessage message = queue.GetMessage();     

6. 调用从队列中删除消息的 **CloudQueueMessage.Delete** 方法。

	    queue.DeleteMessage(message);

## 获取队列长度

以下步骤演示了如何以编程方式获取队列长度（消息数）。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示队列引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要查询其长度的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 调用检索队列的属性（包括其长度）的 **CloudQueue.FetchAttributes** 方法。

		queue.FetchAttributes();

6. 访问 **CloudQueue.ApproximateMessageCount** 属性以获取队列的长度。
 
		int? nMessages = queue.ApproximateMessageCount;

## 删除队列
以下步骤演示了如何以编程方式删除队列。

1. 添加以下 *using* 指令：
   
        using Microsoft.Azure;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示队列服务客户端的 **CloudQueueClient** 对象。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. 获取表示队列引用的 **CloudQueue** 对象。（将 *<queue-name>* 更改为要查询其长度的队列的名称。）

        CloudQueue queue = queueClient.GetQueueReference(<queue-name>);

5. 调用 **CloudQueue.Delete** 方法，删除 **CloudQueue** 对象代表的队列。

	    messageQueue.Delete();

## 后续步骤
[AZURE.INCLUDE [vs-storage-dotnet-queues-next-steps](../../includes/vs-storage-dotnet-queues-next-steps.md)]

<!---HONumber=Mooncake_0103_2017-->