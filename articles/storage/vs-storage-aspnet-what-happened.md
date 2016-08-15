<properties
	pageTitle="我的 ASP.NET 项目发生了什么情况？| Azure | Visual Studio 连接服务"
	description="介绍使用 Visual Studio 连接服务向 ASP.NET 项目添加 Azure 存储后会发生什么情况"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="05/08/2016"
	wacn.date="06/13/2016"/>

# 我的 ASP.NET 项目（Visual Studio Azure 存储连接服务）发生了什么情况？

## 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目。  
此包添加了以下 .NET 引用：

- **Microsoft.Data.Edm**
- **Microsoft.Data.OData**
- **Microsoft.Data.Services.Client**
- **Microsoft.WindowsAzure.Configuration**
- **Microsoft.WindowsAzure.Storage**
- **Newtonsoft.Json**
- **System.Data**
- **System.Spatial**

##已添加 Azure 存储的连接字符串
在项目的 web.config 文件中，已使用选定存储帐户的连接字符串和密钥创建了一个元素。

有关详细信息，请参阅 [ASP.NET](http://www.asp.net)。

<!---HONumber=Mooncake_0606_2016-->