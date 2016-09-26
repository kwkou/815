<properties
	pageTitle="如何使用适用于 Azure 移动应用的 JavaScript SDK"
	description="如何为 Azure 移动应用使用 v"
	services="app-service\mobile"
	documentationCenter="javascript"
	authors="adrianhall"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.date="06/29/2016"
	wacn.date="09/26/2016"/>

# 如何使用适用于 Azure 移动应用的 JavaScript 客户端库

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南介绍如何使用最新的 [Azure 移动应用 JavaScript SDK] 执行常见任务。对于 Azure 移动应用的新手，请先完成 [Azure Mobile Apps Quick Start]（Azure 移动应用快速入门）创建后端和表。本指南着重介绍如何在 HTML/JavaScript Web 应用程序中使用移动后端。

##<a name="Setup"></a>安装与先决条件

本指南假设已创建了包含表的后端。本指南假设该表的架构与这些教程中的表相同。

可以通过 `npm` 命令安装 Azure 移动应用 JavaScript SDK：

```
npm install azure-mobile-apps-client --save
```

安装后，库位于 `node_modules/azure-mobile-apps-client/dist/MobileServices.Web.min.js`。将此文件复制到 Web 区域。

```
<script src="path/to/MobileServices.Web.min.js"></script>
```

也可将库用作 CommonJS 环境（例如 Browserify 和 Webpack）中的 ES2015 模块，或者用作 AMD 库。例如：

```
# For ECMAScript 5.1 CommonJS
var WindowsAzure = require('azure-mobile-apps-client');
# For ES2015 modules
import * as WindowsAzure from 'azure-mobile-apps-client';
```

[AZURE.INCLUDE [app-service-mobile-html-js-library](../../includes/app-service-mobile-html-js-library.md)]

##<a name="auth"></a>如何：对用户进行身份验证

Azure 应用服务支持使用各种外部标识提供者（包括 Microsoft 帐户）对应用用户进行身份验证和授权。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅 [Get started with authentication]（身份验证入门）教程。

支持两种身份验证流：服务器流和客户端流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供程序特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

[AZURE.INCLUDE [app-service-mobile-html-js-auth-library](../../includes/app-service-mobile-html-js-auth-library.md)]

###<a name="configure-external-redirect-urls"></a>如何为外部重定向 URL 配置移动应用服务。

多种类型的 JavaScript 应用程序使用环回功能来处理 OAuth UI 流，例如，在本地运行服务时、在 Ionic 框架中使用实时重新加载，或重定向到应用服务进行身份验证时。这可能造成问题，因为默认情况下，应用服务身份验证只配置为允许从移动应用后端访问。

使用以下步骤更改应用服务设置，允许从 localhost 进行身份验证：

1. 登录到 [Azure 门户预览]，导航到移动应用后端，然后单击“工具”>“资源浏览器”>“转到”，为移动应用后端（站点）打开新的资源浏览器窗口。

2. 展开应用的“config”节点，然后单击“authsettings”>“编辑”，找到 **allowedExternalRedirectUrls** 元素（应为 null）并将它更改为：

         "allowedExternalRedirectUrls": [
             "http://localhost:3000",
             "https://localhost:3000"
         ],

    将数组中的 URL 替换为服务的 URL，在本示例中为本地 Node.js 示例服务的 `http://localhost:3000`。对于 Ripple 服务，也可以根据应用的配置方式，使用 `http://localhost:4400` 或其他某个 URL。
    
3. 在页面顶部，单击“读/写”，然后单击“PUT”保存更新。

    仍要将相同的环回 URL 添加到 CORS 允许列表设置：

4. 返回 [Azure 门户预览]，在移动应用后端，单击“所有设置”>“CORS”，将环回 URL 添加到允许列表，然后单击“保存”。

后端更新后，可以在应用中使用新的环回 URL。

<!-- URLs. -->
[Azure Mobile Apps Quick Start]: /documentation/articles/app-service-mobile-cordova-get-started/
[Get started with authentication]: /documentation/articles/app-service-mobile-cordova-get-started-users/
[Add authentication to your app]: /documentation/articles/app-service-mobile-cordova-get-started-users/

[Azure 移动应用 JavaScript SDK]: https://www.npmjs.com/package/azure-mobile-apps-client
[Query object documentation]: https://msdn.microsoft.com/zh-cn/library/azure/jj613353.aspx

<!---HONumber=Mooncake_0919_2016-->