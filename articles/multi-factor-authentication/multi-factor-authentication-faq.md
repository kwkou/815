<properties
	pageTitle="Azure 多重身份验证常见问题"
	description="提供与 Azure 多重身份验证相关的常见问题与解答列表。Azure 多重身份验证是要求使用多种方式（而不仅仅是用户名和密码）来验证用户身份的一种方法。它为用户登录和事务提供了额外的安全层。"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="billmath"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	ms.author="billmath"
   	wacn.date="10/19/2016"/>

# Azure 多重身份验证常见问题


本“常见问题”文章解答有关 Azure 多重身份验证和使用多重身份验证服务的常见问题，包括计费模式和可用性相关问题。

## 常规

**问：如何获得 Azure 多重身份验证方面的帮助？**

- [搜索 Microsoft 支持知识库](https://www.microsoft.com/zh-cn/Search/result.aspx?form=mssupport&q=phonefactor&form=mssupport)

  在知识库中搜索常见技术问题的解决方法。

- [Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/home?forum=WindowsAzureAD)

  在 [Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)中搜索和浏览来自社区的技术问题与解答，或者提出自己的问题。

## 计费

**问：通过电话或短信对我的用户进行身份验证时，我的组织是否需要付费？**

所有费用都计入服务的每位用户或每次身份验证费用。对于通过 Azure 多重身份验证拨打的每个电话或者向用户发送的每条短信，组织不需要付费。手机主人可能会因为通过其手机服务运营商接听电话或发送短信而产生漫游相关费用或其他费用。

**问：如何对使用 Azure 多重身份验证的组织计费？**

Azure 多重身份验证可作为独立的服务购买，按照每位用户和身份验证次数来计费，也能与 Azure Active Directory Premium、企业移动性套件或企业云套件一同购买。独立服务根据消费量提供，针对组织的 Azure 承诺用量按月计费；或者通过 Microsoft 企业协议、Microsoft 开放式许可计划、Microsoft 云解决方案提供商 (CSP) 计划或 Direct 按用户收取年度授权费。

**问：按用户计费模式的收费依据是配置为使用多重身份验证的用户数目还是执行验证的用户数目？**

计费依据是配置为使用多重身份验证的用户数目。

## 可用性

**问：如果用户未收到电话答复或者无法使用电话，该怎么办？**

如果用户以前配置了备用电话，应在登录页出现提示时选择该电话，然后重试。如果用户未配置其他方法，则应与组织管理员联系，请管理员更新分配用作用户主要电话（手机或办公电话）的号码。


**问：如果用户由于不再能够访问某个帐户而联系管理员，管理员该如何处理？**

管理员可以要求用户再次完成注册过程来重置其帐户。详细了解如何[在云中使用 Azure 多重身份验证管理用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

**问：如果包含应用密码的用户手机遗失或遭窃，管理员该如何处理？**

管理员可以删除该用户的所有应用密码，防止未经授权的访问。用户购买替代设备后，即可重新创建密码。详细了解如何[在云中使用 Azure 多重身份验证管理用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

**问：如果用户无法登录非浏览器应用，该怎么办？**

- 配置为使用多重身份验证的用户需要通过一个应用密码来登录某些非浏览器应用。
- 用户需要清除（删除）登录信息，重新启动该应用，然后使用其用户名和应用密码登录。

获取有关创建应用密码的详细信息，以及其他[有关应用密码的帮助](/documentation/articles/multi-factor-authentication-end-user-app-passwords/)。


>[AZURE.NOTE] 适用于 Office 2013 客户端的新式验证
>
> Office 2013 客户端（包括 Outlook）支持新式验证协议。可以将 Office 2013 配置为支持多重身份验证。配置 Office 2013 之后，Office 2013 客户端就不需要应用密码。有关详细信息，请参阅 [Office 2013 modern authentication public preview announcement](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)（Office 2013 新式验证公共预览版发布声明）。

**问：如果用户未收到短信或者回复了双向短信但验证超时，该怎么办？**

Azure 多重身份验证服务通过短信聚合器发送短信。许多因素可能会影响短信传递和接收的可靠性，包括使用的聚合器、目标国家/地区、用户的手机运营商和信号强度。因此，在双向短信传递过程中，不保证短信和回复能够送达与收悉。建议尽可能使用单向短信，而不要使用双向短信。单向短信更加可靠，可以防止由于回复其他国家/地区发来的短信而给用户造成的全球短信费用。

在某些国家，例如美国和加拿大，短信验证比其他国家或地区也更加可靠。建议在使用 Azure 多重身份验证时难以可靠接收短信的用户改用移动应用或电话呼叫的方法。移动应用身份验证方法非常有利，因为用户可以通过蜂窝网络和 Wi-Fi 连接接收移动应用通知。此外，即使设备根本没有信号，也可以显示移动应用密码。Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。


## 错误

**问：如果用户使用移动应用通知进行身份验证时看到“身份验证请求不适用于已激活的帐户”错误消息，该怎么办？**

请遵循以下过程：

1. 转到 [Azure 门户预览配置文件](https://account.activedirectory.azure.cn/profile/)，然后使用组织帐户登录。
2. 如果需要，请单击“其他验证选项”，然后单击用于帐户验证的其他选项。
3. 单击“其他安全验证”。
4. 从移动应用中删除现有帐户。
5. 单击“配置”，然后按照说明重新配置移动应用。




**问：如果用户在尝试使用非浏览器应用程序登录时看到 0x800434D4L 错误消息，该怎么办？**

目前，用户只能在可以通过其浏览器访问的应用程序和服务中使用其他安全验证。安装在本地计算机上的非浏览器应用程序（也称为*富客户端应用程序*，例如 Windows PowerShell）无法使用需要其他安全验证的帐户。在此情况下，用户可能会看到应用程序生成 0x800434D4L 错误。

此问题的解决方法是使用不同的用户帐户进行管理员相关操作和非管理员操作。稍后，可以在管理员帐户与非管理员帐户之间链接邮箱，以便能够使用非管理员帐户登录到 Outlook。了解如何[使管理员能够打开和查看用户邮箱的内容](http://help.outlook.com/141/gg709759.aspx?sl=1)，获取有关此解决方法的更多详细信息。

<!---HONumber=Mooncake_1010_2016-->
