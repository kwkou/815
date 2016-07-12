<properties
   pageTitle="故障排除：“Active Directory”项缺失或不可用 | Azure"
   description="Azure 经典管理门户中未显示 Active Directory 菜单项时怎么办。"
   services="active-directory"
   documentationCenter="na"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="12/04/2015"
   wacn.date="05/13/2016"/>

# 故障排除：“Active Directory”项缺失或不可用

关于使用 Azure Active Directory 功能和服务的很多说明都以“转到 Azure 经典管理门户并单击‘Active Directory’”开头。 但是，如果未出现 Active Directory 扩展或菜单项或者它被标记为“不可用”，你该怎么办？ 本主题旨在提供帮助。其中描述了 **Active Directory** 未出现或不可用的情况，并解释了如何继续执行操作。

## 缺少 Active Directory

通常，**Active Directory** 项出现在左侧导航菜单中。Azure Active Directory 过程中的说明假定你可以看见该项。

![屏幕截图：Azure 中的 Active Directory](./media/active-directory-troubleshooting/typical-view.png)

如果以下任一情况属实，Active Directory 项将出现在左侧导航菜单中。否则，该项不会出现。

* 当前用户已使用 Microsoft 帐户（以前称为 Windows Live ID）登录。

    或者

* Azure 租户具有目录，且当前帐户为目录管理员。

    或者

* Azure 租户至少有一个 Azure AD 访问控制 (ACS) 命名空间。有关详细信息，请参阅[访问控制命名空间](https://msdn.microsoft.com/library/azure/gg185908.aspx)。

    或者

* Azure 租户至少有一个 Azure Multi-Factor Authentication 提供程序。有关详细信息，请参阅[管理 Azure Multi-Factor Authentication 提供程序](/documentation/articles/multi-factor-authentication-get-started-cloud/)。

若要创建访问控制命名空间或 Multi-Factor Authentication 提供程序，请单击“+新建”>“应用程序服务”>“Active Directory”。

要获取对目录的管理权限，请让管理员为你的帐户分配管理员角色。有关详细信息，请参阅[分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。

## Active Directory 不可用

当你单击“+新建”>“应用程序服务”时，将显示“Active Directory”项。具体而言，当前用户可以使用任何 Active Directory 功能（如“目录”、“访问控制”或“Multi-Factor Auth 提供程序”）时，会显示 Active Directory 项。

但是，在加载页面时，该项将灰显或标记为“不可用”。这是一种暂时性的状态。只需等待几秒，该项便可供使用。如果延迟时间过长，刷新网页通常就会解决问题。

![屏幕截图：Active Directory 不可用](./media/active-directory-troubleshooting/not-available.png)

<!---HONumber=Mooncake_0418_2016-->