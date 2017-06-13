<properties
    pageTitle="Troubleshooting: ''Active Directory'' item is missing or not available | Azure"
    description="Azure 管理门户中未显示 Active Directory 菜单项时怎么办。"
    services="active-directory"
    documentationcenter="na"
    author="bryanla"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="3383020d-6397-43ea-b7aa-c6a9d6a1e3df"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="04/27/2017"
    wacn.date="06/12/2017"
    ms.author="bryanla"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="d4ce90f3fa7cd85d0bc0ed762310d60887f9e266"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="troubleshooting-active-directory-item-is-missing-or-not-available"></a>故障排除：“Active Directory”项缺失或不可用
关于使用 Azure Active Directory 功能和服务的很多说明都以“转到 Azure 管理门户并单击‘Active Directory’”开头。 但是，如果未出现 Active Directory 扩展或菜单项或者它被标记为“不可用”，该怎么办？ 本主题旨在提供帮助。 其中描述了 **Active Directory** 未出现或不可用的情况，并解释了如何继续执行操作。

## <a name="active-directory-is-missing"></a>缺少 Active Directory
通常， **Active Directory** 项出现在左侧导航菜单中。 Azure Active Directory 过程中的说明假定你可以看见该项。

![屏幕截图：Azure 中的 Active Directory](./media/active-directory-troubleshooting/typical-view.png)

如果以下任一情况属实，Active  Directory 项将出现在左侧导航菜单中。 否则，该项不会出现。

- 当前用户已使用 Microsoft 帐户（以前称为 Windows Live ID）登录。
  
    或
- Azure 租户具有目录，且当前帐户为目录管理员。

要获取对目录的管理权限，请让管理员为你的帐户分配管理员角色。 有关详细信息，请参阅[分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。

## <a name="active-directory-is-not-available"></a>Active Directory 不可用
当单击“+新建” > “应用服务”时，将显示“Active Directory”项。 具体而言，当前用户可以使用任何 Active Directory 功能（如“目录”或“访问控制”）时，会显示 Active Directory 项。

但是，在加载页面时，该项将灰显或标记为“不可用”。 这是一种暂时性的状态。 只需等待几秒，该项便可供使用。 如果延迟时间过长，刷新网页通常就会解决问题。

![屏幕截图：Active Directory 不可用](./media/active-directory-troubleshooting/not-available.png)

<!--Update_Description: wording update-->