<properties
    pageTitle="连接到 Azure AD 时对 WebApi 项目所做的更改 | Azure"
    description="介绍使用 Visual Studio 连接到 Azure AD 时，WebApi 项目会发生什么情况"
    services="active-directory"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />

<tags
    ms.assetid="57630aee-26a2-4326-9dbb-ea2a66daa8b0"
    ms.service="active-directory"
    ms.workload="web"
    ms.tgt_pltfrm="vs-what-happened"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/01/2017"
    ms.author="tarcher"
    translationtype="Human Translation"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="fcdfda9645eaef85f364d0315e0cb7ad977cd44d"
    ms.lasthandoff="04/07/2017"
    wacn.date="04/17/2017" />



# <a name="what-happened-to-my-webapi-project-visual-studio-azure-active-directory-connected-service"></a>我的 WebApi 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况
> [AZURE.SELECTOR]
>- [入门](/documentation/articles/vs-active-directory-webapi-getting-started/)
>- [发生了什么情况](/documentation/articles/vs-active-directory-webapi-what-happened/)

## <a name="references-have-been-added"></a>已添加引用
### <a name="nuget-package-references"></a>NuGet 包引用
- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.ActiveDirectory`
- `Microsoft.Owin.Security.Jwt`
- `Microsoft.Owin.Security.OAuth`
- `Owin`
- `System.IdentityModel.Tokens.Jwt`

### <a name="net-references"></a>.NET 引用
- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.ActiveDirectory`
- `Microsoft.Owin.Security.Jwt`
- `Microsoft.Owin.Security.OAuth`
- `Owin`
- `System.IdentityModel.Tokens.Jwt`

## <a name="code-changes"></a>代码更改
### <a name="code-files-were-added-to-your-project"></a>代码文件已添加到您的项目
身份验证启动类 **App_Start/Startup.Auth.cs**（包含 Azure AD 身份验证的启动逻辑）已添加到你的项目。

### <a name="startup-code-was-added-to-your-project"></a>启动代码已添加到您的项目
如果项目中已经有一个 Startup 类，**Configuration** 方法将进行更新，以包括对 `ConfigureAuth(app)` 的调用。 否则，Startup 类已添加到您的项目。

### <a name="your-appconfig-or-webconfig-file-has-new-configuration-values"></a>您的 app.config 或 web.config 文件具有新的配置值。
已添加以下配置条目。

```
<appSettings>
    <add key="ida:ClientId" value="ClientId from the new Azure AD App" />
    <add key="ida:Tenant" value="Your selected Azure AD Tenant" />
    <add key="ida:Audience" value="The App ID Uri from the wizard" />
</appSettings>
```

### <a name="an-azure-ad-app-was-created"></a>已创建 Azure AD 应用
已在您在向导中选定的目录内创建一个 Azure AD 应用程序。

[详细了解 Azure Active Directory](https://www.azure.cn/home/features/identity/)

## <a name="if-i-checked-disable-individual-user-accounts-authentication-what-additional-changes-were-made-to-my-project"></a>如果我选中“*禁用单个用户帐户身份验证*”，会对我的项目进行哪些额外的更改？
NuGet 包引用已删除，文件已删除和备份。 根据你的项目的状态，你可能需要手动删除额外的引用或文件，或者根据需要修改代码。

### <a name="nuget-package-references-removed-for-those-present"></a>删除的 NuGet 包引用（针对已存在的）
- `Microsoft.AspNet.Identity.Core`
- `Microsoft.AspNet.Identity.EntityFramework`
- `Microsoft.AspNet.Identity.Owin`

### <a name="code-files-backed-up-and-removed-for-those-present"></a>备份的和删除的代码文件（针对已存在的）
以下每个文件都已备份并从项目中删除。 备份文件位于项目根目录的“Backup”文件夹中。

- `App_Start\IdentityConfig.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Providers\ApplicationOAuthProvider.cs`

### <a name="code-files-backed-up-for-those-present"></a>备份的的代码文件（针对已存在的）
以下每个文件在替换之前已备份。 备份文件位于项目根目录的“Backup”文件夹中。

- `Startup.cs`
- `App_Start\Startup.Auth.cs`

## <a name="if-i-checked-read-directory-data-what-additional-changes-were-made-to-my-project"></a>如果我选中“*读取目录数据*”，会对我的项目进行其他哪些更改？
### <a name="additional-changes-were-made-to-your-appconfig-or-webconfig"></a>对 app.config 或 web.config 做出的其他更改
添加了以下附加配置条目。

```
<appSettings>
    <add key="ida:Password" value="Your Azure AD App's new password" />
</appSettings>
```

### <a name="your-azure-active-directory-app-was-updated"></a>你的 Azure Active Directory 应用已更新
你的 Azure Active Directory 应用已更新为包括“*读取目录数据*”权限，并已创建一个附加密钥，并在随后用作 `web.config` 文件中的 *ida:Password*。

## <a name="next-steps"></a>后续步骤
- [详细了解 Azure Active Directory](https://www.azure.cn/home/features/identity/)
<!-- Update_Description: wording update -->
