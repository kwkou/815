<properties title="身份验证检测期间的错误" pageTitle="身份验证检测期间的错误" metaKeywords="" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

### 身份验证检测期间的错误

检测以前的身份验证代码时，向导检测到不兼容的身份验证类型。

##### 正在检查哪些内容？

向导尝试检测使用以前版本的 Visual Studio 配置的身份验证代码版本。如果您已收到此错误，它表示您的项目包含不兼容的身份验证类型。此向导将通过以前版本的 Visual Studio 检测以下身份验证类型：

-   Windows 身份验证
-   单个用户帐户
-   组织帐户

要检测 MVC 项目中的 Windows 身份验证，此向导将从您的 **web.config** 文件中查找 `authentication` 元素。

<pre class="prettyprint">
&lt;configuration&gt;
&lt;system.web&gt;
&lt;authentication mode=&quot;Windows&quot; /&gt;
&lt;/system.web&gt;
&lt;/configuration&gt;
</pre>

要检测 MVC 项目中的 Windows 身份验证，此向导将从您的项目的 **.csproj** 文件中查找 `IISExpressWindowsAuthentication` 元素：

<pre class="prettyprint">
&lt;Project&gt;
&lt;PropertyGroup&gt;
&lt;IISExpressWindowsAuthentication&gt;enabled&lt;/IISExpressWindowsAuthentication&gt;
&lt;/PropertyGroup&gt;
&lt;/Project&gt;
</pre>

要检测单个用户帐户身份验证，此向导将从您的 **Packages.config** 文件中查找 package 元素。

<pre class="prettyprint">
&lt;packages&gt;
&lt;package id=&quot;Microsoft.AspNet.Identity.EntityFramework&quot; version=&quot;2.1.0&quot; targetFramework=&quot;net45&quot; /&gt;
&lt;/packages&gt;
</pre>

要检测旧式的组织帐户身份验证，此向导将从 **web.config** 文件中查找以下元素：

<pre class="prettyprint">
&lt;configuration*gt;
&lt;appSettings&gt;
&lt;add key=&quot;ida:Realm&quot; value=&quot;***&quot; /&gt;
&lt;/appSettings&gt;
&lt;/configuration&gt;
</pre>

如需变更身份认证类型，请删除不兼容的身份验证类型，然后再次运行此向导。

有关详细信息，请参阅 [Azure AD 的身份验证方案][Azure AD 的身份验证方案]。

  [Azure AD 的身份验证方案]: http://msdn.microsoft.com/library/azure/dn499820.aspx
