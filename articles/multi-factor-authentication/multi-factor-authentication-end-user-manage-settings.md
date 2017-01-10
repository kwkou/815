<properties
    pageTitle="管理双重验证设置 | Azure"
    description="管理如何使用 Azure 多重身份验证，包括更改联系信息或配置设备。"
    services="multi-factor-authentication"
    keywords="多重身份验证客户端, 身份验证问题, 相关性 ID"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />
<tags
    ms.assetid="d3372d9a-9ad1-4609-bdcf-2c4ca9679a3b"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/10/2017"
    ms.author="kgremban" />  


# 管理双重验证设置
本文回答有关如何更新双重验证或多重身份验证的设置的问题。如果登录到帐户时出现问题，请参考[使用双重验证时遇到问题](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)获取疑难解答帮助。

## 哪里可以找到设置页
具体取决于公司设置 Azure 多重身份验证的方式，可在其中几个位置更改设置，例如电话号码。

如果 IT 管理员发出特定的 URL 或步骤管理双重验证，请按照这些说明进行操作。否则，以下说明应适用于其他所有人。如果按照这些步骤操作，但未看到相同的选项，这意味着工作场所或学校自定义了它们自己的门户。向管理员寻求指向 Azure 多重身份验证门户的链接。

1. 登录到 [https://myapps.microsoft.com](https://myapps.microsoft.com)
2. 在顶部，选择“配置文件”。
3. 选择“其他安全性验证”。

    ![Myapps](./media/multi-factor-authentication-end-user-manage/myapps1.png)  

4. 其他安全性验证页将与设置一同加载。

    ![验证](./media/multi-factor-authentication-end-user-manage/proofup.png)  


## 我想要更改我的电话号码，或添加次要号码
必须配置辅助身份验证电话号码。由于你的主要电话号码和移动应用可能在同一部手机上，因此，当你的手机丢失或被盗时，只能通过辅助电话号码访问你的帐户。

> [AZURE.NOTE]
如果无权访问主电话号码，并且在登入帐户时需要帮助，请参阅[使用双重验证时遇到问题](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)中的帮助主题。
>
>

**更改主电话号码：**

1. 在“其他安全性验证”页中，选择带有当前电话号码的文本框，然后将其编辑为新的电话号码。
2. 选择“保存”。
3. 如果这是用于首选验证选项的电话号码，在保存之前，必须验证新的电话号码。

**添加次要电话号码：**

1. 在“其他安全性验证”页中，勾选“备用身份验证电话”旁的复选框。
2. 在文本框中输入次要电话号码。
3. 选择“保存”，所做的更改已完成。

## 如何从旧设备清除 Microsoft Authenticator 并将其迁移到新设备？
从设备上卸载该应用或重置设备时，不会删除应用在后端的激活。有关详细信息，请参阅 [Microsoft Authenticator](/documentation/articles/multi-factor-authentication-microsoft-authenticator/)。

## 后续步骤
- 获取有关[使用双重验证时遇到问题](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)的故障排除提示和帮助
- 为任何不支持双重验证的应用设置[应用密码](/documentation/articles/multi-factor-authentication-end-user-app-passwords/)。

<!---HONumber=Mooncake_0103_2017-->
