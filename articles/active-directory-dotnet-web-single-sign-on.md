<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="通过 .NET 和 Azure Active Directory 实现的 Web 单一登录" authors="" solutions="" manager="" editor="" />

# 通过 .NET 和 Azure Active Directory 实现的 Web 单一登录

## <a name="introduction"></a>介绍

本教程将向您介绍如何利用 Azure Active Directory 为 Office 365 客户的用户实现单一登录。你将了解如何执行以下操作：

-   在客户的租户中设置 Web 应用程序
-   使用 WS 联合身份验证保护应用程序

### 先决条件

本演练具有以下开发环境先决条件：

-   Visual Studio 2010 SP1
-   Microsoft .NET Framework 4.0
-   [ASP.NET MVC 3][ASP.NET MVC 3]
-   [Windows Identity Foundation 1.0 运行时][Windows Identity Foundation 1.0 运行时]
-   [Windows Identity Foundation 3.5 SDK][Windows Identity Foundation 3.5 SDK]
-   启用了 SSL 的 Internet Information Services (IIS) 7.5
-   Windows PowerShell
-   [Office 365 PowerShell Commandlets][Office 365 PowerShell Commandlets]

### 目录

-   [介绍][介绍]
-   [步骤 1：创建 ASP.NET MVC 应用程序][步骤 1：创建 ASP.NET MVC 应用程序]
-   [步骤 2：在公司的目录租户中设置应用程序][步骤 2：在公司的目录租户中设置应用程序]
-   [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序][步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]
-   [摘要][摘要]

## <a name="createapp"></a>步骤 1：创建 ASP.NET MVC 应用程序

此步骤介绍如何创建可用于表示受保护资源的简单 ASP.NET MVC 3 应用程序。对此资源的访问权限将通过公司的 STS 管理的联合身份验证（将在本教程后面部分介绍）授予。

1.  以管理员的身份打开 Visual Studio。
2.  在“文件”菜单中，单击“新建项目”。
3.  在左侧的模板菜单中，选择“Web”，然后单击“ASP.NET MVC 3 Web 应用程序”。将项目命名为 *OrgIdFederationSample*。
4.  在“新建 ASP.NET MVC 3 项目”对话框中，单击“空”模板，然后单击“确定”。
5.  在 Visual Studio 生成新项目后，请在“解决方案资源管理器”中右键单击“控制器”文件夹，单击“添加”，然后单击“控制器”。
6.  在“添加控制器”对话框中，将新控制器命名为 *HomeController.cs*，然后单击“添加”。
7.  随后将会创建并打开 **HomeController.cs** 文件。在代码文件中，右键单击 ***Index()*** 方法，然后单击“添加视图...”。
8.  在“添加视图”对话框中单击“确定”。随后将在名为“主页”的新文件夹中创建 **Index.cshtml** 文件。
9.  在“解决方案资源管理器”中右键单击你的 **OrgIdFederationSample** 项目，然后单击“属性”。
10. 在属性窗口中，单击左侧菜单中的“Web”，然后选择“使用本地 IIS Web 服务器”。随后可能会出现一个对话框，询问你是否要为项目创建虚拟目录；请在该对话框中单击“是”。
11. 构建并运行应用程序。随后将显示“主页”控制器的“索引”页。

## <a name="provisionapp"></a>步骤 2：在公司的目录租户中设置应用程序

此步骤介绍 Azure Active Directory 客户的管理员如何在其租户中设置 MVC 应用程序以及如何配置单一登录。完成此步骤后，公司的员工可以使用其 Office 365 帐户对 Web 应用程序进行身份验证。

设置过程的第一步是为应用程序创建新的服务主体。Azure Active Directory 将使用服务主体来向目录注册和验证应用程序。

1.  如果尚未下载和安装 Office 365 PowerShell Commandlets，则执行此操作。
2.  在“开始”菜单中，运行“用于 Windows PowerShell 的 Microsoft Online Services 模块”控制台。此控制台提供了用于配置有关 Office 365 租户的属性（如创建和修改服务主体）的命令行环境。
3.  若要导入必需的 **MSOnlineExtended** 模块，键入以下命令并按 Enter：

        Import-Module MSOnlineExtended -Force

4.  若要连接到 Office 365 目录，你需要提供公司的管理员凭据。请键入以下命令并按 Enter，然后在提示符处输入凭据：

        Connect-MsolService

5.  现在，你将为应用程序创建新的服务主体。请键入以下命令并按 Enter：

        New-MsolServicePrincipal -ServicePrincipalNames @("OrgIdFederationSample/localhost") -DisplayName "Federation Sample Web Site" -Type Symmetric -Usage Verify -StartDate "12/01/2012" -EndDate "12/01/2013" 

    此步骤将输出类似于下面的信息：

        The following symmetric key was created as one was not supplied qY+Drf20Zz+A4t2w e3PebCopoCugO76My+JMVsqNBFc=
        DisplayName           : Federation Sample Web Site
        ServicePrincipalNames : {OrgIdFederationSample/localhost}
        ObjectId              : 59cab09a-3f5d-4e86-999c-2e69f682d90d
        AppPrincipalId        : 7829c758-2bef-43df-a685-717089474505
        TrustedForDelegation  : False
        AccountEnabled        : True
        KeyType               : Symmetric
        KeyId                 : f1735cbe-aa46-421b-8a1c-03b8f9bb3565
        StartDate             : 12/01/2012 08:00:00 a.m.
        EndDate               : 12/01/2013 08:00:00 a.m.
        Usage                 : Verify 

    <div class="dev-callout">

    **说明**
    你应该保存此输出，尤其是生成的对称密钥。此密钥仅在服务主体创建期间对你显示，你以后将无法检索它。使用 Graph API 在目录中读取和写入信息还需要其他值。

    </div>

6.  最后一个步骤是为应用程序设置答复 URL。在进行身份验证尝试之后，将在答复 URL 中发送响应。请键入以下命令并按 Enter：

        $replyUrl = New-MsolServicePrincipalAddresses -Address "https://localhost/OrgIdFederationSample" 

        Set-MsolServicePrincipal -AppPrincipalId "7829c758-2bef-43df-a685-717089474505" -Addresses $replyUrl 

现在已经在目录中设置 Web 应用程序，公司员工可以使用它来进行 Web 单一登录。

## <a name="protectapp"></a>步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序

此步骤说明如何使用 Windows Identity Foundation (WIF) 添加对联合登录的支持。你还将添加登录页并配置应用程序和目录租户之间的信任关系。

1.  在 Visual Studio 中，右键单击 **OrgIdFederationSample** 项目并选择“添加 STS 引用...”。当你安装 WIF SDK 时已添加此上下文菜单选项。
2.  在“身份验证实用工具”对话框中的“应用程序 URL”下，你需要提供采用以下格式的 URI：

        spn:<Your AppPrincipalId>@<Your Directory Domain>

    **spn** 指定 URI 为服务主体名称，**Your AppPrincpalId** 是在你创建服务主体时生成的 **AppPrincipalId** GUID 值，**Your Directory Domain** 是目录租户的 \*.partner.onmschina.cn 域。对于本示例应用程序，我们指定了如下所示的完整应用程序 URI 值：

        spn:7829c758-2bef-43df-a685-717089474505@awesomecomputers.partner.onmschina.cn

    输入应用程序 URI 后，单击“下一步”。

3.  随后将显示一个警告对话框，指出“该应用程序未托管在安全的 https 连接上”。发生此警告的原因是联合身份验证实用工具不能识别 **spn:**格式，但应用程序仍要使用安全的 https 连接。单击“是”继续。

4.  在联合身份验证实用工具的下一页，选择“使用现有 STS”，然后在“STS WS 联合身份验证元数据文档位置”下输入 WS 联合身份验证元数据文档的 URL。此 URL 是使用以下格式指定的：

        https://accounts.accesscontrol.windows.net/<Domain Name or Tenant ID>/FederationMetadata/2007-06/FederationMetadata.xml 

    对于本应用程序，已按如下所示指定了 WS 联合身份验证元数据位置：

        https://accounts.accesscontrol.windows.net/fabrikam.onmicrosoft.com/FederationMetadata/2007-06/FederationMetadata.xml 

    输入元数据位置后，单击“下一步”。

5.  在联合身份验证实用工具的下一页，选择“禁用证书链验证”，然后单击“下一步”。

6.  在联合身份验证实用工具的下一页，选择“不加密”，然后单击“下一步”。

7.  联合身份验证实用工具的下一页将显示 STS 提供的声明。请查看声明，然后单击“下一步”。

8.  接下来，你将要配置 Internet Information Services (IIS) 以支持开发环境的 SSL。若要打开 IIS 管理器，你可以在“运行”提示符下键入 *inetmgr*。

9.  在 IIS 管理器中，展开左侧菜单中的“站点”文件夹，然后单击“默认网站主页”。在右侧的“操作”菜单中单击“绑定...”。

10. 在“站点绑定”对话框中单击“添加”。在“添加站点绑定”对话框中，将“类型”下拉列表中的选项更改为“https”，然后在“SSL 证书”中选择“IIS Express 开发证书”****。单击“确定”。

11. 在 IIS 管理器的左侧菜单中单击“应用程序池”，在列表中选择“ASP.NET v4.0”条目，然后在右侧的“操作”菜单中单击“高级设置”。

12. 在“高级设置”对话框中，将“加载用户配置文件”属性设置为 **true**。单击“确定”。

13. 在 Visual Studio 中，从“解决方案资源管理器”打开位于项目根目录的 **Web.config** 文件。

14. 在 **Web.config** 文件中，找到 **wsFederation** 节并添加一个 **reply** 属性，该属性的值与你在创建服务主体时指定的 **$replyUrl** 变量值相同。例如：

        <wsFederation passiveRedirectEnabled="true" issuer="https://accounts.accesscontrol.windows.net/v2/wsfederation" realm="spn: 7829c758-2bef-43df-a685-717089474505" requireHttps="false" reply="https://localhost/OrgIdFederationSample" /> 

15. 在 **system.web** 节中添加一个 **httpRuntime** 节点，该节点的 **requestValidationMode** 属性设置为 **2.0**。例如：

        <system.web>
            <httpRuntime requestValidationMode="2.0" /> 
            ...

    保存 **Web.config** 文件。

16. 在为 Web 应用程序启用身份验证后，应修改“索引”页以显示已经过身份验证的用户的声明。打开“视图”文件夹的“主页”文件夹中的 **Index.cshtml** 文件。

17. 在 **Index.cshtml** 文件中添加以下代码段并保存该文件：

        <p>
            @if (User.Identity.IsAuthenticated)
            {
            <ul>
                @foreach (string claim in ((Microsoft.IdentityModel.Claims.IClaimsIdentity)this.User.Identity).Claims.Select(c => c.ClaimType + " : " + c.Value))
                {
                    <li>@claim</li>
                }
            </ul>
            }
        </p> 

18. 保存对 **Index.cshtml** 文件所做的更改后，按 **F5** 以运行应用程序。你将会重定向到“Office 365 标识提供程序”页，你可在其中使用目录租户凭据进行登录。例如 *john.doe@awesomecomputers.partner.onmschina.cn*。

19. 使用你的凭据登录后，你将会重定向到“主页”控制器的“索引”页，其中显示了你的帐户的声明。这表明某个用户已成功使用 Azure Active Directory 提供的单一登录在应用程序上进行身份验证。

## <a name="summary"></a>摘要

本教程说明了如何创建和配置使用 Azure Active Directory 的单一登录功能的单租户应用程序。你还可以参考以下教程，为 Azure Active Directory 创建多租户应用程序：[使用 Azure Active Directory 开发多租户云应用程序][使用 Azure Active Directory 开发多租户云应用程序]。

  [ASP.NET MVC 3]: http://www.microsoft.com/zh-cn/download/details.aspx?id=4211
  [Windows Identity Foundation 1.0 运行时]: http://www.microsoft.com/zh-cn/download/details.aspx?id=17331
  [Windows Identity Foundation 3.5 SDK]: http://www.microsoft.com/zh-cn/download/details.aspx?id=4451
  [Office 365 PowerShell Commandlets]: http://technet.microsoft.com/zh-cn/library/jj151814.aspx
  [介绍]: #introduction
  [步骤 1：创建 ASP.NET MVC 应用程序]: #createapp
  [步骤 2：在公司的目录租户中设置应用程序]: #provisionapp
  [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]: #protectapp
  [摘要]: #summary
  [使用 Azure Active Directory 开发多租户云应用程序]: http://msdn.microsoft.com/zh-cn/library/azure/dn151789.aspx
