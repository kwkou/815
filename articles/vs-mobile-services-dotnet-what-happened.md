<properties 
	pageTitle="" 
	description="描述 Visual Studio 中的 Azure 移动服务 .NET 项目发生了什么情况" 
	services="mobile-services" 
	documentationCenter="" 
	authors="patshea123" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="05/06/2015" 
	wacn.date="07/25/2015"/>

#  我的项目发生了什么情况？

> [AZURE.SELECTOR]
> - [Getting Started](vs-mobile-services-dotnet-getting-started)
> - [What Happened](vs-mobile-services-dotnet-what-happened)

### <span id="whathappened">我的项目发生了什么情况？</span>

##### 已添加引用

Azure 移动服务 NuGet 包已添加到您的项目。因此，下面的 .NET 引用也已添加到您的项目：

- `Microsoft.WindowsAzure.Mobile`
- `Microsoft.WindowsAzure.Mobile.Ext`
- `Newtonsoft.Json`
- `System.Net.Http.Extensions`
- `System.Net.Http.Primitives` 

##### 移动服务的连接字符串值

在 App.xaml.cs 文件中，已使用选定移动服务的应用程序 URL 和应用程序密钥创建 **MobileServiceClient** 对象。

[详细了解移动服务](/documentation/services/mobile-services)

<!---HONumber=HO63-->