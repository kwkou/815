<properties
    pageTitle="Azure 事件中心管理库 | Azure"
      description="通过 .NET 管理事件中心命名空间和实体"
      services="event-hubs"
      cloud="na"
      documentationcenter="na"
      author="jtaubensee"
      manager="sethm" />
<tags
    ms.assetid=""
      ms.service="event-hubs"
      ms.workload="na"
      ms.tgt_pltfrm="na"
      ms.devlang="dotnet"
      ms.topic="article"
      ms.date="1/6/2017"
      wacn.date="02/24/2017"
      ms.author="jotaub" />  


# 事件中心管理库

事件中心管理库提供用于动态预配事件中心命名空间和实体的功能，因而适用于复杂部署和消息传送方案，这让用户能够以编程方式确定要预配的实体。这些库当前面向 .NET 提供。

## 受支持的功能

* 创建、更新、删除命名空间
* 事件中心创建、更新、删除
* 资源组创建、更新、删除

## 先决条件

若要开始使用事件中心管理库，必须使用 Azure Active Directory \(AAD\) 进行身份验证。AAD 要求以提供 Azure 资源访问权限的服务主体身份进行身份验证。有关创建服务主体的信息，请参阅以下文章之一：

* [使用 Azure 门户预览创建可访问资源的 Active Directory 应用程序和服务主体](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)
* [使用 Azure PowerShell 创建服务主体来访问资源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal)
* [使用 Azure CLI 创建服务主体来访问资源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal-cli)

前面的教程提供 `AppId`（客户端 ID）、`TenantId` 和 `ClientSecret`（身份验证密钥），管理库会使用它们进行身份验证。必须具有要在其中运行的资源组的“所有者”权限。

## 编程模式

所有事件中心资源的操作模式都具有相似性，并遵循一个通用协议：

1. 使用 `Microsoft.IdentityModel.Clients.ActiveDirectory` 库从 Azure Active Directory 获取令牌。

        var context = new AuthenticationContext($"https://login.chinacloudapi.cn/{tenantId}");

        var result = await context.AcquireTokenAsync(
            "https://management.core.chinacloudapi.cn/",
            new ClientCredential(clientId, clientSecret)
        );

2. 创建 `EventHubManagementClient` 对象。

        var creds = new TokenCredentials(token);
        var ehClient = new EventHubManagementClient(creds)
        {
            SubscriptionId = SettingsCache["SubscriptionId"]
        };

3. 将 CreateOrUpdate 参数设置为指定的值。

        var ehParams = new EventHubCreateOrUpdateParameters()
        {
            Location = SettingsCache["DataCenterLocation"]
        };

4. 执行调用。

        await ehClient.EventHubs.CreateOrUpdateAsync(resourceGroupName, namespaceName, EventHubName, ehParams);

## 后续步骤
* [.NET 管理示例](https://github.com/Azure-Samples/event-hubs-dotnet-management/)
* [Microsoft.Azure.Management.EventHub 参考](/dotnet/api/Microsoft.Azure.Management.EventHub)

<!---HONumber=Mooncake_0220_2017-->
<!-- Update_Description: add new article on managing the namespace and entity in the event hubs management libraries -->