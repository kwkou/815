<properties
    pageTitle="Azure AD Connect 同步：启用 AD 回收站 | Azure"
    description="本主题提供有关使用 Azure AD Connect 的 AD 回收站功能的建议。"
    services="active-directory"
    keywords="AD 回收站, 意外删除, 源定位点"
    documentationcenter=""
    author="cychua"
    manager="femila"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="afec4207-74f7-4cdd-b13a-574af5223a90"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/03/2017"
    wacn.date="05/02/2017"
    ms.author="billmath"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="8e0c932c7bc4e260b42e955f89efe06e2a6a804e"
    ms.lasthandoff="04/22/2017" />

# <a name="azure-ad-connect-sync-enable-ad-recycle-bin"></a>Azure AD Connect 同步：启用 AD 回收站
建议为同步到 Azure AD 的本地 Active Directory 启用 AD 回收站功能。 

如果你意外删除了某个本地 AD 用户对象，然后该功能还原该对象，Azure AD 会还原相应的 Azure AD 用户对象。  有关 AD 回收站功能的信息，请参阅[有关还原已删除的 Active Directory 对象的方案概述](https://technet.microsoft.com/zh-cn/library/dd379542.aspx)一文。

## <a name="benefits-of-enabling-the-ad-recycle-bin"></a>启用 AD 回收站的好处
此功能通过以下方式帮助你还原 Azure AD 用户对象：

- 如果你意外删除了某个本地 AD 用户对象，相应的 Azure AD 用户对象将在下一同步周期被删除。 默认情况下，Azure AD 会将删除的 Azure AD 用户对象以软删除状态保留 30 天。

- 如果已启用本地 AD 回收站功能，无需更改已删除的本地 AD 用户对象的“源定位点”值，即可将它还原。 已恢复的本地 AD 用户对象同步到 Azure AD 后，Azure AD 将还原处于软删除状态的相应 Azure AD 用户对象。 有关“源定位点”属性的信息，请参阅 [Azure AD Connect：设计概念](/documentation/articles/active-directory-aadconnect-design-concepts#sourceanchor/)一文。

- 如果未启用本地 AD 回收站功能，可能需要创建一个 AD 用户对象来替换已删除的对象。 如果 Azure AD Connect 同步服务配置为对“源定位点”属性使用系统生成的 AD 属性（例如 ObjectGuid），新建的 AD 用户对象与已删除的 AD 用户对象的“源定位点”值将不相同。 新建的 AD 用户对象同步到 Azure AD 后，Azure AD 将创建新的 Azure AD 用户对象，而不是还原处于软删除状态的 Azure AD 用户对象。

> [AZURE.NOTE]
> 默认情况下，Azure AD 会将处于软删除状态中的已删除 Azure AD 用户对象保留 30 天，此期限过后，将永久删除这些对象。 但是，管理员可以提前删除此类对象。 永久删除这些对象后，即使已启用在本地 AD 回收站功能，也不再可以恢复它们。



## <a name="next-steps"></a>后续步骤
**概述主题**

- [Azure AD Connect 同步：理解和自定义同步](/documentation/articles/active-directory-aadconnectsync-whatis/)

- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

