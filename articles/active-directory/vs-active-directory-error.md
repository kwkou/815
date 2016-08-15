<properties 
	pageTitle="身份验证检测期间的错误" 
	description="Active Directory 连接向导检测到不兼容的身份验证类型" 
	services="active-directory" 
	documentationCenter="" 
	authors="TomArcher" 
	manager="douge" 
	editor=""/>
  
<tags 
	ms.service="active-directory"  
	ms.date="03/28/2016" 
	wacn.date="07/05/2016"/>

# 身份验证检测期间的错误

检测以前的身份验证代码时，向导检测到不兼容的身份验证类型。

##正在检查哪些内容？

**注意：**为了正常检测某个项目中以前的身份验证代码，必须生成该项目。如果你遇到此错误，并且项目中不存在以前的身份验证代码，请重新生成项目并重试。

###项目类型

向导会检查你正在开发的项目类型，以便可以将正确的身份验证逻辑注入到项目。如果项目中有任何派生自 `ApiController` 的控制器，则项目将被视为 WebAPI 项目。如果项目中只有派生自 `MVC.Controller` 的控制器，则项目将被视为 MVC 项目。任何其他项目均不受向导支持。当前不支持 WebForms 项目。

###兼容的身份验证代码

向导还将检查是否存在以前使用向导配置的身份验证设置或与向导兼容的身份验证设置。如果所有这些设置都存在，则将视为可重入情况，向导将打开并显示这些设置。如果只存在某些设置，则将视为错误情况。

在 MVC 项目中，向导将检查是否存在以下任何设置（这些设置是以前使用向导生成的）：

	<add key="ida:ClientId" value="" />
	<add key="ida:Tenant" value="" />
	<add key="ida:AADInstance" value="" />
	<add key="ida:PostLogoutRedirectUri" value="" />

此外，在 Web API 项目中向导还会检查是否存在以下任何设置（这些设置是以前使用向导生成的）：

	<add key="ida:ClientId" value="" />
	<add key="ida:Tenant" value="" />
	<add key="ida:Audience" value="" />

###不兼容的身份验证代码

最后，向导将尝试检测使用以前版本的 Visual Studio 配置的身份验证代码版本。如果您已收到此错误，它表示您的项目包含不兼容的身份验证类型。此向导将通过以前版本的 Visual Studio 检测以下身份验证类型：

* Windows 身份验证 
* 单个用户帐户 
* 组织帐户 
 

为了检测 MVC 项目中的 Windows 身份验证，此向导将从 **web.config** 文件中查找 `authentication` 元素。

<pre>
	&lt;configuration>
	    &lt;system.web>
	        <span style="background-color: yellow">&lt;authentication mode="Windows" /></span>
	    &lt;/system.web>
	&lt;/configuration>
</pre>

为了检测 Web API 项目中的 Windows 身份验证，此向导将从项目的 **.csproj** 文件中查找 `IISExpressWindowsAuthentication` 元素：

<pre>
	&lt;Project>
	    &lt;PropertyGroup>
	        <span style="background-color: yellow">&lt;IISExpressWindowsAuthentication>enabled&lt;/IISExpressWindowsAuthentication></span>
	    &lt;/PropertyGroup>
	&lt;/Project>
</pre>

要检测单个用户帐户身份验证，此向导将从您的 **Packages.config** 文件中查找 package 元素。

<pre>
	&lt;packages>
	    <span style="background-color: yellow">&lt;package id="Microsoft.AspNet.Identity.EntityFramework" version="2.1.0" targetFramework="net45" /></span>
	&lt;/packages>
</pre>

要检测旧式的组织帐户身份验证，此向导将从 **web.config** 文件中查找以下元素：

<pre>
	&lt;configuration>
	    &lt;appSettings>
	        <span style="background-color: yellow">&lt;add key="ida:Realm" value="***" /></span>
	    &lt;/appSettings>
	&lt;/configuration>
</pre>

如需变更身份认证类型，请删除不兼容的身份验证类型，然后再次运行此向导。

有关详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。
 
<!---HONumber=Mooncake_0620_2016-->