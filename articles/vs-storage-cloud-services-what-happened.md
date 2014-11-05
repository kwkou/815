<properties title="Azure 存储入门" pageTitle="Azure 存储入门" metaKeywords="Azure, Getting Started, Storage" description="" services="storage" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="storage" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="09/17/2014" ms.author="ghogen, kempb"></tags>

> [AZURE.SELECTOR]
>
> -   [入门][入门]
> -   [发生了什么情况][发生了什么情况]

### <span id="whathappened">我的项目发生了什么情况？</span>

###### 已添加引用

Azure 存储 NuGet 包已添加到您的 Visual Studio 项目。
此包添加了以下 .NET 引用：

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

  [入门]: /documentation/articles/vs-storage-cloud-services-getting-started-blobs/
  [发生了什么情况]: /documentation/articles/vs-storage-cloud-services-what-happened/
