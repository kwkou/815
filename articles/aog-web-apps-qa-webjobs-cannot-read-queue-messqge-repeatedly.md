<properties
	pageTitle="使用 Web Jobs SDK 无法多次读取 Azure 队列中的消息"
	description="使用 Web Jobs SDK 无法多次读取 Azure 队列中的消息"
	service=""
	resource="webapps"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, Web Jobs, Queue, SDK"​
    cloudEnvironments="MoonCake" />
<tags
	ms.service="app-service-web-aog"
	ms.date=""
	wacn.date="1/20/2017" />
# 使用 Web Jobs SDK 无法多次读取 Azure 队列中的消息

## **问题现象**

使用 Web Jobs SDK 成功读取 Azure 队列中的消息后，这个消息会被自动删除；如果没有读取成功，则还可以继续读取。
这个是 Web Jobs SDK 默认行为，具体请参考：[Dont delete from queue until success](https://github.com/Azure/azure-webjobs-sdk/issues/519) 

## **解决方法**

通过以下两种方法可以解决该问题：

1.	不通过 Web Jobs SDK，而是通过原始的方法来读取 Azure 队列中的消息：[通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/) 
2.	下载 Github 的 SDK 源码，本地修改后在进行使用 : [Azure/azure-webjobs-sdk/Queues/Listeners/QueueListener ](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Queues/Listeners/QueueListener.cs#L247)
