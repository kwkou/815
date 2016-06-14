<properties
    pageTitle="我的云服务项目发生了什么情况？| Azure | Visual Studio 连接服务"
	description="介绍使用 Visual Studio 连接服务连接到 Azure 存储帐户后云服务项目中会发生什么情况"
    services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="05/08/2016"
	wacn.date="06/13/2016"/>
# 我的云服务项目（Visual Studio Azure 存储连接服务）发生了什么情况？

## 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目。此包添加了以下 .NET 引用：

- **Microsoft.Data.Edm**
- **Microsoft.Data.OData**
- **Microsoft.Data.Services.Client**
- **Microsoft.WindowsAzure.Configuration**
- **Microsoft.WindowsAzure.Storage**
- **Newtonsoft.Json**
- **System.Data**
- **System.Spatial**

## 已添加 Azure 存储的连接字符串
已使用选定存储帐户的连接字符串和密钥创建了元素。已对以下文件进行了修改：

- **ServiceDefinition.csdef**
- **ServiceConfiguration.Cloud.cscfg**
- **ServiceConfiguration.Local.cscfg**

<!---HONumber=Mooncake_0606_2016-->