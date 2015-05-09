<properties
pageTitle="如何通过 Azure AD 构建单页面应用程序"
description="演示如何使用适用于 Javascript 的 Active Directory 身份验证库 (ADAL) 保护基于 AngularJS 且实施了 ASP.NET Web API 后端的单页面应用程序，该后端使用 CORS 调用另一个ASP.NET Web API。"
services=""
documentationCenter=""
authors="Justinha"
manager="terrylan"
editor="LisaToft"/>
	
<tags ms.service="AD" ms.date="02/20/2015" wacn.date="04/11/2015"/>
    
# 如何通过 Azure AD 构建单页面应用程序 

本教程演示了如何使用适用于 Javascript 的 Active Directory 身份验证库 (ADAL) 保护基于 AngularJS 且实施了 ASP.NET Web API 后端的单页面应用程序，该后端使用 CORS 调用另一个 ASP.NET Web API。若要查看本教程中使用的代码示例，请参阅 github 上的 [AzureADSamples/SinglePageApp-AngularJS-DotNet](https://github.com/AzureADSamples/SinglePageApp-AngularJS-DotNet)。

适用于 Javascript 的 ADAL 是一个开源库。有关分发选项、源代码和贡献方面的信息，请签出 [ADAL JS 存储库](https://github.com/AzureAD/azure-activedirectory-library-for-js)。

有关协议在此方案和其他方案中如何工作的更多信息，请参阅[针对 Azure AD 的身份验证方案](https://msdn.microsoft.com/zh-CN/library/azure/dn499820.aspx)。

## 如何运行此示例

入门很简单！若要运行此示例，你将需要：

- Visual Studio 2013
- Internet 连接
- Azure 订阅（[1rmb 试用版](/pricing/1rmb-trial/) 足够了）

每个 Azure 订阅都有一个关联的 Azure Active Directory (Azure AD) 租户。此示例使用的所有 Azure AD 功能都可免费使用。

## 克隆或下载此存储库

从 shell 或命令行执行以下操作：
`git clone https://github.com/AzureADSamples/SinglePageApp-WebAPI-AngularJS-DotNet.git`

## 向你的 Azure Active Directory 租户注册"待去地点 API"服务

1. 登录到 [Azure 管理门户](http://go.microsoft.com/fwlink/?LinkId=301902).
2. 在左侧的导航栏中单击"Active Directory"。
3. 单击你希望在其中注册示例应用程序的目录。
4. 单击"应用程序"选项卡。
5. 在抽屉中，单击"添加"。
6. 单击"添加我的组织正在开发的应用程序"。
7. 为应用程序输入一个友好的名称，例如"待去地点 API"，选择"Web 应用程序"和/或"Web API"，然后单击"下一步"。
8. 对于登录 URL，请输入示例的基 URL，默认情况下为 `https://localhost:44327/`。
9. 对于应用程序 ID URI，请输入  `https://<your_directory_name>/ToGoAPI`，并将 `<your_directory_name>` 替换为你的 Azure AD 目录的名称。保存配置。

所有工作都已完成！在前进到下一步骤之前，你需要找到你的 API 的应用程序 ID URI。

1. 仍然在 Azure 门户中，单击你的应用程序的"配置"选项卡。
2. 找到应用程序 ID URI 值并将其复制到剪贴板。

## 配置"待去地点 API"以使用你的 Azure Active Directory 租户

1. 在 Visual Studio 2013 中打开解决方案。
2. 在 ToGoAPI 项目中，打开  `web.config` 文件。
3. 找到应用程序键  `ida:Tenant` 并将该值替换为你的 Azure AD 租户名称。
4. 找到应用程序键  `ida:Audience` 并将该值替换为你从 Azure 门户复制的应用程序 ID URI。
5. 还是在 ToGoAPI 项目中，打开文件  `Controllers/ToGoListController.cs`。在 `[EnableCors...]` 属性中，输入"待办事项 SPA"客户端的位置。默认情况下，它是 `https://localhost:44326`。请务必省略尾部反斜杠。
5. 在 TodoSPA 项目中，打开文件  `App/Scripts/App.js` 并找到  `endpoints` 对象的声明。
6. 输入"待去地点 API"终结点位置到其资源标识符或应用程序 ID URI 的映射。 `endpoints` 对象的该属性名称应当是"待去地点 API"的位置。默认情况下，它是  `https://localhost:44327/`。此属性的值应当是你从门户复制的应用程序 ID URI，例如 `https://<your_tenant_name>/ToGoAPI`。
8. 先不要担心此文件中的其他配置值，我们一会还会回来。
9. 还是在 TodoSPA 项目中，打开文件  `App/Scripts/toGoListSvc.js`。将  `apiEndpoint` 变量的值替换为你的"待去地点 API"的位置。默认情况下，它是 `https://localhost:44327/`。

## 向你的 Azure Active Directory 租户注册"待办事项"单页面应用程序。

1. 再次登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中单击"Active Directory"。
3. 单击你要在其中注册示例应用程序的目录租户。
4. 单击"应用程序"选项卡。
5. 在抽屉中，单击"添加"。
6. 单击"添加我的组织正在开发的应用程序"。
7. 为应用程序输入一个友好的名称，例如"待办事项 SPA"，选择"Web 应用程序"和/或"Web API"，然后单击"下一步"。
8. 对于登录 URL，请输入示例的基 URL，默认情况下为 `https://localhost:44326/`。
9. 对于应用程序 ID URI，请输入  `https://<your_directory_name>/ToDoSPA`，并将 `<your_directory_name>` 替换为你的 Azure AD 目录的名称。
10. 在"针对其他应用程序的权限"部分中，单击"添加应用程序"。在"显示"下拉列表中选择"其他"，然后单击上方的复选标记。找到并单击"待去地点 API"，然后单击底部的复选标记以添加该应用程序。从"委托的权限"下拉列表中选择"访问待去地点 API"，然后保存配置。

所有工作都已完成！在前进到下一步骤之前，你需要找到你的应用程序的客户端 ID。

1. 仍然在 Azure 门户中，单击你的应用程序的"配置"选项卡。
2. 找到客户端 ID 值并将其复制到剪贴板。


## 为你的应用程序启用 OAuth2 隐式授权

默认情况下不允许在 Azure AD 中设置的应用程序使用 OAuth2 隐式授权。为了运行此示例，你需要明确进行选择。

1. 在运行前面的步骤后，你的浏览器应当仍然处于 Azure 管理门户中 - 并且具体而言，显示的是你的应用程序条目的"配置"选项卡。
2. 使用抽屉中的"管理清单"按钮，下载应用程序的清单文件并将其保存到磁盘。
3. 使用文本编辑器打开清单文件。搜索  `oauth2AllowImplicitFlow` 属性。你将发现它设置为  `false`；请将其更改为  `true` 并保存文件。
4. 使用"管理清单"按钮，上载更新的清单文件。保存应用程序的配置。

## 配置"待办事项 SPA"以使用你的 Azure Active Directory 租户。

1. 在 Visual Studio 2013 中打开解决方案。
2. 在 TodoSPA 项目中，打开  `web.config` 文件。
3. 找到应用程序键  `ida:Tenant` 并将该值替换为你的 Azure AD 目录名称。
4. 找到应用程序键  `ida:Audience` 并将该值替换为来自 Azure 门户的客户端 ID。
5. 还是在 TodoSPA 项目中，再次打开文件  `App/Scripts/App.js` 并找到行 `adalAuthenticationServiceProvider.init(`。
6. 将  `tenant` 的值替换为你的 Azure AD 目录名。
7. 将  `clientId` 的值替换为来自 Azure 门户的客户端 ID。

## 运行示例

清除解决方案，重新生成解决方案并运行它。 

你可以通过单击右上角的登录链接或通过直接在"待办事项列表"或"待去地点列表"选项卡上单击来触发登录体验。通过执行以下步骤来探究示例：登录、向"待办事项列表"添加项、删除用户帐户并重新启动。向"待去地点列表"添加地点，使用 CORS 对"待去地点 API"执行 CRUD 操作。

## 如何将此示例部署到 Azure

若要将"待办事项 SPA"和"待去地点 API"部署到 Azure 网站，你将创建两个网站，将每个项目发布到其中一个网站，并更新两个项目来引用新位置而非 IIS Express。

### 创建"待去地点 API"Azure 网站

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn).
2. 在左侧导航栏中单击"网站"。
3. 单击左下角的"新建"，选择"计算">"网站">"自定义创建"，选择宿主计划和区域，为你的网站指定一个名称，例如 togo-contoso.chinacloudsites.cn。选择要使用的数据库，或者创建一个新数据库。单击"创建网站"。
4. 在创建网站后，单击它进行管理。对于此组步骤，请下载 .publishsettings 文件并保存。还可以使用其他部署机制，例如从源控件。有关使用 .publishsettings 文件的更多信息，请参阅[如何：连接到订阅](/documentation/articles/install-configure-powershell/#Connect)。 

### 创建"待办事项 SPA"Azure 网站

1. 导航到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧导航栏中单击"网站"。
3. 单击左下角的"新建"，选择"计算">"网站">"自定义创建"，选择宿主计划和区域，为你的网站指定一个名称，例如 todo-contoso.chinacloudsites.cn。   选择要使用的数据库，与"待去地点 API"使用同一数据库即可。单击"创建网站"。
4. 在创建网站后，单击它进行管理。同样，下载此网站的 .publishsettings 文件并保存。

### 更新两个项目以使用 Azure 网站

1. 在 Visual Studio 中，转到 TodoSPA 项目。
2. 需要执行两处更改。在  `App\Scripts\app.js` 中，将  `endpoints` 对象的属性名称替换为你的"待去地点 API"，例如  `https://togo-contoso.chinacloudsites.cn/`.  In `App\Scripts\toGoListSvc.js`，将  `apiEndpoint` 变量替换为同一值。
3. 在 ToGoAPI 项目中，只需要执行一处更改。在  `Controllers\ToGoListController.cs` 中，更新 `[EnableCors...]` 属性以反映"待办事项 SPA"的新位置，例如  `https://todo-contoso.chinacloudsites.cn`。同样，请务必省略尾部反斜杠。

### 将"待去地点 API"发布到 Azure 网站

1. 切换到 Visual Studio 并转到 ToGoAPI 项目。在解决方案资源管理器中右键单击该项目并选择**发布**。单击"导入"，并导入你下载的"待去地点 API"发布配置文件。
6. 在"连接"选项卡上，更新"目标 URL"，使其采用 https 格式，例如 https://togo-constoso.chinacloudsites.cn。 单击"下一步"。
7. 在"设置"选项卡上，确保未选中"启用组织身份验证"。单击"发布"。
8. Visual Studio 将发布该项目并自动在浏览器中打开该项目的 URL。如果你看到了该项目的默认网页，则说明发布成功。

### 将"待办事项 SPA"发布到 Azure 网站

1. 切换到 Visual Studio 并转到 TodoSPA 项目。在解决方案资源管理器中右键单击该项目并选择**发布**。单击"导入"并导入你下载的"待办事项 SPA".publishsettings 文件。
6. 在"连接"选项卡上，更新"目标 URL"，使其采用 https 格式，例如 https://todo-contoso.chinacloudsites.cn。 单击"下一步"。
7. 在"设置"选项卡上，确保未选中"启用组织身份验证"。单击"发布"。
8. Visual Studio 将发布该项目并自动在浏览器中打开该项目的 URL。如果你看到了该项目的默认网页，则说明发布成功。

### 在目录租户中更新"待办事项 SPA"配置

1. 导航到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中，单击"Active Directory"并选择你的租户。
3. 在"应用程序"选项卡上，选择"待办事项 SPA"应用程序。
4. 在"配置"选项卡上，将"登录 URL"和"答复 URL"字段更新为你的 SPA 的地址，例如 https://todo-contoso.chinacloudsites.cn。 保存配置。

## 关于代码

包含身份验证逻辑的主要文件如下：

- **App.js** - 注入 ADAL 模块依赖项、提供由 ADAL 使用的用于驱动与 Azure AD 进行协议交互的应用程序配置值，并且指定未通过身份验证不应访问的路由。

- **index.html** - 包含对 adal.js 的引用

- **HomeController.js**- 显示了如何利用 ADAL 中的 login() 和 logOut() 方法。

- **UserDataController.js** - 显示了如何从缓存的 id_token 提取用户信息。
   
特别感谢在创建本教程期间提供帮助的 @matvelloso。


## 后续步骤

下面是在使用 Azure AD 向你的 web 应用程序和 web API 添加身份验证和授权方面可以为你提供帮助的一些其他资源： 

+ [Azure Active Directory 代码示例](https://msdn.microsoft.com/zh-CN/library/azure/dn646737.aspx)
+ [Azure AD 的身份验证方案](https://msdn.microsoft.com/zh-CN/library/azure/dn499820.aspx)




<!--HONumber=51-->