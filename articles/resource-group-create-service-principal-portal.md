<properties
   pageTitle="在门户中创建 AD 应用程序和服务主体 | Windows Azure"
   description="介绍如何创建新的 Active Directory 应用程序和服务主体，在 Azure 资源管理器中将此服务主体与基于角色的访问控制配合使用可以管理对资源的访问权限。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="10/29/2015"
   wacn.date="01/21/2016"/>

# 使用门户创建 Active Directory 应用程序和服务主体

## 概述
当你的应用程序需要访问或修改订阅中的资源时，你可以使用门户来创建 Active Directory 应用程序，并将它分配到拥有适当权限的角色。通过门户创建某个 Active Directory 应用程序时，实际上会同时创建该应用程序和一个服务主体。设置权限时，将要使用该服务主体。

本主题说明如何使用 Azure 门户创建新的应用程序和服务主体。目前，你必须使用 Windows Azure 门户来创建新的 Active Directory 应用程序。在以后的版本中，此功能将添加到 Azure 门户。可以使用预览门户将应用程序添加到角色。

## 概念
1. Azure Active Directory (AAD) - 云的标识与访问管理服务生成版。有关详细信息，请参阅：[什么是 Azure Active Directory](/documentation/articles/active-directory-whatis)
2. 服务主体 - 目录中应用程序的实例。
3. AD 应用程序 - AAD 中向 AAD 标识某个应用程序的目录记录。 

有关应用程序和服务主体的详细说明，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects)。
有关 Active Directory 身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios)。


## 创建应用程序对象和服务主体对象

1. 通过[门户](https://manage.windowsazure.cn/)登录到你的 Azure 帐户。

2. 在左侧窗格中选择“Active Directory”。

     ![选择“Active Directory”][1]

3. 选择你要用来创建新应用程序的目录。

     ![选择目录][2]

3. 若要查看目录中的应用程序，请单击“应用程序”。

     ![查看应用程序][11]

4. 如果你之前尚未在该目录中创建应用程序，则应该会看到与下面类似的图像。单击“添加应用程序”。

     ![添加应用程序][6]

     或者，单击底部窗格中的“添加”。

     ![添加][12]

5. 选择你想要创建的应用程序类型。对于本教程，我们将不使用库中的应用程序。

     ![新应用程序][10]

6. 填写应用程序名称，然后选择你要使用的应用程序类型。由于我们想要使用此应用程序的服务主体在 Azure 资源管理器上进行身份验证，因此要选择创建“ WEB 应用和/或 WEB API”，然后单击“下一步”按钮。

     ![命名应用程序][9]

7. 填写应用程序的属性。对于“登入 URL”，请提供用于描述应用程序的 WEB 应用 URI。将不验证 WEB 应用是否存在。对于“应用程序 ID URI”，请提供用于标识应用程序的 URI。
将不验证终结点的唯一性或存在性。单击“完成”创建 AAD 应用程序。

     ![应用程序属性][4]

## 为应用程序创建身份验证密钥
门户现在应该已选择你的应用程序。

1. 单击“配置”选项卡以配置应用程序的密码。

     ![配置应用程序][3]

2. 向下滚动到“密钥”部分，并选择你希望密码保持有效的时间。

     ![密钥][7]

3. 选择“保存”以创建密钥。

     ![保存][13]

     随后将显示保存的密钥，你可以复制该密钥。

     ![保存的密钥][8]

4. 现在，你可以使用该密钥来以服务主体的身份进行身份验证。除了“密钥”以外，你还需要使用“客户端 ID”来登录。转到“客户端 ID”并复制它。
  
     ![客户端 ID][5]


你的应用程序现已准备就绪，并且租户上已创建服务主体。以服务主体身份登录时，请务必使用：

* **客户端 ID** - 与你的用户名相同。
* **密钥** - 与你的密码相同。

## 将应用程序分配到角色

可以使用[Azure 门户](https://manage.windowsazure.cn)将 Active Directory 应用程序分配到有权访问你需要访问的资源的角色。

## 在代码中获取访问令牌

如果你正在使用 .NET，可以通过以下代码检索应用程序的访问令牌。

首先，必须将 Active Directory 身份验证库安装到你的 Visual Studio 项目中。执行此操作的最简单方法是使用 NuGet 包。打开包管理器控制台并键入以下命令。

    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213
    PM> Update-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Safe

在你的应用程序中，添加如下所示的方法以检索令牌。

    public static string GetAccessToken()
    {
        var authenticationContext = new AuthenticationContext("https://login.chinacloudapi.cn/{tenantId or tenant name}");  
        var credential = new ClientCredential(clientId: "{application id}", clientSecret: "{application password}");
        var result = authenticationContext.AcquireToken(resource: "https://management.core.chinacloudapi.cn/", clientCredential:credential);

        if (result == null) {
            throw new InvalidOperationException("Failed to obtain the JWT token");
        }

        string token = result.AccessToken;

        return token;
    }

## 后续步骤

- 若要了解有关指定安全策略的信息，请参阅[管理和审核对资源的访问权限](/documentation/articles/resource-group-rbac)。  
- 若要了解如何使用 Azure PowerShell 或 Azure CLI 来处理 Active Directory 应用程序和服务主体，包括如何使用证书进行身份验证，请参阅[通过 Azure 资源管理器对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal)。
- 有关在 Azure 资源管理器中实现安全性的指南，请参阅 [Azure 资源管理器的安全注意事项](/documentation/articles/best-practices-resource-manager-security)。


<!-- Images. -->
[1]: ./media/resource-group-create-service-principal-portal/active-directory.png
[2]: ./media/resource-group-create-service-principal-portal/active-directory-details.png
[3]: ./media/resource-group-create-service-principal-portal/application-configure.png
[4]: ./media/resource-group-create-service-principal-portal/app-properties.png
[5]: ./media/resource-group-create-service-principal-portal/client-id.png
[6]: ./media/resource-group-create-service-principal-portal/create-application.png
[7]: ./media/resource-group-create-service-principal-portal/create-key.png
[8]: ./media/resource-group-create-service-principal-portal/save-key.png
[9]: ./media/resource-group-create-service-principal-portal/tell-us-about-your-application.png
[10]: ./media/resource-group-create-service-principal-portal/what-do-you-want-to-do.png
[11]: ./media/resource-group-create-service-principal-portal/view-applications.png
[12]: ./media/resource-group-create-service-principal-portal/add-icon.png
[13]: ./media/resource-group-create-service-principal-portal/save-icon.png

<!---HONumber=Mooncake_1221_2015-->