<properties
	pageTitle="管理 Azure AD 目录"
	description="本主题介绍什么是 Azure AD 租户，以及如何管理 Azure AD 目录。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	writer="markvi"
	manager="stevenpo"
	editor="LisaToft"/>

<tags 
	ms.service="active-directory" 
	ms.date="01/25/2016"
	wacn.date="06/03/2016" />

# 管理 Azure AD 目录

## 什么是 Azure AD 租户？

在物理工作区中，可以将“租户”一词定义为占用某个大厦的组或公司。例如，你的组织可能在某个大厦中拥有办公空间。此大厦可能与其他几个组织在一条大街上。你的组织将被视为该大厦的租户。此大厦是你的组织的资产并提供安全性以确保你们可以安全地开展业务。此外，它与你们街道上的其他企业是分隔开的。这可确保你的组织以及其中的资产与其他组织隔离。

在启用云的工作区中，可以将“租户”定义为拥有并管理该云服务的特定实例的客户端或组织。租户使用 Microsoft Azure 提供的标识平台，它只是你的组织在注册 Azure 或 Office 365 等 Microsoft 云服务时接收并拥有的 Azure Active Directory (Azure AD) 专用实例。

每个 Azure AD 目录都是独特的，独立于其他 Azure AD 目录。就像公司办公大楼是你的组织特有的安全资产一样，根据设计，Azure AD 目录也是仅供你的组织使用的安全资产。Azure AD 体系结构隔离了客户数据和身份信息，避免混合存放。这意味着，一个 Azure AD 目录的用户和管理员不可能意外或恶意性地访问另一目录中的数据。

![][1]

## 如何获取 Azure AD 目录？

Azure AD 在大多数 Microsoft 云服务的后面提供核心目录和身份管理功能，其中包括：

- Azure
- Microsoft Office 365
- Microsoft Dynamics CRM Online
- Microsoft Intune

当你注册其中任何一个 Microsoft 云服务时，便会获得一个 Azure AD 目录。你可根据需要创建更多的目录。例如，可以将第一个目录保留为生产目录，然后创建另一个目录进行测试或过渡。 

> [AZURE.NOTE]
> 在你注册第一个服务后，我们建议你在注册其他 Microsoft 云服务时使用与你的组织关联的同一个管理员帐户。

当你首次注册 Microsoft 云服务时，系统将提示你提供有关组织以及组织 Internet 域名注册的详细信息。然后，这些信息将用来为你的组织创建一个新的 Azure AD 目录实例。当你订阅多个 Microsoft 云服务时，将使用这同一个目录对登录尝试进行身份验证。

其他服务将充分利用所有现有的用户帐户、策略、设置或者你配置的本地目录集成，以帮助改进你的组织本地的标识基础结构与 Azure AD 之间的效率。

例如，如果你最初注册了 Windows Intune 订阅并完成了通过部署目录同步和/或单一登录服务器将本地 Active Directory 与 Azure AD 目录进一步集成所需的步骤，则可以注册其他 Microsoft 云服务（例如 Office 365），该服务也可以利用目前用于 Windows Intune 的目录集成优势。

有关将本地目录与 Azure AD 集成的详细信息，请参阅[目录集成](/documentation/articles/active-directory-aadconnect/)。

### 将 Azure AD 目录与新的 Azure 订阅相关联

可以将新的 Azure 订阅，与用于对现有 Office 365 或 Microsoft Intune 订阅的登录进行身份验证的相同目录进行关联。请使用你的工作或学校帐户登录到 Azure 经典管理门户。经典管理门户将返回一条消息，指出找不到该帐户的任何订阅。选择"注册 Azure"，你的目录将可用于在门户中进行管理。有关详细信息，请参阅[在 Azure 中管理 Office 365 订阅的目录](/documentation/articles/active-directory-how-subscriptions-associated-directory/#manage-the-directory-for-your-office-365-subscription-in-azure)。

<!--For a video about common usage questions for Azure AD, see [Azure Active Directory - Common Sign-up, sign-in and usage questions](http://channel9.msdn.com/Series/Windows-Azure-Active-Directory/WAADCommonSignupsigninquestions).-->

### 通过以组织名义注册 Microsoft 云服务来创建 Azure AD 目录

如果你尚未订阅 Microsoft 云服务，请使用下面的链接之一来注册。注册第一个服务后，将自动创建 Azure AD 目录。

- [Azure](https://account.windowsazure.cn/organization)
- [Office 365](http://products.office.com/zh-cn/business/compare-office-365-for-business-plans)
- [Microsoft Intune](https://account.manage.microsoft.com)

### 管理 Azure 设置的默认目录

现在，当你注册 Azure 时，将自动创建一个目录，你的订阅将与该目录相关联。但是，如果你最初是在 2013 年 10 月之前注册 Azure 的，则不会自动创建一个目录。在那种情况下，Azure 可能已为你的帐户进行了"回填"，即为其设置了默认目录。那么，你的订阅已与该默认目录相关联。

在 2013 年 10 月进行了目录回填，作为对 Azure 安全模型总体增强的一部分。它有助于向所有 Azure 客户提供组织标识功能，并确保通过目录中的一个用户即可访问所有 Azure 资源。没有目录就无法使用 Azure。因此，在 2013 年 7 月 7 日之前注册但没有目录的任何用户都必须创建一个目录。如果你已创建了一个目录，则你的订阅已与该目录相关联。 

使用 Azure AD 不收取费用。目录是免费资源。还有一个额外的单独许可的 Azure Active Directory 高级版级别，它提供额外的功能，例如公司品牌和自助密码重置。 

若要更改你的目录的显示名称，请在门户中单击该目录并单击“配置”。如本主题后面所述，你可以添加新目录或删除不再需要的目录。若要将你的订阅与其他目录相关联，请在左侧导航窗格中单击“设置”扩展，然后在“订阅”页的底部单击“编辑目录”。你还可以使用已注册的 DNS 名称而非默认的 *.onmicrosoft.com 域来创建自定义域，对于 SharePoint Online 之类的服务这样做可能更好。

## 如何管理目录数据

作为一个或多个 Microsoft 云服务订阅的管理员，你可以使用 Azure 经典管理门户、Microsoft Intune 帐户门户或 Office 365 管理中心来管理你的组织目录数据。你还可以下载并运行[用于 Windows PowerShell 的 Azure Active Directory 模块](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx) cmdlet 来帮助管理 Azure AD 中存储的数据。 

通过上述任一门户（或 cmdlet），你可以执行以下操作：

- 创建和管理用户帐户和组帐户
- 管理你组织订阅的相关云服务
- 设置与目录服务的本地集成

Azure 经典管理门户、Office 365 管理中心、Microsoft Intune 帐户门户和 Azure AD cmdlet 均从与组织的目录关联的 Azure AD 的单个共享实例读取数据并将数据写入到该实例中，如下图所示。门户（或 cmdlet）通过这种方式充当用于输入和/或修改目录数据的前端接口。

![][2]

这些用于管理用户和组的帐户门户及关联的 Azure AD PowerShell cmdlet 均构建在 Azure AD 平台之上。

如果你在上述服务之一的上下文中登录时使用任何门户（或 cmdlet）更改了组织数据，则当你下次在该服务的上下文中登录时，此更改也会显示在其他门户中，因为此数据在你订阅的 Microsoft 云服务之间共享。例如，如果使用 Office 365 管理中心阻止某个用户登录，则该操作会阻止该用户登录到你的组织当前订阅的任何其他服务。如果在 Microsoft Intune 帐户门户的上下文中请求相同的用户帐户，则会看到该用户被阻止。

## 如何添加和管理多个目录？

可以在 Azure 经典管理门户中添加 Azure AD 目录。在左侧选择“Active Directory”扩展，然后单击“添加”。

可将每个目录作为完全独立的资源进行管理：每个目录都是一个具有完整功能的对等方，在逻辑上独立于你所管理的其他目录；目录之间不存在父子关系。目录之间的这种独立性包括资源独立性、管理独立性和同步独立性。 

- **资源独立性**。在一个目录中创建或删除一个资源不影响另一个目录中的任何资源，但对于外部用户来说，情况并不完全如此，具体如下所述。如果将自定义域“contoso.com”用于一个目录，则不能将它用于任何其他目录。 
- **管理独立性**。如果目录“Contoso”的某个非管理用户创建了测试目录“Test”，那么： 
    - ◦目录同步工具，用于将数据与单个 AD 林同步。 
    - ◦目录“Contoso”的管理员对目录“Test”没有直接管理特权，除非“Test”的管理员专门向其授予了这些特权。“Contoso”的管理员可以控制对目录“Test”的访问，因为他们可以控制创建“Test”的用户帐户。 

    另外，如果你更改（添加或删除）某个用户在一个目录中的管理员角色，这项更改不会影响该用户在另一个目录中可能具有的任何管理员角色。 


- **同步独立性**。可以独立配置每个 Azure AD，以便通过下列任一工具的单个实例同步数据： 
    - 目录同步工具，用于将数据与单个 AD 林同步
    - 适用于 Forefront Identity Manager 的 Azure Active Directory 连接器，用于将数据与一个或多个本地林和/或非 AD 数据源同步。 

另请注意，与其他 Azure 资源不同，你的目录不是 Azure 订阅的子资源。因此，如果取消 Azure 订阅或让其过期，则仍可以使用 Azure AD PowerShell、Azure Graph API 或其他界面（例如 Office 365 管理中心）访问目录数据。还可以将其他订阅与目录相关联。

## 如何删除 Azure AD 目录？
全局管理员可以从门户删除 Azure AD 目录。删除某个目录时，该目录中包含的所有资源也将被删除；因此，在删除之前，应确认你不需要该目录。 

> [AZURE.NOTE]
> 如果用户是使用工作或学校帐户登录的，则该用户不得尝试删除其主目录。例如，如果用户是作为 joe@contoso.onmicrosoft.com 登录的，则该用户不能删除默认域为 contoso.onmicrosoft.com 的目录。

### 删除 Azure AD 目录时必须符合的条件

Azure AD 要求删除目录之前必须符合特定的条件。这可以降低删除目录后对用户或应用程序造成负面影响（例如，影响用户登录 Office 365 或访问 Azure 中的资源）的风险。例如，如果用于订阅的某个目录被无意删除，则用户无法访问该订阅的 Azure 资源。 

需检查以下条件：

- 该目录中的唯一用户是将要删除该目录的全局管理员。只有在删除所有其他用户后，才能删除该目录。如果用户是从本地同步的，则需要关闭同步，并且必须使用经典管理门户或用于 Windows PowerShell 的 Azure 模块从云目录中删除这些用户。不要求删除组或联系人，例如，从 Office 365 管理中心添加的联系人。
- 该目录中不能有任何应用程序。只有在删除所有应用程序后，才能删除该目录。 
- 与该目录关联的任何 Microsoft Online Services（例如 Microsoft Azure、Office 365 或 Azure AD Premium）不能存在任何订阅。例如，如果在 Azure 中为你创建了一个默认目录，并且你的 Azure 订阅仍然依赖于此目录进行身份验证，则你不能删除此目录。类似地，如果其他用户已将订阅与某个目录相关联，则你无法删除该目录。若要将你的订阅与其他目录相关联，请登录到 Azure 经典管理门户，并在左侧导航窗格中单击“设置”。然后，在“订阅”页的底部单击“编辑目录”。有关 Azure 订阅的详细信息，请参阅 [Azure 订阅与 Azure AD 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory/)。 

    > [AZURE.NOTE]
    > 如果用户是使用工作或学校帐户登录的，则该用户不得尝试删除其主目录。例如，如果用户是作为 joe@contoso.onmicrosoft.com 登录的，则该用户不能删除默认域为 contoso.onmicrosoft.com 的目录。

- 不能有任何多重身份验证提供程序链接到该目录。


## 其他资源

- [Azure AD 论坛](https://social.msdn.microsoft.com/Forums/home?forum=WindowsAzureAD)
- [Azure 多重身份验证论坛](https://social.msdn.microsoft.com/Forums/home?forum=windowsazureactiveauthentication)
- [堆栈溢出](http://stackoverflow.com/questions/tagged/azure)
- [以组织身份注册 Azure](/documentation/articles/sign-up-organization/)
- [使用 Windows PowerShell 管理 Azure AD](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx)
- [在 Azure AD 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)

<!--Image references-->
[1]: ./media/active-directory-administer/aad_portals.png
[2]: ./media/active-directory-administer/azure_tenants.png



<!---HONumber=Mooncake_0411_2016-->