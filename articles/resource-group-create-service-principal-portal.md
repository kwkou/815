<properties
   pageTitle="使用 Azure 门户创建新的 Azure 服务主体"
   description="介绍如何创建一个新的 Azure 服务主体，在 Azure 资源管理器中将此服务主体与基于角色的访问控制配合使用可以管理对资源的访问权限。"
   services="na"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/24/2015"
   wacn.date="10/3/2015"/>

# 使用 Azure 门户创建新的 Azure 服务主体

## 概述
服务主体是需要访问其他资源的自动化过程、应用程序或服务。使用 Azure 资源管理器，可以向服务主体授予访问权限并对服务主体进行身份验证，使服务主体可对存在于订阅或用作租户的资源执行允许的管理操作。

本主题说明如何使用 Azure 门户创建新的服务主体。目前，你必须使用 Microsoft Azure 门户来创建新的服务主体。在更新的版本中，此功能将添加到 Azure 预览版门户。

## 概念
1. Azure Active Directory (AAD) - 云的标识与访问管理服务生成版。有关详细信息，请参阅：[什么是 Azure Active Directory](/documentation/articles/active-directory-whatis)
2. 服务主体 - 目录中应用程序的实例。
3. AD 应用程序 - AAD 中向 AAD 标识某个应用程序的目录记录。有关详细信息，请参阅 [Azure AD 中的身份验证基本知识](https://msdn.microsoft.com/zh-cn/library/azure/874839d9-6de6-43aa-9a5c-613b0c93247e#BKMK_Auth)。


## 创建 Active Directory 应用程序
1. 通过[经典门户](https://manage.windowsazure.cn/)登录到你的 Azure 帐户。

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

6. 填写应用程序名称，然后选择你要使用的应用程序类型。由于我们想要使用此应用程序的服务主体在 Azure 资源管理器上进行身份验证，因此要选择创建“WEB 应用程序和/或 WEB API”，然后单击“下一步”按钮。

   ![命名应用程序][9]

7. 填写应用程序的属性。对于“登入 URL”，请提供用于描述应用程序的网站 URI。将不验证网站是否存在。对于“应用程序 ID URI”，请提供用于标识应用程序的 URI。将不验证终结点的唯一性或存在性。单击“完成”创建 AAD 应用程序。

   ![应用程序属性][4]

## 创建服务主体密码
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

## 后续步骤

- 若要了解有关指定安全策略的信息，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->  
- 有关允许服务主体访问资源的步骤，请参阅[通过 Azure 资源管理器对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal)  
- 有关基于角色的访问控制的概述，请参阅 <!--[-->Microsoft Azure 门户中基于角色的访问控制<!--](/documentation/articles/role-based-access-control-configure)-->
- 有关在 Azure 资源管理器中实现安全性的指南，请参阅 [Azure 资源管理器的安全注意事项](/documentation/articles/best-practices-resource-manager-security)


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

<!---HONumber=71-->