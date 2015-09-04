<properties
   pageTitle="什么是 Windows Azure Active Directory 许可？| Windows Azure"
   description="Windows Azure AD 许可、其工作原理，如何入门和最佳做法（包括 Office 365、Microsoft Intune 和 Azure Active Directory Premium 和 Basic 版本）的描述"
   services="active-directory"
   documentationCenter=""
   authors="curtand"
   manager="swadhwa"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/16/2015"
   wacn.date="08/29/2015"/>

# 什么是 Windows Azure Active Directory 许可？

##说明
Azure Active Directory (AD) 是 Microsoft 的标识即服务 (IDaaS) 解决方案和平台。Azure AD 可提供多种功能和技术版本，包括 Azure AD 免费版（随任何 Microsoft 服务（如 Office 365、Dynamics、Microsoft Intune 和 Azure）一起提供，在此模式下，Azure AD 不会产生任何使用费用）、Azure AD 付费版本（如企业移动性套件 (EMS)）、Azure AD Premium 和 Azure AD Basic，以及 Azure Multi-Factor Authentication (MFA)。和许多 Microsoft 在线服务一样，大部分 Azure AD 付费版本就像在 Office 365、Microsoft Intune 和 Azure AD 中一样通过每用户权利提供。在这些情况下，使用一个或多个订阅来表示服务购买，并且每个订阅在你的租户中包括预购买许可证数目。实现每用户权利的方式为：进行许可证分配、创建用户与产品之间的链接、为用户启用服务组件，并使用一个预付许可证。

Azure AD 管理门户是 Azure 管理门户的一部分。尽管使用 Azure AD 不需要任何 Azure 购买内容，但访问此门户需要处于活动状态的 Azure 订阅或 [Azure 试用版订阅](http://www.windowsazure.cn/pricing/1rmb-trial/)。

有关 Azure AD 服务功能的宽泛概述，请参阅[什么是 Azure AD](/documentation/articles/active-directory-whatis)。

> [AZURE.NOTE]Azure 即付即用订阅有所不同：你的目录中表示了这些订阅，它们可创建 Azure 资源，并将这些资源映射到你的付款方法。在这种情况下，没有与订阅关联的许可证计数。通过向用户授予对映射到订阅的 Azure 资源进行操作的权限，可实现用户与订阅的关联以及用户对管理订阅资源的访问权限。

> [了解有关 Azure AD 服务级别的详细信息](http://azure.microsoft.com/zh-cn/support/legal/sla/)

##Azure AD 许可如何工作？

通过激活 Azure AD 目录/服务租户中的订阅，基于权利或基于许可证的 Azure AD 服务可开始工作。一旦订阅处于活动状态，服务功能就可由目录/服务管理员进行管理，并可由获得许可的用户使用。

购买或激活企业性移动套件、Azure AD Premium 或 Azure AD Basic后，将使用该订阅（包括其有效期和预付的许可证）更新你的目录。你的订阅信息（包括状态、下一个生命周期事件以及已分配许可证或可用许可证的数目）可在 Azure AD 管理门户中特定目录的“许可证”选项卡下获取。这也是管理许可证分配的最佳位置。

每个订阅包含一个或多个服务计划，每个服务计划表示包含的服务类型功能级别；例如，Azure AD、Azure MFA、Microsoft Intune、Exchange Online 或 SharePoint Online。Azure AD 许可证管理不需要服务计划级别管理。这不同于 Office 365（依赖此高级配置模式来管理对包括的服务的访问）。Azure AD 依赖于服务中的配置来启用功能和管理单独的权限。

通常情况下，通过 Azure 管理门户来管理 Azure AD 订阅信息，此信息位于特定目录的“许可证”选项卡下。Azure AD 订阅（Azure AD Premium 除外）不会显示在 Office 门户中。

> [AZURE.IMPORTANT]Azure AD Premium、Azure AD Basic 和企业移动性套件订阅限于其预配的目录/租户。无法在目录之间拆分订阅，也无法将订阅用于为其他目录中的用户授权。可在目录之间移动订阅，但这需要提交支持票证或取消并重新购买（在直接购买的情况下）。

付费 Azure AD 功能可拓宽目录。示例包括：- 基于组分配到应用程序（可在你正在管理的特定应用程序下启用）。- 高级和自助服务组管理功能在目录配置下提供或在特定组内可用。-“高级安全报告”位于“报告”选项卡中。-“云应用程序发现”显示在 Azure 预览门户中的“标识”下。

###分配许可证 (Assigning licenses)
尽管仅需获取订阅即可配置付费功能，但使用 Azure AD 付费功能要求向适当的人员分配许可证。通常，必须向将有权访问 Azure AD 付费功能的用户或通过 Azure AD 付费功能进行管理的用户分配许可证。许可证分配是用户与所购买服务（如 Azure AD Premium、Basic 或企业移动性套件）之间的映射。

管理目录中哪些用户应具有许可证非常简单。它可以通过以下方式实现：将许可证分配给一个组，通过 Azure AD 管理门户创建分配规则；或通过门户、PowerShell 或 API 直接向适当的个人分配许可证。将许可证分配给一个组时，将向所有组成员分配许可证。如果向组添加了用户或从组中删除了用户，则将向这些用户分配相应的许可证或删除相应的许可证。组分配可以利用可用的任何组管理，并且与基于组分配到应用程序一致。可使用此方法设置规则，以便自动为目录中的所有用户进行分配、确保具有相应职务的每个人都具有许可证，甚至将决策委派给组织中的其他管理人员。

使用基于组的许可证分配，缺少使用位置的任何用户都将在分配过程中继承目录位置。管理员可随时更改此位置。如果自动分配因出现错误而失败，则该许可证类型下的用户信息将反映该状态。

##Azure AD 许可入门

Azure AD 许可入门非常轻松；你始终可在注册到免费 Azure 试用版的过程中创建自己的目录。[了解有关以组织身份注册的详细信息](/documentation/articles/sign-up-organization)。以下内容可帮助确保你的目录与你可能正在使用或计划使用的其他 Microsoft 服务以及获取服务的目标保持最佳一致。

此处提供了一些最佳做法：- 如果你已在使用任何 Microsoft 组织服务，则你已经具有 Azure AD 目录。在此情况下，你应该继续对其他服务使用相同的目录，以便可跨服务利用核心标识管理（包括预配和混合 SSO）。你的用户将具有单一登录体验，并将受益于更丰富的跨服务功能。因此，如果你决定为员工购买 Azure AD 付费服务，我们建议使用相同的目录来执行此操作。- 如果你打算对一组不同的用户（合作伙伴和客户等）使用 Azure AD，或者如果想要评估 Azure AD 服务并且想要在单独的生产服务中执行此操作，或者如果你想要为你的服务设置沙盒环境，我们建议你首先通过 Azure 管理门户创建一个新目录。[了解有关在 Azure 管理门户中创建新 Azure AD 目录的详细信息](/documentation/articles/active-directory-licensing-directory-independence)。将使用你的帐户作为具有全局管理员权限的外部用户创建新目录。使用此帐户登录到 Azure 管理门户时，你将能够查看此目录和访问所有目录管理任务。我们建议你创建具有管理其他 Microsoft 服务（不可通过 Azure管理门户访问的服务）的相应权限的本地帐户。[了解有关在 Azure AD 中创建用户帐户的详细信息](/documentation/articles/active-directory-create-users)。

> [AZURE.NOTE]Azure AD 支持“外部用户”，即使用另一个目录中的 Microsoft 帐户 (MSA) 或 Azure AD 标识创建的 Azure AD 实例中的用户帐户。尽管我们一直致力于将此功能扩展到所有 Microsoft 组织服务中，但部分服务体验尚不支持这些帐户；例如，Office 365 管理门户当前不支持这些用户。因此，使用 Microsoft 帐户的外部用户将完全不能访问 Office 365 管理门户，而其他 Azure AD 目录中的外部用户将被忽略。在后一种情况下，仅可通过这些体验访问用户的本地帐户、Azure AD 或最初在其中创建用户的 Office 365 目录。

正如所指出的那样，Azure AD 具有不同的付费版本。这些版本在其购买可用性方面具有一些细微的差异：


| 产品 | EA/VL | 开放 | 	CSP | 	MPN 使用权限 | 	直接购买 | 试用版 |
|---|---|---|---|---|---|---|
| 企业移动性套件 |	X |	X |	X |	X | |	 	X |
| Azure AD Premium | X | X | X | | X | X |
| Azure AD Basic | X | X | X | X | <br /> | <br /> |

###选择一个或多个许可证试用版
 在所有情况下，你都可以通过在目录中的“许可证”选项卡上选择你想要的特定试用版来激活 Azure AD Premium 或企业移动性套件试用订阅。任一试用版都包含具有 100 份许可证的 30 天订阅。

![Azure Active Directory 试用版许可证计划](./media/active-directory-licensing-what-is/trial_plans.png)

![企业移动性套件试用版许可证计划](./media/active-directory-licensing-what-is/EMS_trial_plan.png)

![激活试用版许可证计划](./media/active-directory-licensing-what-is/active_license_trials.png)

###分配许可证 (Assign licenses)
激活试用版后，应将许可证分配给自己并刷新浏览器以确保看到所有功能。下一步是为需要访问付费 Azure AD 功能或需要包含在此类功能内的用户分配许可证。正如我们在上文的“分配许可证 (Assigning licenses)”中所述，执行此操作的最佳方法是标识表示所需受众的组，并向该组分配许可证；采用这种方式时，对于在组的生命周期内添加到组或从组中删除的用户，将向其分配许可证或删除其许可证。

若要向一个组或单个用户分配许可证，请选择你想要分配的许可证计划，然后单击命令栏上的“分配”。

![激活试用版许可证计划](./media/active-directory-licensing-what-is/assign_licenses.png)

在所选计划的“分配”对话框中，你可以选择用户并将其添加到右侧的“分配”列。可以使用用户网格右上角的窥镜翻阅用户列表或搜索特定个人。若要分配组，请从“显示”菜单中选择“组”，然后单击右侧的勾选按钮以刷新显示的分配。

![向组分配许可证](./media/active-directory-licensing-what-is/assign_licenses_to_groups.png)

现在可以采用相同的方式搜索或翻阅组，并将其添加到“分配”列。你可以使用这些方法在一次操作中同时分配用户和组。若要完成分配过程，请单击页面右下角的勾选按钮。

![许可证分配进度消息](./media/active-directory-licensing-what-is/license_assignment_progress_message.png)

分配组后，其成员在 30 分钟内（但通常在 1-2 分钟之内）继承许可证。

在 Azure AD 门户许可证分配过程中，可能出现分配错误，但相对稀少。潜在的分配错误限于：- 分配冲突 - 以前向用户分配的许可证与当前许可证不兼容。在这种情况下，分配新的许可证将需要删除之前的许可证。- 超出可用许可证数 - 当已分配组中的用户数超过可用许可证数目时，用户的分配状态将反映由于缺少许可证而分配失败。

###查看已分配许可证

已分配许可证（包括可用许可证、已分配许可证和下一个订阅生命周期事件）的摘要视图显示在“许可证”选项卡上。

![查看已分配许可证数目](./media/active-directory-licensing-what-is/view_assigned_licenses.png)

当导航到许可证计划时，将可看到已分配用户和组的详细列表，包括分配状态和路径（直接或从一个或多个组继承）。

![许可证计划的已分配许可证详细信息显示](./media/active-directory-licensing-what-is/assigned_licenses_detail.png)

删除许可证与分配许可证一样简单。如果用户直接获得分配或属于分配的组，则删除许可证的步骤为：选择许可证类型、选择“删除”、将用户或组添加到删除列表，然后确认操作。或者，可以打开许可类型、选择特定用户或组，然后点击命令栏上的“删除”。若要结束用户从组继承许可证，只需从组中删除用户。

###延长试用期

Office 365 门户中以自助服务的形式向客户提供试用版延期。客户管理员可导航到 [Office 门户](https://portal.office.com/#Billing)（访问权限取决于 Office 门户的权限），然后选择你的 Azure AD Premium 试用版。单击“延长试用期”链接，然后按照说明进行操作。你将需要输入信用卡号，但不会收费。

![在 Office 门户中延长许可证试用期](./media/active-directory-licensing-what-is/extend_license_trial.png)

客户还可以通过提交支持请求来请求试用版延期。客户管理员可导航到 Office 门户[支持页](http://aka.ms/extendAADtrial)（访问权限取决于 Office 门户支持页的权限）。在此页上选择“功能”下的“订阅和试用版”和“症状”下的“试用版问题”。最后，输入不同情况的信息

![使用支持请求延长许可证试用期](./media/active-directory-licensing-what-is/alternate_office_aad_trial_extension.png)

## 后续步骤

现在你可能已准备好配置并使用 Azure AD Premium 中的某些功能。

- [自助密码重置](/documentation/articles/active-directory-manage-passwords)
- [自助服务组管理](/documentation/articles/active-directory-accessmanagement-self-service-group-management)
- [Azure AD 连接运行状况](/documentation/articles/active-directory-aadconnect-health)
- [对应用程序的组分配](/documentation/articles/active-directory-manage-groups)
- [Azure Multi-Factor Authentication](/documentation/articles/multi-factor-authentication)

<!---HONumber=67-->