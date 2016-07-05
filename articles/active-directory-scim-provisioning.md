<properties
	pageTitle="使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配 | Azure"
	description="Azure Active Directory 可以使用 SCIM 协议规范中定义的接口，自动将用户和组预配到以 Web 服务为前端的任何应用程序或标识存储"
	services="active-directory"
	documentationCenter=""
	authors="asmalser-msft"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/09/2016"
	wacn.date="04/28/2016"/>

#使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配

##概述

Azure Active Directory 可以使用 [SCIM 2.0 协议规范](https://tools.ietf.org/html/draft-ietf-scim-api-19)中定义的接口，将用户和组自动预配到 Web 服务前端的任何应用程序或标识存储。Azure Active Directory 可将请求发送到此 Web 服务以创建、修改和删除分配的用户与组，然后，Web 服务可将这些请求转换为针对目标标识存储的操作。

![][1]
图：通过 Web 服务从 Azure Active Directory 预配到标识存储

此功能可配合 Azure AD 中的“[自带应用](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)”功能，为提供 SCIM Web 服务或位于该服务后端的应用程序启用单一登录和自动用户预配。

Azure Active Directory 中的 SCIM 有两种使用方案：

* **将用户和组预配到支持 SCIM 的应用程序** — 支持 SCIM 2.0 并使用 OAuth 持有者令牌进行身份验证的应用程序可直接与 Azure AD 配合工作。

* **为支持其他基于 API 的预配的应用程序构建自己的预配解决方案** - 对于非 SCIM 应用程序，可以创建一个 SCIM 终结点用于在 Azure AD 的 SCIM 终结点与应用程序为用户预配支持的任何 API 之间进行转换。为了帮助开发 SCIM 终结点，我们连同代码示例提供了 CLI 库，说明如何提供 SCIM 终结点和转换 SCIM 消息。

##将用户和组预配到支持 SCIM 的应用程序

Azure Active Directory 可配置为将已分配的用户和组预配到实现[跨域标识管理系统 2 (SCIM)](https://tools.ietf.org/html/draft-ietf-scim-api-19) Web 服务、并接受使用 OAuth 持有者令牌进行身份验证的应用程序。在 SCIM 2.0 规范中，应用程序必须符合以下要求：

* 支持根据 SCIM 协议第 3.3 部分创建用户和/或组。  

* 支持根据 SCIM 协议第 3.5.2 部分修改具有修补请求的用户和/或组。

* 支持根据 SCIM 协议第 3.4.1 部分检索已知资源。

*  支持根据 SCIM 协议第 3.4.2 部分查询用户和/或组。默认情况下，用户是根据 externalId 查询的，组是根据 displayName 查询的。

* 支持根据 SCIM 协议第 3.4.2 部分，按 ID 和管理员查询用户。

* 支持根据 SCIM 协议第 3.4.2 部分，按 ID 和成员查询组。

* 接受根据 SCIM 协议第 2.1 部分使用 OAuth 持有者令牌进行授权。

你应该咨询应用程序提供者，或参阅应用程序提供者文档中的说明，以了解是否符合这些要求。
 
###入门

支持上述 SCIM 配置文件的应用程序可以使用 Azure AD 应用程序库中的“自定义”应用功能连接到 Azure Active Directory。连接后，Azure AD 将每隔 5 分钟运行同步过程，此过程将为分配的用户和组查询应用程序的 SCIM 终结点，并根据分配详细信息创建或修改这些用户和组。

**连接到支持 SCIM 的应用程序：**

1.	在 Web 浏览器中，从 https://manage.windowsazure.cn 启动 Azure 经典管理门户。
2.	浏览到“Active Directory”>“目录”>“[你的目录]”>“应用程序”，然后选择“添加”>“从库中添加应用程序”。
3.	选择左侧的“自定义”选项卡，输入应用程序的名称，然后单击复选标记图标以创建应用对象。

![][2]

4.	在出现的屏幕中，选择第二个“配置帐户预配”按钮。
5.	在“预配终结点 URL”字段中，输入应用程序的 SCIM 终结点的 URL。
6.	如果 SCIM 终结点需要来自非 Azure AD 颁发者的 OAuth 持有者令牌，那么便将所需的 OAuth 持有者令牌复制到“身份验证令牌(可选)”字段。如果此字段留空，则 Azure AD 将在每个请求中包含从 Azure AD 颁发的 OAuth 持有者令牌。使用 Azure AD 作为标识提供者的应用可以验证 Azure AD 颁发的此令牌。
7.	单击“下一步”，然后单击“开始测试”按钮，使 Azure Active Directory 尝试连接到 SCIM 终结点。如果尝试失败，将显示诊断信息。  
8.	如果尝试连接到应用程序成功，请在余下的屏幕中单击“下一步”，然后单击“完成”以退出对话框。
9.	在出现的屏幕中，选择第三个“分配帐户”按钮。在出现的“用户和组”部分中，分配你要预配到应用程序的用户或组。
10.	分配用户和组后，单击屏幕顶部附近的“配置”选项卡。
11.	在“帐户预配”下，确认“状态”设置为“打开”。 
12.	在“工具”下，单击“重新开始帐户预配”以开始预配过程。

请注意，预配过程可能需要 5-10 分钟才能开始将请求发送到 SCIM 终结点。应用程序的“仪表板”选项卡上提供了连接尝试的摘要，可以从目录的“报告”选项卡下载预配活动报告和任何预配错误。

##为任何应用程序构建你自己的预配解决方案

创建可与 Azure Active Directory 交互的 SCIM Web 服务后，可为提供 REST 或 SOAP 用户预配 API 的几乎所有应用程序启用单一登录和自动用户预配。

工作方式如下：

1.	Azure AD 提供名为 [Microsoft.SystemForCrossDomainIdentityManagement](https://www.nuget.org/packages/Microsoft.SystemForCrossDomainIdentityManagement/) 的公共语言基础结构库。系统集成商和开发商可以使用此库来创建与部署能够将 Azure AD 连接到任何应用程序的标识存储的、基于 SCIM 的 Web 服务终结点。
2.	将在 Web 服务中实现映射，以将标准化用户架构映射到用户架构和应用程序所需的协议。
3.	终结点 URL 在 Azure AD 中注册为应用程序库中自定义应用程序的一部分。
4.	用户和组在 Azure AD 中分配到此应用程序。分配后，它们将被放入队列，以同步到目标应用程序。处理队列的同步过程每隔 5 分钟运行一次。

###代码示例

为了简化此过程，我们提供了一组[代码示例](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master)，用于创建 SCIM Web 服务终结点并演示自动预配。其中一个示例是维护包含逗号分隔值行（代表用户和组的）的文件的提供程序。另一个是在 Amazon Web 服务标识与访问管理服务上运行的提供程序。

**先决条件**

* Visual Studio 2013 或更高版本
* [Azure SDK for .NET](/downloads/)
* 支持将 ASP.NET Framework 4.5 用作 SCIM 终结点的 Windows 计算机。必须能够从云访问此计算机
* [具有 Azure AD Premium 试用版或许可版的 Azure 订阅](/documentation/services/identity/)
* Amazon AWS 示例需要 [AWS Toolkit for Visual Studio](http://docs.aws.amazon.com/AWSToolkitVS/latest/UserGuide/tkv_setup.html) 中的库。请参阅示例随附的自述文件以获取其他详细信息

###入门

实现可以接受来自 Azure AD 的预配请求的 SCIM 终结点的最简单方法是构建和部署将预配的用户输出逗号分隔值 (CSV) 文件的代码示例。

**创建示例 SCIM 终结点：**

1.	从 [https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master) 下载代码示例包
2.	将包解压缩，并将它放在 Windows 计算机上的某个位置，例如 C:\\AzureAD-BYOA-Provisioning-Samples。
3.	在此文件夹中，使用 Visual Studio 启动 FileProvisioningAgent 解决方案。
4.	选择“工具”>“库程序包管理器”>“程序包管理器控制台”并执行以下命令，使 FileProvisioningAgent 项目解析解决方案引用：

    Install-Package Microsoft.SystemForCrossDomainIdentityManagement
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    Install-Package Microsoft.Owin.Diagnostics
    Install-Package Microsoft.Owin.Host.SystemWeb

5.	构建 FileProvisioningAgent 项目。
6.	在 Windows 中启动命令提示符应用程序（以管理员身分），并使用 **cd** 命令将目录切换到 **\\AzureAD-BYOA-Provisioning-Samples\\ProvisioningAgent\\bin\\Debug** 文件夹。
7.	运行以下命令，并将 <ip-address> 替换为 Windows 计算机的 IP 或域名。

    FileAgnt.exe http://<ip-address>:9000 TargetFile.csv

8.	在 Windows 中，于“Windows 设置”>“网络和 Internet 设置”下面，选择“Windows 防火墙”>“高级设置”，然后创建允许对端口 9000 进行入站访问的“入站规则”。
9.	如果 Windows 计算机位于路由器后面，则你需要将路由器配置为在面向 Internet 的端口 9000 与 Windows 计算机上的端口 9000 之间执行网络访问转换。为了使 Azure AD 能够在云中访问此终结点，必须执行此操作。


**在 Azure AD 中注册示例 SCIM 终结点：**

1.	在 Web 浏览器中，从 https://manage.windowsazure.cn 启动 Azure 经典管理门户。
2.	浏览到“Active Directory”>“目录”>“[你的目录]”>“应用程序”，然后选择“添加”>“从库中添加应用程序”。
3.	选择左侧的“自定义”选项卡，输入类似于“SCIM 测试应用”的名称，然后单击复选标记图标以创建应用对象。请注意，创建的应用程序对象代表要预配和实现登一登入的目标应用程序，而不只是 SCIM 终结点。

![][2]

4.	在出现的屏幕中，选择第二个“配置帐户预配”按钮。
5.	在对话框中，输入面向 Internet 的 URL 和 SCIM 终结点的端口。这类似于 http://testmachine.contoso.com:9000 或 http://<ip-address>:9000/，其中 <ip-address> 是面向 Internet 的 IP 地址。  
6.	单击“下一步”，然后单击“开始测试”按钮，使 Azure Active Directory 尝试连接到 SCIM 终结点。如果尝试失败，将显示诊断信息。  
7.	如果尝试连接到 Web 服务成功，请在余下的屏幕中单击“下一步”，然后单击“完成”以退出对话框。
8.	在出现的屏幕中，选择第三个“分配帐户”按钮。在出现的“用户和组”部分中，分配你要预配到应用程序的用户或组。
9.	分配用户和组后，单击屏幕顶部附近的“配置”选项卡。
10.	在“帐户预配”下，确认“状态”设置为“打开”。 
11.	在“工具”下，单击“重新开始帐户预配”以开始预配过程。

请注意，预配过程可能需要 5-10 分钟才能开始将请求发送到 SCIM 终结点。应用程序的“仪表板”选项卡上提供了连接尝试的摘要，可以从目录的“报告”选项卡下载预配活动报告和任何预配错误。

验证此示例的最后一步是打开 Windows 计算机上 \\AzureAD-BYOA-Provisioning-Samples\\ProvisioningAgent\\bin\\Debug 文件夹中的 TargetFile.csv 文件。运行预配过程后，此文件将显示所有已分配和预配的用户与组的详细信息。

###开发库

若要开发自己的符合 SCIM 规范的 Web 服务，请先熟悉 Microsoft 提供的、有助于加速开发过程的以下库：

**1：**提供公共语言基础结构库以配合基于该基础结构的语言，例如 C#。其中一个库 [Microsoft.SystemForCrossDomainIdentityManagement.Service](https://www.nuget.org/packages/Microsoft.SystemForCrossDomainIdentityManagement/) 声明接口 Microsoft.SystemForCrossDomainIdentityManagement.IProvider，如下图所示。使用这些库的开发人员将对某个类（一般称为提供程序）实现该接口。库可让开发人员轻松部署符合 SCIM 规范的 Web 服务，无论该服务是托管在 Internet 信息服务还是任何可执行的通用语言基础结构程序集中。对该 Web 服务的请求将转换为对提供程序方法的调用，这些方法由开发人员编程，以便对某些标识存储执行操作。

![][3]

**2：**提供 [ExpressRoute 处理程序](http://expressjs.com/guide/routing.html)用于分析代表对 node.js Web 服务的调用（由 SCIM 规范定义）的 node.js 请求对象。

###构建自定义 SCIM 终结点

开发人员可以使用上述库将其服务托管在任何可执行的通用语言基础结构程序集或 Internet 信息服务中。以下代码示例用于将服务托管在地址为 http://localhost:9000 的可执行程序集中：

    private static void Main(string[] arguments)
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IProvider and 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  
    
    Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
      new DevelopersMonitor();
    Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
      new DevelopersProvider(arguments[1]);
    Microsoft.SystemForCrossDomainIdentityManagement.Service webService = null;
    try
    {
        webService = new WebService(monitor, provider);
        webService.Start("http://localhost:9000");

        Console.ReadKey(true);
    }
    finally
    {
        if (webService != null)
        {
            webService.Dispose();
            webService = null;
        }
    }
    }

    public class WebService : Microsoft.SystemForCrossDomainIdentityManagement.Service
    {
    private Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor;
    private Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider;

    public WebService(
      Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitoringBehavior, 
      Microsoft.SystemForCrossDomainIdentityManagement.IProvider providerBehavior)
    {
        this.monitor = monitoringBehavior;
        this.provider = providerBehavior;
    }

    public override IMonitor MonitoringBehavior
    {
        get
        {
            return this.monitor;
        }

        set
        {
            this.monitor = value;
        }
    }

    public override IProvider ProviderBehavior
    {
        get
        {
            return this.provider;
        }

        set
        {
            this.provider = value;
        }
    }
    }

请务必注意，此服务必须具有 HTTP 地址，其服务器身份验证凭证的根证书颁发机构是下列其中之一：

* CNNIC
* Comodo
* CyberTrust
* DigiCert
* GeoTrust
* GlobalSign
* Go Daddy
* Verisign
* WoSign

可以使用网络 shell 实用程序将服务器身份验证证书绑定到 Windows 主机上的某个端口，例如：

    netsh http add sslcert ipport=0.0.0.0:443 certhash=0000000000003ed9cd0c315bbb6dc1c08da5e6 appid={00112233-4455-6677-8899-AABBCCDDEEFF}  
 
此处，为 certhash 参数提供的值为证书指纹，为 appid 参数提供的值为任意全局唯一标识符。

若要将服务托管在 Internet 信息服务中，开发人员需构建一个通用语言基础结构代码库程序集，并在该程序集的默认命名空间中使用名为 Startup 的类。以下是这种类的示例：

    public class Startup
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor and  
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  

    Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter starter;

    public Startup()
    {
        Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
          new DevelopersMonitor();
        Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
          new DevelopersProvider();
        this.starter = 
          new Microsoft.SystemForCrossDomainIdentityManagement.WebApplicationStarter(
            provider, 
            monitor);
    }

    public void Configuration(
      Owin.IAppBuilder builder) // Defined in in Owin.dll.  
    {
        this.starter.ConfigureApplication(builder);
    }
    }

###处理终结点身份验证

来自 Azure Active Directory 的请求包括 OAuth 2.0 持有者令牌。接收请求的任何服务应该代表所需的 Azure Active Directory 租户将颁发者作为 Azure Active Directory 进行身份验证，以访问 Azure Active Directory 的 Graph Web 服务。在令牌中，颁发者由 iss 声明，例如："iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/" 。在此示例中，声明值的基地址 https://sts.windows.net 将 Azure Active Directory 标识为颁发者，而相对地址段 cbb1a5ac-f33b-45fa-9bf5-f37db0fed422 代表颁发令牌时 Azure Active Directory 租户的唯一标识符。如果颁发的令牌用于访问 Azure Active Directory 的 Graph Web 服务，该服务的标识符 00000002-0000-0000-c000-000000000000 应在令牌的 aud 声明值中。

使用 Microsoft 提供的通用语言基础结构库构建 SCIM 服务的开发人员可以按照以下步骤使用 Microsoft.Owin.Security.ActiveDirectory 包对 Azure Active Directory 的请求进行身份验证：

**1：**在提供程序中，通过每次启动服务时让服务返回要调用的方法来实现 Microsoft.SystemForCrossDomainIdentityManagement.IProvider.StartupBehavior 属性：

    public override Action<Owin.IAppBuilder, System.Web.Http.HttpConfiguration.HttpConfiguration> StartupBehavior
    {
      get
      {
        return this.OnServiceStartup;
      }
    }

    private void OnServiceStartup(
      Owin.IAppBuilder applicationBuilder,  // Defined in Owin.dll.  
      System.Web.Http.HttpConfiguration configuration)  // Defined in System.Web.Http.dll.  
    {
    }

**2：**将以下代码添加到该方法，以代表指定的租户对所有服务终结点的所有请求进行身份验证，以确定它们是否持有 Azure Active Directory 颁发的、用于访问 Azure Active Directory 的 Graph Web 服务的令牌：

    private void OnServiceStartup(
      Owin.IAppBuilder applicationBuilder IAppBuilder applicationBuilder, 
      System.Web.Http.HttpConfiguration HttpConfiguration configuration)
    {
      // IFilter is defined in System.Web.Http.dll.  
      System.Web.Http.Filters.IFilter authorizationFilter = 
        new System.Web.Http.AuthorizeAttribute(); // Defined in System.Web.Http.dll.configuration.Filters.Add(authorizationFilter);

      // SystemIdentityModel.Tokens.TokenValidationParameters is defined in    
      // System.IdentityModel.Token.Jwt.dll.
      SystemIdentityModel.Tokens.TokenValidationParameters tokenValidationParameters =     
        new TokenValidationParameters()
        {
          ValidAudience = "00000002-0000-0000-c000-000000000000"
        };

      // WindowsAzureActiveDirectoryBearerAuthenticationOptions is defined in 
      // Microsoft.Owin.Security.ActiveDirectory.dll
      Microsoft.Owin.Security.ActiveDirectory.
      WindowsAzureActiveDirectoryBearerAuthenticationOptions authenticationOptions =
        new WindowsAzureActiveDirectoryBearerAuthenticationOptions()    {
        TokenValidationParameters = tokenValidationParameters,
        Tenant = "03F9FCBC-EA7B-46C2-8466-F81917F3C15E" // Substitute the appropriate tenant’s 
                                                      // identifier for this one.  
      };

      applicationBuilder.UseWindowsAzureActiveDirectoryBearerAuthentication(authenticationOptions);
    }

##用户和组架构

Azure Active Directory 可将两种类型的资源预配到 SCIM Web 服务。这些类型的资源是用户和组。

用户资源由协议规范 http://tools.ietf.org/html/draft-ietf-scim-core-schema 中包含的架构标识符 urn:ietf:params:scim:schemas:extension:enterprise:2.0:User 标识。以下表 1 提供了 Azure Active Directory 中用户属性与 urn:ietf:params:scim:schemas:extension:enterprise:2.0:User 资源属性之间的默认映射。

组资源由架构标识符 http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group 标识。下面的表 2 显示了 Azure Active Directory 中组属性与 http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group 资源属性之间的默认映射。

###表 1：默认用户属性映射

| Azure Active Directory 用户 | urn:ietf:params:scim:schemas:extension:enterprise:2.0:User |
| ------------- | ------------- |
| IsSoftDeleted | 活动 |
| displayName | displayName |
| Facsimile-TelephoneNumber | phoneNumbers[type eq "fax"].value |
| givenName | name.givenName |
| jobTitle | title |
| mail | emails[type eq "work"].value |
| mailNickname | externalId |
| manager | manager |
| mobile | phoneNumbers[type eq "mobile"].value |
| objectId | id |
| postalCode | addresses[type eq "work"].postalCode |
| proxy-Addresses | emails[type eq "other"].Value |
| physical-Delivery-OfficeName | addresses[type eq "other"].Formatted |
| streetAddress | addresses[type eq "work"].streetAddress |
| surname | name.familyName |
| telephone-Number | phoneNumbers[type eq "work"].value |
| user-PrincipalName | userName |


###表 2：默认组属性映射

| Azure Active Directory 组 | http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group |
| ------------- | ------------- |
| displayName | externalId |
| mail | emails[type eq "work"].value |
| mailNickname | displayName |
| members | members |
| objectId | id |
| proxyAddresses | emails[type eq "other"].Value |


##用户预配和取消预配

下图显示了 Azure Active Directory 将发送到 SCIM 服务以管理用户在其他标识存储中的生命周期的消息。该图还显示了使用 Microsoft 提供的、用于构建此类服务的通用语言基础结构库所实现的 SCIM 服务如何将这些请求转换为对提供程序的方法调用。

![][4] 
图：用户预配和撤销顺序

**1：**Azure Active Directory 将在服务中查询是否有某个用户的 externalId 属性值与 Azure Active Directory 中用户的 mailNickname 属性值匹配。查询以类似的超文本传输协议请求表示，其中，jyoung 是 Azure Active Directory 中某个用户的 mailNickname 示例：

    GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
    Authorization: Bearer ...

如果使用 Microsoft 提供的、用于实现 SCIM 服务的通用语言基础结构库构建了服务，则将请求转换为对服务提供者的 Query 方法调用。以下是该方法的签名：

    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Protocol.  

    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource[]> Query(
      Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters parameters, 
      string correlationIdentifier);

以下是 Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters 接口的定义：

    public interface IQueryParameters: 
      Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
    {
        System.Collections.Generic.IReadOnlyCollection <Microsoft.SystemForCrossDomainIdentityManagement.IFilter> AlternateFilters 
        { get; }
    }

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
	{
      system.Collections.Generic.IReadOnlyCollection<string> ExcludedAttributePaths 
      { get; }
      System.Collections.Generic.IReadOnlyCollection<string> RequestedAttributePaths 
      { get; }
      string SchemaIdentifier 
	  { get; }
    }

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IFilter
    {
        Microsoft.SystemForCrossDomainIdentityManagement.IFilter AdditionalFilter 
          { get; set; }
        string AttributePath 
          { get; } 
        Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator FilterOperator 
          { get; }
        string ComparisonValue 
          { get; }
    }
    
    public enum Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator
    {
        Equals
    }

在上述查询具有给定 externalId 属性值的用户的示例中，传递给 Query 方法的参数值将是：

* parameters.AlternateFilters.Count: 1
* parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
* parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"
* correlationIdentifier: System.Net.Http.HttpRequestMessage.GetOwinEnvironment["owin.RequestId"] 

**2：**如果在服务中查询是否有某个用户的 externalId 属性值与 Azure Active Directory 中用户的 mailNickname 属性值匹配时，该查询的响应未返回任何用户，Azure Active Directory 将请求服务预配与 Azure Active Directory 中的用户相对应的用户。以下是此类请求的示例：

    POST https://.../scim/Users HTTP/1.1
    Authorization: Bearer ...
    Content-type: application/json
    {
      "schemas":
      [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0User"],
      "externalId":"jyoung",
      "userName":"jyoung",
      "active":true,
      "addresses":null,
      "displayName":"Joy Young",
      "emails": [
        {
          "type":"work",
          "value":"jyoung@Contoso.com",
          "primary":true}],
      "meta": {
        "resourceType":"User"},
       "name":{
        "familyName":"Young",
        "givenName":"Joy"},
      "phoneNumbers":null,
      "preferredLanguage":null,
      "title":null,
      "department":null,
      "manager":null}

Microsoft 提供的、用于实现 SCIM 服务的通用语言基础结构库将请求转换为对服务提供者的 Create 方法调用。Create 方法具有此签名：

    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  

    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource> Create(
      Microsoft.SystemForCrossDomainIdentityManagement.Resource resource, 
      string correlationIdentifier);

如果请求预配用户，资源参数的值将是 Microsoft.SystemForCrossDomainIdentityManagement 的实例。Core2EnterpriseUser 类，在 Microsoft.SystemForCrossDomainIdentityManagement.Schemas 库中定义。如果预配用户的请求成功，则方法的实现应返回 Microsoft.SystemForCrossDomainIdentityManagement 的实例。Core2EnterpriseUser 类，其 Identifier 属性值设置为新预配用户的唯一标识符。

**3：**为了更新已知存在于前端为 SCIM 的标识存储中的用户，Azure Active Directory 将通过类似于下面的请求向服务请求该用户的当前状态来继续处理：

    GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...

如果使用 Microsoft 提供的、用于实现 SCIM 服务的通用语言基础结构库构建了服务，则将请求转换为对服务提供者的 Retrieve 方法调用。以下是 Retrieve 方法的签名：

    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource and 
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 
    // are defined in Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource> 
       Retrieve(
         Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 
           parameters, 
           string correlationIdentifier);
    
    public interface 
      Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters:   
        IRetrievalParameters
        {
          Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier 
            ResourceIdentifier 
              { get; }
    }
    public interface Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier
    {
        string Identifier 
          { get; set; }
        string Microsoft.SystemForCrossDomainIdentityManagement.SchemaIdentifier 
          { get; set; }
    }

对于上述检索用户当前状态的请求示例，作为参数自变量值提供的对象属性值如下所示：

* Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

**4：**若要更新引用属性，Azure Active Directory 将查询服务以判断前端为该服务的标识存储中引用属性的当前值是否已经与 Azure Active Directory 中该属性的值相匹配。对于用户，以这种方式查询当前值的唯一属性是 manager 属性。确定特定用户对象的 manager 属性当前是否具有特定值的请求示例如下：

    GET ~/scim/Users?filter=id eq 54D382A4-2050-4C03-94D1-E769F1D15682 and manager eq 2819c223-7f76-453a-919d-413861904646&attributes=id HTTP/1.1
    Authorization: Bearer ...

属性查询参数 id 的值，表示如果满足提供为筛选器查询参数值的表达式的用户对象存在，则服务应以 urn:ietf:params:scim:schemas:core:2.0:User 或 urn:ietf:params:scim:schemas:extension:enterprise:2.0:User 资源做出响应（仅包括该资源的 id 属性值）。当然，请求者知道 id 属性的值 — 它包含在筛选器查询参数的值中；请求它的目的实际上是请求满足筛选表达式的资源的精简表示形式（指示是否存在任何此类对象）。

如果使用 Microsoft 提供的、用于实现 SCIM 服务的通用语言基础结构库构建了服务，则将请求转换为对服务提供者的 Query 方法调用。作为参数自变量值提供的对像属性值如下：

* parameters.AlternateFilters.Count: 2
* parameters.AlternateFilters.ElementAt(x).AttributePath: "id"
* parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
* parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
* parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
* parameters.RequestedAttributePaths.ElementAt(0): "id"
* parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

此处，索引 x 的值可以是 0 并且索引 y 的值可以是 1，或者，x 值可以是 1 并且 y 的值可以是 0，具体根据筛选器查询参数表达式的顺序而定。

**5：**以下是从 Azure Active Directory 向 SCIM 服务发出更新用户请求的示例：

    PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
    Content-type: application/json
    {
      "schemas": 
      [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
      "Operations":
      [
        {
          "op":"Add",
          "path":"manager",
          "value":
            [
              {
                "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
                "value":"2819c223-7f76-453a-919d-413861904646"}]}]}

用于实现 SCIM 服务的 Microsoft 通用语言基础结构库将请求转换为对服务提供者的 Update 方法调用。以下是该方法的签名：

    // System.Threading.Tasks.Tasks and 
    // System.Collections.Generic.IReadOnlyCollection<T>
    // are defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IPatch, 
    // Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
    // Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation, 
    // Microsoft.SystemForCrossDomainIdentityManagement.OperationName, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IPath and 
    // Microsoft.SystemForCrossDomainIdentityManagement.OperationValue 
    // are all defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 

    System.Threading.Tasks.Task Update(
      Microsoft.SystemForCrossDomainIdentityManagement.IPatch patch, 
      string correlationIdentifier);

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IPatch
    {
    Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase 
      PatchRequest 
        { get; set; }
    Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier 
      ResourceIdentifier 
        { get; set; }        
    }

    public class PatchRequest2: 
      Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase
    {
    public System.Collections.Generic.IReadOnlyCollection
      <Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation> 
        Operations
        { get;}

    public void AddOperation(
      Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation operation);
    }

    public class PatchOperation
    {
    public Microsoft.SystemForCrossDomainIdentityManagement.OperationName 
      Name
      { get; set; }
    
    public Microsoft.SystemForCrossDomainIdentityManagement.IPath 
      Path
      { get; set; }

    public System.Collections.Generic.IReadOnlyCollection
      <Microsoft.SystemForCrossDomainIdentityManagement.OperationValue> Value
      { get; }

    public void AddValue(
      Microsoft.SystemForCrossDomainIdentityManagement.OperationValue value);
    }

    public enum OperationName
    {
      Add,
      Remove,
      Replace
    }

    public interface IPath
    {
      string AttributePath { get; }
      System.Collections.Generic.IReadOnlyCollection<IFilter> SubAttributes { get; }
      Microsoft.SystemForCrossDomainIdentityManagement.IPath ValuePath { get; }
    }

    public class OperationValue
    {
      public string Reference
      { get; set; }
      
      public string Value
      { get; set; }
    }



对于上述更新用户的请求示例，作为修补参数值提供的对象将具有这些属性值：

* ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
* (PatchRequest as PatchRequest2).Operations.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName: OperationName.Add
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath: "manager"
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference: http://.../scim/Users/2819c223-7f76-453a-919d-413861904646
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value: 2819c223-7f76-453a-919d-413861904646

**6：**为了从前端为 SCIM 服务的标识存储撤销用户，Azure Active Directory 将发送如下请求：

    DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
	
如果使用 Microsoft 提供的、用于实现 SCIM 服务的通用语言基础结构库构建了服务，则将请求转换为对服务提供者的 Delete 方法调用。该方法具有以下签名：

    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
    // is defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 
    System.Threading.Tasks.Task Delete(
      Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier  
        resourceIdentifier, 
      string correlationIdentifier);
 
在上述取消预配用户的请求示例中，作为 resourceIdentifier 参数值提供的对象将具有以下属性值：

* ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

##组预配和取消预配

下图显示了 Azure Active Directory 将发送到 SCIM 服务以管理组在其他标识存储中的生命周期的消息。这些消息在以下三个方面与用户相关的消息不同：

* 组资源的架构标识为 http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group。  
* 检索组的请求规定将成员属性从请求响应中提供的任何资源中排除。  
* 确定引用属性是否具有特定值的请求将是有关成员属性的请求。  

![][5]
图：组预配和撤销顺序

	
<!--Image references-->
[1]: ./media/active-directory-scim-provisioning/scim-figure-1.PNG
[2]: ./media/active-directory-scim-provisioning/scim-figure-2.PNG
[3]: ./media/active-directory-scim-provisioning/scim-figure-3.PNG
[4]: ./media/active-directory-scim-provisioning/scim-figure-4.PNG
[5]: ./media/active-directory-scim-provisioning/scim-figure-5.PNG

<!---HONumber=Mooncake_0620_2016-->