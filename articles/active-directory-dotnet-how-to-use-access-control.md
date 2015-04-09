<properties linkid="dev-net-how-to-access-control" urlDisplayName="Access Control" pageTitle="如何使用 Access Control (.NET) - Azure 功能指南" metaKeywords="Azure Access Control Service authentication C#" description="了解如何在 Azure 应用程序中使用访问控制服务 (ACS) 在用户尝试获取对 Web 应用程序的访问权限时对其进行身份验证。" metaCanonical="" services="active-directory" documentationCenter=".NET" title="如何使用 Azure Active Directory 访问控制对 Web 用户进行身份验证" authors="juneb" solutions="" manager="" editor="" />

# 如何使用 Azure Active Directory 访问控制对 Web 用户进行身份验证

本指南演示如何使用 Azure Active Directory 访问控制（也称为访问控制服务或 ACS）在标识提供程序（如 Microsoft 和 Yahoo）用户尝试获取对 Web 应用程序的访问权限时对其进行身份验证。

## <span class="short-header">目录</span>

-   [什么是 ACS？][什么是 ACS？]
-   [概念][概念]
-   [先决条件][先决条件]
-   [创建 Access Control 命名空间][创建 Access Control 命名空间]
-   [创建 ASP.NET MVC 应用程序][创建 ASP.NET MVC 应用程序]
-   [将您的 Web 应用程序与 ACS 集成][将您的 Web 应用程序与 ACS 集成]
-   [测试与 ACS 的集成][测试与 ACS 的集成]
-   [查看 ACS 发送的声明][查看 ACS 发送的声明]
-   [在 ACS 管理门户中查看应用程序][在 ACS 管理门户中查看应用程序]
-   [添加标识提供程序][添加标识提供程序]
-   [后续步骤][后续步骤]

## <span class="short-header">什么是 ACS？</span>

大多数开发人员都不是标识专家，他们都不想花时间开发针对其应用程序和服务的身份验证和授权机制。ACS 是一项 Azure 服务，可用于轻松对访问您的 Web 应用程序和服务的用户进行身份验证，而不必将复杂的身份验证逻辑添加到代码中。

ACS 具有以下可用功能：

-   与 Windows Identity Foundation (WIF) 集成。
-   支持常用 Web 标识提供程序 (IP)，包括 Microsoft 帐户（以前称为 Windows Live ID）和 Yahoo。
-   支持 Active Directory 联合身份验证服务 (AD FS) 2.0。
-   一项基于开放数据协议 (OData) 的管理服务，该服务提供对 ACS 设置的
    编程访问。
-   一个允许对 ACS
    设置进行管理访问的管理门户。

有关 ACS 的详细信息，请参阅 [Access Control 服务 2.0][Access Control 服务 2.0]。

## <span class="short-header">概念</span>

ACS 在基于声明的标识主体的基础上构建，它是一种创建针对本地运行或在云中运行的应用程序的身份验证机制的一致性方法。通常，应用程序和服务可使用基于声明的标识来获取所需的有关其组织内、其他组织内以及 Internet 上的用户的标识信息。

若要完成本指南中的任务，您应了解本指南中使用的以下术语和概念：

**客户端** - 尝试获取对 Web 应用程序的访问权限的浏览器。

**信赖方 (RP) 应用程序** - 您的 Web 应用程序。RP 应用程序是一个将身份验证外包给外部权威机构的网站或服务。用标识行话来说，RP 信任该权威机构。本指南说明如何将您的应用程序配置为信任 ACS。

**令牌** - 用户可通过提供由 RP 应用程序信任的颁发机构颁发的有效令牌来获取对 RP 应用程序的访问权限。对客户端进行身份验证时颁发的安全数据集合。它包含一组声明，这些声明是经过身份验证的用户的特性，例如，用户的姓名或年龄或者用户角色的标识符。已对令牌进行数字签名，这样便能标识其颁发者，并且无法更改其内容。

**标识提供程序 (IP)** - 验证用户标识和颁发安全令牌的权威机构（例如，Microsoft 帐户 (Windows Live ID) 和 Active Directory）。在将 ACS 配置为信任某个 IP 后，它将接受并验证该 IP 颁发的令牌。由于 ACS 可同时信任多个 IP，因此，如果您的应用程序信任 ACS，则您的应用程序将使用户能够选择通过 ACS 代表您信任的任何 IP 进行身份验证。

**联合提供者 (FP)** - 标识提供程序 (IP) 了解用户的直接信息，使用用户凭据对用户进行身份验证并发布有关用户的声明。联合提供者 (FP) 是一种不同类型的权威机构。FP 代理身份验证，而不是直接对用户进行身份验证。它还充当依赖方应用程序与一个或多个 IP 之间的中介。ACS 是一个联合提供者 (FP)。

**ACS 规则引擎** - 声明转换规则将转换来自受信任的 IP 的令牌中的声明，使 RP 能够使用这些声明。ACS 包含一个规则引擎，该引擎应用为您的 RP 指定的声明转换规则。

**Access Control 命名空间** - 提供在应用程序中对 ACS 资源进行寻址的唯一范围。该命名空间包含您的设置（例如您信任的 IP、您需要服务的 RP 应用程序、您对传入令牌应用的规则），并且它显示应用程序和开发人员用来与 ACS 进行通信的终结点。

下图演示 ACS 身份验证如何使用 Web 应用程序：

![][0]

1.  客户端（在此示例中为浏览器）从 RP 请求页面。
2.  由于尚未对请求进行身份验证，因此 RP 会将
    用户重定向到它信任的权威机构（即 ACS）。ACS 让
    用户选择已为此 RP 指定的 IP。
    用户选择适当的 IP。
3.  客户端浏览到该 IP 的身份验证页，并提示
    用户登录。
4.  
    在对客户端进行身份验证（例如，输入标识凭据）后，IP 将颁发一个安全令牌。
5.  颁发安全令牌后，IP 会定向客户端以将 IP 所颁发的安全令牌发送给 ACS。
6.  ACS 验证 IP 所颁发的安全令牌，将此令牌中的
    标识声明输入 ACS 规则引擎，计算
    输出标识声明，并颁发
    包含这些输出声明的新安全令牌。
7.  ACS 定向客户端以将 ACS 所颁发的安全令牌发送给 RP。RP 将验证安全令牌上的签名，提取应用程序业务逻辑所使用的声明，并返回最初请求的页面。

## <span class="short-header">先决条件</span>

若要完成本指南中的任务，你将需要：

-   Azure 订阅
-   Microsoft Visual Studio 2012
-   Identity and Access Tool for Visual Studio 2012（若要下载，请参阅 [身份验证和访问工具][身份验证和访问工具]）

## <span class="short-header">创建 Access Control 命名空间</span>

要在 Azure 中使用 Active Directory Access Control，请创建 Access Control 命名空间。该命名空间提供了一个唯一范围，用于
在应用程序中对 ACS 资源进行寻址。

1.  登录到 [Azure 管理门户][Azure 管理门户] (https://manage.WindowsAzure.cn)。

2.  单击“Active Directory”。

    ![][1]

3.  要创建新的 Access Control 命名空间，请单击**“新建”**。这将选中“应用服务”和“Access Control”。单击“快速创建”。

    ![][2]

4.  输入该命名空间的名称。Azure 将验证该名称是否是唯一的。

5.  选择在其中使用该命名空间的区域。若要获得最佳性能，请使用要在其中部署应用程序的区域，然后单击**“创建”**。

Azure 将创建并激活该命名空间。

## <span class="short-header">创建 ASP.NET MVC 应用程序</span>

在此步骤中，您将创建一个 ASP.NET MVC 应用程序。在后续步骤中，我们将此简单的 Web 窗体应用程序与 ACS 集成。

1.  启动 Visual Studio 2012 或 Visual Studio Express for Web 2012（本教程不适用于早期版本的 Visual Studio）。
2.  单击**“文件”**，然后单击**“新建项目”**。
3.  选择 Visual C#/Web 模板，然后选择**“ASP.NET MVC 4 Web 应用程序”**。

    在本指南中，我们将使用 MVC 应用程序，但您可以使用任何 Web 应用程序类型来执行该任务。

    ![][3]

4.  在**名称**中，键入**MvcACS**，然后单击**确定**。
5.  在下一个对话框中，选择**Internet 应用程序**，然后单击**确定**。
6.  编辑 *Views\\Shared\_LoginPartial.cshtml* 文件，并将内容替换为下列代码：

        @if (Request.IsAuthenticated)
        {
            string name = "Null ID";
            if (!String.IsNullOrEmpty(User.Identity.Name))
            {
                name = User.Identity.Name;
            }    
            <text>
            Hello, @Html.ActionLink(name, "Manage", "Account", routeValues: null, htmlAttributes: new { @class = "username", title = "Manage" })!
                    @using (Html.BeginForm("LogOff", "Account", FormMethod.Post, new { id = "logoutForm" }))
                    {
                        @Html.AntiForgeryToken()
                        <a href="javascript:document.getElementById('logoutForm').submit()">Log off</a>
                    }
            </text>
        }
        else
        {
            <ul>
                <li>@Html.ActionLink("Register", "Register", "Account", routeValues: null, htmlAttributes: new { id = "registerLink" })</li>
                <li>@Html.ActionLink("Log in", "Login", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
            </ul>
        }

目前，ACS 未设置 User.Identity.Name，因此我们需要进行上述更改。

1.  按 F5 运行应用程序。默认的 ASP.NET MVC 应用程序将显示在 Web 浏览器中。

## <span class="short-header">将您的 Web 应用程序与 ACS 集成</span>

在此任务中，您将 ASP.NET web 应用程序与 ACS 集成。

1.  在“解决方案资源管理器”中，右键单击 MvcACS 项目，然后选择**身份验证和访问**。

    如果**身份验证和访问”**选项未显示在上下文菜单上，请安装“身份验证和访问工具”。有关信息，请参阅[身份验证和访问工具][身份验证和访问工具]。

    ![][4]

2.  在**提供程序**选项卡上，选择**使用 Windows Azure Access Control 服务**。

    ![][5]

3.  单击**配置**链接。

    ![][6]

    Visual Studio 请求有关 Access Control 命名空间的信息。输入您之前创建的命名空间名称（在上述图像中对此进行测试，但您将具有其他命名空间）。切换回 Azure 管理门户来获取对称密钥。

    ![][7]

4.  在 Azure 管理门户中，单击 Access Control 命名空间，然后单击**管理**。

    ![][8]

5.  单击**管理服务**，然后单击**管理客户端**。

    ![][9]

6.  单击**对称密钥**，再单击**显示密钥**，然后复制该密钥值。然后，单击**取消**退出“编辑管理客户端”页面而不进行任何更改。

    ![][10]

7.  在 Visual Studio 中，将密钥粘贴到**输入命名空间的管理密钥**字段中，并单击**保存管理密钥**，然后单击**确定**。

    ![][11]

    Visual Studio 使用有关命名空间的信息来连接到 ACS 管理门户，并获取命名空间的设置，包括标识提供程序、领域和返回 URL。

8.  选择 **Windows Live ID**（Microsoft 帐户）并单击“确定”。

    ![][12]

## <span class="short-header">测试与 ACS 的集成</span>

本任务说明如何测试 RP 应用程序与 ACS 的集成。

-   在 Visual Studio 中，按 F5 运行应用程序。

在将应用程序与 ACS 集成并已选择 Windows Live ID（Microsoft 帐户）后，会将您的浏览器重定向到 Microsoft 帐户的登录页，而不是打开默认的 ASP.NET Web 窗体应用程序。当您使用有效的用户名和密码登录时，您会被重定向到 MvcACS 应用程序。

![][13]

祝贺你！您已成功将 ACS 与 ASP.NET Web 应用程序集成。ACS 现在正在使用用户的 Microsoft 帐户凭据处理用户的身份验证。

## <a name="bkmk_viewClaims"></a>查看 ACS 发送的声明

在本节中，我们将修改应用程序以查看 ACS 发送的声明。身份验证和访问工具已创建一个将 IP 中的所有声明传递给应用程序的规则组。请注意，不同的标识提供程序会发送不同的声明。

1.  打开 *Controllers\\HomeController.cs* 文件。为 **System.Threading** 添加 **using** 语句：

    using System.Threading;

2.  在 HomeController 类中，添加 *Claims* 方法：

    public ActionResult Claims()
    {
     ViewBag.Message = "Your claims page.";

        ViewBag.ClaimsIdentity = Thread.CurrentPrincipal.Identity;

        return View();

    }

3.  右键单击 *Claims* 方法，然后选择**添加视图**。

![][14]

1.  单击“添加”。

2.  将 *Views\\Home\\Claims.cshtml* 文件的内容替换为下列代码：

        @{
            ViewBag.Title = "Claims";
        }
        <hgroup class="title">
            <h1>@ViewBag.Title.</h1>
            <h2>@ViewBag.Message</h2>
        </hgroup>
        <h3>Values from Identity</h3>
        <table>
            <tr>
                <td>
                    IsAuthenticated: 
                </td>
                <td>
                    @ViewBag.ClaimsIdentity.IsAuthenticated 
                </td>
            </tr>
            <tr>
                <td>
                    Name: 
                </td>        
                <td>
                    @ViewBag.ClaimsIdentity.Name
                </td>        
            </tr>
        </table>
        <h3>Claims from ClaimsIdentity</h3>
        <table>
            <tr>
                <th>
                    Claim Type
                </th>
                <th>
                    Claim Value
                </th>
            </tr>
                @foreach (System.Security.Claims.Claim claim in ViewBag.ClaimsIdentity.Claims ) {
            <tr>
                <td>
                    @claim.Type
                </td>
                <td>
                    @claim.Value
                </td>
            </tr>
        }
        </table>

3.  运行应用程序并导航到 *Claims* 方法：

![][15]

有关在应用程序中使用声明的详细信息，请参阅 [Windows Identity Foundation 库][Windows Identity Foundation 库]。

## <a name="bkmk_VP"></a>在 ACS 管理门户中查看应用程序

Visual Studio 中的身份验证和访问工具会自动将您的应用程序与 ACS 集成。

在选择“使用 Azure Access Control”选项并运行应用程序时，身份验证和访问工具会将应用程序添加为信赖方，将应用程序配置为使用选定的标识提供程序，并生成和选择应用程序的默认声明转换规则。

您可以在 ACS 管理门户中检查和更改这些配置设置。使用下列步骤可查看门户中的更改。

1.  登录到 Windows [Azure 管理门户][Azure 管理门户]。

2.  单击“Active Directory”。

    ![][8]

3.  选择一个 Access Control 命名空间，然后单击**管理**。此操作将打开 ACS 管理门户。

    ![][16]

4.  单击**信赖方应用程序**。

    新的 MvcACS 应用程序将出现在信赖方应用程序列表中。该领域将自动设置为应用程序主页。

    ![][17]

5.  单击 **MvcACS**。

    “编辑信赖方应用程序”页包含 MvcACS Web 应用程序的配置设置。在此页面上更改设置并进行保存后，这些更改将立即应用于应用程序。

    ![][18]

6.  向下滚动页面以查看 MvcACS 应用程序的剩余配置设置，包括标识提供程序和声明转换规则。

    ![][19]

在下一节中，我们将使用 ACS 管理门户的功能来对 Web 应用程序进行更改 -- 只是为了说明此操作是多么容易完成。

## <span class="short-header">添加标识提供程序</span>

下面我们使用 ACS 管理门户来更改 MvcACS 应用程序的身份验证。在此示例中，我们将添加 Google 作为 MvcACS 的标识提供程序。

1.  单击**标识提供程序**（位于导航菜单中），然后单击**添加**。

    ![][20]

2.  单击 **Google**，然后单击**下一步**。默认选中的是 MvcACS 应用程序复选框。

    ![][21]

3.  单击“保存”。

    ![][22]

完成！如果您返回 Visual Studio，请打开 MvcACS 应用程序的项目，然后单击**身份验证和访问**，该工具将同时列出 Windows Live ID 和 Google 标识提供程序。

![][23]

然后，当您运行应用程序时，您将看到起到的效果。当应用程序支持多个标识提供程序时，首先会将用户的浏览器定向到 ACS 承载的页面，该页面提示用户选择标识提供程序。

![][24]

在用户选择标识提供程序后，浏览器将转到标识提供程序登录页。

## <span class="short-header">后续步骤</span>

您已创建与 ACS 集成的 Web 应用程序。但这只是开始！您可以在此方案的基础上进行扩展。

例如，您可以为此 RP 添加多个标识提供程序或允许企业目录（例如 Active Directory 域服务）中注册的用户登录到 Web 应用程序。

还可以向命名空间中添加规则，这些规则确定将哪些声明发送到应用程序以便在应用程序业务逻辑中进行处理。

若要进一步了解 ACS 的功能并尝试将其用于更多方案，请参阅 [Access Control 服务 2.0][Access Control 服务 2.0]。

  [什么是 ACS？]: #what-is
  [概念]: #concepts
  [先决条件]: #pre
  [创建 Access Control 命名空间]: #create-namespace
  [创建 ASP.NET MVC 应用程序]: #create-web-app
  [将您的 Web 应用程序与 ACS 集成]: #Identity-Access
  [测试与 ACS 的集成]: #Test-ACS
  [查看 ACS 发送的声明]: #bkmk_viewClaims
  [在 ACS 管理门户中查看应用程序]: #bkmk_VP
  [添加标识提供程序]: #add-IP
  [后续步骤]: #whats-next
  [Access Control 服务 2.0]: http://go.microsoft.com/fwlink/?LinkID=212360
  [0]: ./media/active-directory-dotnet-how-to-use-access-control/acs-01.png
  [身份验证和访问工具]: http://go.microsoft.com/fwlink/?LinkID=245849
  [Azure 管理门户]: http://manage.WindowsAzure.cn
  [1]: ./media/active-directory-dotnet-how-to-use-access-control/acsCreateNamespace.png
  [2]: ./media/active-directory-dotnet-how-to-use-access-control/acsQuickCreate.png
  [3]: ./media/active-directory-dotnet-how-to-use-access-control/rzMvc.png
  [4]: ./media/active-directory-dotnet-how-to-use-access-control/rzIA.png
  [5]: ./media/active-directory-dotnet-how-to-use-access-control/rzPT.png
  [6]: ./media/active-directory-dotnet-how-to-use-access-control/rzC.png
  [7]: ./media/active-directory-dotnet-how-to-use-access-control/acsConfigAcsNamespace.png
  [8]: ./media/active-directory-dotnet-how-to-use-access-control/acsClickManage.png
  [9]: ./media/active-directory-dotnet-how-to-use-access-control/acsManagementService.png
  [10]: ./media/active-directory-dotnet-how-to-use-access-control/acsShowKey.png
  [11]: ./media/active-directory-dotnet-how-to-use-access-control/acsConfigAcsNamespace2.png
  [12]: ./media/active-directory-dotnet-how-to-use-access-control/acsIdAndAccess1.png
  [13]: ./media/active-directory-dotnet-how-to-use-access-control/acsMSFTAcct.png
  [14]: ./media/active-directory-dotnet-how-to-use-access-control/rzAv.png
  [15]: ./media/active-directory-dotnet-how-to-use-access-control/rzCl.png
  [Windows Identity Foundation 库]: http://msdn.microsoft.com/library/hh377151.aspx
  [16]: ./media/active-directory-dotnet-how-to-use-access-control/acsACSPortal.png
  [17]: ./media/active-directory-dotnet-how-to-use-access-control/acsRPPage.png
  [18]: ./media/active-directory-dotnet-how-to-use-access-control/acsEdit-RP.png
  [19]: ./media/active-directory-dotnet-how-to-use-access-control/acsEdit-RP2.png
  [20]: ./media/active-directory-dotnet-how-to-use-access-control/acsAdd-Idp.png
  [21]: ./media/active-directory-dotnet-how-to-use-access-control/acsAdd-Google.png
  [22]: ./media/active-directory-dotnet-how-to-use-access-control/acsSave-Google.png
  [23]: ./media/active-directory-dotnet-how-to-use-access-control/acsIdAndA-after.png
  [24]: ./media/active-directory-dotnet-how-to-use-access-control/acsSignIn.png
