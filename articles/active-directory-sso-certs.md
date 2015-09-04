<properties
	pageTitle="如何在 Azure AD 中管理联合身份验证证书 |Windows Azure"
	description="了解如何自定义联合身份验证证书的到期日期以及如何续订即将过期的证书。"
	services="active-directory"
	documentationCenter=""
	authors="liviodlc"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/01/2015"
	wacn.date="08/29/2015"/>

#在 Azure Active Directory 中管理联合单一登录的证书

本文介绍了与 Azure Active Directory 为建立 SaaS 应用程序的联合单一登录 (SSO) 而创建的证书相关的常见问题。

本文仅与配置为使用 **Azure AD 单一登录**的应用相关，如以下示例所示：

![Azure AD 单一登录](./media/active-directory-sso-certs/fed-sso.PNG)

##如何自定义联合身份验证证书的到期日期

默认情况下，证书设置为两年后过期。可以通过执行以下步骤为证书选择不同的到期日期。所包含的屏幕截图使用 ServiceNow 作为示例，但这些步骤可应用于任何联合 SaaS 应用。

1. 在 Azure Active Directory 中，在应用程序的“快速启动”页上，单击“配置单一登录”。

	![打开 SSO 配置向导。](./media/active-directory-sso-certs/config-sso.png)

2. 选择“Azure AD 单一登录”，然后单击“下一步”。

3. 键入应用程序的“登录 URL”，然后选中“配置用于联合单一登录的证书”复选框。然后单击“下一步”。

	![Azure AD 单一登录](./media/active-directory-sso-certs/new-app-config-sso.PNG)

4. 在下一页上，选择“生成新证书”，然后选择希望证书有效的时间。然后单击“下一步”。

	![生成新证书](./media/active-directory-sso-certs/new-app-config-cert.PNG)

5. 接下来，单击“下载证书”。若要了解如何将证书上载到特定 SaaS 应用，请单击“查看配置说明”。

	![下载然后上载证书](./media/active-directory-sso-certs/new-app-config-app.PNG)

6. 在选中对话框底部的确认复选框并按“提交”后，才会启用证书。

##如何续订即将过期的证书

想情况下，下面显示的续订步骤不会对用户造成较长的故障时间。此部分使用的屏幕截图使用 Salesforce 作为示例，但这些步骤可应用于任何联合 SaaS 应用。

1. 在 Azure Active Directory 中，在应用程序的“快速启动”页上，单击“配置单一登录”。

	![打开 SSO 配置向导](./media/active-directory-sso-certs/renew-sso-button.PNG)

2. 在对话框的第一页上，应已选择了“Azure AD 单一登录”，因此单击“下一步”。

3. 在第二页上，选中“配置用于联合单一登录的证书”复选框。然后单击“下一步”。

	![Azure AD 单一登录](./media/active-directory-sso-certs/renew-config-sso.PNG)

4. 在下一页上，选择“生成新证书”，然后选择希望新证书有效的时间。然后单击“下一步”。

	![生成新证书](./media/active-directory-sso-certs/new-app-config-cert.PNG)

5. 单击“下载证书”。若要成功续订证书，必须执行以下两个步骤：

	- 将新证书上载到 SaaS 应用的单一登录配置屏幕。若要了解如何为特定 SaaS 应用执行此操作，请单击“查看配置说明”。

	- 在 Azure AD 中，选择对话框底部的确认复选框以启用新证书，然后单击“下一步”提交。

	> [AZURE.IMPORTANT]完成其中任何一步后，将禁用应用的单一登录，但将在完成第二步后再次启用。因此，为了最大限度减少故障时间，请准备在短时间间隔内完成这两个步骤。

	![下载然后上载证书](./media/active-directory-sso-certs/renew-config-app.PNG)

##另请参阅

[Azure AD 中的应用程序访问和单一登录](/documentation/articles/active-directory-appssoaccess-whatis)

<!---HONumber=67-->