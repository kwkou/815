<properties
	pageTitle="Azure AD iOS 入门 | Windows Azure"
	description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 iOS 应用程序。"
	services="active-directory"
	documentationCenter="ios"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/28/2015"
	wacn.date="06/16/2015"/>

# 将 Azure AD 集成到 iOS 应用程序中

[AZURE.INCLUDE [active-directory-devguide](../includes/active-directory-devguide.md)]

如果你要开发桌面应用程序，Azure AD 可让你简单直接地使用用户的 Active Directory 帐户对其进行身份验证。它还可以让应用程序安全地使用 Azure AD 保护的任何 Web API，例如 Office 365 API 或 Azure API。

对于需要访问受保护资源的 iOS 客户端，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用程序获取访问令牌。为了演示这种简便性，我们生成了一个 Objective C 待办事项列表应用程序，其中包括：

-	使用 [OAuth 2.0 身份验证协议](https://msdn.microsoft.com/zh-cn/library/azure/dn645545.aspx)获取调用 Azure AD Graph API 的访问令牌。
-	在目录中搜索具有给定别名的用户。
-	将用户注销。

若要生成完整的工作应用程序，你需要：

2. 将应用程序注册到 Azure AD。
3. 安装并配置 ADAL。
5. 使用 ADAL 从 Azure AD 获取令牌。

若要开始，请[下载应用程序框架](https://github.com/AzureADQuickStarts/NativeClient-iOS/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/NativeClient-iOS/archive/complete.zip)。你还需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant)。

## 步骤 1：下载并运行 .Net 或 Node.js REST API TODO 示例服务器

专门编写这个示例是为了与用于生成 Windows Azure Active Directory 的单租户 To-Do REST API 的现有示例配合工作。这是本快速入门教程的先决条件。

有关如何设置的信息，请访问我们的现有示例：

* [适用于 Node.js 的 Windows Azure Active Directory 示例 REST API 服务](/documentation/articles/active-directory-devquickstarts-webapi-nodejs)

## 步骤 2：向 Windows Azure AD 租户注册 Web API

**我正在执行什么操作？**

*Microsoft Active Directory 支持添加两种类型的应用程序。Web API，向访问这些 Web API 的用户和应用程序（在 Web 上或者设备中运行的应用程序上）提供服务。在此步骤中，你将注册你在本地运行的用于测试此示例的 Web API。通常，此 Web API 是一个 REST 服务，它提供应用程序需要访问的功能。Windows Azure Active Directory 可以保护任何终结点！*

*此处我们假设你要注册上面引用的 TODO REST API，但这也适用于你希望 Azure Active Directory 保护的任何 Web API。*

在 Windows Azure AD 中注册 Web API 的步骤

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中单击“Active Directory”。
3. 单击你要在其中注册示例应用程序的目录租户。
4. 单击“应用程序”选项卡。
5. 在抽屉中，单击“添加”。
6. 单击“添加我的组织正在开发的应用程序”。
7. 为应用程序输入一个友好的名称，例如“TodoListService”，选择“Web 应用程序和/或 Web API”，然后单击“下一步”。
8. 对于登录 URL，请输入示例的基 URL，默认情况下为 `https://localhost:8080`。
9. 对于应用程序 ID URI，请输入 `https://<your_tenant_name>/TodoListService`，并将 `<your_tenant_name>` 替换为你的 Azure AD 租户的名称。单击“确定”完成注册。
10. 仍然在 Azure 门户中，单击你的应用程序的“配置”选项卡。
11. **查找“客户端 ID”值并将它复制到某个位置**，稍后配置应用程序时需要用到它。

## 步骤 3：注册示例 iOS 本机客户端应用程序

注册 Web 应用程序是第一步。接下来，你还需要告诉 Azure Active Directory 有关应用程序的情况。这样，应用程序便能够与刚刚注册的 Web API 通信

**我正在执行什么操作？**

*如前所述，Windows Azure Active Directory 支持添加两种类型的应用程序。Web API，向访问这些 Web API 的用户和应用程序（在 Web 上或者设备中运行的应用程序上）提供服务。在此步骤中，你要将应用程序注册到此示例。只有执行了此操作，此应用程序才能请求访问你刚刚注册的 Web API。除非注册了应用程序，否则 Azure Active Directory 甚至可能会拒绝应用程序请求登录！ 这是模型安全功能的一部分。*

*此处我们假设你要注册上面引用的这个示例应用程序，但这也适用于你正在开发的任何应用程序。*

**为什么要将应用程序和 Web API 放在一个租户中？**

*如你可能猜到的那样，你可以生成一个从其他租户访问 Azure Active Directory 中注册的外部 API 的应用程序。如果你这样做，系统将提示你的客户许可使用该应用程序中的 API。令人欣慰的是，适用于 iOS 的 Active Directory 身份验证库将负责为你处理此许可！ 在了解更高级的功能后，你将发现，这是从 Azure 和 Office 以及任何其他服务提供程序访问 Microsoft API 套件所需工作的重要部分。现在，由于你已将 Web API 和应用程序注册到同一个租户下，因此，你不会看到任何许可提示。如果你只是在为自己的公司开发要使用的应用程序，则通常就是这种情况。*

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 在左侧的导航栏中单击“Active Directory”。
3. 单击你要在其中注册示例应用程序的目录租户。
4. 单击“应用程序”选项卡。
5. 在抽屉中，单击“添加”。
6. 单击“添加我的组织正在开发的应用程序”。
7. 为应用程序输入一个友好的名称，例如“TodoListClient-iOS”，选择“本机客户端应用程序”，然后单击“下一步”。
8. 对于“重定向 URI”，请输入 `http://TodoListClient`。单击“完成”。
9. 单击应用程序的“配置”选项卡。
10. 查找“客户端 ID”值并将它复制到某个位置，稍后配置应用程序时需要用到它。
11. 在“针对其他应用程序的权限”中，单击“添加应用程序”。 在“显示”下拉列表中选择“其他”，然后单击上方的复选标记。找到并单击“TodoListService”，然后单击底部的复选标记以添加该应用程序。从“委托的权限”下拉列表中选择“访问 TodoListService”，然后保存配置。


## 步骤 4：下载 iOS 本机客户端示例代码

* `$ git clone git@github.com:AzureADSamples/NativeClient-iOS.git`

## 步骤 5：下载 ADAL for iOS 并将其添加到 XCode 工作区

#### 下载 ADAL for iOS

* `git clone git@github.com:MSOpenTech/azure-activedirectory-library-for-ios.git`

#### 将库导入工作区

在 XCode 中，右键单击你的项目目录，然后选择“将文件添加到 iOS 示例”...

出现提示时，选择 ADAL for iOS 所克隆到的目录

#### 将 libADALiOS.a 添加到链接的框架和库

单击“链接的框架和库”下面的添加按钮，然后从导入的框架添加库文件。

生成项目，以确保正确编译所有内容。


## 步骤 6：使用 Web API 信息配置 settings.plist 文件

在“支持文件”下，可以看到 settings.plist 文件。该文件包含以下信息：

	XML
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
		<key>authority</key>
		<string>https://login.windows.net/common/oauth2/token</string>
		<key>clientId</key>
		<string>xxxxxxx-xxxxxx-xxxxxxx-xxxxxxx</string>
		<key>resourceString</key>
		<string>https://localhost/todolistservice</string>
		<key>redirectUri</key>
		<string>http://demo_todolist_app</string>
		<key>userId</key>
		<string>user@domain.com</string>
		<key>taskWebAPI</key>
		<string>https://localhost/api/todolist/</string>
	</dict>
	</plist>


将 plist 文件中的信息替换你的 Web API 设置。

##### 说明

当前设置的默认值可以配合[适用于 Node.js 的 Azure Active Directory 示例 REST API 服务](https://github.com/AzureADSamples/WebAPI-Nodejs)。但是，你需要指定 Web API 的 clientID。如果你正在运行自己的 API，则需要根据情况更新终结点。

## 步骤 7：生成并运行应用程序。

你应能够连接到 REST API 终结点，系统会提示输入 Azure Active Directory 帐户的凭据。

有关更多资源，请参阅：
- [GitHub 上的 AzureADSamples >](https://github.com/AzureAdSamples)
- [www.windowsazure.cn >](/documentation/services/identity/)上的 Azure AD 文档 

<!---HONumber=60-->