<properties
	pageTitle="我的 WebJob 项目（Visual Studio Azure 存储连接服务）发生了什么情况？| Windows Azure"
	description="介绍使用 Visual Studio 连接服务连接到存储帐户后 Azure WebJob 项目中会发生什么情况" 
	services="storage"
	documentationCenter=""
	authors="patshea123"
	manager="douge"
	editor="tglee"/>

<tags 
	ms.service="storage"
	ms.date="09/03/2015"
	wacn.date="10/17/2015"/>

# 我的 WebJob 项目（Visual Studio Azure 存储连接服务）发生了什么情况？

> [AZURE.SELECTOR]
> - [入门](/documentation/articles/vs-storage-webjobs-getting-started-blobs)
> - [发生了什么情况](/documentation/articles/vs-storage-webjobs-what-happened)


##### 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目或在其中更新。  
此包添加了以下 .NET 引用：

- `Microsoft.Data.Edm`
- `Microsoft.Data.OData`
- `Microsoft.Data.Services.Client`
- `Microsoft.WindowsAzure.ConfigurationManager`
- `Microsoft.WindowsAzure.Storage`
- `Newtonsoft.Json`
- `System.Data`
- `System.Spatial`

## 已添加 Azure 存储的连接字符串
在项目的 App.config 文件中，已使用选定存储帐户的连接字符串和密钥更新 **AzureWebJobsStorage** 和 **AzureWebJobsDashboard** 条目。

有关详细信息，请参阅[Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources)。

<!---HONumber=74-->