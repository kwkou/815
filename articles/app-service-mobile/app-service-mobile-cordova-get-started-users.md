<properties
	pageTitle="使用移动应用在 Apache Cordova 中添加身份验证 | Azure 应用服务"
	description="了解如何使用 Azure 应用服务中的移动应用，通过各种标识提供者对 Apache Cordova 应用的用户进行身份验证。"
	services="app-service\mobile"
	documentationCenter="javascript"
	authors="adrianhall"
	manager="erikre"
	editor=""/>  


<tags
	ms.service="app-service-mobile"
	ms.workload="na"
	ms.tgt_pltfrm="mobile-html"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="12/26/2016"
	ms.author="adrianha"/>

# 将身份验证添加到 Apache Cordova 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

## 摘要

本教程介绍如何使用支持的标识提供者将身份验证添加到 Apache Cordova 上的待办事项列表快速入门项目。本教程基于 [Get started with Mobile Apps]（移动应用入门）教程，必须先完成该教程。

## <a name="register"></a>注册应用以进行身份验证并配置应用服务
[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

现在，可以验证是否已禁用对后端的匿名访问。在 Visual Studio 中打开完成 [Get started with Mobile Apps]（移动应用入门）教程时创建的项目，然后在 **Google Android Emulator** 中运行应用程序，验证启动应用后是否显示“意外的连接失败”。

接下来，需要更新应用程序，以便在从移动应用后端请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

1. 在 **Visual Studio** 中打开项目，然后打开 `www/index.html` 文件进行编辑。

2. 找到 head 节中的 `Content-Security-Policy` 元标记。需要将 OAuth 主机添加到允许的源列表。

    | 提供程序 | SDK 提供程序名称 | OAuth 主机 |
    | :--------------------- | :---------------- | :-------------------------- |
    | Azure Active Directory | aad | https://login.chinacloudapi.cn |
    | Microsoft | microsoftaccount | https://login.live.com |

    下面显示了 Content-Security-Policy（针对 Azure Active Directory 实现）的示例：

        <meta http-equiv="Content-Security-Policy" content="default-src 'self'
			data: gap: https://login.chinacloudapi.cn https://yourapp.chinacloudsites.cn; style-src 'self'">

    应该将 `https://login.chinacloudapi.cn` 替换为上表中的 OAuth 主机。有关此元标记的详细信息，请参阅 [Content-Security-Policy 文档]。

    请注意，在相应的移动设备上使用时，某些身份验证提供程序不需要 Content-Security-Policy 更改。

3. 打开 `www/js/index.js` 文件进行编辑，找到 `onDeviceReady()` 方法，然后在客户端创建代码下面添加以下内容：

        // Login to the service
        client.login('SDK_Provider_Name')
            .then(function () {

                // BEGINNING OF ORIGINAL CODE

                // Create a table reference
                todoItemTable = client.getTable('todoitem');

                // Refresh the todoItems
                refreshDisplay();

                // Wire up the UI Event Handler for the Add Item
                $('#add-item').submit(addItemHandler);
                $('#refresh').on('click', refreshDisplay);

                // END OF ORIGINAL CODE

            }, handleError);

    请注意，此代码将替换用于创建表引用和刷新 UI 的现有代码。

    login() 方法开始对提供程序进行身份验证。login() 方法是返回 JavaScript Promise 的异步函数。初始化的剩余部分放置在 promise 响应中，因此在 login() 方法完成之前不会执行。

4. 在刚刚添加的代码中，将 `SDK_Provider_Name` 替换为登录提供程序的名称。例如，对于 Azure Active Directory，请使用 `client.login('aad')`。

4. 运行项目。当项目完成初始化时，应用程序将针对所选的身份验证提供程序显示 OAuth 登录页。

## <a name="next-steps"></a>后续步骤

* 了解[有关 Azure 应用服务身份验证]的详细信息。
* 将[推送通知]添加到 Apache Cordova 应用，继续学习本教程。

了解如何使用 SDK。

* [Apache Cordova SDK]
* [ASP.NET Server SDK]
* [Node.js Server SDK]

<!-- URLs. -->
[Get started with Mobile Apps]: /documentation/articles/app-service-mobile-cordova-get-started/
[Content-Security-Policy 文档]: https://cordova.apache.org/docs/en/latest/guide/appdev/whitelist/index.html
[推送通知]: /documentation/articles/app-service-mobile-cordova-get-started-push/
[有关 Azure 应用服务身份验证]: /documentation/articles/app-service-mobile-auth/
[Apache Cordova SDK]: /documentation/articles/app-service-mobile-codova-how-to-use-client-library/
[ASP.NET Server SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Node.js Server SDK]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/

<!---HONumber=Mooncake_1219_2016-->