<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="使用 Azure Active Directory 开发多租户云应用程序" authors="" solutions="" manager="" editor="" />

# 使用 Azure Active Directory 开发多租户云应用程序

## <a name="introduction"></a>介绍

Azure Active Directory (Azure AD) 是一项基于 REST 的新型服务，它可为云应用程序提供标识管理和访问控制功能。Azure AD 可与云服务，以及 Azure、Microsoft Office 365、Dynamics CRM Online 和 Windows Intune 轻松集成。现有的本地 Active Directory 部署还可以充分利用 Azure AD。要了解详细信息，请参阅 [windowsazure.cn][windowsazure.cn] 上的[标识页][标识页]。

本演练面向想要将多租户应用程序与 Azure AD 集成的 .NET 开发人员。你将了解如何执行以下操作：

-   让客户能够使用 Azure AD 注册您的应用程序
-   启用 Azure AD 的单一登录 (SSO)
-   查询使用 Azure AD Graph API 的客户的目录数据

本演练的配套示例应用程序可在[此处下载][此处下载]。该示例无需更改即可运行，但您可能需要更改自己的 [Visual Studio 中的端口分配][Visual Studio 中的端口分配]以使用 https。请按照链接中的说明进行操作，但在 ApplicationHost.config 文件的绑定部分中将绑定协议设置为 "https"。下面步骤中的所有代码段都从示例提取。

> [WACOM.NOTE]
> 提供多租户目录的应用程序示例仅供说明目的。此示例（包括其帮助程序库类）不应用在生产中。

### 先决条件

本演练具有以下开发人员先决条件：

-   [Visual Studio 2012][Visual Studio 2012]
-   [WCF Data Services for OData][WCF Data Services for OData]

### 目录

-   [介绍][介绍]
-   [第 1 部分：为访问 Azure AD 获取客户端 ID][第 1 部分：为访问 Azure AD 获取客户端 ID]
-   [第 2 部分：允许客户使用 Azure AD 注册][第 2 部分：允许客户使用 Azure AD 注册]
-   [第 3 部分：启用单一登录][第 3 部分：启用单一登录]
-   [第 4 部分：访问 Azure AD Graph][第 4 部分：访问 Azure AD Graph]
-   [第 5 部分：发布应用程序][第 5 部分：发布应用程序]
-   [摘要][摘要]

## <a name="getclientid"></a>第 1 部分：为访问 Azure AD 获取客户端 ID

本部分介绍在您创建 Microsoft Seller Dashboard 帐户后如何获取客户端 ID 和客户端机密。客户端 ID 是应用程序的唯一标识符，客户端机密是发出使用客户端 ID 请求时所需的相关密钥。两项都需要才能将您的应用程序与 Azure AD 集成。

### 步骤 1：使用 Microsoft Seller Dashboard 创建一个帐户

要开发和发布与 Azure AD 集成的应用程序，必须注册一个 [Microsoft Seller Dashboard][Microsoft Seller Dashboard] 帐户。然后系统将提示您作为一家公司或个人[创建帐户配置文件][创建帐户配置文件]。此配置文件用于在 Azure Marketplace 或其他市场发布应用程序，生成客户端 ID 和客户端机密也需要它。

新帐户处于“帐户等待批准”状态。此状态不会阻止您开始开发 - 您仍可创建客户端 ID 以及草稿应用程序列表。但帐户本身经过审批后，应用程序列表只能提交以供审核。获得批准后，提交的应用程序列表只对 Azure Marketplace 中的客户可见。

### 步骤 2：为您的应用程序获取客户端 ID

需要客户端 ID 和客户端机密才能将您的应用程序与 Azure AD 集成。客户端 ID 是应用程序的唯一标识符，主要用于标识应用程序的单一登录或对 Azure AD Graph 进行身份验证调用。有关获取客户端 ID 和客户端机密的详细信息，请参阅[在 Microsoft Seller Dashboard 中创建客户端 ID 和机密][在 Microsoft Seller Dashboard 中创建客户端 ID 和机密]。

> [WACOM.NOTE]
> 本演练中稍后将需要您的客户端 ID 和客户端机密，因此请确保记录它们。

要生成客户端 ID 和客户端机密，您需要在 Microsoft Seller Dashboard 中输入以下属性：

**应用域**：应用程序的主机名，例如 "contoso.com"。此属性不得包含任何端口号。在开发期间，此属性应设置为 "localhost"。

**应用程序重定向 URL**：用户登录后并当一个组织已授权您的应用程序时，Azure AD 将发送响应的重定向 URL，例如："https://contoso.com/"。在开发期间，此属性应设置为 "https://localhost: \< 端口号 \>"

### 步骤 3：配置应用程序以使用客户端 ID 和客户端机密

此步骤需要在 Seller Dashboard 上的注册过程中生成的客户端 ID 和客户端机密。客户端 ID 用于 SSO，并且客户端 ID 和客户端机密将用于在稍后获取访问令牌以调用 Azure AD Graph API。

示例应用程序以预先连接以使用 Azure AD，并从 config 加载客户端 ID 和客户端机密。在示例应用程序中的 **Web.config** 文件里，进行以下更改：

1.  在 **appSettings** 节点中，用 "clientId" 和 "SymmetricKey" 的值替您的客户端 ID、客户端机密和应用域：

        <appSettings>
            <add key="clientId" value="(Your Client ID value)"/>
            <add key="SymmetricKey" value="(Your Client Secret value)"/>
            <add key="AppHostname" value="(Your App Domain)"/>
        </appSettings>

2.  在 **system.identityModel** 的 **audienceUris** 节点中，在 "spn:" 后面插入您的客户端 ID：

        <system.identityModel>
            <audienceUris>
                <add value="spn:(Your Client ID value)" />
            </audienceUris>

## <a name="enablesignup"></a>第 2 部分：允许客户使用 Azure AD 注册

本部分介绍如何让客户使用 Azure AD 注册您的应用程序。客户可使用与 Azure AD 集成的应用程序前，租户管理员必须授权该应用程序。此授权流程从您的应用程序到 Azure 的后续请求开始，这将生成必须由您的应用程序处理的一个响应。以下步骤介绍如何生成后续请求并处理响应。

本部分中的步骤使用来自示例应用程序的帮助器类。这些类包含在示例的 *Microsoft.IdentityModel.WAAD.Preview* 库中，并且可以使用户更容易专注于应用程序代码而非标识和协议详情。

### 步骤 1：为您的应用程序请求同意

下面的示例交互演示为您的应用程序请求同意的过程：

1.  客户单击您的应用程序中的网页上的“注册使用 Azure AD”链接。
2.  您将客户重定向到 Azure AD 授权页面，应用程序的信息附加到该请求。
3.  客户可授权或拒绝您的应用程序的同意。
4.  Azure AD 将客户重定向到您指定的“应用重定向 URL”。在 Microsoft Seller Dashboard 上生成客户端 ID 和客户端机密时，您指定此 URL。重定向请求表示同意请求的结果，如果授予同意则包括有关其租户的信息。

要在上面的步骤 2 中生成重定向请求，必须为 Azure AD 授权页将查询字符串参数附加到以下 URL：*http://activedirectory.windowsazure.com/Consent/AuthorizeApplication.aspx*

查询字符串参数的描述如下：

**ApplicationID**：（必需）在 Seller Dashboard 中收到的 **ClientID** 值。

**RequestedPermissions**：（可选）必须按租户对您的应用程序授予权限。
在开发期间，这些权限用于测试未发布的应用程序。对于发布的应用程序，将忽略此参数。相反，将使用您的应用程序列表中所请求的权限。有关此列表的详细信息，请参阅第 5 部分。
此参数有三个可能的值：

**DirectoryReader**：授予权限以读取目录数据，例如用户帐户、组和有关您组织的信息。启用 SSO。

**UserAccountAdministrator**：授予权限以读取和写入数据，如用户、组和有关您组织的信息。启用 SSO。

**None**：启用单一登录，但禁用对目录数据的访问。

如果未指定参数或参数指定错误，则默认值为 "None"。

以下是一个有效同意请求 URL 的示例：
*https://activedirectory.windowsazure.com/Consent/AuthorizeApplication.aspx?ApplicationId=33E48BD5-1C3E-4862-BA79-1C0D2B51FB26&RequestedPermissions=DirectoryReader*

在示例应用程序中，“注册”链接包含用于同意请求的类似 URL，如下所示：

![登录][登录]

> [WACOM.NOTE]
> 测试您未发布的应用程序时，您将经历类似于客户的同意体验。但是，未发布应用程序的授权页外观不同于已发布应用程序的授权页。发布的应用程序显示您的应用程序名称、徽标和发布者的详细信息，而未发布的应用程序没有。

### 步骤 2：处理同意响应

客户授予或拒绝同意您的应用程序后，Azure AD 将响应发送到您的应用程序的应用重定向 URL。此响应包含下面的查询字符串参数：

**TenantId**：已授权您的应用程序的 Azure AD 租户的唯一 GUID。只有客户已授权您的应用程序，才能指定此参数。

**Consent**：如果已授权应用程序，值将设置为 "Granted"；如果拒绝该请求则为 "Denied"。

下面是对表示已授权应用程序的同意请求的有效响应示例：
*https://app.litware.com/redirect.aspx&TenantId=7F3CE253-66DB-4AEF-980A-D8312D76FDC2&Consent=Granted*

您的应用程序将需要保持上下文，以便发送到 Azure AD 授权页面的请求与该响应关联（并且将拒绝不带相关联请求的任何响应）。

<div class="dev-callout"><strong>说明</strong><p>授予同意后，Azure AD 可能需要一些时间再设置 SSO 和 Graph 访问。设置完成前，每个组织中注册您应用程序的第一个用户将看到登录错误。</p></div>

客户授予同意您的应用程序后，存储并将应用程序中新创建的租户与同意响应返回的 TenantId 关联很重要。示例应用程序包含 *Microsoft.IdentityModel.WAAD.Preview.Consent* 命名空间中的 *HttpModule*，它将 TenantId 自动记录到所有成功同意响应上的客户/TenantId“数据存储”。下面包括用于它的代码，并通过 *TrustedIssuers.Add* 方法执行 TenantId 到客户/TenantId“数据存储”的记录。

    private void Application_BeginRequest(Object source,
             EventArgs e)
    {
        HttpApplication application = (HttpApplication)source;
        HttpRequest req = application.Context.Request;             

        if((!string.IsNullOrEmpty(req.QueryString["TenantId"]) && (!string.IsNullOrEmpty(req.QueryString["Consent"]))))
        { 
            if(req.QueryString["Consent"].Equals("Granted",StringComparison.InvariantCultureIgnoreCase))
            {
                // For this sample we store the consenting tenants in
                // an XML file. We strongly recommend that you change
                // this to use your DataStore
                TrustedIssuers.Add(req.QueryString["TenantId"]; 
            }
        }            
    }

可以对您的应用程序测试同意请求/响应代码之前，必须获取一个 Azure AD 目录租户。

### 步骤 3：获取一个 Azure AD 租户以测试您的应用程序

要测试您的应用程序与 Azure AD 集成的功能，您需要一个 Azure AD 租户。如果您已具有用于测试其他应用程序的租户，则可以重复使用它。我们建议至少获取两个租户，以确保多个租户测试和使用您的应用程序。不建议将生产租户用于此目的。[获取 Azure AD 租户][获取 Azure AD 租户]。

一旦获得 Azure AD 租户，通过按 **F5** 可以生成并运行该应用程序。此外，可以尝试使用新租户注册您的应用程序。

<div class="dev-callout"><strong>说明</strong><p>如果您的客户注册为新的 Azure AD 租户，可能需要一些时间才能完全设置该租户。设置完成前，用户可能在同意页上看到错误。</p></div>

## <a name="enablesso"></a>第 3 部分：启用单一登录

本部分演示如何启用单一登录 (SSO)。过程从将登录请求构建到 Azure AD 开始，对您应用程序的用户进行身份验证，然后在登录响应中验证客户是否属于已授权您的应用程序的租户。登录请求从 Seller Dashboard 要求您的客户端 ID 和客户组织的租户 ID。

登录请求特定于一个目录租户，并且必须包括 TenantID。TenantID 可从 Azure AD 目录租户的域名来确定。有两种常用的方法可在最终用户登录时获取他们的域名：

-   如果应用程序的 URL 是 *https://contoso.myapp.com* 或 *https://myapp.com/contoso.com*，*contoso* 和 *contoso.com* 表示 Azure AD 域名，而 *myapp.com* 表示应用程序的 URL。
-   您的应用程序可提示用户的电子邮件地址或其 Azure AD 域名。此方法用在示例应用程序中，用户必须在其中输入 Azure AD 域名，如下所示：

![登录][登录]

### 步骤 1：查找租户 ID

使用客户提供的 Azure AD 域名，您可以通过分析 Azure AD 联合元数据查找其租户 ID。在示例应用程序中，此过程通过 *Microsoft.IdentityModel.WAAD.Preview.TenantUtils.Globals* 类的 *Domain2TenantId* 方法处理。

为演示这一过程，以下步骤使用 contoso.com 域名。

1.  为 Azure AD 租户获取 **FederationMetadata.xml** 文件。例如：
    *https://accounts.accesscontrol.windows.net/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml*
2.  在 **FederationMetadata.xml** 文件中，找到**实体描述符**条目。租户 ID 包含作为 **entityID** 属性的一部分，遵循 "https://sts.windows.net"，如下所示：

         <EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" entityID="https://sts.windows.net/a7456b11-6fe2-4e5b-bc83-67508c201e4b/" ID="_cba45203-f8f4-4fc3-a3bb-0b136a2bafa5"> 

    在这种情况下，TenantID 值是 **a7456b11-6fe2-4e5b-bc83-67508c201e4b**。

3.  在您的应用程序的客户/TenantId“数据存储区”，您应存储域及其相关联的 TenantID。这两个值可一起用于将来的登录请求，并且无需每次获取 **FederationMetadata.xml**。示例应用程序并不介绍这种优化。

### 步骤 2：生成登录请求

当客户登录您的应用程序时，例如通过单击登录按钮，登录请求必须通过使用客户的租户 ID 和应用程序的客户端 ID 生成。在示例应用程序中，此请求由 *Microsoft.IdentityModel.WAAD.Preview.WebSSO.URLUtils* 类的 *GenerateSignInMessage* 方法生成。此方法验证客户的 TenantID 是否代表已授权您的应用程序的组织，并且它为登录按钮生成目的地 URL，如下所示：

![登录][登录]

单击该按钮会将用户浏览器的登录页导航到 Azure AD。一旦登录，Azure AD 将向应用程序返回登录响应。

### 步骤 3：验证登录响应中的传入令牌的颁发者

当客户登录您的应用程序时，您需要验证他们的租户已授权您的应用程序。其登录响应包含一个令牌，并且该令牌包含您的应用程序可检查的属性和声明。

要执行此验证，您必须从令牌中的“颁发者”属性获取 TenantID，并确保它存在于客户/TenantId“数据存储区”中。在示例应用程序中，此验证通过创建自定义令牌处理程序类实现，它扩展 Windows Identity Foundation 的 *Saml2SecurityTokenHandler*。自定义令牌处理程序检查传入安全令牌并将其声明和属性提供给应用程序，以便验证 TenantID。此类的代码段如下所示。

在示例应用程序中，可在 *Microsoft.IdentityModel.WAAD.Preview.WebSSO* 命名空间下面找到原始代码。令牌处理程序还使用 *Microsoft.IdentityModel.WAAD.Preview.WebSSO.TrustedIssuers* 类的 Contains 方法，它验证 TenantID 是否保留在客户/TenantId“数据存储区”。

    /// <summary>
    /// Extends the out of the box SAML2 token handler by ensuring
    /// that incoming tokens have been issued by registered tenants 
    /// </summary>
    public class ConfigurationBasedSaml2SecurityTokenHandler : Saml2SecurityTokenHandler
    {
        public override ReadOnlyCollection<System.Security.Claims.ClaimsIdentity> ValidateToken(SecurityToken token)
        {
            ReadOnlyCollection<System.Security.Claims.ClaimsIdentity> aa = base.ValidateToken(token);
            Saml2SecurityToken ss = token as Saml2SecurityToken;
            string tenant = ss.Assertion.Issuer.Value.Split('/')[3];
            if (!TrustedIssuers.Contains(tenant))
            {
                throw new SecurityTokenValidationException(string.Format("The tenant {0} is not registered with the application", tenant));
            }
            return aa;
        }
    }

验证令牌后，用户登录应用程序。运行该应用程序，并使用以前创建的已同意租户中的 Azure AD 帐户登录。

## <a name="accessgraph"></a>第 4 部分：访问 Azure AD Graph

本部分介绍如何获取访问令牌和调用 Azure AD Graph API，以访问租户的目录数据。例如，尽管在登录过程中获取的令牌包含用户信息（如姓名和电子邮件地址），您的应用程序可能需要组成员身份或用户的管理程序名称的信息。使用 Graph API，可从租户的目录获取此信息。有关 Graph API 的详细信息，请参阅[本主题][本主题]。

应用程序可调用 Azure AD Graph 前，它必须对自身进行身份验证并获取访问令牌。访问令牌通过客户端 ID 和客户端机密验证应用程序的身份获取。以下步骤将介绍如何操作：

1.  使用生成的代理类调用 Azure AD Graph
2.  使用 Azure 身份验证库 (AAL) 获取访问令牌
3.  调用 Azure AD Graph，获取租户用户的列表

<div class="dev-callout"><strong>说明</strong><p>示例应用程序帮助程序库 Microsoft.IdentityModel.WAAD.Preview 已经包含一个自动生成的代理类（通过将服务引用添加到调用 GraphService 的 https://graph.windows.net/your-domain-name 创建）。该应用程序将使用此代理类来调入 Azure AD Graph 服务。</p></div>

### 步骤 1：使用代理类调用 Azure AD Graph

在此步骤中，我们将使用示例应用程序说明如何操作：

1.  创建特定于租户的 Azure AD Graph 终结点
2.  使用终结点实例化代理以调用 Graph
3.  将授权标头添加到请求并获取令牌。

在示例应用程序中，这些对 API 的调用由 *Microsoft.IdentityModel.WAAD.Preview.Graph.GraphInterface* 类中的 GraphInterface 方法处理，如下面的代码中所示。

    public GraphInterface()
    {
        // 1a: When the customer was signed in, we get a security token 
        // that contains a tenant id. Extract that here
        TenantDomainName = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/ws/2012/10/identity/claims/tenantid").Value;

        // 1b: We generate a URL (https://graph.windows.net/<CustomerDomainName>)
        // to access the Azure AD Graph API endpoint for the tenant 
        connectionUri = new Uri(string.Format(@"https://{0}/{1}", TenantUtils.Globals.endpoint, TenantDomainName));

        // 2: create an instance of the AzureAD Service proxy with the connection URL
        dataService = new DirectoryDataService(connectionUri);

        // This flags ignores the resource not found exception
        // If AzureAD Service throws this exception, it returns null
        dataService.IgnoreResourceNotFoundException = true;
        dataService.MergeOption = MergeOption.OverwriteChanges;
        dataService.AddAndUpdateResponsePreference = DataServiceResponsePreference.IncludeContent;

        // 3: This adds the default required headers to each request
        AddHeaders(GetAuthorizationHeader());
    }

### 步骤 2：使用 Azure 身份验证库获取访问令牌

示例应用程序使用 Azure 身份验证库 (AAL) 获取用于访问 Graph API 的令牌。令牌获取流程由 *Microsoft.IdentityModel.WAAD.Preview.Graph.GraphInterface* 类中的 *GetAuthorizationHeader* 方法管理，如下所示。

<div class="dev-callout"><strong>说明</strong><p>AAL 可作为 NuGet 程序包，并且可安装在 Visual Studio 中。</p></div>

    /// <summary>
    /// Method to get the Oauth2 Authorization header from ACS
    /// </summary>
    /// <returns>AOauth2 Authorization header</returns>
    private string GetAuthorizationHeader()
    {
        // AAL values
        string fullTenantName = TenantUtils.Globals.StsUrl + TenantDomainName;
        string serviceRealm = string.Format("{0}/{1}@{2}", TenantUtils.Globals.GraphServicePrincipalId, TenantUtils.Globals.GraphServiceHost, TenantDomainName);
        string issuingResource = string.Format("{0}@{1}", Globals.ClientId, TenantDomainName);
        string clientResource = string.Format("{0}/{1}@{2}", Globals.ClientId, Globals.AppHostname, TenantDomainName);

        string authzHeader = null;
        AuthenticationContext _authContext = new AuthenticationContext(fullTenantName);

        try
        {
            ClientCredential credential = new ClientCredential(issuingResource, clientResource, Globals.ServicePrincipalKey);
            AssertionCredential _assertionCredential = _authContext.AcquireToken(serviceRealm, credential);
            authzHeader = _assertionCredential.CreateAuthorizationHeader();
        }
        catch (Exception ex)
        {
            AALException aex = ex as AALException;
            string a = aex.InnerException.Message;
        }

        return authzHeader;
    }

以下信息用于获取访问令牌，如上面的代码所示：

1.  应用程序的信息（ClientID、ServicePrincipalKey 和 AppHostname）
2.  目标信息，就是 Graph，即上述的 ServiceRealm
3.  您之前获得的 TenantDomainName

### 步骤 3：调用 Azure AD Graph 以获取用户的列表

*Microsoft.IdentityModel.WAAD.Preview.Graph.GraphInterface* 类中的以下方法使用之前生成的 *DataService* 代理，获取您租户的所有用户的列表。

    public List<User> GetUsers()
    {
        // Add the page size using top
        var users = dataService.Users.AddQueryOption("$top", 20);

        // Execute the Query
        var userQuery = users.Execute();

        // Get the return users list
        return userQuery.ToList();
    }

这种方法从 **HomeController.cs** 文件调用，以在 web 应用程序中的“用户”选项卡上显示用户列表。

## <a name="publish"></a>第 5 部分：发布应用程序

一旦已全面测试您的应用程序，您可以创建应用程序列表并在 Azure Marketplace 上发布应用程序。在 Microsoft Seller Dashboard 上执行这些步骤。

<div class="dev-callout"><strong>说明</strong><p>您的应用程序负责管理您的客户的任何付费关系。Azure Marketplace 仅提供到您的应用程序网站和有关信息的链接。</p></div>

### 步骤 1：创建应用程序清单和应用程序列表

在创建应用程序列表前，您必须为应用程序的生产版本生成新的客户端 ID 和客户端机密。在本演练的第 1 部分，您生成适用于应用程序的测试版的客户端 ID 和客户端机密。重复这些步骤并配置应用程序以使用新值，确保设置生产“应用域”和“应用重定向 URL”。

接下来，您必须创建一个应用程序清单，列出您的应用程序将请求客户同意的权限。此清单采用通过 XSD 文件控制的 XML 格式编写。作为您创建应用程序列表的一部分，必须上传该清单。

有三个不同的权限级别，如本演练的第 1 部分中所述：

**DirectoryReader**：授予权限以读取目录数据，例如用户帐户、组和有关您组织的信息。启用 SSO。

**UserAccountAdministrator**：授予权限以读取和写入数据，如用户、组和有关您组织的信息。启用 SSO。

**None**：启用单一登录，但禁用对目录数据的访问。

下面包括了两个应用程序清单示例。第一个演示仅 SSO 应用程序的权限，第二个演示只读应用程序的权限：

    <?xml version="1.0" encoding="utf-16"?>
    <AppRequiredPermissions>
      <AppPermissionRequests Policy="AppOnly">
        <AppPermissionRequest Right="None" Scope="http://directory" />
      </AppPermissionRequests>
    </AppRequiredPermissions>


    <?xml version="1.0" encoding="utf-16"?>
    <AppRequiredPermissions>
      <AppPermissionRequests Policy="AppOnly">
        <AppPermissionRequest Right="Directory Reader" Scope="http://directory">
          <Reason culture="zh-cn" value="Needs to read the app"/>
        </AppPermissionRequest>
      </AppPermissionRequests>
    </AppRequiredPermissions>

上面示例中的*策略*属性描述请求的应用程序权限的类型。目前仅支持 "AppOnly" 权限属性。此权限类型表示该应用程序只访问作为自身的目录。

可选*原因*元素允许您指定（在多个区域性）对所需的权限级别的判定。此文本显示在同意页上，在客户批准或拒绝您的应用程序时提供帮助。

使用新的客户端 ID 和您的应用程序清单，可以创建应用程序列表，请按[在 Microsoft Seller Dashboard 中添加应用程序][在 Microsoft Seller Dashboard 中添加应用程序]中的说明操作。创建应用程序列表时，确保选择 Azure AD 应用程序类型。一旦完成创建您的应用程序列表，单击“提交”将应用程序发布到 Azure Marketplace。需要等到您的应用程序得到批准，发布才告完成。

<div class="dev-callout"><strong>说明</strong><p>如果系统提示您&ldquo;添加税务和支出信息&rdquo;，可以跳过此步骤中，因为您直接向客户而非通过 Microsoft 销售您的应用程序。</p></div>

### 步骤 2：完成测试并公开您的应用程序

一旦您的应用程序列表经过审批，应再次端到端测试您的应用程序。例如，确保已经使用生产 ClientID 和客户端机密更新您的应用程序。通过最后一次测试核对清单，确保同意页现在显示您的应用程序列表中包含的信息。

## <a name="summary"></a>摘要

在本演练中，您学习了如何将多租户应用程序与 Azure AD 集成。该过程包括三个步骤：

-   允许客户使用 Azure AD 注册您的应用程序
-   启用 Azure AD 的单一登录 (SSO)
-   查询使用 Azure AD Graph API 的客户的目录数据

与 Azure AD 集成可以让您的客户注册应用程序，并通过他们所维系的标识管理系统登录，以便减少或消除您的应用程序执行单独标识管理任务的需要。此功能为您的客户在使用应用程序时提供更加无缝的体验，解放管理任务所花费的时间。

  [标识页]: http://azure.microsoft.com/zh-cn/services/multi-factor-authentication/
  [此处下载]: http://go.microsoft.com/fwlink/?LinkId=271213
  [Visual Studio 中的端口分配]: http://msdn.microsoft.com/zh-cn/library/ms178109(v=vs.100).aspx
  [Visual Studio 2012]: http://www.visualstudio.com/downloads/download-visual-studio-vs
  [WCF Data Services for OData]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29306
  [介绍]: #introduction
  [第 1 部分：为访问 Azure AD 获取客户端 ID]: #getclientid
  [第 2 部分：允许客户使用 Azure AD 注册]: #enablesignup
  [第 3 部分：启用单一登录]: #enablesso
  [第 4 部分：访问 Azure AD Graph]: #accessgraph
  [第 5 部分：发布应用程序]: #publish
  [摘要]: #summary
  [Microsoft Seller Dashboard]: https://sellerdashboard.microsoft.com/
  [创建帐户配置文件]: http://msdn.microsoft.com/zh-cn/library/jj552460.aspx
  [在 Microsoft Seller Dashboard 中创建客户端 ID 和机密]: http://msdn.microsoft.com/zh-cn/library/jj552461.aspx
  [登录]: ./media/active-directory-dotnet-integrate-multitent-cloud-applications/login.png
  [获取 Azure AD 租户]: https://account.windowsazure.cn/organization
  [本主题]: http://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx
  [在 Microsoft Seller Dashboard 中添加应用程序]: http://msdn.microsoft.com/zh-cn/library/jj552465.aspx
