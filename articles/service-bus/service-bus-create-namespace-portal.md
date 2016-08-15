<properties
   pageTitle="使用 Azure 门户创建服务总线命名空间 | Azure"
   description="为开始使用服务总线，你将需要一个命名空间。下面介绍了如何使用 Azure 门户创建一个。"
   services="service-bus"
   documentationCenter=".net"
   authors="jtaubensee"
   manager="timlt"
   editor="sethmanheim"/>

<tags
   ms.service="service-bus"
   ms.date="06/07/2016"
   wacn.date="07/25/2016"/>

#使用 Azure 门户创建服务总线命名空间
命名空间是一个适用于所有消息传送组件的公用容器。多个队列和主题可位于单个命名空间，且命名空间通常用作应用程序容器。目前有 2 种创建服务总线命名空间的方式。

1.	Azure 门户（这篇文章）

2.	[ARM 模板][create-namespace-using-arm]

##在 Azure 门户中创建命名空间
[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

祝贺你！ 现已创建一个服务总线命名空间。

##后续步骤

查看我们的 GitHub 存储库，其中包含展示了某些更高级的 Azure 服务总线消息传递功能的示例。
[https://github.com/Azure-Samples/azure-servicebus-messaging-samples][github-samples]

[create-namespace-using-arm]: /documentation/articles/service-bus-resource-manager-overview/
[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples
<!---HONumber=Mooncake_0718_2016-->