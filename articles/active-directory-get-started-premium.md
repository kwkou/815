<properties
	pageTitle="Azure Active Directory 高级版入门"
	description="本主题介绍如何注册 Azure Active Directory Premium Edition。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="stevenpo" 
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="05/25/2016"
	wacn.date="07/26/2016"/>

# Azure Active Directory Premium 入门


若要注册 Active Directory Premium，可以使用多个选项：

**Azure 或 Office 365** - Azure 或 Office 365 订户可以在线购买 Active Directory Premium。 
有关详细步骤，请参阅 [How to Purchase Azure Active Directory Premium - Existing Customers（如何购买 Azure Active Directory Premium - 现有客户）](https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/How-to-Purchase-Azure-Active-Directory-Premium-Existing-Customer)或 [How to Purchase Azure Active Directory Premium - New Customers（如何购买 Azure Active Directory Premium - 新客户）](https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/How-to-Purchase-Azure-Active-Directory-Premium-New-Customers)。  

**企业移动套件** - 企业移动性套件提供一种经济高效的方式，让组织可以在一个许可计划下同时使用以下服务：Active Directory Premium、Azure Rights Management 和 Microsoft Intune。有关详细信息，请参阅[企业移动套件](https://www.microsoft.com/zh-cn/server-cloud/enterprise-mobility/overview.aspx)网站。若要获取 30 天免费试用版，请单击[此处](https://portal.office.com/Signup/Signup.aspx?OfferId=2E63A04D-BE0B-4A0F-A8CF-407C1C299221&dl=EMS&ali=1#0)。


**Microsoft 批量许可** - 可以通过 [Microsoft 企业协议](https://www.microsoft.com/zh-cn/licensing/licensing-programs/enterprise.aspx)（250 或更多许可证）或[开放批量许可](https://www.microsoft.com/zh-cn/licensing/licensing-programs/open-license.aspx)（5-250 个许可证）计划获取 Azure Active Directory Premium。


本主题说明如何开始使用通过批量许可计划购买的 Azure Active Directory Premium。如果你还不熟悉 Azure Active Directory 的不同版本，请参阅 [Azure Active Directory editions（Azure Active Directory 版本）](active-directory-editions.md)。

> [AZURE.NOTE]
在中国，使用 Azure Active Directory 全球实例的客户可以使用 Azure Active Directory 高级和基本版。由中国 21Vianet 运营的 Microsoft Azure 服务目前不支持 Azure Active Directory 高级和基本版。有关详细信息，请在 [Azure Active Directory 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)与我们联系。




## 步骤 1：注册 Active Directory Premium

若要注册，请参阅[批量许可](http://www.microsoft.com/zh-cn/licensing/how-to-buy/how-to-buy.aspx)网站。

## 步骤 2：激活许可计划

这是否为通过 Microsoft 的企业批量许可计划购买的第一个许可计划？
如果是，则在完成购买后，你将收到一封确认电子邮件。
必须通过这封电子邮件才能激活你的第一个许可计划。
以后针对此目录购买任何许可证时，都会在同一目录中自动激活许可证。



**若要激活你的许可计划，请执行以下步骤之一：**


1. 若要开始激活，请单击“登录”或“注册”。

    ![登录][1]



    - 如果你已有现有的租户，请单击“登录”以使用现有的管理员帐户登录。需要从必须激活许可证的目录中使用全局管理员凭据登录。

    - 如果你想创建新的 Azure Active Directory 租户以便与许可计划搭配使用，请单击“注册”以打开“创建帐户配置文件”对话框。

        ![创建帐户配置文件][2]

完成时将显示下列对话框，以确认激活你的租户的许可计划。

![确认][3]

## 步骤 3：激活 Azure Active Directory 访问权限

如果你以前用过 Microsoft Azure，则可以转到[步骤 4](#step-4-assign-license-to-user-accounts)。

为你的目录预配许可证时，你会收到**欢迎电子邮件**。 
此电子邮件会确认你可以开始管理 Azure Active Directory Premium 或企业移动性套件的许可证和功能。

如果在收到欢迎电子邮件之前尝试激活 Azure Active Directory 访问权限，你会看到以下错误消息。

![无法访问][9]

请在收到该电子邮件后的几分钟内重试。

订阅中的新管理员也可以通过此链接激活他们对 Azure 经典门户的访问权限。






**若要激活 Azure Active Directory 访问权限，请执行以下步骤：**

1. 在**欢迎电子邮件**中单击“登录”。
    
    ![欢迎电子邮件][4]

2. 成功登录后，需要以手机验证的方式完成二重身份验证：

    ![手机验证][5]

激活可能需要几分钟的时间。激活访问权限后，棕色的进度条便会消失，而你就可以单击“门户”。

![正在设置，请稍候][6]

在这种情况下，Azure 访问权限将限制为 Azure Active Directory。

![Azure 功能][7]

你可能已在以前的使用过程中获得了对 Azure 的访问权限，此外，你可以通过激活更多的 Azure 订阅将“Azure Active Directory 访问权限”升级为 Azure 完全访问权限。在这些情况下，Azure 经典门户会提供更多功能。

![Azure 功能][8]



## 步骤 4：向用户帐户分配许可证

在开始使用购买的计划之前，需要先手动将许可证分配到组织中的用户帐户，以便他们能够使用 Premium 随附的丰富功能。请使用以下步骤向用户分配许可证，以便他们能够使用 Azure Active Directory Premium 功能。

**若要将许可证分配给用户，请执行以下步骤：**

1. 以你想要自定义的目录的全局管理员身份登录 Azure 经典门户。
2. 单击“Active Directory”，然后选择你要在其中分配许可证的目录。
3. 选择“许可证”选项卡，选择“Active Directory Premium”或“企业移动套件”，然后单击“分配”。

    ![许可计划][10]

4. 在对话框中，选择要向其分配许可证的用户，然后单击复选标记图标以保存更改。

    ![分配许可证][11]

### 许可限制

某些许可计划是其他许可计划的子集或超集。通常情况下，如果已经向某个用户分配了某个许可计划，则不能再次向其分配该许可计划。如果你打算分配作为超集的许可计划，则需要先删除子集许可计划。

### 许可要求

向用户分配许可证时，可以在其帐户的属性中指定主要使用位置。如果未指定使用位置，则会将租户的位置自动分配给该用户。

![用户位置][12]

用于 Microsoft 云服务的服务和功能可用性根据国家或地区的不同而异。某种服务（例如 IP 语音 (VoIP)）可能在一个国家或地区可用，但在另一个国家或地区却不可用。出于特定国家或地区法律要求的原因，服务中的功能可能会受到限制。若要了解提供的某个服务或功能是否存在限制，请在该服务的许可限制站点上查找你所在的国家或地区。

## 后续步骤

- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding/)


<!--Image references-->
[1]: ./media/active-directory-get-started-premium/MOLSEmail.png
[2]: ./media/active-directory-get-started-premium/MOLSAccountProfile.png
[3]: ./media/active-directory-get-started-premium/MOLSThankYou.png
[4]: ./media/active-directory-get-started-premium/AADEmail.png
[5]: ./media/active-directory-get-started-premium/SignUppage.png
[6]: ./media/active-directory-get-started-premium/Subscriptionspage.png
[7]: ./media/active-directory-get-started-premium/Premiuminportal.png
[8]: ./media/active-directory-get-started-premium/Premiuminportal_large.png
[9]: ./media/active-directory-get-started-premium/Signuppage_oops.png
[10]: ./media/active-directory-get-started-premium/contosolicenseplan.png
[11]: ./media/active-directory-get-started-premium/Assignlicensespicker.png
[12]: ./media/active-directory-get-started-premium/Usagelocation.png

<!---HONumber=AcomDC_0718_2016-->