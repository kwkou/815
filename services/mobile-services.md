<properties linkid="dev-net-Mobile-Service" urlDisplayName="Windows Azure Mobile Service" pageTitle="移动服务 - Azure 微软云" metaKeywords="Mobile Service,Azure 移动服务,iOS,Windows Phone,Android,HTML,PhoneGap,Sencha,Office 365,AD,Active Directory,.Net,Node.js,Web API,动态受众细分,推送,通知,推送通知,服务端用户授权,后端作业,移动服务SDK,REST API" description="本页面是微软Azure云服务中移动服务的入口页。通过本页面，你可以找到有关Azure移动服务的相关内容。包括：iOS、Windows Phone、Android等主流移动操作系统上如何使用Azure移动服务提供的SDK、API等，使用你熟悉的.Net、Node.js等架构开发应用程序；如何将应用程序注册到Windows应用商店；如何链接到AD、SQL Database、MongoDB、Office 365、SharePoint；如何添加身份认证，如何对客户端进行授权等。" metaCanonical="" services="Mobile Service" documentationCenter="Services" title="Add a cloud backend to your app in minutes" authors="" solutions="" manager="" editor="" />
<tags ms.service="Mobile Service"
    ms.date=""
    wacn.date=""
    />     


#移动服务<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase;" class="wa-previewTag">预览</sup>

####只需几分钟即可向应用程序添加云后端

-   通过全天候监视和管理承载 .NET 或 Node.js Web API
-   将单一登录用于 Active Directory
-   向个人用户发送推送通知和进行动态受众细分
-   在 SQL、表存储和 MongoDB 中存储数据
-   访问本地系统、Office 365 和 SharePoint
-   使用基于云的同步构建可以脱机运行的应用程序

####创建您的第一个移动服务

-   [iOS 入门教程][iOS getstarted]
-   [Windows Phone 入门教程][WP getstarted]
-   [Windows应用商店 入门教程][Windows Store getstarted]
-   [Android 入门教程][Android getstarted]
-   [HTML 入门教程][HTML getstarted]
-   [PhoneGap 入门教程][PhoneGap getstarted]
-   [Sencha 入门教程][Sencha getstarted]

<!--
##移动服务指南

平台 | .Net | JavaScript
------------ | ------------- | ------------
iOS | [iOS/.Net指南]  | [iOS/JavaScript指南]
Windows Phone | [WP/.Net指南]  | [WP/Javascript指南]
Windows应用商店C# | [Windows应用商店C#/.Net]指南 | [Windows应用商店C#/Javascript]指南           
Windows应用商店JavaScript | 
Xamarin iOS |
Xamarin Android |
Android |
HTML |
PhoneGap |
Sencha |
-->

##教程和指南

###开始使用      

####[移动服务入门](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started/)

了解如何使用 Azure 移动服务将基于云的后端服务快速添加到应用程序中。

####[如何将移动服务客户端库用于 iOS](/zh-cn/documentation/articles/mobile-services-ios-how-to-use-client-library/)

了解如何使用 iOS 客户端库执行常见方案，包括查询数据，插入、更新和删除数据，对用户进行身份验证，处理错误，以及将数据上载到 Blob 存储。

###用户

####[向应用程序添加身份验证](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/)

了解如何通过各种标识提供者（包括 Microsoft）在应用中对用户进行身份验证，然后利用配置文件数据添加各种功能，例如按姓名向用户进行问候。

####[在服务端对用户进行授权](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-authorize-users-in-scripts/)

本主题演示如何对经过身份验证的用户进行授权，使其有权访问你的移动服务中的数据。在本教程中，你将向控制器中的数据访问方法添加代码，从而根据经过身份验证的用户的 userID 筛选查询，确保每位用户都只可以看到自己的数据。

###服务
        
####[使用混合连接从 Azure 移动服务连接到本地 SQL Server](/zh-cn/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started/)

了解如何使用混合连接安全地连接到本地资源。

####[从客户端调用自定义 API](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-call-custom-api/)

了解如何从 Windows 应用商店应用程序调用自定义 API。

####[在移动服务中安排后端作业](/zh-cn/documentation/articles/mobile-services-dotnet-backend-schedule-recurring-tasks/)

了解如何使用移动服务作业计划程序功能定义按您所设定的计划执行的服务器端代码

####[在发生灾难情形时恢复](/zh-cn/documentation/articles/mobile-services-disaster-recovery/)

了解如何使用 Azure 内置功能在出现可用性问题时确保业务连续性。            

###工具

####[使用命令行工具自动实施移动服务](/zh-cn/documentation/articles/mobile-services-manage-command-line-interface/)

本主题演示如何使用 Azure 命令行工具来自动创建和管理移动服务。其中介绍如何安装工具以及如何执行常规任务（包括新建移动服务、创建表、注册表操作脚本、删除表和现有移动服务）。

####[管理移动服务的命令](/zh-cn/documentation/articles/command-line-tools/#Commands_to_manage_mobile_services)

查找移动服务命令以便为您的应用程序实现后端功能。

###参考

####[iOS 客户端库](http://dl.windowsazure.cn/iosdocs/)

Azure 移动服务 SDK 包括支持进行 Windows 应用商店和 iOS 应用程序开发的客户端库。本节提供了有关使用这些客户端库的详细信息。

####[如何将移动服务客户端库用于 iOS](/zh-cn/documentation/articles/mobile-services-ios-how-to-use-client-library/)

了解如何使用 iOS 客户端库执行常见方案，包括查询数据，插入、更新和删除数据，对用户进行身份验证，处理错误，以及将数据上载到 Blob 存储。

####[.NET 服务库](http://msdn.microsoft.com/zh-cn/library/azure/dn632690.aspx)

利用 Windows Azure 移动服务的 .NET 服务库，可以开发使用 .NET 后端的移动服务。本节提供库的详细参考。

####[REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/jj710108.aspx)

移动服务提供一系列 REST API，可用于访问和更改表数据以及检索经过身份验证的登录信息。此参考提供有关使用移动服务 API 的常规信息，以及每个可用操作的特定参考信息。

<!----------- Links ---------->
[iOS getstarted]:/zh-cn/documentation/articles/mobile-services-ios-get-started/
[WP getstarted]:/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started/
[Windows Store getstarted]:/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[Xamarin.iOS getstarted]:/zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started/
[Xamarin.Android getstarted]:/zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started/
[Android getstarted]:/zh-cn/documentation/articles/mobile-services-android-get-started/
[HTML getstarted]:/zh-cn/documentation/articles/mobile-services-html-get-started/
[PhoneGap getstarted]:/zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/
[Sencha getstarted]:/zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/


