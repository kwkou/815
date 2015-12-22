<properties 
	pageTitle="Azure Multi-Factor Authentication 常见问题" 
	description="Azure Multi-Factor Authentication 是要求使用多种方式（而不仅仅是用户名和密码）对你的身份进行验证的一种方法。它为用户登录和事务提供了额外的安全层。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.date="08/24/2015" 
	wacn.date="12/17/2015"/>

# Azure 多重认证常见问题


本“常见问题”部分将回答 Azure 多重认证的相关问题。其中涵盖了有关使用服务的问题，包括计费模式和可用性。

## 常规

**问：如何获得 Azure 多重认证方面的帮助？**

[搜索 Microsoft 知识库 (KB)](http://search.microsoft.com/zh-cn/supportresults.aspx?form=mssupport&q=phonefactor)

- 在 Microsoft 知识库 (KB) 中搜索有关 Microsoft Azure 多重认证服务器 (Phone Factor) 支持的常见故障维修服务问题的技术解决方案。

[Microsoft Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/home?forum=WindowsAzureAD)

- 单击[此处](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)，在社区中搜索和浏览技术问题与答案，或提出自己的问题。

[密码重置](mailto:phonefactorsupport@microsoft.com)

- 如果旧版 Phonefactor 客户有任何关于重置密码的疑问，或如果需要获取密码重置方面的帮助，请使用下面的链接建立支持案例。

[Microsoft Azure 多重认证服务器 (Phone Factor) 客户支持](https://support.microsoft.com/oas/default.aspx?prid=14947)

- 使用此链接可联系 Microsoft 支持专业人员。我们将询问几个问题，以帮助确定有哪些支持选项。支持选项可能包括电子邮件、在线提交或电话支持。 

[有关 Microsoft Azure 多重认证服务器 (Phone Factor) 的一般咨询](http://azure.microsoft.com/services/multi-factor-authentication)

- 若要深入了解 Microsoft Azure Multi-Factor Authentication 服务器 (Phone Factor)，或者有关于如何购买产品及其他可用支持选项的疑问，请访问我们的网站，或者向 [pfsales@microsoft.com](mailto:pfsales@microsoft.com) 发送电子邮件。 



**问：Azure 多重认证服务器如何处理用户数据？**

如果你在本地使用多重认证服务器，用户的数据将存储在本地服务器中。云中不会持久存储任何用户数据。当用户执行双因素身份验证时，MFA 服务器会将数据发送到 Azure MFA 云服务，以执行身份验证。将这些身份验证请求发送到云服务时，会在请求和日志中发送以下字段，以便在客户的身份验证/使用情况报告中使用。某些字段是可选的，可以在多重认证服务器中启用或禁用。从 MFA 服务器到 MFA 云服务的通信使用基于出站端口 443 的 SSL/TLS。这些字段是：

- 唯一 ID - 用户名或内部 MFA 服务器 ID
- 名字和姓氏 - 可选
- 电子邮件地址 - 可选
- 电话号码 - 用于语音通话或短信身份验证
- 设备令牌 - 用于移动应用身份验证
- 身份验证模式 
- 身份验证结果 
- MFA 服务器名称 
- MFA 服务器 IP 
- 客户端 IP – 如果可用



除了上述字段，身份验证结果（成功/拒绝）和任何拒绝的原因还与身份验证数据一起存储，可通过身份验证/使用情况报告获取。




## 计费

**问：通过电话或短信对我的用户进行身份验证时，我的组织是否需要付费？**

所有费用都计入服务的每位用户或每次身份验证费用。组织不需支付使用 Azure 多重认证时每次拨打的电话或发送给用户的短信费用。电话主人可能会因为通过其电话运营商接听电话或发送短信而产生漫游相关的费用或其他费用。

**问：如何对使用 Azure 多重认证的组织计费？**

在 Azure 管理门户中创建 Multi-Factor Auth Provider 时，选择“按用户”或“按身份验证”计费/使用模式。这是基于消耗量并针对组织的 Azure 订阅计费的资源，就像虚拟机、网站等都是针对订阅计费。

**问：“按用户”计费模式的收费是以启用多重认证的用户数目还是执行身份验证的用户数目为基础？**

计费以启用多重认证的用户数目为基础。

## 可用性

**问：如果我没有接到回复来电，或忘了接电话，该怎么办？**

如果先前已配置备用电话，请在登录页出现提示时选择该电话，然后再试一次。如果没有配置其他方法，请联系管理员并请求他们更新分配给主要电话（移动或办公）的号码。

**问：我已删除用户的管理员角色，但忘了禁用多重认证，而现在它未显示在列表中，该如何删除这项功能？**

- 根据你使用的门户，在左窗格中单击“用户”或“用户和组”。
- 选中你要编辑的用户旁边的复选框，然后单击“编辑”或“编辑”图标。
- 单击“设置”，在“分配角色”下选择“是”，并将用户添加回到以前的管理员角色。
- 转到 Multi-Factor Authentication 页。该帐户现在应会显示在页面上的列表中。 
- 遵循上述步骤，禁用帐户的 Multi-Factor Authentication。 
- 此时，你便可以从管理员角色中删除该帐户。


**问：我是一名管理员，如果某个用户联系我，但我的帐户已被锁定，我该怎么办？**

强制用户再次完成注册过程，即可重置用户。为此，请参阅[在云使用 Azure 多重认证管理用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices)

**问：如果用户的设备丢失或被盗，而设备使用了应用密码，我该怎么做？**

你可以删除所有用户应用密码，以防止任何未经授权的访问。在更换设备后，用户可以重建密码。为此，请参阅[在云使用 Azure Multi-Factor Authentication 管理用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices)

**问：如果用户无法登录非浏览器应用，该怎么办？**

- 已启用多重认证的用户需要应用密码，才能登录一些非浏览器应用。
- 用户需要清除登录信息（删除登录信息），重新启动应用程序，然后使用其用户名和应用密码登录。 

有关创建应用密码的信息，请参阅[有关使用应用密码的帮助](/documentation/articles/multi-factor-authentication-end-user-app-passwords)


>[AZURE.NOTE]适用于 Office 2013 客户端的现代身份验证
>
> Office 2013 客户端（包括 Outlook）现在支持新的身份验证协议，并且可以启用对多重认证的支持。这意味着一旦启用后，就不需要对 Office 2013 客户端使用应用密码。有关详细信息，请参阅 [Office 2013 现代身份验证公共预览版发布声明](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。

**问：如果我未收到短信，或回复双向短信时验证超时，该怎么办？**

Azure 多重认证服务通过短信聚合器发送短信。许多因素可能会影响短信传递和接收的可靠性，包括使用的聚合器、目标国家/地区、移动电话运营商和信号强度。因此，在执行双向短信传递时，不保证能够传递短信和接收短信回复。建议尽量使用单向短信而不是双向短信，因为前者更加可靠，并且可以防止由于回复其他国家/地区发来的短信而给用户造成的全球短信费用。

在某些国家，例如美国和加拿大，短信验证比其他国家/地区也更加可靠。建议在使用 Azure 多重认证时难以可靠接收短信的用户改为选择移动应用程序或电话呼叫的方法。移动应用程序非常有利，因为可以通过蜂窝网络和 Wi-Fi 连接接收移动应用程序通知，并且，即使设备根本没有信号，也可以显示移动应用程序密码。Azure 验证器应用可用于 [Windows Phone](http://www.windowsphone.com/store/app/azure-authenticator/03a5b2bf-6066-418f-b569-e8aecbc06e50)、[Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) 和 [iOS](https://itunes.apple.com/us/app/azure-authenticator/id983156458)。

**问：我是否可以在 Azure MFA 服务器上使用硬件令牌？**

如果你正在使用 Azure MFA 服务器，可以导入第三方 OATH TOTP 令牌并将其用于 MFA。目前，我们支持导入 Gemalto 为其令牌生成的早期 PSKC 格式的第三方 OATH TOTP 令牌，并支持导入 CSV 格式的令牌。导入 CSV 格式的令牌时，CSV 文件必须包含序列号、Base32 格式的机密密钥和时间间隔（通常为 30 秒）。

因此，如果 ActiveIdentity 令牌是 TOTP OATH 令牌并且你可以将机密密钥提取到可导入 Azure MFA 服务器的 CSV 文件，则可以使用这些令牌。OATH 令牌可以配合 AD FS、RADIUS（如果客户端系统能够处理访问质询响应）和基于 IIS 窗体的身份验证使用。


## 错误

**问：如果我在使用移动应用程序通知进行身份验证时看到“身份验证请求不适用于已激活的帐户”错误，该怎么办？**

- 转到 [https://account.activedirectory.windowsazure.com/profile/](https://account.activedirectory.windowsazure.com/profile/)，然后使用组织帐户登录。
- 如果需要，请单击“其他验证选项”并选择用于完成帐户验证的不同选项。
- 单击“其他安全性验证”。
- 从移动应用程序中删除现有帐户。
- 单击“配置”，然后按照说明重新配置移动应用程序。




**当我尝试使用非浏览器应用程序登录时，显示 0x800434D4L 错误，该怎么办？**

目前，附加安全性验证只能用于可通过浏览器访问的应用程序/服务。安装在本地计算机上的非浏览器应用程序（也称为富客户端应用程序，例如 Windows Powershell）将无法使用附加安全性验证所需的帐户。在这种情况下，你可能会看到应用程序生成错误 0x800434D4L。

此问题的解决方法是，管理员相关操作与非管理员操作使用单独的用户帐户。稍后你可以链接管理员帐户和非管理员帐户的邮箱，以便使用非管理员帐户登录到 Outlook。有关此解决方法的更多详细信息，请参阅 [使管理员能够打开和查看用户邮箱内容](http://help.outlook.com/141/gg709759(d=loband).aspx?sl=1)。

<!---HONumber=Mooncake_1207_2015-->