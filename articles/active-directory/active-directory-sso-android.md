<properties
    pageTitle="如何使用 ADAL 在 Android 上启用跨应用 SSO | Azure"
    description="如何使用 ADAL SDK 的功能跨应用程序启用单一登录。"
    services="active-directory"
    documentationcenter=""
    author="xerners"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="40710225-05ab-40a3-9aec-8b4e96b6b5e7"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="android"
    ms.devlang="java"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="brandwe" />  


# 如何使用 ADAL 在 Android 上启用跨应用 SSO
客户现在都希望提供单一登录 (SSO)，以便用户只需输入一次凭据，并使这些凭据能够自动跨应用程序工作。要用户在一个小屏幕上输入用户名和密码，这本身就是一项很难的操作，通常还伴有其他身份验证方式 (2FA)，例如电话呼叫或短信代码，如果用户必须针对你的产品多次执行这些操作，将会很快导致不满。

此外，如果你利用其他应用程序也可能使用的标识平台（例如 Microsoft 帐户或 Office365 工作帐户），客户会希望不管供应商是谁，都可以在所有应用程序中使用这些凭据。

Microsoft 标识平台以及 Microsoft 标识 SDK 能够为你完成所有这些繁琐的工作，并让你能够在自己的应用程序套件中使用 SSO 以让客户感到满意，或者使用中转站功能和验证器应用程序在整个设备上提供良好的用户体验。

本演练将介绍如何在应用程序中配置 SDK，以便向客户提供此项优点。



请注意，以下文档假设你已了解如何[在旧版门户中为 Azure Active Directory 预配应用程序](/documentation/articles/active-directory-how-to-integrate/)，并且已将应用程序与 [Microsoft Identity Android SDK](https://github.com/AzureAD/azure-activedirectory-library-for-android) 集成。

## Microsoft 标识平台中的 SSO 概念
### Microsoft 标识中转站
Microsoft 为每个移动平台提供了应用程序，可在来自不同供应商的应用程序之间桥接凭据，还支持需要一个安全位置来验证凭据的特殊增强功能。我们称之为**中转站**。在 iOS 和 Android 平台上，它们通过可下载的应用程序提供，客户可以独立安装这些应用程序，也可以由负责管理其员工的部分或全部设备的公司推送到设备。根据 IT 管理员的需要，这些中转站支持管理部分应用程序或整个设备的安全性。在 Windows 环境中，此功能由内置于操作系统的帐户选择器提供，在技术上被称为 Web 身份验证中转站。

为了理解我们如何使用这些中转站以及它们如何在 Microsoft 标识平台的登录流中向客户呈现，请继续阅读以了解详细信息。

### 移动设备上的登录模式
在设备上访问凭据遵循 Microsoft 标识平台的两种基本模式：

- 无中转站辅助的登录
- 中转站辅助的登录

#### 无中转站辅助的登录
无中转站辅助的登录是一种与应用程序内联的登录体验，使用设备上针对该应用程序的本地存储。此存储可以跨应用程序共享，但凭据紧密绑定到使用该凭据的应用或应用套件。这是最有可能在许多移动应用程序中遇到的体验，其中你只在应用程序中输入用户名和密码。

这种登录具有以下优点：

- 用户体验完全存在于应用程序中。
- 凭据可在由同一证书签名的应用程序间共享，从而为应用程序套件提供单一登录体验。
- 关于登录体验的控件在登录之前和之后提供给应用程序。

这种登录具有以下缺点：

- 用户不能在使用 Microsoft 标识的所有应用中体验单一登录，而只能在使用应用程序拥有并且已配置的 Microsoft 标识的应用中体验单一登录。
- 应用程序不可以使用更高级的业务功能，如条件访问，或使用 InTune 产品套件。
- 应用程序不能支持业务用户使用基于证书的身份验证。

下面介绍了 Microsoft 标识 SDK 如何与应用程序的共享存储配合工作以启用 SSO：


	+------------+ +------------+  +-------------+
	|            | |            |  |             |
	|   App 1    | |   App 2    |  |   App 3     |
	|            | |            |  |             |
	|            | |            |  |             |
	+------------+ +------------+  +-------------+
	| Azure SDK  | | Azure SDK  |  | Azure SDK   |
	+------------+-+------------+--+-------------+
	|                                            |
	|            App Shared Storage              |
	+--------------------------------------------+


#### 中转站辅助的登录
中转站辅助的登录是一种发生在中转站应用程序中的登录体验，使用中转站的存储和安全性在设备上利用 Microsoft 身份平台的所有应用程序间共享凭据。这意味着，应用程序将依赖于中转站才能将用户登录。在 iOS 和 Android 上，它们通过可下载的应用程序提供，客户可以独立安装这些应用程序，也可以由负责为其用户管理设备的公司推送到设备。这种类型的应用程序有 iOS 上的 Azure 验证器应用程序。在 Windows 环境中，此功能由内置于操作系统的帐户选择器提供，在技术上被称为 Web 身份验证中转站。具体的体验因平台而异，如果未正确管理，有时会给用户带来麻烦。如果你已安装 Facebook 应用程序并在另一个应用程序中使用 Facebook 登录功能，可能很熟悉这种模式。Microsoft 标识平台也利用相同的模式。

对于 iOS，这会导致“转换”动画，其中，应用程序将发送到后台，而 Azure 验证器应用程序将发送到前台，让用户选择要用来登录的帐户。

对于 Android 和 Windows，帐户选择器将会显示在应用程序顶部，这给用户带来的麻烦较少。

#### 如何调用中转站
如果在设备上安装了 Azure 验证器应用程序等兼容的中转站，当用户指出希望在 Microsoft 标识平台上使用任何帐户登录时，Microsoft 标识 SDK 将自动为你调用中转站。这可以是个人 Microsoft 帐户、工作或学校帐户。通过使用非常安全的算法和加密，我们可确保以安全的方式要求用户提供凭据并将其发送回应用程序。除了由 Apple 和 Google 协作开发外，关于这些机制的确切技术细节并未发布。

**开发人员可以选择 Microsoft 标识 SDK 是调用中转站，还是使用无中转站辅助的流。** 但是，如果开发人员选择不使用中转站辅助的流，他们将会失去利用用户可能已添加到设备的 SSO 凭据的优势，并阻止其应用程序使用 Microsoft 为客户提供的业务功能，例如条件访问、Intune 管理功能和基于证书的身份验证。

这种登录具有以下优点：

- 无论供应商是谁，用户都可以在所有应用程序中体验 SSO。
- 应用程序可以利用更高级的业务功能，如条件访问，或使用 InTune 产品套件。
- 应用程序可以支持业务用户使用基于证书的身份验证。
- 登录体验要安全得多，因为中转站应用程序将使用附加的安全算法和加密来验证应用程序与用户的标识。

这种登录具有以下缺点：

- 在 iOS 中，当用户选择凭据时，会被转换到应用程序的体验之外。
- 无法管理客户在应用程序中的登录体验。

下面介绍了 Microsoft 标识 SDK 如何与中转站应用程序配合工作以启用 SSO：

	
	+------------+ +------------+   +-------------+
	|            | |            |   |             |
	|   App 1    | |   App 2    |   |   Someone   |
	|            | |            |   |    Else's   |
	|            | |            |   |     App     |
	+------------+ +------------+   +-------------+
	| Azure SDK  | | Azure SDK  |   | Azure SDK   |
	+-----+------+-+-----+------+-  +-------+-----+
	      |              |                  |
	      |       +------v------+           |
	      |       |             |           |
	      |       | Microsoft   |           |
	      +-------> Broker      |^----------+
	              | Application
	              |             |
	              +-------------+
	              |             |
	              |   Broker    |
	              |   Storage   |
	              |             |
	              +-------------+
	

了解这些背景信息后，你应该可以更好地理解 SSO 并使用 Microsoft 标识平台和 SDK 在应用程序中实现 SSO。

## 使用 ADAL 启用跨应用 SSO
这里，我们将使用 ADAL Android SDK 来执行以下操作：

- 为应用套件启用无中转站辅助的 SSO
- 启用对中转站辅助 SSO 的支持

### 针对无中转站辅助 SSO 启用 SSO
对于跨应用程序的无中转站辅助 SSO，Microsoft 标识 SDK 可帮你管理大部分复杂的 SSO。这包括在缓存中查找适当的用户，以及维护已登录用户列表以供你查询。

若要跨拥有的应用程序启用 SSO，需要执行以下操作：

1. 确保所有应用程序都使用相同的客户端 ID 或应用程序 ID。
2. 确保所有应用程序都具有相同的 SharedUserID 集。
3. 确保所有应用程序都共享来自 Google Play Store 的同一签名证书，以便可以共享存储。

#### 步骤 1：对应用套件中的所有应用程序使用相同的客户端 ID/应用程序 ID
为了让 Microsoft 标识平台知道可以跨应用程序共享令牌，每个应用程序都需要共享同一个客户端 ID 或应用程序 ID。这是在门户中注册第一个应用程序时为你提供的唯一标识符。

如果应用使用相同的应用程序 ID，你可能想要知道如何在 Microsoft 标识服务中标识不同的应用。答案是使用“重定向 URI”。每个应用程序都可以在登记门户中注册多个重定向 URI。套件中的每个应用都具有不同的重定向 URI。下面显示了这种情况的示例：

App1 重定向 URI：`msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D`

App2 重定向 URI：`msauth://com.example.userapp1/KmB7PxIytyLkbGHuI%2UitkW%2Fejk%4E`

App3 重定向 URI：`msauth://com.example.userapp2/Pt85PxIyvbLkbKUtBI%2SitkW%2Fejk%9F`

....

这些应用嵌套在同一个客户端 ID/应用程序 ID 下，可以根据你在 SDK 配置中返回给我们的重定向 URI 来查找。


	+-------------------+
	|                   |
	|  Client ID        |
	+---------+---------+
	          |
	          |           +-----------------------------------+
	          |           |  App 1 Redirect URI               |
	          +----------^+                                   |
	          |           +-----------------------------------+
	          |
	          |           +-----------------------------------+
	          +----------^+  App 2 Redirect URI               |
	          |           |                                   |
	          |           +-----------------------------------+
	          |
	          +----------^+-----------------------------------+
	                      |  App 3 Redirect URI               |
	                      |                                   |
	                      +-----------------------------------+
	
	


*请注意，下面介绍了这些重定向 URI 的格式。你可以使用任何重定向 URI，除非你想要支持中转站，在这种情况下，它们必须如上所示*

#### 步骤 2：在 Android 中配置共享存储
设置 `SharedUserID` 超出了本文档的范围，但可以阅读有关[清单](http://developer.android.com/guide/topics/manifest/manifest-element.html)的 Google Android 文档来了解具体操作。重要的是，你需要决定将要调用的 sharedUserID，并在所有应用程序中使用。

在所有应用程序中设置 `SharedUserID` 后，即可使用 SSO。

> [AZURE.WARNING]
在应用程序之间共享存储之后，任何应用程序都可以删除用户，更糟的是，删除应用程序中的所有令牌。如果应用程序依赖于这些令牌来执行后台工作，后果将特别严重。要共享存储，就必须十分警惕通过 Microsoft 标识 SDK 执行的任意和所有删除操作。
> 
> 

就这么简单！ Microsoft 标识 SDK 现在将在所有应用程序之间共享凭据。此外还将在应用程序实例之间共享用户列表。

### 针对中转站辅助 SSO 启用 SSO
**默认关闭**应用程序使用安装在设备上的任何中转站的功能。若要让应用程序使用中转站，必须执行一些额外配置，并将一些代码添加到应用程序。

要遵循的步骤如下：

1. 在应用程序代码对 MS SDK 的调用中启用中转站模式
2. 建立新的重定向 URI，并向应用和应用注册提供该 URI
3. 在 Android 清单中设置正确的权限

#### 步骤 1：在应用程序中启用中转站模式
为身份验证实例创建“设置”或初始设置时，即会打开应用程序使用中转站的功能。在代码中设置 ApplicationSettings 类型即可执行此操作：


	AuthenticationSettings.Instance.setUseBroker(true);



#### 步骤 2：使用 URL 方案建立新的重定向 URI
为了确保始终将凭据令牌返回到正确的应用程序，我们需要确保以 Android 操作系统可以验证的方式回调应用程序。Android 操作系统使用 Google Play Store 中的证书哈希。此哈希不会被恶意应用程序欺骗。因此，我们会利用这种哈希以及中转站应用程序的 URI 来确保令牌返回到正确的应用程序。我们需要你在应用程序中建立这种唯一的重定向 URI，并设置为开发人员门户中的重定向 URI。

重定向 URI 必须采用正确的格式：

`msauth://packagename/Base64UrlencodedSignature`  


例如：*msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D*

需要使用 [Azure 经典管理门户](https://manage.windowsazure.cn/)在应用注册中指定此重定向 URI。有关 Azure AD 应用注册的详细信息，请参阅[与 Azure Active Directory 集成](/documentation/articles/active-directory-how-to-integrate/)。

#### 步骤 3：在应用程序中设置正确的权限
Android 中的中转站应用程序使用 Android OS 的帐户管理器功能来跨应用程序管理凭据。若要在 Android 中使用中转站，应用清单必须有权使用 AccountManager 帐户。[Google 帐户管理器文档](http://developer.android.com/reference/android/accounts/AccountManager.html)中对此进行了详细介绍

具体而言，这些权限如下：

	
	GET_ACCOUNTS
	USE_CREDENTIALS
	MANAGE_ACCOUNTS


### 你已配置好 SSO！
现在，Microsoft 标识 SDK 将自动在应用程序之间共享凭据并调用中转站（如果在设备上存在）。

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->