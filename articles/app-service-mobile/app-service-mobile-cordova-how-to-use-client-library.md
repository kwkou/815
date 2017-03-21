<properties
	pageTitle="如何使用适用于 Azure 移动应用的 Apache Cordova 插件"
	description="如何使用适用于 Azure 移动应用的 Apache Cordova 插件"
	services="app-service\mobile"
	documentationCenter="javascript"
	authors="adrianhall"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-html"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="10/30/2016"
	wacn.date="03/20/2017"
	ms.author="adrianha"/>

# 如何使用适用于 Azure 移动应用的 Apache Cordova 客户端库

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南介绍如何使用最新的[适用于 Azure 移动应用的 Apache Cordova 插件]执行常见任务。对于 Azure 移动应用的新手，请先完成 [Azure Mobile Apps Quick Start]（Azure 移动应用快速入门），创建后端、创建表并下载预先生成的 Apache Cordova 项目。本指南侧重于客户端 Apache Cordova 插件。

## 支持的平台

此 SDK 在 iOS、Android 和 Windows 设备上支持 Apache Cordova v6.0.0 和更高版本。平台支持如下所示：

* Android API 19-24（KitKat 到 Nougat）
* iOS 8.0 和更高版本。
* Windows Phone 8.1。
* 通用 Windows 平台。

##<a name="Setup"></a>安装与先决条件

本指南假设已创建了包含表的后端。本指南假设该表的架构与这些教程中的表相同。本指南还假设已将 Apache Cordova 插件添加到代码。如果尚未这样做，可以在命令行中将 Apache Cordova 插件添加到项目：

    cordova plugin add cordova-plugin-ms-azure-mobile-apps

有关创建[第一个 Apache Cordova 应用]的详细信息，请参阅相关文档。

## <a name="ionic"></a>设置 Ionic v2 应用

若要正确配置 Ionic v2 项目，首先需创建一个基本应用，并添加 Cordova 插件：


	ionic start projectName --v2
	cd projectName
	ionic plugin add cordova-plugin-ms-azure-mobile-apps


向 `app.component.ts` 中添加以下行以创建客户端对象：


	declare var WindowsAzure: any;
	var client = new WindowsAzure.MobileServiceClient("https://yoursite.azurewebsites.cn");


现在可以在浏览器中生成并运行项目：


	ionic platform add browser
	ionic run browser


Azure 移动应用 Cordova 插件同时支持 Ionic v1 和 v2 应用。只有 Ionic v2 应用需要额外的针对 `WindowsAzure` 对象的声明。

[!INCLUDE [app-service-mobile-html-js-library.md](../../includes/app-service-mobile-html-js-library.md)]

## <a name="auth"></a>如何：对用户进行身份验证

Azure 应用服务支持使用各种外部标识提供者（包括 Microsoft 帐户）对应用用户进行身份验证和授权。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅 [Get started with authentication]（身份验证入门）教程。

在 Apache Cordova 应用中使用身份验证时，以下 Cordova 插件必须可用：

* [cordova-plugin-device]
* [cordova-plugin-inappbrowser]

支持两种身份验证流：服务器流和客户端流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

[AZURE.INCLUDE [app-service-mobile-html-js-auth-library.md](../../includes/app-service-mobile-html-js-auth-library.md)]

### <a name="configure-external-redirect-urls"></a>如何为外部重定向 URL 配置移动应用服务。

有多种类型的 Apache Cordova 应用程序使用环回功能来处理 OAuth UI 流。Localhost 上的 OAuth UI 流会导致问题，因为默认情况下，身份验证服务只知道如何利用服务。有问题的 OAuth UI 流的示例包括：

- Ripple 模拟器。
- 使用 Ionic 的实时重新加载。
- 在本地运行移动后端
- 在提供身份验证服务以外的其他 Azure App Service 中运行移动后端。

请遵循以下说明将本地设置添加到配置中：

1. 登录到 [Azure 门户预览]
2. 选择“所有资源”或“应用服务”，然后单击移动应用的名称。
3. 单击“工具”
4. 单击“观察”菜单中的“资源浏览器”，然后单击“转到”。会打开新窗口或选项卡。
5. 在左侧导航栏中，展开站点的“config”、“authsettings”节点。
6. 单击“编辑”
7. 查找“allowedExternalRedirectUrls”元素。它可能被设置为 null 或值数组。将该值更改为以下值：

         "allowedExternalRedirectUrls": [
             "http://localhost:3000",
             "https://localhost:3000"
         ],

    将 URL 替换为自己服务的 URL。示例包括“http://localhost:3000”（适用于 Node.js 示例服务）或“http://localhost:4400”（适用于 Ripple 服务）。但是，这些 URL 是示例，根据不同的情况（包括示例中提到的服务）可能会有差异。
8. 单击屏幕右上角的“读/写”按钮。
9. 单击绿色的“PUT”按钮。

此时会保存设置。在保存完设置之前，请不要关闭浏览器窗口。同时将以下环回 URL 添加到应用服务的 CORS 设置：

1. 登录到 [Azure 门户预览]。
2. 选择“所有资源”或“应用服务”，然后单击移动应用的名称。
3. “设置”边栏选项卡会自动打开。如果没有打开，请单击“所有设置”。
4. 在“API”菜单下面单击“CORS”。
5. 在提供的框中输入想要添加的 URL，然后按 Enter。
6. 根据需要输入其他 URL。
7. 单击“保存”保存这些设置。

大约需要 10-15 秒时间才能使新设置生效。

## <a name="register-for-push"></a>如何注册推送通知

安装 [phonegap-plugin-push] 即可处理推送通知。在命令行中使用 `cordova plugin add` 命令，或者在 Visual Studio 内通过 Git 插件安装程序，即可轻松添加此插件。Apache Cordova 应用中的以下代码为设备注册推送通知：


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


使用通知中心 SDK 从服务器发送推送通知。永远不要直接从客户端发送推送通知。此做法可能会被用于触发针对通知中心或 PNS 的拒绝服务攻击。PNS 可能会因此类攻击而禁止相关流量。

## 详细信息

可在 [API 文档](http://azure.github.io/azure-mobile-apps-js-client/)中找到有关 API 的详细信息。

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

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update wording-->