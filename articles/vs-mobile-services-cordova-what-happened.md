<properties title="移动服务入门" pageTitle="" metaKeywords="Azure, Getting Started, Mobile Services" description="" services="mobile-services" documentationCenter="" authors="ghogen, kempb" />

<tags 
wacn.date="04/11/2015"
ms.service="mobile-services" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="02/02/2015" ms.author="ghogen, kempb"></tags>

> [AZURE.SELECTOR]
>
> -   [入门][入门]
> -   [发生了什么情况][发生了什么情况]

### <span id="whathappened">我的项目发生了什么情况？</span>

##### 已添加引用

已启用所有多个设备混合应用随附的 Azure 移动服务客户端插件。

##### 移动服务的连接字符串值

在 `services\mobileServices\settings` 下，已生成具有 **MobileServiceClient** 的新 JavaScript (.js) 文件，其中包含选定的移动服务的应用程序 URL 和应用程序密钥。该文件包含移动服务客户端对象的初始化，类似于以下代码。

    var mobileServiceClient;
    document.addEventListener("deviceready", function() {
            mobileServiceClient = new WindowsAzure.MobileServiceClient(
            "<your mobile service name>.azure-mobile.cn",
            "<insert your key>"
        );

[详细了解移动服务][详细了解移动服务]

  [入门]: /documentation/articles/vs-mobile-services-cordova-getting-started/
  [发生了什么情况]: /documentation/articles/vs-mobile-services-cordova-what-happened/
  [详细了解移动服务]: http://www.windowsazure.cn/manage/services/mobile-services/
