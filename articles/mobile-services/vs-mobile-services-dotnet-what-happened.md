<properties
	pageTitle="使用 Visual Studio 连接服务添加移动服务后，我的 .NET 项目发生了什么情况？| Microsoft Azure"
	description="描述使用“连接服务”添加 Azure 移动服务后，Visual Studio .NET 项目发生了什么情况"
	services="mobile-services"
	documentationCenter=""
	authors="mlhoop"
	manager="douge"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/05/2016"
	wacn.date="03/21/2016"/>

# 使用“连接服务”添加 Azure 移动服务后，我的 Visual Studio .NET 项目发生了什么情况？


## 已添加引用

Azure 移动服务 NuGet 包已添加到您的项目。因此，下面的 .NET 引用也已添加到您的项目：

- **Microsoft.WindowsAzure.Mobile**
- **Microsoft.WindowsAzure.Mobile.Ext**
- **Newtonsoft.Json**
- **System.Net.Http.Extensions**
- **System.Net.Http.Primitives** 

## 移动服务的连接字符串值

在 App.xaml.cs 文件中，已使用选定移动服务的应用程序 URL 和应用程序密钥创建 **MobileServiceClient** 对象。

## 已添加移动服务项目

如果在连接服务提供程序中创建了 .NET 移动服务，则已创建移动服务项目并已将其添加到解决方案中。


[详细了解移动服务](/documentation/services/mobile-services/)

<!---HONumber=Mooncake_0215_2016-->