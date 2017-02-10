<properties
    pageTitle="Azure AD Cordova 入门 | Azure"
    description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 Cordova 应用程序。"
    services="active-directory"
    documentationcenter=""
    author="vibronet"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="b1a8d7bd-7ad6-44d5-8ccb-5255bb623345"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="javascript"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="vittorib" />  


# 将 Azure AD 与 Apache Cordova 应用集成
[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Apache Cordova 可让你开发能够在移动设备上运行的完全成熟的 HTML5/JavaScript 本机应用程序。
使用 Azure AD，可以向 Cordova 应用程序添加企业级身份验证功能。得益于在 iOS、Android、Windows 应用商店和 Windows Phone 上包装 Azure AD 本机 SDK 的 Cordova 插件，你可以增强应用程序以支持使用用户的 AD 帐户登录，访问 Office 365 和 Azure API，甚至保护对你自己的自定义 Web API 的调用。

在本教程中，我们将使用适用于 Active Directory 身份验证库 (ADAL) 的 Apache Cordova 插件来改进一个具有以下功能的简单应用：

- 只需编写几行代码，就能对 AD 用户进行身份验证并获取用于调用 Azure AD 图形 API 的令牌。
- 然后，使用该令牌调用图形 API 以查询目录并显示结果
- 利用 ADAL 令牌缓存来最大限度地减少用户身份验证提示。

为此，你需要：

1. 将应用程序注册到 Azure AD
2. 将代码添加到应用以请求令牌
3. 添加代码以使用该令牌查询图形 API 并显示结果。
4. 使用所需的所有目标平台和 Cordova ADAL 插件创建 Cordova 部署项目，并在模拟器中测试解决方案。

## 0\.先决条件
若要完成本教程，你需要：

- 一个 Azure AD 租户，你在其中有一个具有应用开发权限的帐户
- 一个配置为使用 Apache Cordova 的开发环境

如果你已经设置了这两个项目，请直接转到步骤 1。

如果你没有 Azure AD 租户，可以[在此处了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

如果你没有在计算机上设置 Apache Cordova，请安装以下组件：

- [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [NodeJS](https://nodejs.org/download/)
- [Cordova CLI](https://cordova.apache.org/)
  （可以通过 NPM 包管理器轻松安装：`npm install -g cordova`）

请注意，这些组件应可在 PC 和 Mac 上工作。

每个目标平台都有不同的先决条件。

- 生成并运行 Windows Tablet/PC 或 Phone 应用版本
  - [Visual Studio 2013 for Windows with Update 2 或更高版本](http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs#d-express-windows-8)（Express 或其他版本）或 [Visual Studio 2015](https://www.visualstudio.com/downloads/download-visual-studio-vs#d-community)。

- 为 iOS 生成并运行
  
  - Xcode 6.x 或更高版本。在 http://developer.apple.com/downloads 或 [Mac 应用商店](http://itunes.apple.com/cn/app/xcode/id497799835?mt=12)上下载该软件
  - [ios-sim](https://www.npmjs.org/package/ios-sim) - 用于通过命令行在 iOS 模拟器中启动 iOS 应用（可以通过终端轻松安装：`npm install -g ios-sim`）
- 生成并运行适用于 Android 的应用程序
  
  - 安装 [Java 开发工具包 (JDK) 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 或更高版本。请确保根据 JDK 安装路径（例如 C:\\Program Files\\Java\\jdk1.7.0\_75）正确设置 `JAVA_HOME`（环境变量）。
  - 安装 [Android SDK](http://developer.android.com/sdk/installing/index.html?pkg=tools) 并将 `<android-sdk-location>\tools` 位置（例如 C:\\tools\\Android\\android-sdk\\tools）添加到 `PATH` 环境变量。
  - 打开 Android SDK Manager（例如，通过终端：`android`）并安装
    - *Android 5.0.1 (API 21)* 平台 SDK
    - *Android SDK Build-tools* 19.1.0 或更高版本
    - *Android 支持存储库* (Extras)
  
  Android SDK 不提供任何默认模拟器实例。如果你要在模拟器上运行 Android 应用程序，请从终端运行 `android avd`，然后选择“创建...”，以创建一个新实例。建议的“API 级别”为 19 或更高。有关 Android 模拟器和创建选项的详细信息，请参阅 [AVD Manager](http://developer.android.com/tools/help/avd-manager.html)。

## 1\.将一个应用程序注册到 Azure AD
此步骤是可选的。本教程提供了预先设置的值，使你无需在自己的租户中进行任何设置，就能查看示例的运行情况。但是，建议你执行此步骤并熟悉相关的过程，因为在你创建自己的应用程序时，需要执行该过程。

Azure AD 只会向已知的应用程序颁发令牌。在从应用使用 Azure AD 之前，你需要在租户中为该应用创建一个条目。在租户中注册新的应用程序：

-	登录到 [Azure 管理门户](https://manage.windowsazure.cn)
-	在左侧的导航栏中单击“Active Directory”
-	选择你要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	按提示操作，并创建新的**本机客户端应用程序**（尽管 Cordova 应用以 HTML 为基础，我们仍在此创建本机客户端应用程序，因此必须选择`Native Client Application`选项，否则应用程序无法运行）。
    -	应用程序的“名称”向最终用户描述你的应用程序
    -	“重定向 URI”是用于向应用返回令牌的 URI。输入 `http://MyDirectorySearcherApp`。

完成注册后，AAD 将为应用分配唯一的客户端标识符。在学习后面的部分时，你需要用到此值：可以在新建应用的“配置”选项卡中找到此值。

若要运行 `DirSearchClient Sample`，请向新建应用授予查询 *Azure AD 图形 API* 的权限：

在“设置”页上，依次选择“所需的权限”和“添加”。对于“Azure Active Directory”应用程序，选择“Microsoft Graph”作为 API，在“委派的权限”下添加“以已登录用户的身份访问目录”权限。这样，你的应用程序便可以在图形 API 中查询用户。

## 2\.克隆教程所需的示例应用存储库
在 shell 或命令行中键入以下命令：

    git clone -b skeleton https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-Cordova.git

## 3\.创建 Cordova 应用
可通过多种方式创建 Cordova 应用程序。在本教程中，我们将使用 Cordova 命令行界面 (CLI)。
在 shell 或命令行中键入以下命令：

     cordova create DirSearchClient

它将为 Cordova 项目创建文件夹结构和基架。
切换到新的 DirSearchClient 文件夹

    cd .\DirSearchClient
    
并使用文件管理器或 shell 中的以下命令在 www 子文件夹中复制初学者项目的内容：

  - Windows：`xcopy ..\NativeClient-MultiTarget-Cordova\DirSearchClient www /E /Y`
  - Mac：`cp -r  ../NativeClient-MultiTarget-Cordova/DirSearchClient/* www`

然后添加调用图形 API 时所需的允许列表插件。

     cordova plugin add cordova-plugin-whitelist

接下来，添加你要支持的所有平台。为了获得一个有效示例，至少需要执行下列其中一个命令。请注意，你无法在 Windows 上模拟 iOS，或者在 Mac 上模拟 Windows/Windows Phone。

    cordova platform add android
    cordova platform add ios
    cordova platform add windows

最后，可以将 ADAL for Cordova 插件添加到项目。

    cordova plugin add cordova-plugin-ms-adal

## 4\.添加代码以便对用户进行身份验证并从 AAD 获取令牌
你在本教程中开发的应用程序将提供一个精简的目录搜索功能，最终用户可以使用此功能键入目录中任一用户的别名，并可视化一些基本属性。初学者项目包含应用的基本用户界面定义（在 www/index.html 中），以及用于串联基本应用事件周期、用户界面绑定和结果显示逻辑的基架（在 www/js/index.js 中）。你唯一要做的事情就是添加实现标识任务的逻辑。

要做的第一件事是，在代码中引入 AAD 用来识别应用和所需资源的协议值。稍后将要使用这些值来构造令牌请求。在 index.js 文件的顶部插入以下代码段。

javascript

    var authority = "https://login.chinacloudapi.cn/common",
    redirectUri = "http://MyDirectorySearcherApp",
    resourceUri = "https://graph.chinacloudapi.cn",
    clientId = "a5d92493-ae5a-4a9f-bcbf-9f1d354067d3",
    graphApiVersion = "2013-11-08";


`redirectUri` 和 `clientId` 值应与 AAD 中用于描述应用程序的值匹配。如本教程前面的步骤 1 中所述，你可以在 Azure 门户预览的“配置”选项卡中找到这些值。
注意：如果你选择不在自己的租户中注册新应用，则只需按原样粘贴上述预配置值 - 这样你便可以看到示例的运行情况，不过，在生产环境中，你始终要为应用创建自己的条目。

接下来，需要添加实际的令牌请求代码。在 `search `和 `renderdata `定义之间插入以下代码段。

javascript

    // Shows user authentication dialog if required.
    authenticate: function (authCompletedCallback) {

        app.context = new Microsoft.ADAL.AuthenticationContext(authority);
        app.context.tokenCache.readItems().then(function (items) {
            if (items.length > 0) {
                authority = items[0].authority;
                app.context = new Microsoft.ADAL.AuthenticationContext(authority);
            }
            // Attempt to authorize user silently
            app.context.acquireTokenSilentAsync(resourceUri, clientId)
            .then(authCompletedCallback, function () {
                // We require user credentials so triggers authentication dialog
                app.context.acquireTokenAsync(resourceUri, clientId, redirectUri)
                .then(authCompletedCallback, function (err) {
                    app.error("Failed to authenticate: " + err);
                });
            });
        });

    },

让我们通过将它分解成两个主要部分来检查运行情况。
此示例应适用于任何租户，而不只是与某个特定租户相关。它使用了“/common”终结点，因此，用户在身份验证时可以输入任何帐户，并可将请求定向到它所属的租户。
该方法的第一部分将检查 ADAL 缓存，以确定是否已有一个存储的令牌 - 如果有，则使用该令牌的来源租户重新初始化 ADAL。要避免额外的提示，必须执行此操作，因为使用“/common”总会导致要求用户输入新帐户。

javascript

        app.context = new Microsoft.ADAL.AuthenticationContext(authority);
        app.context.tokenCache.readItems().then(function (items) {
            if (items.length > 0) {
                authority = items[0].authority;
                app.context = new Microsoft.ADAL.AuthenticationContext(authority);
            }

该方法的第二部分将执行适当的 tokewn 请求。
`acquireTokenSilentAsync` 方法请求 ADAL 返回指定资源的令牌，且不显示任何 UX。如果缓存中已经存储了一个适当的访问令牌，或者有一个刷新令牌可用于获取新访问令牌且不显示任何提示，则可能会发生这种情况。
如果该尝试失败，我们将在 `acquireTokenAsync` 上回退 - 这会以可视方式提示用户进行身份验证。

javascript

            // Attempt to authorize user silently
            app.context.acquireTokenSilentAsync(resourceUri, clientId)
            .then(authCompletedCallback, function () {
                // We require user credentials so triggers authentication dialog
                app.context.acquireTokenAsync(resourceUri, clientId, redirectUri)
                .then(authCompletedCallback, function (err) {
                    app.error("Failed to authenticate: " + err);
                });
            });

现在，我们已经获得了令牌，最后，我们可以调用图形 API 并执行所需的搜索查询。在 `authenticate` 定义的正下方插入以下代码段。

javascript

	// Makes Api call to receive user list.
		requestData: function (authResult, searchText) {
			var req = new XMLHttpRequest();
			var url = resourceUri + "/" + authResult.tenantId + "/users?api-version=" + graphApiVersion;
			url = searchText ? url + "&$filter=mailNickname eq '" + searchText + "'" : url + "&$top=10";

			req.open("GET", url, true);
			req.setRequestHeader('Authorization', 'Bearer ' + authResult.accessToken);

			req.onload = function(e) {
				if (e.target.status >= 200 && e.target.status < 300) {
					app.renderData(JSON.parse(e.target.response));
					return;
				}
				app.error('Data request failed: ' + e.target.response);
			};
			req.onerror = function(e) {
				app.error('Data request failed: ' + e.error);
			}

			req.send();
		},


起点文件提供了一个精简 UX，用于在文本框中输入用户的别名。此方法使用该值来构造查询，将该查询与访问令牌相结合，然后将其发送到 Graph 并分析结果。起点文件中已提供了一个负责可视化结果的 renderData 方法。

## 5\.运行
你的应用终于准备好运行了！ 操作非常简单：启动应用后，在文本框中输入要查找的用户的别名，然后单击按钮。系统将提示你进行身份验证。身份验证和搜索成功后，将显示搜索到的用户的属性。后续运行将执行搜索而不显示任何提示，这是因为缓存中已经存储了以前获取的令牌。
运行应用的具体步骤因平台而异。

#### Windows 10：
   平板电脑/电脑：`cordova run windows --archs=x64 -- --appx=uap`

   移动版（需要连接到电脑的 Windows10 移动版设备）：`cordova run windows --archs=arm -- --appx=uap --phone`

   **注意**：在首次运行期间，系统可能会要求你登录以获得开发人员许可证。有关详细信息，请参阅[开发人员许可证](https://msdn.microsoft.com/zh-cn/library/windows/apps/hh974578.aspx)。

#### Windows 8.1 平板电脑/电脑：
   `cordova run windows`

   **注意**：在首次运行期间，系统可能会要求你登录以获得开发人员许可证。有关详细信息，请参阅[开发人员许可证](https://msdn.microsoft.com/zh-cn/library/windows/apps/hh974578.aspx)。

#### Windows Phone 8.1：
   在连接的设备上运行：`cordova run windows --device -- --phone`

   在默认的模拟器上运行：`cordova emulate windows -- --phone`

   使用 `cordova run windows --list -- --phone` 可查看所有可用目标，使用 `cordova run windows --target=<target_name> -- --phone` 可在特定的设备或模拟器上运行应用程序（例如 `cordova run windows --target="Emulator 8.1 720P 4.7 inch" -- --phone`）。

#### Android：
   在连接的设备上运行：`cordova run android --device`

   在默认的模拟器上运行：`cordova emulate android`

   **注意**：请确保根据“先决条件”部分中所示，使用 *AVD Manager* 创建模拟器实例。

   使用 `cordova run android --list` 可查看所有可用目标，使用 `cordova run android --target=<target_name>` 可在特定的设备或模拟器上运行应用程序（例如 `cordova run android --target="Nexus4_emulator"`）。

#### iOS：
   在连接的设备上运行：`cordova run ios --device`

   在默认的模拟器上运行：`cordova emulate ios`

   **注意**：请确保安装 `ios-sim` 包以在模拟器上运行。有关详细信息，请参阅“先决条件”部分。

   使用 `cordova run ios --list` 可查看所有可用目标，使用 `cordova run ios --target=<target_name>` 可在特定的设备或模拟器上运行应用程序（例如 `cordova run android --target="iPhone-6"`）。

使用 `cordova run --help` 可查看其他生成和运行选项。

[此处](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-Cordova/tree/complete/DirSearchClient)提供了已完成示例（无需配置值）供你参考。现在，你可以转到更高级（也更有趣）的方案。你可能想要尝试：

[使用 Azure AD 保护 Node.js Web API>>](/documentation/articles/active-directory-devquickstarts-webapi-nodejs/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->