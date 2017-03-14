<properties
    pageTitle="将应用程序与 Azure Active Directory 集成 | Azure"
    description="有关如何在 Azure Active Directory (Azure AD) 中添加、更新或删除应用程序的详细信息。"
    services="active-directory"
    documentationcenter=""
    author="bryanla"
    manager="mbaldwin"
    editor="mbaldwin" />
<tags
    ms.assetid="ae637be5-0b71-4b1e-b1fe-b83df3eb4845"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/08/2017"
    wacn.date="03/13/2017"
    ms.author="mbaldwin;bryanla" />  


# 将应用程序与 Azure Active Directory 集成

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

企业开发人员和软件即服务 (SaaS) 提供商可以开发能够与 Azure Active Directory (Azure AD) 集成的商业云服务或业务线应用程序，以针对其服务提供安全的登录和授权。若要将应用程序或服务与 Azure AD 集成，开发人员必须首先通过 Azure 经典管理门户向 Azure AD 注册关于其应用程序的详细信息。

本文说明如何在 Azure AD 中添加、更新或删除应用程序。你将了解可与 Azure AD 集成的不同类型的应用程序，以及如何将应用程序配置为访问其他资源（如 Web API 等等）。

若要详细了解表示已注册应用程序的两个 Azure AD 对象及其关系，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)；若要详细了解利用 Azure Active Directory 开发应用程序时应使用的品牌准则，请参阅[适用于集成应用的品牌准则](/documentation/articles/active-directory-branding-guidelines/)。

## 添加应用程序 <a name="BKMK_Native"></a><a name="adding-an-application"></a>
任何想要使用 Azure AD 功能的应用程序都必须先在 Azure AD 租户中注册。此注册过程涉及到提供有关应用程序的 Azure AD 详细信息，例如，该应用程序所在位置的 URL、对用户进行身份验证后用于发送答复的 URL、用于标识应用程序的 URI，等等。

如果正在构建的 Web 应用程序只要求支持用户在 Azure AD 中登录，则只需按照以下说明操作。如果应用程序需要使用凭据或权限访问 Web API，或者需要允许其他 Azure AD 租户中的用户访问该应用程序，请参阅[更新应用程序](#updating-an-application)部分继续配置应用程序。

### 在 Azure 经典管理门户注册新应用程序
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中单击“应用程序”。如果尚未将任何应用添加到你的目录，则此页只会显示“添加应用”链接。单击该链接，或者单击命令栏上的“添加”按钮。
4. 在“要执行什么操作”页上，单击“添加我的组织正在开发的应用程序”链接。
5. 在“向我们说明你的应用程序”页上，必须指定应用程序的名称并指明你要注册到 Azure AD 的应用程序类型。可以从 [Web 应用程序/客户端](/documentation/articles/active-directory-dev-glossary/#client-application/)和/或 [Web 资源/API](/documentation/articles/active-directory-dev-glossary/#resource-server/) 应用程序或[本机客户端](/documentation/articles/active-directory-dev-glossary/#native-client/)应用程序中选择。完成后，单击页面右下角的箭头图标。
6. 在“应用属性”页上，提供“登录 URL”和“应用 ID URI”（如果正在注册 Web 应用程序）或者只提供“重定向 URI”（针对本机客户端应用程序），然后单击页面右下角的复选框。
7. 现已添加你的应用程序，此时你将转到应用程序的“快速启动”页。根据你的应用程序是 Web 应用程序还是本机应用程序，你将看到有关如何向应用程序添加更多功能的不同选项。添加应用程序后，你可以开始更新应用程序，使用户能够登录、访问其他应用程序中的 Web API，或者配置多租户应用程序（允许其他组织访问你的应用程序）。

> [AZURE.NOTE]
默认情况下，新建的应用程序注册已配置为允许你的目录中的用户登录到你的应用程序。
> 
> 

## 更新应用程序 <a name="updating-an-application"></a>
将应用程序注册到 Azure AD 后，可能需要更新该应用程序，以提供对 Web API 的访问权限、使其可在其他组织中使用，等等。本部分介绍可以通过哪些不同的方法来进一步配置应用程序。首先，我们将概述同意框架。如果你要构建资源/API 应用程序，并且这些应用程序将由你的组织或其他组织中的开发人员构建的应用程序使用，则必须了解此框架。

有关如何在 Azure AD 中进行身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。

### 同意框架概述 <a name="overview-of-the-consent-framework"></a>
使用 Azure AD 的同意架构可以轻松开发需要访问 Azure AD 租户保护的多租户 Web API 的 Web 应用程序和本机客户端应用程序（不同于注册客户端应用程序的框架）。这些 Web API 包括图形 API、Office 365 和其他 Microsoft 服务，以及你自己的 Web API。该框架基于某个用户或管理员，该用户或管理员允许某个应用程序在其目录中注册，这可能涉及到访问目录数据。

例如，如果某个 Web 客户端应用程序需要调用 Office 365 Web API/资源应用程序以读取有关用户的日历信息，则该用户需要同意该客户端应用程序。同意后，该客户端应用程序能够代表该用户调用 Office 365 Web API，并根据需要使用日历信息。

同意框架使用公共或机密客户端，建立在 OAuth 2.0 及其各种流程的基础之上，例如，代码授权和客户端凭据授权。通过使用 OAuth 2.0，可以在 Azure AD 中生成多种不同类型的客户端应用程序（例如手机、平板电脑、服务器或 Web 应用程序），并获取对所需资源的访问权限。

有关同意框架的更多详细信息，请参阅 [Azure AD 中的 OAuth 2.0](/documentation/articles/active-directory-protocols-oauth-code/)、[Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)和 Office 365 主题[了解如何使用 Office 365 API 进行身份验证](https://msdn.microsoft.com/office/office365/howto/common-app-authentication-tasks)。

#### 同意体验示例
以下步骤将说明同意体验对应用程序开发人员和用户的工作方式。

1. 在 Azure 经典管理门户中 Web 客户端应用程序的配置页上，使用“针对其他应用程序的权限”控件中的下拉菜单设置应用程序所需的权限。
   
    ![针对其他应用程序的权限](./media/active-directory-integrating-applications/permissions.png)  

2. 考虑已更新应用程序的权限，该应用程序正在运行，并且某个用户即将首次使用该应用程序。如果应用程序尚未获取访问令牌或刷新令牌，则该应用程序需要转到 Azure AD 的授权终结点，以获取可用于取得新访问令牌和刷新令牌的授权代码。
3. 如果用户尚未经过身份验证，系统将要求他们登录到 Azure AD。
   
    ![用户或管理员登录到 Azure AD](./media/active-directory-integrating-applications/useradminsignin.png)  

4. 用户登录后，Azure AD 将决定是否要向该用户显示同意页。此决定基于该用户（或其组织的管理员）是否已授予应用程序许可。如果尚未授予许可，Azure AD 将提示用户授予许可，并显示运行该应用程序所需的权限。同意对话框中显示的权限集与在 Azure 经典管理门户的“针对其他应用程序的权限”控件中选择的权限集相同。
   
    ![用户同意体验](./media/active-directory-integrating-applications/userconsent.png)  

5. 用户授予许可后，授权代码将返回到你的应用程序，应用程序可凭此获取访问令牌和刷新令牌。有关此流程的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)主题中的[从 Web 应用程序到 Web API](/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api/) 部分。

### 将客户端应用程序配置为访问 Web API
为使 Web/机密客户端应用程序能够参与要求身份验证的授权流程（以及获取访问令牌），必须建立安全凭据。Azure 门户预览支持的默认身份验证方法是客户端 ID + 对称密钥。本部分介绍需要执行哪些配置步骤来提供客户端凭据的机密密钥。

在此，在客户端访问资源应用程序公开的 Web API（例如 Azure AD 图形 API）之前，同意框架将确保客户端根据请求的权限获取所需的授权。默认情况下，所有应用程序可以从 Azure Active Directory（图形 API）和 Azure 服务管理 API 中选择权限，并且已选择 Azure AD 的“启用登录并读取用户的配置文件”权限。如果客户端应用程序已在 Office 365 Azure AD 租户中注册，则也可以选择 SharePoint 与 Exchange Online 的 Web API 和权限。可以从所需 Web API 旁边的下拉菜单中的[两种类型的权限](/documentation/articles/active-directory-dev-glossary/#permissions/)中进行选择：

- 应用程序权限：客户端应用程序需要亲自直接访问 Web API（无用户上下文）。此类型的权限需要管理员同意，并且不可用于本机客户端应用程序。
- 委托的权限：客户端应用程序需要以登录用户的身份访问 Web API，但访问权限受所选权限的限制。除非权限已配置为需要管理员同意，否则用户可以授予此类型的权限。

> [AZURE.NOTE]
将委托权限添加到应用程序不会自动向租户中的用户授予许可。 用户仍必须手动许可在运行时添加委托权限。

#### 添加用于访问 Web API 的凭据或权限
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的应用程序。“快速启动”页将显示单一登录信息和其他配置信息。单击页面顶部的“配置”链接。随后将转到应用程序的配置页。
4. 若要添加 Web 应用程序凭据的密钥，请向下滚动到“密钥”部分。
   
   - 首先单击“选择持续时间”下拉列表，然后选择 1 年或 2 年的持续时间。
   - 随后将显示一个新行，其中填充了“生效日期”和“过期日期”。
   - 保存配置更改（如下所示）后，最右边的列将包含密钥值。点击保存后请务必返回到此部分并复制该值，因为在运行时进行身份验证的过程中，需要在客户端应用程序中用到该值。
5. 若要添加用于从客户端访问资源 API 的权限，请向下滚动到“针对其他应用程序的权限”部分。
   
   - 首先单击“添加应用程序”按钮。
   - 使用“显示”下拉列表选择所需的资源类型。
   - 在第一列中，可以从目录中提供的、公开 Web API 的资源应用程序中进行选择。单击所需的资源，然后单击右下角的勾选标记。
   - 选择后，将会看到该资源已添加到“针对其他应用程序的权限”列表中。
   - 使用“应用程序权限”和“委托的权限”下拉列表选择客户端应用程序的所需权限。
6. 完成后，单击命令栏上的“保存”按钮。如果为应用程序指定了密钥，则单击“保存”还会生成该密钥。

>[AZURE.NOTE] 单击“保存”按钮还会基于你配置的“对其他应用程序的权限”自动设置对你目录中的应用程序的权限。可以在应用程序的“属性”选项卡中查看这些应用程序权限。

### 将资源应用程序配置为公开 Web API
可以开发一个 Web API，并通过公开访问权限[范围](/documentation/articles/active-directory-dev-glossary/#scopes/)和[角色](/documentation/articles/active-directory-dev-glossary/#roles/)，使其可供其他客户端应用程序使用。可以像提供其他 Microsoft Web API（包括图形 API 和 Office 365 API）一样提供正确配置的 Web API。可通过[应用程序的清单](/documentation/articles/active-directory-dev-glossary/#application-manifest/)公开访问权限范围和角色。该清单是表示应用程序标识配置的 JSON 文件。

以下部分说明如何通过修改资源应用程序的清单公开访问范围。

#### 将访问权限范围添加到资源应用程序
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的资源应用程序。“快速启动”页将显示单一登录信息和其他配置信息。
4. 在命令栏中单击“管理清单”按钮，然后选择“下载清单”。
5. 打开 JSON 应用程序清单文件，并将 “oauth2Permissions” 节点替换为以下 JSON 代码段。此示例代码片段演示如何公开称为“用户模拟”的范围，它允许资源所有者为客户端应用程序分配某种类型的、对资源的委托访问权限。请确保根据自己的应用程序更改文本和值：
   
        "oauth2Permissions": [
        {
            "adminConsentDescription": "Allow the application full access to the Todo List service on behalf of the signed-in     user",
            "adminConsentDisplayName": "Have full access to the Todo List service",
            "id": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
            "isEnabled": true,
            "type": "User",
            "userConsentDescription": "Allow the application full access to the todo service on your behalf",
            "userConsentDisplayName": "Have full access to the todo service",
            "value": "user_impersonation"
            }
        ],
   
    ID 值必须是使用 [GUID 生成工具](https://msdn.microsoft.com/zh-cn/library/ms241442%28v=vs.80%29.aspx)或以编程方式新生成的 GUID。它表示 Web API 公开的权限的唯一标识符。将客户端适当地配置为请求访问 Web API 并调用 Web API 后，该客户端将显示一个 OAuth 2.0 JWT 令牌，该令牌的作用域 (scp) 声明已设置为上面指定的值（在本例中为 user\_impersonation）。
   
	>[AZURE.NOTE] 以后你可以根据需要公开其他范围。请考虑你的 Web API 可能要公开与各种不同功能关联的多个范围。现在，你可以通过使用所收到的 OAuth 2.0 JWT 令牌中的范围 (scp) 声明来控制对 Web API 的访问。

6. 保存更新的 JSON 文件，然后将其上载，方法是单击命令栏中的“管理清单”按钮，选择“上载清单”，浏览到更新的清单文件，然后将其选中。上载后，你的 Web API 便已配置为可供目录中的其他应用程序使用。

#### 验证 Web API 是否已公开给目录中的其他应用程序
1. 在顶部菜单中，单击“应用程序”，选择要为其配置 Web API 访问权限的所需客户端应用程序，然后单击“配置”。
2. 向下滚动到“对其他应用程序的权限”部分。单击“选择应用程序”下拉菜单，然后便可以选择你刚刚为其公开了权限的 Web API。从“委托的权限”下拉菜单中选择新的权限。

![显示“待办事项”权限](./media/active-directory-integrating-applications/listpermissions.png)  


#### 有关应用程序清单的更多信息
应用程序清单实际上充当了用于更新应用程序实体的机制，它定义了 Azure AD 应用程序的标识配置的所有属性，包括本文所述的 API 访问权限范围。有关应用程序实体的详细信息，请参阅 [Graph API Application entity documentation](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity)（图形 API 应用程序实体文档）。在该文档中，可以找到用于指定 API 权限的应用程序实体成员的完整参考信息：

- appRoles 成员，可用于定义 Web API 的**应用程序权限**的 [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) 实体集合
- oauth2Permissions 成员，可用于定义 Web API 的**委托权限**的 [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) 实体集合

有关应用程序清单概念的一般详细信息，请参阅[了解 Azure Active Directory 应用程序清单](/documentation/articles/active-directory-application-manifest/)。

### 访问 Azure AD Graph 和 Office 365 API
如前文所述，除了在自己的资源应用程序上公开/访问 API 以外，你还可以更新客户端应用程序以访问 Microsoft 资源公开的 API。默认情况下，Azure AD 图形 API（在其他应用程序的“权限”列表中称为“Azure Active Directory”）可供注册到 Azure AD 的所有应用程序使用。如果要在 Office 365 设置的 Azure AD 租户中注册客户端应用程序，你也可以访问 API 向各种 Office 365 资源公开的所有权限。

有关以下各项公开的访问权限范围的完整介绍：

- Azure AD 图形 API，请参阅[权限范围 | Graph API 概念](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)一文。
- Office 365 API，请参阅 [Authentication and Authorization using Common Consent Framework](https://msdn.microsoft.com/office/office365/howto/application-manifest)（使用通用同意框架进行身份验证和授权）一文。有关如何生成可与 Office 365 API 集成的客户端应用的详细介绍，请参阅 [Set up your Office 365 development environment](https://msdn.microsoft.com/office/office365/HowTo/setup-development-environment)（设置 Office 365 开发环境）。

> [AZURE.NOTE]
由于当前存在的限制，如果本机客户端应用程序使用“访问组织的目录”权限，则它们只能调用 Azure AD 图形 API。此限制不适用于 Web 应用程序。
> 
> 

### 配置多租户应用程序
将应用程序添加到 Azure AD 时，你可能希望你的应用程序只能由你组织中的用户访问。或者，你可能希望你的应用程序可由外部组织中的用户访问。这两种应用程序分别称为单租户应用程序和多租户应用程序。你可以修改单租户应用程序的配置，使其成为多租户应用程序，本部分将对此进行介绍。

请务必注意单租户与多租户应用程序之间的差别：

- 单租户应用程序旨在用于一家组织中。它们通常是企业开发人员编写的业务线 (LoB) 应用程序。单租户应用程序只需要供单个目录中的用户进行访问，因此，只需要将其设置在单个目录中。
- 多租户应用程序旨在用于多家组织中。它们通常是独立软件供应商 (ISV) 编写的软件即服务 (SaaS) Web 应用程序。多租户应用程序需要设置在将使用它们的每个目录中，需要经过用户或管理员许可才能注册它们，并通过 Azure AD 同意框架获得支持。请注意，默认情况下，所有本机客户端应用程序都是多租户的，因为它们安装在资源所有者的设备上。有关同意框架的更多详细信息，请参阅上面的“同意框架概述”部分。

#### 使外部用户能够向你的应用程序授予对其资源的访问权限
如果正在编写一个要供组织外部的客户或合作伙伴使用的应用程序，则需要在 Azure 经典管理门户中更新应用程序定义。

> [AZURE.NOTE]
在启用多租户时，必须确保应用程序的应用 ID URI 属于已验证的域。此外，返回 URL 必须以 https:// 开头。有关详细信息，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。
> 
> 

若要为外部用户启用对你应用访问权限，请执行以下操作：

1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的应用程序。“快速启动”页将显示配置选项。
4. 展开“快速启动”的“配置多租户应用程序”部分，然后单击“启用访问权限”部分中的“立即配置它”链接。此时将显示应用程序属性页。
5. 单击“应用程序是多租户的”旁边的“是”按钮，然后单击命令栏上的“保存”按钮。

完成上述更改后，其他组织中的用户和管理员可以向你的应用程序授予对其目录和其他数据的访问权限。

#### 在运行时触发 Azure AD 同意框架
若要使用同意框架，多租户客户端应用程序必须使用 OAuth 2.0 请求授权。我们提供了[代码示例](https://azure.microsoft.com/documentation/samples/?service=active-directory&term=multi-tenant/)用于说明 Web 应用程序、本机应用程序或服务器/后台应用程序如何请求授权代码和访问令牌来调用 Web API。

Web 应用程序也可为用户提供注册体验。如果你确实提供注册体验，用户应会单击注册按钮，此时会将浏览器重定向到 Azure AD OAuth2.0 授权终结点或 OpenID Connect userinfo 终结点。这些终结点允许应用程序通过检查 id\_token 来获取有关新用户的信息。在注册阶段后，用户将看到一个同意提示，与上面“同意框架概述”一节中所示的同意提示类似。

或者，你的 Web 应用程序还可能会提供允许管理员“注册我的公司”的体验。这种体验也会将用户重定向到 Azure AD OAuth 2.0 授权终结点。不过在此情况下，你会向授权终结点传递一个 prompt=admin\_consent 参数来强制提供管理员同意体验，此时管理员会代表其组织授予同意。只有使用属于全局管理员角色的帐户进行身份验证的用户才能提供同意；其他人会收到错误。成功同意后，响应中将包含 admin\_consent=true。当换取访问令牌时，你还会收到一个 id\_token，其中提供了有关注册你的应用程序的组织和管理员的信息。

### 为单页面应用程序启用 OAuth 2.0 隐式授权
通常将单页面应用程序 (SPA) 构建为一个在浏览器中运行的 JavaScript 重型前端，该前端调用应用程序的 Web API 后端来执行其业务逻辑。对于托管在 Azure AD 中的 SPA，可以使用 OAuth 2.0 隐式授权对具有 Azure AD 的用户进行身份验证，并获取可用来保护从应用程序 JavaScript 客户端到其后端 Web API 的调用的令牌。用户授予同意之后，可以使用同一个身份验证协议来获取令牌以保护客户端与针对应用程序配置的其他 Web API 资源之间的调用。若要了解有关隐式授权的详细信息，并帮助确定其是否适合应用程序方案，请参阅[了解 Azure Active Directory 中的 OAuth2 隐式授权流](/documentation/articles/active-directory-dev-understanding-oauth2-implicit-grant/)。

默认情况下，为应用程序禁用了 OAuth 2.0 隐式授权。可以通过在[应用程序清单](/documentation/articles/active-directory-application-manifest/)（表示应用程序标识配置的 JSON 文件）中设置 `oauth2AllowImplicitFlow` 值，来为应用程序启用 OAuth 2.0 隐式授权。

#### 启用 OAuth 2.0 隐式授权
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的应用程序。“快速启动”页将显示单一登录信息和其他配置信息。
4. 在命令栏中单击“管理清单”按钮，然后选择“下载清单”。打开 JSON 应用程序清单文件并将“oauth2AllowImplicitFlow”值设为“true”。默认情况下，值为“false”。
   
    `"oauth2AllowImplicitFlow": true,`  

5. 保存更新的 JSON 文件，然后将其上载，方法是单击命令栏中的“管理清单”按钮，选择“上载清单”，浏览到更新的清单文件，然后将其选中。上载后，你的 Web API 即已配置为使用 OAuth 2.0 隐式授权来对用户进行身份验证。


## 删除应用程序
本部分说明如何从 Azure AD 租户中删除应用程序。

### 删除组织编写的应用程序
这些应用程序是 Azure AD 租户的“应用程序”主页面上“我的公司拥有的应用程序”筛选器下显示的应用程序。从技术层面讲，这些是通过 Azure 经典管理门户以手动方式注册的应用程序，或通过 PowerShell 或图形 API 以编程方式注册的应用程序。更具体地说，它们由租户中的应用程序与服务主体对象表示。有关详细信息，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。

#### 从目录中删除单租户应用程序
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的应用程序。“快速启动”页将显示单一登录信息和其他配置信息。
4. 单击命令栏中的“删除”按钮。
5. 在确认消息中单击“是”。

#### 从目录中删除多租户应用程序
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧菜单中单击“Active Directory”图标，然后单击所需的目录。
3. 在顶部菜单中，单击“应用程序”，然后单击要配置的应用程序。“快速启动”页将显示单一登录信息和其他配置信息。
4. 在“应用程序是多租户的”部分中，单击“否”。这会将应用程序转换为单租户应用程序，但该应用程序仍保留在已同意它的组织中。
5. 单击命令栏中的“删除”按钮。
6. 在确认消息中单击“是”。

### 删除其他组织授权的多租户应用程序
这些应用程序是 Azure AD 租户的“应用程序”主页面上“我的公司使用的应用程序”筛选器下显示的一部分应用程序，特别是未列在“我的公司拥有的应用程序”列表下的应用程序。在技术层面讲，它们是在同意处理期间注册的多租户应用程序。更具体地说，它们仅由租户中的服务主体对象表示。有关详细信息，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。

公司管理员若要删除多租户应用程序对目录的访问权限（在授予许可后），必须拥有 Azure 订阅才能通过 Azure 经典管理门户删除该访问权限。公司管理员也可以使用 [Azure AD PowerShell Cmdlet](http://go.microsoft.com/fwlink/?LinkId=294151) 删除访问权限。

## 后续步骤
- 有关应用的视觉指南的提示，请参阅[集成应用的品牌准则](/documentation/articles/active-directory-branding-guidelines/)。
- 有关应用程序的应用程序对象与服务主体对象之间关系的详细信息，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。
- 若要了解有关应用程序清单扮演的角色的详细信息，请参阅[了解 Azure Active Directory 应用程序清单](/documentation/articles/active-directory-application-manifest/)
- 请参阅 [Azure AD developer glossary](/documentation/articles/active-directory-dev-glossary/)（Azure AD 开发人员术语表），了解某些核心的 Azure Active Directory (AD) 开发人员概念的定义。
- 请参阅 [Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)了解与所有开发人员相关内容的概述。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->