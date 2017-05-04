<properties
    pageTitle="Azure AD Connect 同步：更改 AD DS 帐户密码 | Azure"
    description="本主题文档介绍如何在更改 AD DS 帐户的密码以后更新 Azure AD Connect。"
    services="active-directory"
    keywords="AD DS 帐户, Active Directory 帐户, 密码"
    documentationcenter=""
    author="cychua"
    manager="femila"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="76b19162-8b16-4960-9e22-bd64e6675ecc"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/10/2017"
    wacn.date="05/02/2017"
    ms.author="billmath"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="2dcd02274c284a1bc5f5c897c5147fb8ba94d53b"
    ms.lasthandoff="04/22/2017" />

# <a name="changing-the-ad-ds-account-password"></a>更改 AD DS 帐户密码
AD DS 帐户是指 Azure AD Connect 用来与本地 Active Directory 通信的用户帐户。 如果更改 AD DS 帐户的密码，则必须使用新密码更新 Azure AD Connect 同步服务。 否则，同步服务将再也不能正确地通过本地 Active Directory 进行同步，你会遇到以下错误：

- 在 Synchronization Service Manager 中，任何通过本地 AD 进行的导入或导出操作都会失败，出现 **no-start-credentials** 错误。

- 在 Windows 事件查看器下，应用程序事件日志包含**事件 ID 为 6000** 且内容为“管理代理 "contoso.com" 无法运行，因为凭据无效”的错误。


## <a name="how-to-update-the-synchronization-service-with-new-password-for-ad-ds-account"></a>如何使用 AD DS 帐户的新密码更新同步服务
若要使用新密码更新同步服务，请执行以下操作：

1. 启动 Synchronization Service Manager（“开始”→“同步服务”）。
</br>![Sync Service Manager](./media/active-directory-aadconnectsync-service-manager-ui/startmenu.png)  

2. 转到“连接器”选项卡。

3. 选择“AD 连接器”，该连接器对应于其密码已更改的 AD DS 帐户。

4. 在“操作”下面，选择“属性”。

5. 在弹出对话框中，选择“连接到 Active Directory 林”：

6. 在“密码”文本框中输入 AD DS 帐户的新密码。

7. 单击“确定”保存新密码并关闭弹出对话框。

8. 在 Windows 服务控制管理器下重新启动 Azure AD Connect 同步服务。 这是为了确保从内存缓存中删除对旧密码的任何引用。

## <a name="next-steps"></a>后续步骤
**概述主题**

- [Azure AD Connect 同步：理解和自定义同步](/documentation/articles/active-directory-aadconnectsync-whatis/)

- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

