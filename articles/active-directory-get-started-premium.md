<properties 
	pageTitle="Azure Active Directory Premium 入门" 
	description="本主题介绍如何注册 Azure Active Directory Premium 版本。" 
	services="active-directory" 
	documentationCenter="" 
	authors="Justinha"  
	manager="TerryLan" 
	editor="LisaToft"/>

<tags 
	ms.service="active-directory" 
	ms.date="05/05/2015"
	wacn.date="08/29/2015"/>

# Azure Active Directory Premium 入门

Azure Active Directory 有三个版本：免费版 (Free)、基本版 (Basic) 和高级版 (Premium)。Azure 或 Office 365 订阅随附了免费版。可以通过 [Microsoft 企业协议](https://www.microsoft.com/en-us/licensing/licensing-programs/enterprise.aspx)或[开放式批量许可](https://www.microsoft.com/en-us/licensing/licensing-programs/open-license.aspx)计划获取基本和高级版。Azure 和 Office 365 订阅者还可以在线购买 Active Directory Premium。[在此处登录](https://portal.office.com/Commerce/Catalog.aspx)进行购买。

> [AZURE.NOTE]在中国，使用 Azure Active Directory 全球实例的客户可以使用 Azure Active Directory 高级和基本版。由中国 21Vianet 运营的 Windows Azure 服务目前不支持 Azure Active Directory 高级和基本版。有关详细信息，请在 [Azure Active Directory 论坛](http://feedback.azure.com/forums/169401-azure-active-directory)与我们联系。

**企业移动性套件**中也包含了 Azure Active Directory Premium。通过企业移动性套件，组织能够根据一个许可计划，以经济高效的方式结合使用 Microsoft Intune、Azure 权限管理和 Active Directory Premium 服务。有关详细信息，请参阅[企业移动性套件](https://www.microsoft.com/en-us/server-cloud/enterprise-mobility/overview.aspx)网站。

若要立即开始使用 Azure Active Directory Premium 功能，请按照以下步骤操作。这些步骤也适用于 Azure Active Directory 基本版。

## 步骤 1：注册 Active Directory Premium

若要注册，请参阅[批量许可](http://www.microsoft.com/en-us/licensing/how-to-buy/how-to-buy.aspx)网站。

## 步骤 2：激活许可计划

如果这是你第一次通过 Microsoft 的企业批量许可计划购买许可计划，则需要先激活你的许可计划，然后才能开始从 Azure Active Directory 目录内部分配许可证。为此，你必须单击完成首次许可计划购买后收到的确认电子邮件（参阅下图）中的“登录”或“注册”链接。在针对此目录的任何后续购买时，都会在同一目录中自动激活许可证。

![][1]

如果你拥有一个现有租户，请选择“登录”链接以使用现有的管理员帐户登录。应该从必须激活许可证的目录中使用全局管理员凭据登录，这一点很重要。

如果你想要创建与许可计划配合使用的新 Azure Active Directory 租户，则应选择“注册”链接，这样将会转到以下屏幕。

![][2]

完成从电子邮件启动的注册过程或登录过程后，你将会看到以下屏幕，其中确认正在为你的租户激活许可计划。

![][3]

## 步骤 3：激活 Azure Active Directory 访问权限

将许可证设置到目录中后，你将会收到一封欢迎电子邮件（参阅下图），其中确认你可以开始管理 Azure Active Directory Premium 或企业移动性套件的许可证和功能。如果你以前用过 Windows Azure，则可以转到 http://manage.windowsazure.cn 以分配新的许可证（有关更多说明，请参阅下面的“步骤 4”）。如果你不熟悉 Windows Azure，则选择电子邮件中的“登录”链接或转到[访问 Azure Active Directory 激活页](https://account.windowsazure.cn/signup?offer=MS-AZR-0110P)时，系统将引导你完成一系列步骤，以帮助你通过 Azure 管理门户访问目录。

![][4]

成功登录后，需要通过提供手机号码并对其进行验证，来完成第二因素身份验证屏幕（参阅下图）上的操作。完成此操作后，便可以通过选择“注册”来激活对 Azure Active Directory 的访问权限。

![][5]

激活可能需要几分钟时间（如下图所示），激活访问权限后，棕色的进度条将会消失，现在你可以单击右上角的“门户”链接或导航到 [Azure 管理门户](http://manage.windowsazure.cn)。

![][6]

在这种情况下，Azure 访问权限将限制为 Azure Active Directory。

![][7]

你可能在以前的使用中已获取了对 Azure 的访问权限；此外，你可以通过激活更多的 Azure 订阅将“Azure Active Directory 访问权限”升级为 Azure 完全访问权限。在这种情况下，管理门户中的功能将比下图中所示的功能要多。

![][8]

如果在收到上述欢迎电子邮件之前尝试激活 Azure Active Directory 访问权限，则可能会看到以下错误消息。请在收到该电子邮件后的几分钟内重试。

![][9]

订阅中的新管理员也可以通过此链接激活他们对管理门户的访问权限。

## 步骤 4：向用户帐户分配许可证

在开始使用购买的计划之前，需要先手动将许可证分配到组织中的用户帐户，以便他们能够使用高级版随附的丰富功能。请使用以下步骤向用户分配许可证，以便他们能够使用 Azure Active Directory Premium 功能。

向用户分配许可证：

1. 以你想要自定义的目录的全局管理员身份登录管理门户。
2. 单击“Active Directory”，然后选择你要在其中分配许可证的目录。
3. 选择“许可证”选项卡，选择“Active Directory Premium”或“企业移动性套件”，然后单击“分配”。

    ![][10]

4. 在对话框中，选择要向其分配许可证的用户，然后单击复选标记图标以保存更改。

    ![][11]

## 许可限制

某些许可计划是其他许可计划的子集或超集。在大多数情况下，如果已经向某位用户分配了某个许可计划，则不能再次向其分配该许可计划。如果你打算分配作为超集的许可计划，则需要先删除子集许可计划。

## 许可要求

向用户分配许可证时，可以在其帐户的属性中为其指定主要使用位置，如下图所示。如果未指定使用位置，则会将租户的位置自动分配给该用户。

![][12]

用于 Microsoft 云服务的服务和功能可用性因国家或地区而异。某项服务（例如 IP 语音 (VoIP)）可能在一个国家或地区可用，但在另一个国家或地区却不可用。出于特定国家或地区法律要求的原因，服务中的功能可能会受到限制。若要了解某项服务或功能是否存在限制，请在该服务的许可限制站点上查找你所在的国家或地区。

## 后续步骤

- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding)
- [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports)

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

<!---HONumber=67-->