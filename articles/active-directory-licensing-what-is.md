<properties
    pageTitle="什么是 Microsoft Azure Active Directory 许可？| Azure"
    description="介绍 Microsoft Azure Active Directory 许可及其工作原理、入门方法和最佳实践，涉及的产品包括 Office 365、Microsoft Intune 及 Azure Active Directory Premium 和 Basic 版本"
    services="active-directory"
    keywords="Azure AD 许可"
    documentationCenter=""
    authors="curtand"
    manager="stevenpo"
    editor=""/>

<tags
   ms.service="active-directory"
   ms.date="05/16/2016"
   wacn.date="06/23/2016"/>

# 什么是 Microsoft Azure Active Directory 许可？

##说明
Azure Active Directory (Azure AD) 是 Microsoft 的标识即服务 (IDaaS) 解决方案与平台。Azure AD 提供许多功能与技术版本，从可配合任何 Microsoft 服务（例如 Office 365、Dynamics、Microsoft Intune 和 Azure）使用的 Azure AD 免费版（Azure AD 在此模式中不产生任何使用费），到许多 Azure AD 付费版，例如 Enterprise Mobility Suite (EMS)、Azure AD Premium 与 Basic，以及 Azure Multi-Factor Authentication (MFA)。与许多 Microsoft 联机服务（如 Office 365、Microsoft Intune 和 Azure AD）一样，大多数 Azure AD 付费版都是通过向每位用户授权来提供。在这种情况下，购买的服务以一个或多个订阅来表示，每个订阅在租户中包含预先购买数目的许可证。基于用户的权利是通过许可证分配、在用户与产品之间创建关联、为用户启用服务组件并占用一个预付费许可证来实现的。

[立即试用 Azure AD 高级版。](https://portal.office.com/Signup/Signup.aspx?OfferId=01824d11-5ad8-447f-8523-666b0848b381&ali=1#0)

> [AZURE.NOTE] Azure AD 经典管理门户是 Azure 经典管理门户的一部分。尽管使用 Azure AD 无需购买任何 Azure，但访问此门户需要有效的 Azure 订阅或 [Azure 试用版订阅](/pricing/free-trial/)。

有关 Azure AD 服务功能的一般概述，请参阅[什么是 Azure AD](/documentation/articles/active-directory-whatis/)。
[了解有关 Azure AD 服务级别的详细信息](/support/legal/sla/)

> [AZURE.NOTE]  Azure 即付即用订阅则不相同：尽管也在目录中显示，但是这些订阅可以创建 Azure 资源并将资源映射到付款方式。在这种情况下，不存在与订阅关联的许可证计数。用户与订阅的关联就是用户管理订阅资源的访问权限，是通过授予用户权限，使他们能够操作已映射到订阅的 Azure 资源来实现的。


##Azure AD 许可的工作原理

基于许可证（基于权利）的 Azure AD 服务的工作原理是在 Azure AD 目录/服务租户中激活订阅。一旦激活订阅，服务功能就可以由目录/服务管理员管理，并可由许可的用户使用。

当购买或激活 Enterprise Mobility Suite、Azure AD Premium 或 Azure AD Basic 时，目录会随着订阅更新，包括其有效期和预付许可证。可以在 Azure 经典管理门户中特定目录的“许可证”选项卡下查看订阅信息，包括状态、下一生命周期事件，以及已分配或可用的许可证数目。这也是管理许可证分配的最佳位置。

每个订阅都包含一个或多个服务计划，每个计划映射服务类型（例如，Azure AD、Azure MFA、Microsoft Intune、Exchange Online 或 SharePoint Online）包含的功能级别。Azure AD 许可证管理不需要服务计划级别管理。这与 Office 365 不同，Office 365 依赖于此高级配置模式来管理对包含的服务的访问。Azure AD 则依赖于服务配置来启用功能和管理单个权限。

一般而言，可以在 Azure 经典管理门户中特定目录的“许可证”选项卡下管理 Azure AD 订阅信息。Azure AD 订阅（Azure AD Premium 除外）不会显示在 Office 门户中。

> [AZURE.IMPORTANT] Azure AD Premium 和 Basic 以及 Enterprise Mobility Suite 订阅局限于其预配的目录/租户。订阅不能在目录之间拆分，也不能用于向其他目录中的用户授权。在目录之间可以移动订阅，但是需要提交支持票证，如果订阅是直接购买的，则需要取消并重新购买。

> 通过批量许可购买 Azure AD 或 Enterprise Mobility Suite 时，如果协议包含其他 Microsoft Online Services（例如 Office 365），将自动激活订阅。

付费 Azure AD 功能可以扩展目录的范围。示例包括：
- 基于组的应用程序分配，可在你管理的特定应用程序下启用。
- 目录配置下或特定组内提供高级和自助式组管理功能。
- 高级安全报告位于“报告”选项卡上
- 云应用程序发现在 Azure 门户的“标识”下显示。

###分配许可证
只要获取了订阅，你就能配置付费功能；但是，要使用 Azure AD 付费功能，就必须将许可证分配给适当的个人。一般而言，应访问 Azure AD 付费功能或者通过 Azure AD 付费功能管理的人员都必须被分配许可证。许可证分配是用户与购买的服务（例如 Azure AD Premium、Basic 或 Enterprise Mobility Suite）之间的映射。

管理目录中哪些用户应该拥有许可证很简单。只要通过 Azure AD 经典管理门户分配给组来创建分配规则，或者通过门户、PowerShell 或 API 直接将许可证分配给适当的人员即可。将许可证分配给组时，将对所有组成员分配许可证。如果在组中添加或删除用户，则会相应地分配或删除其许可证。组分配可以使用可用的任何组管理功能，而且与基于组的应用程序分配一致。使用这种方法，你可以设置规则以便为目录中所有用户进行自动分配，确保有适当职务的每个人都获得许可证，甚至委派决策权给组织中的其他管理人员。

通过基于组的许可证分配，缺少使用位置的任何用户在分配期间将继承目录位置。管理员可以随时更改此位置。在自动分配因错误而失败的情况下，该许可证类型下的用户信息将反映该状态。

##Azure AD 许可入门

Azure AD 很容易入门；你始终可以在注册免费 Azure 试用版过程中创建目录。[了解有关以组织身份注册的详细信息。](/documentation/articles/sign-up-organization/)以下内容可帮助你确保目录最适合你可能要使用或打算使用的其他 Microsoft 服务，以及获取服务的目标。

以下是几项最佳实践：
- 如果你已在使用 Microsoft 的任何组织服务，则已经获得了一个 Azure AD 目录。在此情况下，应该为其他服务继续使用相同的目录，以便可以跨服务使用核心标识管理（包括预配和混合 SSO）。用户将获得单一登录体验，并且可以受益于更丰富的跨服务功能。因此，如果你决定为员工购买 Azure AD 付费服务，建议使用相同的目录。
- 如果打算为不同的一组用户（合作伙伴、客户等）使用 Azure AD，或者想要评估 Azure AD 服务，并想要隔离生产服务，或者想要为服务设置沙箱环境，则建议先通过 Azure 经典管理门户创建新的目录。[详细了解如何在 Azure 经典管理门户中创建新的 Azure AD 目录](/documentation/articles/active-directory-licensing-directory-independence/)。你可以使用自己的帐户，以具有全局管理员权限的外部用户身份创建新目录。当你使用此帐户登录 Azure 经典管理门户时，将会看到这个目录，并且能够访问所有的目录管理任务。我们建议创建一个具有适当权限的本地帐户，用于管理其他 Microsoft 服务（即无法通过 Azure 经典管理门户访问的服务）。[详细了解如何在 Azure AD 中创建用户帐户](/documentation/articles/active-directory-create-users/)。

> [AZURE.NOTE] Azure AD 支持“外部用户”，这是在 Azure AD 实例中使用 Microsoft 帐户 (MSA) 或另一个目录中的 Azure AD 标识所创建的用户帐户。尽管我们正在努力将此功能扩展到所有 Microsoft 的组织服务，但目前某些服务体验不支持这些帐户；例如，Office 365 系统经典管理门户目前不支持这些用户。因此，使用 Microsoft 帐户的外部用户将无法访问 Office 365 系统经典管理门户，而来自其他 Azure AD 目录的外部用户将被忽略。在后一种情况下，只有用户的本地帐户（最初创建用户的 Azure AD 或 Office 365 目录）才能通过这些体验进行访问。

如前所述，Azure AD 有不同的付费版本。这些版本在其可购性方面略有不同：


| 产品 | EA/VL | 打开 | 	CSP | 	MPN 使用权限 | 	直接购买 | 试用 |
|---|---|---|---|---|---|---|
| Enterprise Mobility Suite |	X |	X |	X |	X | |	 	X |
| Azure AD Premium | X | X | X | | X | X |
| Azure AD Basic | X | X | X | X | <br /> | <br /> |

###选择一个或多个许可证试用版
 在所有情况下，都可以通过在目录中的“许可证”选项卡上选择所需的特定试用版，来启用 Azure AD Premium 或 Enterprise Mobility Suite 试用版订阅。任一试用版都包含 30 天的订阅和 100 个许可证。

![Azure Active Directory 试用版许可计划](./media/active-directory-licensing-what-is/trial_plans.png)

![Enterprise Mobility Suite 试用版许可计划](./media/active-directory-licensing-what-is/EMS_trial_plan.png)

![活动试用版许可计划](./media/active-directory-licensing-what-is/active_license_trials.png)

###分配许可证
激活订阅之后，你应该分配一个许可证给自己，然后刷新浏览器以确保可以看到所有功能。下一步是将许可证分配给需要访问或要包含在 Azure AD 付费功能中的用户。如前面在“分配许可证”中所述，执行这项操作的最好方法是识别代表目标对象的组，并将许可证分配给该组；这样，在组的生命周期内添加到组或从组中删除的用户将被分配许可证或删除许可证。

若要将许可证分配给组或个人用户，请选择想要分配的许可证计划，然后在命令栏上单击“分配”。

![活动试用版许可计划](./media/active-directory-licensing-what-is/assign_licenses.png)

在所选计划的分配对话框中，你可以选择用户，并将他们添加到右侧的“分配”列。你可以逐页查看用户列表，或者使用用户网格右上角的放大镜来搜搜索特定的个人。若要分配组，请从“显示”菜单中选择“组”，然后单击右侧的复选标记以刷新显示的分配。

![将许可证分配到组](./media/active-directory-licensing-what-is/assign_licenses_to_groups.png)

现在可以搜索或逐页查看组，并以相同的方式将其添加到“分配”列。你可以使用这种方式，通过单个操作分配用户与组的组合。若要完成分配过程，请单击页面右下角的勾选按钮。

![许可证分配进度消息](./media/active-directory-licensing-what-is/license_assignment_progress_message.png)

分配组后，其成员将在 30 分钟内（通常为 1-2 分钟）继承许可证。

在 Azure AD 中执行许可证分配期间，可能会发生分配错误，但这种情况相对很少见。潜在的分配错误仅限于：
- 分配冲突 — 以前为用户分配的许可证与最新许可证不兼容。在此情况下，若要分配新的许可证，必须先删除以前的许可证。
- 超过可用的许可证数目 — 当分配组中的用户数目超过可用的许可证数目时，用户的分配状态将反映因缺少许可证而发生的分配失败。

###查看已分配的许可证

“许可证”选项卡上会显示已分配许可证的摘要视图，其中包括可用许可证数、已分配许可证数，以及下一订阅生命周期事件。

![查看已分配的许可证数目](./media/active-directory-licensing-what-is/view_assigned_licenses.png)

当你在许可证计划中导航时，将看到已分配用户和组的详细列表，其中包括分配状态和路径（直接分配或者继承自一个或多个组）。

![为许可计划分配的许可证详细视图](./media/active-directory-licensing-what-is/assigned_licenses_detail.png)

删除许可证的操作与分配许可证一样简单。如果用户是直接分配的或属于某个已分配的组，则你可以通过选择许可证类型，选择“删除”，将用户或组添加到删除列表，然后确认操作来删除许可证。或者，你可以打开许可证类型，选择特定的用户或组，然后在命令栏上点击“删除”。若要终止用户从某个组继承许可证，只需从该组中删除该用户。

###延期试用

通过 Office 365 门户可以自助延长客户试用期限。客户管理员可以导航到 [Office 门户](https://portal.office.com/#Billing)（访问权限取决于对 Office 门户的权限），然后选择“Azure AD Premium 试用版”。单击“延期试用”链接，然后按照说明操作。你需要输入信用卡，但无需付费。

![在 Office 门户中延长许可证试用期限](./media/active-directory-licensing-what-is/extend_license_trial.png)

客户还可以通过提交支持请求来请求来延长试用期限。客户管理员可以导航到 Office 365 门户[支持页](http://aka.ms/extendAADtrial)（访问权限取决于对 Office 支持页的权限）。在此页上的“功能”下面选择“订阅和试用”，并在“症状”下面选择“试用问题”。最后，根据情况输入信息

![使用支持请求延长许可证试用期限](./media/active-directory-licensing-what-is/alternate_office_aad_trial_extension.png)

## 后续步骤

现在，你可能已准备好配置和使用某些 Azure AD Premium 功能。

- [自助密码重置](/documentation/articles/active-directory-manage-passwords/)
- [自助组管理](/documentation/articles/active-directory-accessmanagement-self-service-group-management/)
- [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health/)
- [将组分配到应用程序](/documentation/articles/active-directory-manage-groups/)
- [Azure Multi-Factor Authentication](/documentation/articles/multi-factor-authentication/)
- [直接购买 Azure AD Premium 许可证](http://aka.ms/buyaadp)

<!---HONumber=Mooncake_0613_2016-->