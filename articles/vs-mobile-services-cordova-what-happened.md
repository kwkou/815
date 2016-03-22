<properties 
	pageTitle="我的 Cordova 项目发生了什么情况（Visual Studio 连接服务）| Microsoft Azure" 
	description="描述使用 Visual Studio 连接服务添加 Azure 移动服务后，Azure Cordova 项目发生了什么情况" 
	services="mobile-services" 
	documentationCenter="na" 
	authors="mlhoop" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/05/2016" 
	wacn.date="03/21/2016"/>

# 使用 Visual Studio 连接服务添加 Azure 移动服务后，我的 Azure Cordova 项目发生了什么情况？

##已添加引用

已启用所有多个设备混合应用随附的 Azure 移动服务客户端插件。
  
##移动服务的连接字符串值

在 `services\mobileServices\settings` 下，已生成包含 **MobileServiceClient** 的新 JavaScript (.js) 文件，其中包含选定的移动服务的应用程序 URL 和应用程序密钥。该文件包含移动服务客户端对象的初始化，类似于以下代码。

	var mobileServiceClient;
	document.addEventListener("deviceready", function() {
            mobileServiceClient = new WindowsAzure.MobileServiceClient(
	        "<your mobile service name>.azure-mobile.net",
	        "<insert your key>"
	    );

[详细了解移动服务](/documentation/services/mobile-services)

<!---HONumber=Mooncake_0215_2016-->