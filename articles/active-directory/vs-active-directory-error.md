<properties
    pageTitle="如何使用 Azure Active Directory 连接向导诊断错误"
    description="Active Directory 连接向导检测到不兼容的身份验证类型"
    services="active-directory"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />

<tags
    ms.assetid="dd89ea63-4e45-4da1-9642-645b9309670a"
    ms.service="active-directory"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/05/2017"
    ms.author="tarcher"
    translationtype="Human Translation"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="33379b2ff35d8e1aa71100924e5dff5c8dcd996e"
    ms.lasthandoff="04/07/2017"
    wacn.date="04/17/2017" />



# <a name="diagnosing-errors-with-the-azure-active-directory-connection-wizard"></a>使用 Azure Active Directory 连接向导诊断错误
检测以前的身份验证代码时，向导检测到不兼容的身份验证类型。   

## <a name="what-is-being-checked"></a>正在检查哪些内容？
**注意：**为了正常检测某个项目中以前的身份验证代码，必须生成该项目。  如果你遇到此错误，并且项目中不存在以前的身份验证代码，请重新生成项目并重试。

### <a name="project-types"></a>项目类型
向导会检查你正在开发的项目类型，以便可以将正确的身份验证逻辑注入到项目。  如果项目中有控制器派生自 `ApiController`，则该项目将被视为 WebAPI 项目。  如果项目中的控制器均派生自 `MVC.Controller`，则项目将被视为 MVC 项目。  任何其他项目均不受向导支持。

### <a name="compatible-authentication-code"></a>兼容的身份验证代码
向导还将检查是否存在以前使用向导配置的身份验证设置或与向导兼容的身份验证设置。  如果所有这些设置都存在，则将视为可重入情况，向导将打开并显示这些设置。  如果只存在某些设置，则将视为错误情况。

在 MVC 项目中，向导将检查是否存在以下任何设置（这些设置是以前使用向导生成的）：

```
<add key="ida:ClientId" value="" />
<add key="ida:Tenant" value="" />
<add key="ida:AADInstance" value="" />
<add key="ida:PostLogoutRedirectUri" value="" />
```

此外，在 Web API 项目中向导还会检查是否存在以下任何设置（这些设置是以前使用向导生成的）：

```
<add key="ida:ClientId" value="" />
<add key="ida:Tenant" value="" />
<add key="ida:Audience" value="" />
```

### <a name="incompatible-authentication-code"></a>不兼容的身份验证代码
最后，向导将尝试检测使用以前版本的 Visual Studio 配置的身份验证代码版本。 如果您已收到此错误，它表示您的项目包含不兼容的身份验证类型。 此向导将通过以前版本的 Visual Studio 检测以下身份验证类型：

- Windows 身份验证 
- 单个用户帐户 
- 组织帐户 

为了检测 MVC 项目中的 Windows 身份验证，此向导将从 **web.config** 文件中查找 `authentication` 元素。

<pre>
    &lt;configuration&gt;
        &lt;system.web&gt;
            <span style="background-color: yellow">&lt;authentication mode="Windows" /&gt;</span>
        &lt;/system.web&gt;
    &lt;/configuration&gt;
</pre>

为了检测 Web API 项目中的 Windows 身份验证，此向导将从项目的 **.csproj** 文件中查找 `IISExpressWindowsAuthentication` 元素：

<pre>
    &lt;Project&gt;
        &lt;PropertyGroup&gt;
            <span style="background-color: yellow">&lt;IISExpressWindowsAuthentication&gt;enabled&lt;/IISExpressWindowsAuthentication&gt;</span>
        &lt;/PropertyGroup>
    &lt;/Project&gt;
</pre>

要检测单个用户帐户身份验证，此向导将从您的 **Packages.config** 文件中查找 package 元素。

<pre>
    &lt;packages&gt;
        <span style="background-color: yellow">&lt;package id="Microsoft.AspNet.Identity.EntityFramework" version="2.1.0" targetFramework="net45" /&gt;</span>
    &lt;/packages&gt;
</pre>

要检测旧式的组织帐户身份验证，此向导将从 **web.config**文件中查找以下元素：

<pre>
    &lt;configuration&gt;
        &lt;appSettings&gt;
            <span style="background-color: yellow">&lt;add key="ida:Realm" value="***" /&gt;</span>
        &lt;/appSettings&gt;
    &lt;/configuration&gt;
</pre>

如需变更身份认证类型，请删除不兼容的身份验证类型，然后再次运行此向导。

有关详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。

#<a name="next-steps"></a>后续步骤
- [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)
<!-- Update_Description: wording update -->
