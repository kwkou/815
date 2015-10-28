<properties
   pageTitle="Azure AD 中的自动 SaaS 应用用户设置 | Microsoft Azure"
   description="介绍可以如何使用 Azure AD 进行自动化设置、取消设置，并不断跨多个第三方 SaaS 应用程序更新用户帐户。"
   services="active-directory"
   documentationCenter=""
   authors="liviodlc"
   manager="TerryLanfear"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/14/2015"
   wacn.date="08/29/2015"/>

#Azure Active Directory SaaS 应用程序的自动化用户设置和取消设置

##什么是 SaaS 应用的自动化用户设置？

Azure Active Directory (Azure AD) 允许你自动执行创建、维护和删除云 (SaaS) 应用程序，如 Dropbox、Salesforce、ServiceNow 等中的用户标识。

**以下是此功能操作的一些示例：**

- 在正确的 SaaS 应用中为新人（加入团队时）自动创建新帐户。
- 人员不可避免离开团队时，将帐户自动从 SaaS 应用中删除。
- 确保你的 SaaS 应用中的标识基于目录中的更改保持最新。
- 设置非用户对象，例如组以及支持它们的 SaaS 应用。

**自动化用户设置还包括以下功能：**

- 匹配 Azure AD 和 SaaS 应用之间现有标识的能力。
- 用于帮助 Azure AD 适应组织当前使用的 SaaS 应用配置的自定义选项。
- 可选电子邮件警报，用于设置错误。
- 报表和活动日志，用于帮助进行监视和故障排除。

##为什么要使用自动化设置？

使用此功能的一些常见动机包括：

- 避免与手动设置过程相关的成本、低效和人为错误。
- 通过在用户离开组织时立即删除其在关键 SaaS 应用中的标识，从而保护你的组织。
- 轻松将大量用户导入特定的 SaaS 应用程序。
- 体验设置解决方案运行为 Azure AD 单一登录定义的相同应用访问策略的便利。

##常见问题

**Azure AD 将目录更改写入到 SaaS 应用的频率是多少？**

Azure AD 每五到十分钟进行一次更改检查。如果 SaaS 应用返回几个错误（例如无效的管理员凭据），则 Azure AD 将逐渐降低频率到每天一次，直到完成错误修复。

**设置我的用户需要多长时间？**

增量更改将立即发生，但如果你尝试设置目录的大部分，则时长取决于所拥有的用户和组数量。较小目录只需几分钟、中等规模目录可能需要数分钟，而非常大的目录可能需要几小时。

**可以如何跟踪当前设置作业的进度？**

可以查看目录中“报表”部分下的“帐户设置报表”。另一种方法是，访问正在设置的 SaaS 应用程序的仪表板选项卡，并查找页面底部附近的“集成状态”部分。

**我如何知道用户无法获取正确设置？**

在设置配置向导结束时，存在订阅电子邮件通知获取设置失败的选项。你可以查看“设置错误报表”以了解设置失败的用户以及原因。

**Azure AD 是否可以将更改从 SaaS 应用重新写回目录？**

对于多数 SaaS 应用而言，设置仅支持出站处理，这意味着用户可以从目录写入到应用程序，而无法从应用程序将更改写回目录。然而，对于 [Workday](https://msdn.microsoft.com/zh-cn/library/azure/dn762434.aspx) 而言，设置仅支持入站处理，这意味着用户可从 Workday 导入到目录，同样，目录中的更改执行操作无法写回到 Workday。

**如何向工程团队提交反馈？**

请通过 [Azure Active Directory 反馈论坛](http://feedback.azure.com/forums/169401-azure-active-directory)与我们联系。

##自动化设置是如何工作的？

Azure AD 通过连接到由每个应用程序供应商所提供的设置终结点设置 SaaS 应用的用户。这些终结点允许 Azure AD 以编程方式创建、更新和删除用户。下面是 Azure AD 自动化设置执行的不同步骤的简要概述。

1. 首次为应用程序启用设置时，将执行以下操作：
 - Azure AD 会尝试将 SaaS 应用中的任何现有用户与其在目录中相应的标识进行匹配。匹配用户时，它们将*不*自动启用单一登录。为了使用户具有访问该应用程序的权限，必须将其显式分配到 Azure AD 中的应用，可直接分配或通过组成员身份进行分配。
 - 如果已指定应分配到应用程序的用户，并且如果 Azure AD 无法找到这些用户的现有帐户，则 Azure AD 将为它们在应用程序中配置新帐户。
2. 如上所述完成初始同步操作后，Azure AD 将每隔 10 分钟检查一次以下更改：
 - 如果已将新用户分配到应用程序（直接或通过组成员身份），则将在 SaaS 应用中为其设置新帐户。
 - 如果已删除用户的访问权限，则其在 SaaS 应用中的帐户将标记为禁用（绝不会完全删除用户，这有助于在发生配置错误时避免数据丢失）。
 - 如果用户是最近分配到应用程序并且已在 SaaS 应用中拥有一个帐户，则该帐户将标记为启用，并且如果它们与目录对比已过期，则可能会更新某些用户的属性。
 - 如果已在目录中更改用户的信息（如电话号码、办公地点，等等），则 SaaS 应用程序中也将更新该信息。

有关特性如何在 Azure AD 和 SaaS 应用之间映射的更多详细信息，请参阅[自定义特性映射](https://msdn.microsoft.com/zh-cn/library/azure/dn872469.aspx)一文。

##支持自动化用户设置的应用列表

单击应用以查看如何为其配置自动化设置的教程：

- [Box](https://msdn.microsoft.com/zh-cn/library/azure/dn308589.aspx)
- [Citrix GoToMeeting](https://msdn.microsoft.com/zh-cn/library/azure/dn440168.aspx)
- [Concur](https://msdn.microsoft.com/zh-cn/library/azure/dn308592.aspx)
- [Docusign](https://msdn.microsoft.com/zh-cn/library/azure/dn510973.aspx)
- [Dropbox for Business](https://msdn.microsoft.com/zh-cn/library/azure/dn510978.aspx)
- [Google Apps](/documentation/articles/active-directory-saas-google-apps-tutorial)
- [Jive](https://msdn.microsoft.com/zh-cn/library/azure/dn510977.aspx)
- [Salesforce](/documentation/articles/active-directory-saas-salesforce-tutorial)
- [Salesforce Sandbox](https://msdn.microsoft.com/zh-cn/library/azure/dn798671.aspx)
- [ServiceNow](https://msdn.microsoft.com/zh-cn/library/azure/dn510971.aspx)
- [Workday](https://msdn.microsoft.com/zh-cn/library/azure/dn762434.aspx)（入站设置）

为了使应用程序支持自动化用户设置，它必须首先提供必要的、允许外部程序自动执行创建、维护和删除用户操作的终结点。因此，不是所有的 SaaS 应用都能兼容此功能。对于不支持此功能的应用，Azure AD 工程团队将能够构建连接到这些应用的设置连接器，并按当前和潜在客户需求设置优先级。

如需联系 Azure AD 工程团队以请求其他应用程序的设置支持，请通过 [Azure Active Directory 反馈论坛](http://feedback.azure.com/forums/169401-azure-active-directory)提交消息。

<!---HONumber=67-->