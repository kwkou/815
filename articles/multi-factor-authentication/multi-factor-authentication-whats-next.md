<properties
    pageTitle="配置 Azure MFA | Microsoft 文档"
    description="这个有关 Azure 多重身份验证的页面介绍了使用 MFA 可以执行的后续步骤。  其中包括报告、欺诈警报、一次性跳过、自定义语音消息、缓存，受信任的 IP 和应用密码。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />
<tags
    ms.assetid="75af734e-4b12-40de-aba4-b68d91064ae8"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/21/2017"
    ms.author="kgremban"
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="c258b00bc7641714a5d80e7a3f524e1b5cc36856"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="configure-azure-multi-factor-authentication-settings"></a>配置 Azure 多重身份验证设置
在启动并运行 Azure 多重身份验证后，可以参考本文进行管理。  本文涵盖了各种主题，可帮助你充分利用 Azure 多重身份验证。  并非所有版本的 Azure 多重身份验证都提供了这些功能。

| 功能 | 说明 | 
|:--- |:--- |
| [可选择验证方法](#selectable-verification-methods) |允许选择可供用户使用的身份验证方法。 |

## <a name="selectable-verification-methods"></a>可选择的验证方法
可以选择用户可以使用哪些验证方法。 下表提供了每种方法的简要概述。

当用户注册 MFA 帐户时，可以选择除已启用的选项以外的首选验证方法。 [设置我的多重身份验证帐户](/documentation/articles/multi-factor-authentication-end-user-first-time/)中介绍了注册过程指导

| 方法 | 说明 |
|:--- |:--- |
| 拨打电话 |拨打自动语音电话。 用户接听电话，并按电话键盘上的 # 进行身份验证。 此电话号码不会同步到本地 Active Directory。 |
| 向手机发送短信 |发送包含验证码的短信。 系统会提示用户使用验证码回复短信或在登录界面中输入验证码。 |
| 通过移动应用发送通知 |向你的手机或已注册设备发送推送通知。 用户将查看通知并选择**验证**来完成验证。 <br>Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。 |
| 通过移动应用发送验证码 |Microsoft Authenticator 应用每隔三十秒会生成一个新的 OATH 验证码。 用户将此验证码输入到登录界面中。<br>Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。 |

### <a name="how-to-enabledisable-authentication-methods"></a>如何启用/禁用身份验证方法
1. 登录 [Azure 经典管理门户](https://manage.windowsazure.cn/)。
2. 在左侧单击“Active Directory”。
3. 在 Active Directory 中，单击要对其启用或禁用身份验证方法的目录。
4. 在选择的目录上，单击“配置”。
5. 在“多重身份验证”部分中，单击“管理服务设置”。
6. 在“服务设置”页上的验证选项下，选择/取消选择要使用的选项。</br></br>
   ![验证选项](./media/multi-factor-authentication-whats-next/authmethods.png)
7. 单击“保存” 。
8. 单击“**关闭**”。

<!---Update_Description: wording update -->