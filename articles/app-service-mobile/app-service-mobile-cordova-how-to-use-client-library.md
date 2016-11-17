<properties
	pageTitle="如何使用适用于 Azure 移动应用的 Apache Cordova 插件"
	description="如何使用适用于 Azure 移动应用的 Apache Cordova 插件"
	services="app-service\mobile"
	documentationCenter="javascript"
	authors="ggailey777"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-html"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="10/17/2016"
	ms.author="adrianha"/>

# 如何使用适用于 Azure 移动应用的 Apache Cordova 客户端库

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南介绍如何使用最新的[适用于 Azure 移动应用的 Apache Cordova 插件]执行常见任务。对于 Azure 移动应用的新手，请先完成 [Azure Mobile Apps Quick Start]（Azure 移动应用快速入门），创建后端、创建表并下载预先生成的 Apache Cordova 项目。本指南侧重于客户端 Apache Cordova 插件。

##<a name="Setup"></a>安装与先决条件

本指南假设已创建了包含表的后端。本指南假设该表的架构与这些教程中的表相同。本指南还假设已将 Apache Cordova 插件添加到代码。如果尚未这样做，可以在命令行中将 Apache Cordova 插件添加到项目：

```
cordova plugin add cordova-plugin-ms-azure-mobile-apps
```

有关创建[第一个 Apache Cordova 应用]的详细信息，请参阅相关文档。

[AZURE.INCLUDE [app-service-mobile-html-js-library.md](../../includes/app-service-mobile-html-js-library.md)]

##<a name="auth"></a>如何：对用户进行身份验证

Azure 应用服务支持使用各种外部标识提供者（包括 Microsoft 帐户）对应用用户进行身份验证和授权。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅 [Get started with authentication]（身份验证入门）教程。

在 Apache Cordova 应用中使用身份验证时，以下 Cordova 插件必须可用：

* [cordova-plugin-device]
* [cordova-plugin-inappbrowser]

支持两种身份验证流：服务器流和客户端流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

[AZURE.INCLUDE [app-service-mobile-html-js-auth-library.md](../../includes/app-service-mobile-html-js-auth-library.md)]

###<a name="configure-external-redirect-urls"></a>如何为外部重定向 URL 配置移动应用服务。

有多种类型的 Apache Cordova 应用程序使用环回功能来处理 OAuth UI 流。这会造成问题，因为身份验证服务默认只知道如何利用服务。本示例使用 Ripple 模拟器，在本地或不同的 Azure 应用服务中运行服务，但重定向到 Azure 应用服务进行身份验证，或使用 Ionic 实时重新加载。请遵循以下说明将本地设置添加到配置中：

1. 登录到 [Azure 门户预览]。
2. 选择“所有资源”或“应用服务”，然后单击移动应用的名称。
3. 单击“工具”
4. 在“观察”菜单中单击“资源浏览器”，然后单击“转到”。此时将打开一个新窗口或选项卡。
5. 在左侧导航栏中，展开站点的“config”、“authsettings”节点。
6. 单击“编辑”
7. 查找“allowedExternalRedirectUrls”元素。该元素将设置为 null。将它更改为：

         "allowedExternalRedirectUrls": [
             "http://localhost:3000",
             "https://localhost:3000"
         ],

    将 URL 替换为自己服务的 URL。示例包括“http://localhost:3000”（适用于 Node.js 示例服务）或“http://localhost:4400”（适用于 Ripple 服务）。但这是一些示例 - 根据不同的情况（包括示例中提到的服务）可能会有差异。
8. 单击屏幕右上角的“读/写”按钮。
9. 单击绿色的“PUT”按钮。

此时将保存设置。在保存完设置之前，请不要关闭浏览器窗口。还需要将以下环回 URL 添加到 CORS 设置：

1. 登录到 [Azure 门户预览]。
2. 选择“所有资源”或“应用服务”，然后单击移动应用的名称。
3. “设置”边栏选项卡随即自动打开。如果没有打开，请单击“所有设置”。
4. 在“API”菜单下面单击“CORS”。
5. 在提供的框中输入想要添加的 URL，然后按 Enter。
6. 根据需要输入其他 URL。
7. 单击“保存”以保存设置。

大约需要 10-15 秒时间才能使新设置生效。

##<a name="register-for-push"></a>如何注册推送通知

安装 [phonegap-plugin-push] 即可处理推送通知。在命令行中使用 `cordova plugin add` 命令，或者在 Visual Studio 内通过 Git 插件安装程序，即可轻松添加此插件。Apache Cordova 应用中的以下代码将为设备注册推送通知：

```
var pushOptions = {
    android: {
        senderId: '<from-gcm-console>'
    },
    ios: {
        alert: true,
        badge: true,
        sound: true
    },
    windows: {
    }
};
pushHandler = PushNotification.init(pushOptions);

pushHandler.on('registration', function (data) {
    registrationId = data.registrationId;
    // For cross-platform, you can use the device plugin to determine the device
    // Best is to use device.platform
    var name = 'gcm'; // For android - default
    if (device.platform.toLowerCase() === 'ios')
        name = 'apns';
    if (device.platform.toLowerCase().substring(0, 3) === 'win')
        name = 'wns';
    client.push.register(name, registrationId);
});

pushHandler.on('notification', function (data) {
    // data is an object and is whatever is sent by the PNS - check the format
    // for your particular PNS
});

pushHandler.on('error', function (error) {
    // Handle errors
});
```

使用通知中心 SDK 从服务器发送推送通知。切勿直接从客户端发送推送通知，否则可能会针对通知中心或 PNS 触发拒绝服务攻击。

<!-- URLs. -->
[Azure 门户预览]: https://portal.azure.cn
[Azure Mobile Apps Quick Start]: /documentation/articles/app-service-mobile-cordova-get-started/
[Get started with authentication]: /documentation/articles/app-service-mobile-cordova-get-started-users/
[Add authentication to your app]: /documentation/articles/app-service-mobile-cordova-get-started-users/

[适用于 Azure 移动应用的 Apache Cordova 插件]: https://www.npmjs.com/package/cordova-plugin-ms-azure-mobile-apps
[第一个 Apache Cordova 应用]: http://cordova.apache.org/#getstarted
[phonegap-plugin-push]: https://www.npmjs.com/package/phonegap-plugin-push
[cordova-plugin-device]: https://www.npmjs.com/package/cordova-plugin-device
[cordova-plugin-inappbrowser]: https://www.npmjs.com/package/cordova-plugin-inappbrowser
[Query object documentation]: https://msdn.microsoft.com/zh-cn/library/azure/jj613353.aspx

<!---HONumber=Mooncake_0919_2016-->