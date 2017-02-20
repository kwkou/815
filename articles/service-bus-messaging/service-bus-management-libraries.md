<properties
    pageTitle="Azure 服务总线管理库 | Azure"
      description="通过 .NET 管理服务总线命名空间和实体"
      services="service-bus-messaging"
      cloud="na"
      documentationcenter="na"
      author="jtaubensee"
      manager="sethm" />
<tags
    ms.assetid="ms.service: service-bus-messaging"
      ms.workload="na"
      ms.tgt_pltfrm="na"
      ms.devlang="dotnet"
      ms.topic="article"
      ms.date="01/06/2016"
      wacn.date="02/20/2017"
      ms.author="jotaub" />  


# 服务总线管理库

服务总线管理库提供用于动态预配服务总线命名空间和实体的功能，因而适用于复杂部署和消息传送方案，这让用户能够以编程方式确定要预配的实体。这些库当前面向 .NET 提供。

## 受支持的功能

* 创建、更新、删除命名空间
* 创建、更新、删除队列
* 创建、更新、删除主题
* 创建、更新、删除订阅

## 先决条件

若要开始使用服务总线管理库，必须使用 Azure Active Directory (AAD) 进行身份验证。AAD 要求以提供 Azure 资源访问权限的服务主体身份进行身份验证。有关创建服务主体的信息，请参阅以下文章之一：

* [使用 Azure 门户创建可访问资源的 Active Directory 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)
* [使用 Azure PowerShell 创建服务主体来访问资源](/documentation/articles/resource-group-authenticate-service-principal/)
* [使用 Azure CLI 创建服务主体来访问资源](/documentation/articles/resource-group-authenticate-service-principal-cli/)

前面的教程提供 `AppId`（客户端 ID）、`TenantId` 和 `ClientSecret`（身份验证密钥），管理库会使用它们进行身份验证。必须具有要在其中运行的资源组的“所有者”权限。

## 编程模式

所有服务总线资源的操纵模式都具有相似性，并遵循常用协议：

1. 使用 `Microsoft.IdentityModel.Clients.ActiveDirectory` 库从 Azure Active Directory 获取令牌。

	    var context = new AuthenticationContext($"https://login.windows.cn/{tenantId}");

	    var result = await context.AcquireTokenAsync(
	        "https://management.core.windows.net/",
	        new ClientCredential(clientId, clientSecret)
	    );


1. 创建 `ServiceBusManagementClient` 对象。

	    var creds = new TokenCredentials(token);
	    var sbClient = new ServiceBusManagementClient(creds)
	    {
	        SubscriptionId = SettingsCache["SubscriptionId"]
	    };


1. 将 CreateOrUpdate 参数设置为指定的值。

	    var queueParams = new QueueCreateOrUpdateParameters()
	    {
	        Location = SettingsCache["DataCenterLocation"],
	        EnablePartitioning = true
	    };


1. 执行调用。

	    await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, QueueName, queueParams);


## 后续步骤
* [.NET 管理示例](https://github.com/Azure-Samples/service-bus-dotnet-management/)
* [Microsoft.Azure.Management.ServiceBus 参考](https://docs.microsoft.com/en-us/dotnet/api/Microsoft.Azure.Management.ServiceBus)

<!---HONumber=Mooncake_0213_2017-->