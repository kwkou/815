<properties
	pageTitle="Azure 存储入门"
	description="介绍在 Visual Studio 云服务项目中使用 Azure 存储时会发生什么情况"
	services="storage"
	documentationCenter=""
	authors="patshea123"
	manager="douge"
	editor="tglee"/>

<tags ms.service="storage"
	
	ms.date="07/22/2015"
	wacn.date="09/16/2015"/>

# 我的项目发生了什么情况？

> [AZURE.SELECTOR]
> - [Getting started](/documentation/articles/vs-storage-cloud-services-getting-started-blobs)
> - [What happened](/documentation/articles/vs-storage-cloud-services-what-happened)

###我的项目发生了什么情况？

###### 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目。此包添加了以下 .NET 引用：

-   `Microsoft.Data.Edm`
-   `Microsoft.Data.OData`
-   `Microsoft.Data.Services.Client`
-   `Microsoft.WindowsAzure.Configuration`
-   `Microsoft.WindowsAzure.Storage`
-   `Newtonsoft.Json`
-   `System.Data`
-   `System.Spatial`

###### 已添加 Azure 存储的连接字符串

已使用选定存储帐户的连接字符串和密钥创建了元素。已对以下文件进行了修改：

-   `ServiceDefinition.csdef`
-   `ServiceConfiguration.Cloud.cscfg`
-   `ServiceConfiguration.Local.cscfg`

<!---HONumber=69-->