<properties 
	pageTitle="Azure 存储入门" 
	description="介绍在 Visual Studio Azure WebJob 项目中创建 Azure 存储时发生了什么情况" 
	services="storage" 
	documentationCenter="" 
	authors="patshea123" 
	manager="douge" 
	editor="tglee"/>

<tags 
	ms.service="storage"
	
	ms.date="07/13/2015" 
	wacn.date=""/>

# 我的项目发生了什么情况？

> [AZURE.SELECTOR]
> - [Getting Started](/documentation/articles/vs-storage-webjobs-getting-started-blobs)
> - [What Happened](/documentation/articles/vs-storage-webjobs-what-happened)

###<span id="whathappened">我的项目发生了什么情况？</span>

##### 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目或在其中更新。此包添加了以下 .NET 引用：

- `Microsoft.Data.Edm`
- `Microsoft.Data.OData`
- `Microsoft.Data.Services.Client`
- `Microsoft.WindowsAzure.ConfigurationManager`
- `Microsoft.WindowsAzure.Storage`
- `Newtonsoft.Json`
- `System.Data`
- `System.Spatial`

#####已添加 Azure 存储的连接字符串 
在项目的 App.config 文件中，已使用选定存储帐户的连接字符串和密钥更新 `AzureWebJobsStorage` 和 `AzureWebJobsDashboard`。

有关详细信息，请参阅[Azure WebJobs 推荐资源](http://www.windowsazure.cn/documentation/articles/websites-webjobs-resources/)。

<!---HONumber=67-->