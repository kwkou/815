<properties
	pageTitle="在 Azure 多重身份验证中使用移动应用作为联系方法 | Azure"
	description="本文介绍了如何使用移动应用作为 Azure 多重身份验证的主要联系方法。"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="billmath"
	manager="stevenp"
	editor="curtland"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/30/2016"
	ms.author="billmath"
   	wacn.date="10/19/2016"/>  


# 在 Azure 多重身份验证中使用移动应用作为联系方法

如果要使用 Microsoft Authenticator 应用作为主要联系方法，可以参考本文。本文介绍如何设置 Azure 多重身份验证，以使用移动应用作为主要联系方法。

Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

## 使用 Microsoft Authenticator 作为联系方法


1. 在“其他安全性验证”屏幕上，从下拉列表中选择“移动应用”。
2. 选择“通知”或“一次性密码”，然后选择“设置”。

	![“其他安全性验证”屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/mobileapp.png)  


3. 在已安装 Microsoft Authenticator 应用的手机上，打开应用并选择 **+** 以添加帐户。
4. 指定要添加工作帐户或学校帐户。这将打开 QR 码扫描仪。如果相机未正常工作，可以选择手动输入公司信息。有关详细信息，请参阅[手动添加帐户](#add-an-account-manually)。

	![用于选择帐户的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan.png)  


	![用于选择是要扫描 QR 码还是要手动输入帐户的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan4.png)  


5. 扫描与用于配置移动应用的屏幕一起显示的 QR 码图片。选择“完成”关闭 QR 码屏幕。

	![QR 码屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan2.png)  


	如果无法获取要扫描的 QR 码，请手动输入信息。

	![用于手动输入信息的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/barcode.png)  


6. 在手机上完成激活后，选择“与我联系”。此步骤会将通知或验证码发送到手机。选择“验证”。

	![在其中选择“验证”以登录的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/verify.png)  


7. 如果公司需要 PIN 才能批准登录验证，请输入它。

	![用于输入 PIN 的框](./media/multi-factor-authentication-end-user-first-time-mobile-app/scan3.png)  


8. 完成 PIN 条目后，选择“关闭”。此时，验证应已成功。
9. 建议输入手机号码，以免无法访问移动应用。通过下拉列表指定国家/地区，然后在国家/地区名称旁边的框中输入手机号码。选择“下一步”。
10. 此时，已设置联系方法。现在可以为非浏览器应用（例如，Outlook 2010 或更低版本）设置应用密码。如果不使用这些应用，请选择“完成”。否则，继续执行下一步。

	![用于创建应用密码的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/step4.png)  


11. 如果正在使用非浏览器应用，请复制提供的应用密码，然后将该密码粘贴到这些应用。有关在单个应用（例如 Outlook 和 Lync）上执行的步骤，请参阅“如何将电子邮件中的密码更改为应用密码”和“如何将应用程序中的密码更改为应用密码”。
12. 选择“完成”。


## 手动添加帐户
如果要手动添加帐户，请执行以下操作：

1. 选择“手动输入帐户”按钮。

	![用于输入代码和 URL 的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/addaccount.png)  


	![用于输入代码和 URL 的屏幕](./media/multi-factor-authentication-end-user-first-time-mobile-app/addaccount2.png)  


2. 如果帐户已启用 Azure 多重身份验证，请输入显示条形码的同一页面上所提供的代码和 URL。此信息将会填入移动应用上的“代码”和“URL”框中。

	![设置](./media/multi-factor-authentication-end-user-first-time-mobile-app/barcode2.png)  


3. 激活完成后，选择“与我联系”。此步骤会将通知或验证码发送到手机。选择“验证”。

<!---HONumber=Mooncake_1010_2016-->
