<properties
   pageTitle="教程：将 Salesforce 与 Azure Active Directory 集成 | Windows Azure"
   description="了解如何将 Salesforce 与 Azure Active Directory 配合使用，以启用单一登录和自动化设置等等！"
   services="active-directory"
   documentationCenter=""
   authors="liviodlc"
   manager="TerryLanfear"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/15/2015"
   wacn.date="08/29/2015"/>

#教程：如何将 Salesforce 与 Azure Active Directory 集成

本教程将演示如何将 Salesforce 环境连接到 Azure Active Directory。你将了解如何配置对 Salesforce 的单一登录、如何启用自动化用户设置，以及如何向用户分配对 Salesforce 访问权限。

##先决条件

1. 若要通过 [Azure 管理门户](https://manage.windowsazure.cn)访问 Azure Active Directory，首先必须拥有有效的 Azure 订阅。

2. 必须在 [Salesforce.com](https://www.salesforce.com/) 中拥有有效的租户。

> [AZURE.IMPORTANT]如果你正使用 Salesforce.com **试用**帐户，则将不能配置自动化用户设置。在购买前，试用帐户无法启用所需的 API 访问。
> 
> 可通过使用[免费的开发人员帐户](https://developer.salesforce.com/signup)避开此限制以完成本教程。

如果你使用的是 Salesforce 沙盒环境，请参阅 [Salesforce 沙盒集成教程](https://msdn.microsoft.com/zh-cn/library/azure/dn798671.aspx?f=255&MSPPError=-2147217396)。


##步骤 1：将 Salesforce 添加到目录

1. 在 [Azure 管理门户](https://manage.windowsazure.cn)的左侧导航窗格中，单击“Active Directory”。

	![从左侧导航窗格选择“Active Directory”。][0]

2. 从**“目录”**列表中选择想要将 Salesforce 添加到的目录。

3. 单击顶部菜单中的“应用程序”。

	![单击“应用程序”。][1]

4. 单击页面底部的“添加”。

	![单击“添加”以添加新应用程序。][2]

5. 在“想要执行的操作”对话框中，单击“从库添加应用程序”。

	![单击“从库添加应用程序”。][3]

6. 在**搜索框**中，键入 **Salesforce**。然后从结果中选择“Salesforce”，并单击“完成”以添加该应用程序。

	![添加 Salesforce。][4]

7. 你现在应看到 Salesforce 的“快速启动”页：

	![Azure AD 中 Salesforce 的“快速启动”页][5]

##步骤 2：启用单一登录

1. 在可以配置单一登录前，必须为 Salesforce 环境设置并部署自定义域。有关如何执行该操作的说明，请参阅[设置域名称](https://help.salesforce.com/HTViewHelpDoc?id=domain_name_setup.htm&language=en_US)。

2. 在 Azure AD 中，在 Salesforce 的“快速启动”页上单击“配置单一登录”按钮。

	![配置单一登录按钮][6]

3. 将打开一个对话框，其中显示询问“你希望用户如何登录到 Salesforce？” 选择“Azure AD 单一登录”，然后单击“下一步”。

	![选择“Azure AD 单一登录”][7]

	> [AZURE.NOTE]若要了解有关不同单一登录选项的详细信息，请[单击此处](https://msdn.microsoft.com/zh-cn/library/azure/dn308588.aspx)

4. 在“配置应用设置”页上，按照以下格式在“登录 URL”中键入 Salesforce 域 URL：
 - 企业帐户：`https://<domain>.my.salesforce.com`
 - 开发人员帐户：`https://<domain>-dev-ed.my.salesforce.com` 

	![键入你的登录 URL][8]

5. 在“在 Salesforce 上配置单一登录”页上，单击“下载证书”，然后将证书文件保存在本地计算机上。

	![下载证书][9]

6. 在浏览器中打开新选项卡并登录 Salesforce 管理员帐户。

7. 在“管理员”导航窗格下，单击“安全控制”以展开相关部分。然后单击“单一登录设置”。

	![单击“安全控制”下的“单一登录设置”][10]

8. 在“单一登录设置”页上，单击“编辑”按钮。

	![单击“编辑”按钮][11]

	> [AZURE.NOTE]如果无法为 Salesforce 帐户启用单一登录设置，则可能需要联系 Salesforce 支持部门以便启用该功能。

9. 选择“SAML 已启用”，然后单击“保存”。

	![选择“SAML 已启用”][12]

10. 若要配置 SAML 单一登录设置，请单击“新建”。

	![选择“SAML 已启用”][13]

11. 在“SAML 单一登录设置编辑”页上，进行以下配置：

	![应进行的配置的屏幕截图][14]

 - 在“名称”字段中，键入此配置的友好名称。提供的“名称”值将自动填充“API 名称”文本框。

 - 在 Azure AD 中，复制“颁发者 URL”值，然后将其粘贴到 Salesforce 中的“颁发者”字段。

 - 在“实体 ID”文本框中，按照以下模式键入 Salesforce 域名：
     - 企业帐户：`https://<domain>.my.salesforce.com`
     - 开发人员帐户：`https://<domain>-dev-ed.my.salesforce.com`

 - 单击“浏览”或“选择文件”以打开“选择要上载的文件”对话框，选择 Salesforce 证书，然后单击“打开”以上载证书。

 - 对于“SAML 标识类型”，选择“断言包含用户的 salesforce.com 用户名”。

 - 对于“SAML 标识位置”，选择“标识位于主题陈述的 NameIdentifier 元素中”

 - 在 Azure AD 中，复制“远程登录 URL”值，然后将其粘贴到 Salesforce 中的“标识提供者登录 URL”字段。

 - 对于“服务提供商发起的请求绑定”，选择“HTTP 重定向”。

 - 最后，单击“保存”以应用你的 SAML 单一登录设置。

12. 在 Salesforce 中的左侧导航窗格中，单击“域管理”以展开相关的部分，然后单击“我的域”。

	![单击“我的域”][15]

13. 向下滚动到“身份验证配置”部分，然后单击“编辑”按钮。

	![单击“编辑”按钮][16]

14. 在“身份验证服务”部分，选择 SAML SSO 配置的友好名称，然后单击“保存”。

	![选择 SSO 配置][17]

	> [AZURE.NOTE]如果选择了多个身份验证服务，则当用户尝试启动对 Salesforce 环境的单一登录时，系统将提示他们选择用于登录的身份验证服务。如果不希望发生这种情况，则应**将所有其他身份验证服务保持未选中状态**。

15. 在 Azure AD 中，选中单一登录配置确认复选框以启用上载到 Salesforce 的证书。然后单击“下一步”。

	![选中确认复选框][18]

16. 如果想要接收与此单一登录配置维护相关的错误和警告电子邮件通知，请在对话框的最后一页上键入电子邮件地址。

	![键入电子邮件地址。][19]

17. 单击“完成”关闭对话框。若要测试配置，请参阅下面名为[将用户分配到 Salesforce](#step-4-assign-users-to-salesforce) 的部分。

##步骤 3：启用自动化用户设置

1. 在 Azure AD 中 Salesforce 的“快速启动”页上，单击”配置用户设置“按钮。

	![单击“配置用户设置”按钮][20]

2. 在“配置用户设置”对话框中，键入你的 Salesforce 管理员用户名和密码。

	![键入管理员用户名或密码][21]

	> [AZURE.NOTE]如果正在配置生产环境，最佳做法是在 Salesforce 中专门为此步骤创建新的管理员帐户。在 Salesforce 中，此帐户必须具有向它分配的“系统管理员”配置文件。

3. 若要获取 Salesforce 安全令牌，请打开新选项卡并登录到同一 Salesforce 管理员帐户。在页面的右上角，单击你的名称，然后单击“我的设置”。

	![单击你的名称，然后单击“我的设置”][22]

4. 在左侧导航窗格中，单击“个人”以展开相关的部分，然后单击“重置我的安全令牌”。

	![单击你的名称，然后单击“我的设置”][23]

5. 在“重置我的安全令牌”页上，单击“重置安全令牌”按钮。

	![阅读警告。][24]

6. 检查与此管理员帐户关联的电子邮件收件箱。查找 Salesforce.com 发送的包含新安全令牌的电子邮件。

7. 复制该令牌，转到 Azure AD 窗口，并将其粘贴到“用户安全令牌”字段。然后单击“下一步”。

	![粘贴安全令牌][25]

8. 在确认页上，可以选择接收有关设置故障发生时间的电子邮件通知。单击“完成”关闭对话框。

	![键入接收通知的电子邮件地址][26]

##步骤 4：将用户分配到 Salesforce

1. 若要测试配置，首先在目录中创建新的测试帐户。

2. 在 Salesforce“快速启动”页上，单击“分配用户”按钮。

	![单击“分配用户”][27]

3. 选择测试用户，然后单击屏幕底部的“分配”按钮：

 - 如果尚未启用自动化用户设置，则系统将提示你确认：

		![Confirm the assignment.][28]

 - 如果已启用自动化用户设置，则系统将提示你定义用户应具有的 Salesforce 配置文件。几分钟后，新设置的用户应显示在 Salesforce 环境中。

		![Confirm the assignment.][29]

		> [AZURE.IMPORTANT] If you are provisioning to a Salesforce **developer** environment, you will have a very limited number of licenses available for each profile. Therefore, it's best to provision users to the **Chatter Free User** profile, which has 4,999 licenses available.

4. 若要测试单一登录设置，请打开“访问面板”（网址为 [https://myapps.microsoft.com](https://myapps.microsoft.com/)），然后登录到测试帐户，并单击“Salesforce”。

##另请参阅

- [SaaS 应用程序集成教程列表](/documentation/articles/active-directory-saas-tutorial-list)
- [Azure AD 中的应用程序访问](https://msdn.microsoft.com/zh-cn/library/azure/dn308590.aspx)
- [访问面板简介](https://msdn.microsoft.com/zh-cn/library/azure/dn308586.aspx)

[0]: ./media/active-directory-saas-salesforce-tutorial/azure-active-directory.png
[1]: ./media/active-directory-saas-salesforce-tutorial/applications-tab.png
[2]: ./media/active-directory-saas-salesforce-tutorial/add-app.png
[3]: ./media/active-directory-saas-salesforce-tutorial/add-app-gallery.png
[4]: ./media/active-directory-saas-salesforce-tutorial/add-salesforce.png
[5]: ./media/active-directory-saas-salesforce-tutorial/salesforce-added.png
[6]: ./media/active-directory-saas-salesforce-tutorial/config-sso.png
[7]: ./media/active-directory-saas-salesforce-tutorial/select-azure-ad-sso.png
[8]: ./media/active-directory-saas-salesforce-tutorial/config-app-settings.png
[9]: ./media/active-directory-saas-salesforce-tutorial/download-certificate.png
[10]: ./media/active-directory-saas-salesforce-tutorial/sf-admin-sso.png
[11]: ./media/active-directory-saas-salesforce-tutorial/sf-admin-sso-edit.png
[12]: ./media/active-directory-saas-salesforce-tutorial/sf-enable-saml.png
[13]: ./media/active-directory-saas-salesforce-tutorial/sf-admin-sso-new.png
[14]: ./media/active-directory-saas-salesforce-tutorial/sf-saml-config.png
[15]: ./media/active-directory-saas-salesforce-tutorial/sf-my-domain.png
[16]: ./media/active-directory-saas-salesforce-tutorial/sf-edit-auth-config.png
[17]: ./media/active-directory-saas-salesforce-tutorial/sf-auth-config.png
[18]: ./media/active-directory-saas-salesforce-tutorial/sso-confirm.png
[19]: ./media/active-directory-saas-salesforce-tutorial/sso-notification.png
[20]: ./media/active-directory-saas-salesforce-tutorial/config-prov.png
[21]: ./media/active-directory-saas-salesforce-tutorial/config-prov-dialog.png
[22]: ./media/active-directory-saas-salesforce-tutorial/sf-my-settings.png
[23]: ./media/active-directory-saas-salesforce-tutorial/sf-personal-reset.png
[24]: ./media/active-directory-saas-salesforce-tutorial/sf-reset-token.png
[25]: ./media/active-directory-saas-salesforce-tutorial/got-the-token.png
[26]: ./media/active-directory-saas-salesforce-tutorial/prov-confirm.png
[27]: ./media/active-directory-saas-salesforce-tutorial/assign-users.png
[28]: ./media/active-directory-saas-salesforce-tutorial/assign-confirm.png
[29]: ./media/active-directory-saas-salesforce-tutorial/assign-sf-profile.png

<!---HONumber=67-->