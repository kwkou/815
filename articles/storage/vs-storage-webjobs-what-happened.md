<properties
    pageTitle="我的 WebJob 项目（Visual Studio Azure 存储空间连接服务）发生了什么情况？| Azure"
    description="介绍使用 Visual Studio 连接服务连接到存储帐户后 Azure WebJob 项目中会发生什么情况"
    services="storage"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />
<tags
    ms.assetid="36ae7ff7-c22c-47eb-b220-049d61618c74"
    ms.service="storage"
    ms.workload="web"
    ms.tgt_pltfrm="vs-what-happened"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/02/2016"
    wacn.date="01/06/2017"
    ms.author="tarcher" />  


# 我的 WebJob 项目（Visual Studio Azure 存储连接服务）发生了什么情况？

## 已添加引用

Azure 存储 NuGet 包已添加到你的 Visual Studio 项目或在其中更新。
此包添加了以下 .NET 引用：

- **Microsoft.Data.Edm**
- **Microsoft.Data.OData**
- **Microsoft.Data.Services.Client**
- **Microsoft.WindowsAzure.ConfigurationManager**
- **Microsoft.WindowsAzure.Storage**
- **Newtonsoft.Json**
- **System.Data**
- **System.Spatial**

## 已添加 Azure 存储的连接字符串
在项目的 App.config 文件中，已使用选定存储帐户的连接字符串和密钥更新 **AzureWebJobsStorage** 和 **AzureWebJobsDashboard** 条目。

有关详细信息，请参阅[Azure Web 作业文档资源](/documentation/articles/websites-webjobs-resources/)。

<!---HONumber=Mooncake_0103_2017-->