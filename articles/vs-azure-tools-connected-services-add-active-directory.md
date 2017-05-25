<properties
    pageTitle="在 Visual Studio 中使用连接服务添加 Azure Active Directory | Azure"
    description="使用 Visual Studio 中的“添加连接服务”对话框添加 Azure Active Directory"
    services="visual-studio-online"
    documentationcenter="na"
    author="TomArcher"
    manager="douge"
    editor="" />
<tags
    ms.assetid="f599de6b-e369-436f-9cdc-48a0165684cb"
    ms.service="active-directory"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/01/2017"
    wacn.date="05/22/2017"
    ms.author="tarcher"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="159ed9293df1ae5a88fcfca1600cab96229b9d39"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="adding-an-azure-active-directory-by-using-connected-services-in-visual-studio"></a>在 Visual Studio 中使用连接服务添加 Azure Active Directory
## <a name="overview"></a>概述
通过使用 Azure Active Directory (Azure AD)，您可以对 ASP.NET MVC Web 应用程序或 Web API 服务中的 AD 身份验证支持单一登录 (SSO)。 通过 Azure AD 身份验证，您的用户可以使用其帐户从 Azure AD 连接到您的 Web 应用程序。 通过 Web API 的 Azure AD 身份验证的优点包括，从 Web 应用程序公开 API 时提供增强的数据安全性。 通过 Azure AD，您不需要使用其自己的帐户和用户管理来管理单独的身份验证系统。

## <a name="supported-project-types"></a>支持的项目类型
您可以在以下项目类型中使用“连接服务”对话框连接到 Azure AD。

- ASP.NET MVC 项目
- ASP.NET Web API 项目

### <a name="connect-to-azure-ad-using-the-connected-services-dialog"></a>使用“连接服务”对话框连接到 Azure AD
1. 确保您具有 Azure 帐户。 如果你没有 Azure 帐户，可以注册 [试用版](http://go.microsoft.com/fwlink/?LinkId=518146)。
2. 在 Visual Studio 中，打开项目中“引用”节点的快捷菜单，然后选择“添加连接服务”。
3. 选择“Azure AD 身份验证”，然后选择“配置”。

    ![选择“添加 Azure AD 身份验证”](./media/vs-azure-tools-connected-services-add-active-directory/connected-services-add-active-directory.png)
4. 在“配置 Azure AD 身份验证”的第一页上，选中“使用 Azure AD 配置单一登录”。

    如果项目配置有另一个身份验证配置，向导将警告继续操作将禁用以前的配置。

    ![在向导中配置 Azure AD](./media/vs-azure-tools-connected-services-add-active-directory/configure-azure-ad-wizard-1.png)
5. 在第二页上，从“域”  下拉列表中选择一个域。 域列表包含在“帐户设置”对话框中列出的帐户可以访问的所有域。 如果没有找到你要查找的域，作为替代方法，可以输入域名，如 mydomain.partner.onmschina.cn。 您可以选择用于创建一个新 Azure AD 应用程序的选项，也可以使用现有 Azure AD 应用程序中的设置。 

    ![在向导中配置 Azure AD](./media/vs-azure-tools-connected-services-add-active-directory/configure-azure-ad-wizard-2.png)
6. 在向导的第三页上，确保已选中“读取目录数据”  。 该向导将填写“客户端密码” 。 

    ![在向导中配置 Azure AD](./media/vs-azure-tools-connected-services-add-active-directory/configure-azure-ad-wizard-3.png)
7. 单击“完成”  按钮。 对话框将添加必要的配置代码和引用，以便为 Azure AD 身份验证启用项目。
8. 查看浏览器中显示的“入门”页以了解后续步骤，查看“发生了什么情况”页以了解如何修改了你的项目。 如果想要检查一切是否正常运作，请打开已修改的配置文件之一并验证其中是否存在“发生了什么情况”中提及的设置。 例如，ASP.NET MVC 项目中的主 web.config 会添加这些设置：

        <appSettings> 
            <add key="ida:ClientId" value="ClientId from the new Azure AD App" />
            <add key="ida:AADInstance" value="https://login.chinacloudapi.cn/" />
            <add key="ida:Domain" value="Your selected domain" />
            <add key="ida:TenantId" value="The Id of your selected Azure AD Tenant" />
            <add key="ida:PostLogoutRedirectUri" value="The default redirect URI from the project" />
        </appSettings>

## <a name="how-your-project-is-modified"></a>您的项目的修改情况
当运行向导时，Visual Studio 会将 Azure AD 和关联的引用添加到您的项目。 还会修改您项目中的配置文件和代码文件以添加对 Azure AD 的支持。 Visual Studio 所做的特定修改取决于项目类型。 有关 ASP.NET MVC 项目修改情况的详细信息，请参阅 [完成的操作 - MVC 项目](http://go.microsoft.com/fwlink/p/?LinkID=513809)。 对于 Web API 项目，请参阅 [完成的操作 - Web API 项目](http://go.microsoft.com/fwlink/p/?LinkId=513810)。

## <a name="next-steps"></a>后续步骤
- [Azure Active Directory 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=WindowsAzureAD)
- [Azure Active Directory 文档](/documentation/services/identity/)
- [博客文章：Azure Active Directory 简介](http://blogs.msdn.com/b/brunoterkaly/archive/2014/03/03/introduction-to-windows-azure-active-directory.aspx)

<!-- Update_Description: wording update -->