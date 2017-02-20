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
    ms.date="12/15/2016"
    wacn.date="02/20/2017"
    ms.author="kgremban" />  


# 管理双重验证设置
本文回答有关如何更新双重验证或多重身份验证的设置的问题。如果登录到帐户时出现问题，请参考[使用双重验证时遇到问题](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)获取疑难解答帮助。

## 哪里可以找到设置页
具体取决于公司设置 Azure 多重身份验证的方式，可在其中几个位置更改设置，例如电话号码。

如果 IT 管理员发出特定的 URL 或步骤管理双重验证，请按照这些说明进行操作。否则，以下说明应适用于其他所有人。如果按照这些步骤操作，但未看到相同的选项，这意味着工作场所或学校自定义了它们自己的门户。向管理员寻求指向 Azure 多重身份验证门户的链接。

1. 登录到 [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn)

	![1](./media/multi-factor-authentication-end-user-manage/1.png)  

	输入账户和密码后，点击 “Sign in"。

2. 选择你需要的验证方式。

	- 默认是 “Authentication phone”,即电话验证，“Method” 部分你可以选择“短信验证”或“电话验证”。

		![2](./media/multi-factor-authentication-end-user-manage/2.png)  

	- 也可以选择 “Office phone”,即办公电话验证。

		![3](./media/multi-factor-authentication-end-user-manage/3.png)  

	- 还可以选择 “Mobile phone”,即在手机上安装 “Azure Authentication app" 来验证。

		![4](./media/multi-factor-authentication-end-user-manage/4.png)  

> [AZURE.NOTE] 如果用户使用的使用的是安装Android系统的手机，在安装 “Azure Authentication app" 后,
> 会提示你安装 “Google Play Services",这个目前在中国不支持。 使用“IOS”系统的手机没有这个问题。
> 

## 我想要更改我的电话号码

如果用户想要更改电话号码，可以在登录[经典管理门户](https://manage.windowsazure.cn)后，从 “Active Directory” 进入到 “Multi-factor authentication” 配置页面，在 “users" 子页面上，选中你的用户名，然后点击右侧的 “Manage user settings”，在弹出的页面上选择 “Require selected user to provide contact methods again”，然后点击 “Save” 保存。 用户下次登录后，可以选择新的验证方式和电话号码。

![5](./media/multi-factor-authentication-end-user-manage/5.png)  

![6](./media/multi-factor-authentication-end-user-manage/6.png)  

## 如何从旧设备清除 Microsoft Authenticator 并将其迁移到新设备？
从设备上卸载该应用或重置设备时，不会删除应用在后端的激活。有关详细信息，请参阅 [Microsoft Authenticator](/documentation/articles/microsoft-authenticator-app-how-to/)。

## 后续步骤
- 获取有关[使用双重验证时遇到问题](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)的故障排除提示和帮助
- 为任何不支持双重验证的应用设置[应用密码](/documentation/articles/multi-factor-authentication-end-user-app-passwords/)。

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: link update-->