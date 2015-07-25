<properties 
	pageTitle="" 
	description="描述 Cordova 中的 Azure 移动服务项目发生了什么情况" 
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
> - [Getting Started](vs-mobile-services-cordova-getting-started)
> - [What Happened](vs-mobile-services-cordova-what-happened)

### <span id="whathappened">我的项目发生了什么情况？</span>

##### 已添加引用

已启用所有多个设备混合应用随附的 Azure 移动服务客户端插件。
  
##### 移动服务的连接字符串值

在 `services\mobileServices\settings` 下，已生成包含 **MobileServiceClient** 的新 JavaScript (.js) 文件，其中包含选定的移动服务的应用程序 URL 和应用程序密钥。该文件包含移动服务客户端对象的初始化，类似于以下代码。

	var mobileServiceClient;
	document.addEventListener("deviceready", function() {
            mobileServiceClient = new WindowsAzure.MobileServiceClient(
	        "<your mobile service name>.azure-mobile.net",
	        "<insert your key>"
	    );

[详细了解移动服务](/documentation/services/mobile-services)

<!---HONumber=HO63-->