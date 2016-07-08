<properties
   pageTitle="如何将应用程序添加到 Azure Active Directory。"
   description="本文介绍如何将应用程序添加到 Azure Active Directory 的实例。"
   services="active-directory"
   documentationCenter=""
   authors="shoatman"
   manager="kbrint"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="02/09/2016"
   wacn.date="06/06/2016"/>

# 如何以及为何将应用程序添加到 Azure AD

在 Azure Active Directory 实例中查看应用程序列表时，最初让令人费解的事情之一是不知道应用程序来自何处，以及它们为何会出现在那里。本文将全面概述如何在目录中表示应用程序，并提供上下文帮助你了解应用程序如何进入你的目录。

## Azure AD 为应用程序提供哪些服务？

应用程序将添加到 Azure AD，以利用 Azure AD 提供的一个或多个服务。这些服务包括：

* 应用程序身份验证和授权
* 用户身份验证和授权
* 使用联合身份验证或密码的单一登录 (SSO)
* 用户设置和同步
* 基于角色的访问控制；使用目录定义应用程序角色，以便在应用程序中执行基于角色的授权检查。
* oAuth 授权服务（Office 365 和其他 Microsoft 应用程序使用这些服务来授予对 API/资源的访问权限。）
* 应用程序发布和代理；将应用程序从专用网络发布到 Internet

## 如何在目录中表示应用程序？

Azure AD 中的应用程序是使用 2 个对象表示的：应用程序对象和服务主体对象。“home”/“owner”或“publishing”目录中注册了一个应用程序对象；此外，有一个或多个服务主体对象表示运行应用程序的每个目录中的应用程序。

应用程序对象向 Azure AD（多租户服务）描述应用，可能包括下列任何项：（注意：此列表并不完整。）

* 名称、徽标和发布者
* 机密（用于对应用程序进行身份验证的对称和/或非对称密钥）
* API 依赖关系 (oAuth)
* 发布的 API/资源/作用域 (oAuth)
* 应用程序角色 (RBAC)
* SSO 元数据和配置 (SSO)
* 用户设置元数据和配置
* 代理元数据和配置

服务主体是运行应用程序的每个目录（包括其主目录）中的应用程序记录。服务主体：

* 通过应用程序 id 属性向后引用应用程序对象
* 记录本地用户和组的应用程序角色分配
* 记录授予应用程序的本地用户和管理员权限
    * 例如：应用程序访问特定用户电子邮件的权限
* 记录本地策略，包括条件性访问策略
* 记录应用程序的本地替代设置
    * 声明转换规则
    * 属性映射（用户设置）
    * 租户特定的应用程序角色（如果应用程序支持自定义角色）
    * 名称/徽标

### 不同目录中应用程序对象和服务主体的关系图

![演示应用程序对象和服务主体如何与 Azure AD 实例共存的关系图。][apps_service_principals_directory]

从上面的关系图中可以看出，Microsoft 在内部维护了两个用于发布应用程序的目录（左侧）。
 
* 一个目录用于 Microsoft 应用程序（Microsoft 服务目录）
* 一个目录用于预先集成的第三方应用程序（应用程序库目录）

与 Azure AD 集成的应用程序发布者/供应商需要有一个发布目录。（某个 SAAS 目录）。

你自己添加的应用程序包括：

* 你开发的应用程序（与 AAD 集成）
* 为了进行单一登录而连接的应用程序
* 使用 Azure AD 应用程序代理发布的应用程序。

### 说明和例外情况

* 并非所有服务主体都会往后指向应用程序对象。是吗？ 最初生成 Azure AD 时，提供给应用程序的服务存在很多的限制，使用服务主体便足以建立应用程序标识。原始服务主体在形式上更接近于 Windows Server Active Directory 服务帐户。出于此原因，你仍可以使用 Azure AD PowerShell 创建服务主体，而无需首先创建应用程序对象。Graph API 在创建服务主体之前需要一个应用程序对象。
* 上述信息当前并非全部都是以编程方式公开的。只能在 UI 中使用以下功能：
    * 声明转换规则
    * 属性映射（用户设置）
* 有关服务主体和应用程序对象的详细信息，请参阅 Azure AD Graph REST API 参考文档。提示：目前，阅读 Azure AD Graph API 文档是获得 Azure AD 架构参考信息的最佳捷径。  
    * [应用程序](https://msdn.microsoft.com/zh-cn/library/azure/dn151677.aspx)
    * [服务主体](https://msdn.microsoft.com/zh-cn/library/azure/dn194452.aspx)


## 如何将应用程序添加到 Azure AD 实例？
可以使用多种方法将应用程序添加到 Azure AD：

<!--* Add an app from the [Azure Active Directory App Gallery](/updates/azure-active-directory-over-1000-apps/)-->
* 注册/登录与 Azure Active Directory 集成的第三方应用程序（例如：[Smartsheet](https://app.smartsheet.com/b/home) 或 [DocuSign](https://www.docusign.net/member/MemberLogin.aspx)）
    * 在注册/登录期间，系统会要求用户向应用程序授予访问其配置文件的权限和其他权限。第一个授权者会导致生成一个服务主体，表示要添加到目录中的应用程序。
* 注册/登录到 [Office 365](http://products.office.com/zh-CN) 等  Microsoft Online Services
    * 当你订阅 Office 365 或开始试用时，将在目录中创建一个或多个服务主体，表示传递所有与 Office 365 关联的功能的各种服务。
    * 某些 Office 365 服务（如 SharePoint）会不断地创建服务主体，以允许在组件（包括工作流）之间进行安全通信。
* 在 Azure 经典管理门户中添加你正在开发的应用程序，具体请参阅：https://msdn.microsoft.com/zh-cn/library/azure/dn132599.aspx
* 使用 Visual Studio 添加你正在开发的应用程序，具体请参阅：
    * [ASP.Net 身份验证方法](http://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions)
    * [连接的服务](http://blogs.msdn.com/b/visualstudio/archive/2014/11/19/connecting-to-cloud-services.aspx)

* 连接应用程序，以使用使用 SAML 或密码 SSO 进行单一登录
* 其他许多功能包括 Azure 中的各种开发人员体验，以及开发人员中心的 API 资源管理器体验

## 谁有权向我的 Azure AD 实例添加应用程序？

只有全局管理员可以：

* 从 Azure AD 应用程序库添加应用程序（预先集成的第三方应用程序）
* 使用 Azure AD 应用程序代理发布应用程序

你目录中的所有用户都有权添加他们正在开发的应用程序，并决定要共享哪些应用程序/授予对其组织数据的访问权限。请记住，用户注册/登录应用程序和授权可能会导致创建服务主体。

一开始这听上去可能令人忧虑，不过，请记住以下事项：

* 多年以来，应用程序一直可以利用 Windows Server Active Directory 进行用户身份验证，而无需在目录中注册/记录应用程序。现在，组织改进了可见性，完全知道有多少应用程序正在使用目录以及为何使用目录。
* 无需管理员驱动的应用程序发布/注册过程。过去，在使用 Active Directory 联合身份验证服务时，管理员可能必须代表开发人员添加一个应用程序作为信赖方。现在，开发人员可以自行解决问题。
* 用户为了业务目的使用其组织帐户登录/注册应用程序是一个好现象。如果他们以后离开了组织，他们将无法访问所用应用程序中的帐户。
* 记录与哪个应用程序共享了哪些数据是一个很好的做法。数据的流动性比以往更明显，因此，明确记录哪个用户与哪些应用程序共享了哪些数据会很有用。
* 为 oAuth 使用 Azure AD 的应用程序将明确决定用户可向应用程序授予哪些权限，以及哪些权限需要管理员的许可。不言而喻，只有管理员才能授予较大范围的更重要的权限。
* 添加应用程序和允许应用程序访问其数据的用户将会添加到审核事件，以便你可以在 Azure 经典管理门户中查看审核报告，以确定应用程序是如何添加到目录中的。

**注意：**到目前为止，Microsoft 本身已使用默认配置运行了好几个月。

总而言之，我们可以防止目录中的用户添加应用程序，并可防止他们通过在 Azure 经典管理门户中修改目录配置，来决定要与应用程序共享哪些信息。可以在 Azure 经典管理门户中通过目录的“配置”选项卡访问以下配置。

![用于配置集成应用程序设置的 UI 屏幕截图][app_settings]

## 后续步骤

了解有关如何将应用程序添加到 Azure AD 以及如何为应用程序配置服务的详细信息。

* 开发人员：[在 Github 上查看与 Azure Active Directory 集成的应用程序的示例代码](https://github.com/AzureAD)
* 开发人员和 IT 专业人员：[查看 Azure Active Directory Graph API 的 REST API 文档](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)


<!--Image references--> 

[apps_service_principals_directory]: ./media/active-directory-how-applications-are-added/HowAppsAreAddedToAAD.jpg
[app_settings]: ./media/active-directory-how-applications-are-added/IntegratedAppSettings.jpg

<!---HONumber=Mooncake_0516_2016-->