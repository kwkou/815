<properties
   pageTitle="教程：将 NetSuite 与 Azure Active Directory 集成 | Windows Azure"
   description="了解如何将 NetSuite 与 Azure Active Directory 配合使用，以启用单一登录和自动化设置等等！"
   services="active-directory"
   documentationCenter=""
   authors="liviodlc"
   manager="TerryLanfear"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/15/2015"
   wacn.date="08/29/2015"/>

#教程：如何将 NetSuite 与 Azure Active Directory 集成

本教程将演示如何将 NetSuite 环境连接到 Azure Active Directory (Azure AD)。你将了解如何配置对 NetSuite 的单一登录、如何启用自动化用户设置，以及如何分配用户以访问 NetSuite。

##先决条件

1. 若要通过 [Azure 管理门户](https://manage.windowsazure.cn)访问 Azure Active Directory，首先必须拥有有效的 Azure 订阅。

2. 你必须具有对 [NetSuite](http://www.netsuite.com/portal/home.shtml) 订阅的管理员访问权限。免费试用帐户可用于任一服务。

##步骤 1：将 NetSuite 添加到目录

1. 在 [Azure 管理门户](https://manage.windowsazure.cn)的左侧导航窗格中，单击“Active Directory”。

	![从左侧导航窗格选择“Active Directory”。][0]

2. 从“目录”列表中，选择想要将 NetSuite 添加到的目录。

3. 单击顶部菜单中的“应用程序”。

	![单击“应用程序”。][1]

4. 单击页面底部的“添加”。

	![单击“添加”以添加新应用程序。][2]

5. 在“想要执行的操作”对话框中，单击“从库添加应用程序”。

	![单击“从库添加应用程序”。][3]

6. 在**搜索框**中，键入 **NetSuite**。然后从结果中选择“NetSuite”，并单击“完成”以添加该应用程序。

	![添加 NetSuite。][4]

7. 你现在应看到 NetSuite 的“快速启动”页：

	![Azure AD 中 NetSuite 的“快速启动”页][5]

##步骤 2：启用单一登录

1. 在 Azure AD 中，在 NetSuite 的“快速启动”页上单击“配置单一登录”按钮。

	![配置单一登录按钮][6]

2. 将打开一个对话框，其中显示询问“你希望用户如何登录到 NetSuite?” 选择“Azure AD 单一登录”，然后单击“下一步”。

	![选择“Azure AD 单一登录”][7]

	> [AZURE.NOTE]若要了解有关不同单一登录选项的详细信息，请[单击此处](https://msdn.microsoft.com/zh-cn/library/azure/dn308588.aspx)

3. 在“配置应用设置”页上，按照以下格式之一在“登录 URL”字段中键入 NetSuite 租户 URL：
	- `https://<tenant-name>.netsuite.com`
	- `https://<tenant-name>.na1.netsuite.com`
	- `https://<tenant-name>.na2.netsuite.com`
	- `https://<tenant-name>.sandbox.netsuite.com`
	- `https://<tenant-name>.na1.sandbox.netsuite.com`
	- `https://<tenant-name>.na2.sandbox.netsuite.com`

	![键入租户 URL][8]

4. 在“在 NetSuite 上配置单一登录”页上，单击“下载元数据”，然后将证书文件保存在本地计算机上。

	![下载元数据。][9]

5. 在浏览器中打开新选项卡，并以管理员身份登录到 Netsuite 公司站点。

6. 在页面顶部的工具栏中，单击“设置”，然后单击“设置管理器”。

	![转到“设置管理器”][10]

7. 从“设置任务”列表中，选择“集成”。

	![转到“集成”][11]

8. 在“管理身份验证”部分中，单击“SAML 单一登录”**“SAML 单一登录”**。

	![转到“SAML 单一登录”][12]

9. 在“SAML 设置”页上，执行以下步骤：

	- 在 Azure Active Directory 中，复制“远程登录 URL”值，然后将其粘贴到 NetSuite 中的“标识提供者登录页”字段。

		![“SAML 设置”页。][13]

	- 在 NetSuite 中，选择“主要身份验证方法”。

	- 对于标记为“SAMLV2 标识提供者元数据”的字段，选择“上载 IDP 元数据文件”。然后单击“浏览”以上载在步骤 4 中下载的元数据文件。

		![上载元数据][16]

	- 单击“提交”。

9. 在 Azure AD 中，选中单一登录配置确认复选框以启用上载到 NetSuite 的证书。然后单击“下一步”。

	![选中确认复选框][14]

10. 如果想要接收与此单一登录配置维护相关的错误和警告电子邮件通知，请在对话框的最后一页上键入电子邮件地址。

	![键入电子邮件地址。][15]

11. 单击“完成”关闭对话框。接下来，单击页面顶部的“属性”。

	![单击“属性”。][17]

12. 单击“添加用户属性”。

	![单击“添加用户属性”。][18]

13. 在“属性名”字段中，键入 `account`。在“属性值”字段中，键入你的 NetSuite 帐户 ID。下面包括了有关如何查找帐户 ID 的说明：

	![添加 NetSuite 帐户 ID。][19]

	- 在 NetSuite 中，在顶部导航菜单中单击**“设置”**。
	- 然后在左侧导航菜单的“设置任务”部分下单击，选择“集成”部分，并单击“Web 服务首选项”。
	- 复制 NetSuite 帐户 ID 并将其粘贴到 Azure AD 中的“属性值”字段。

		![获取帐户 ID][20]

14. 在 Azure AD 中，单击“完成”以结束添加 SAML 属性。然后单击底部菜单上的“应用更改”。

	![保存所做更改。][21]

15. 必须首先在 NetSuite 中向用户分配相应的权限，然后用户才可对 NetSuite 执行单一登录。按照下面的说明来分配这些权限。

	- 在顶部导航菜单上，单击“设置”，然后单击“设置管理器”。

	![转到“设置管理器”][10]

	- 在左侧导航菜单上，选择“用户/角色”，然后单击“管理角色”。

	![转到“管理角色”][22]

	- 单击“新建角色”。

	- 键入新角色的**名称**，然后选中“仅单一登录”复选框。

	![为新角色命名。][23]

	- 单击“保存”。

	- 在顶部菜单中，单击“权限”。然后单击“设置”。

	![转到“权限”][24]

	- 选择“设置 SAM 单一登录”，然后单击“添加”。

	- 单击“保存”。

	- 在顶部导航菜单上，单击“设置”，然后单击“设置管理器”。

	![转到“设置管理器”][10]

	- 在左侧导航菜单上，选择“用户/角色”，然后单击“管理用户”。

	![转到“管理用户”][25]

	- 选择一个测试用户。然后单击“编辑”。

	![转到“管理用户”][26]

	- 在“角色”对话框中，选择已创建的角色，然后单击“添加”。

	![转到“管理用户”][27]

	- 单击“保存”。

19. 若要测试配置，请参阅下面名为[将用户分配到 NetSuite](#step-4-assign-users-to-netsuite) 的部分。

##步骤 3：启用自动化用户设置

1. 在 Azure Active Directory 中，在 NetSuite 的“快速启动”页上，单击“配置用户设置”。

	![配置用户设置][28]

2. 在打开的对话框中，键入 NetSuite 的管理员凭据，然后单击“下一步”。

	![键入 NetSuite 管理员凭据。][29]

3. 在确认页中，键入用于接收设置失败通知的电子邮件地址。

	![确认。][30]

4. 单击“完成”关闭对话框。

##步骤 4：将用户分配到 NetSuite

1. 若要测试配置，首先在目录中创建新的测试帐户。

2. 在 NetSuite“快速启动”页上，单击“分配用户”按钮。

	![单击“分配用户”][31]

3. 选择测试用户，然后单击屏幕底部的“分配”按钮：

 - 如果尚未启用自动化用户设置，则系统将提示你确认：

		![Confirm the assignment.][32]

 - 如果已启用自动化用户设置，则系统将提示你定义用户在 NetSuite 中应具有的角色类型。几分钟后，新设置的用户应显示在 NetSuite 环境中。

4. 若要测试单一登录设置，请打开“访问面板”（网址为 [https://myapps.microsoft.com](https://myapps.microsoft.com/)），然后登录到测试帐户，并单击“NetSuite”。

##另请参阅

- [SaaS 应用程序集成教程列表](/documentation/articles/active-directory-saas-tutorial-list)
- [Azure AD 中的应用程序访问](https://msdn.microsoft.com/zh-cn/library/azure/dn308590.aspx)
- [访问面板简介](https://msdn.microsoft.com/zh-cn/library/azure/dn308586.aspx)

[0]: ./media/active-directory-saas-netsuite-tutorial/azure-active-directory.png
[1]: ./media/active-directory-saas-netsuite-tutorial/applications-tab.png
[2]: ./media/active-directory-saas-netsuite-tutorial/add-app.png
[3]: ./media/active-directory-saas-netsuite-tutorial/add-app-gallery.png
[4]: ./media/active-directory-saas-netsuite-tutorial/add-netsuite.png
[5]: ./media/active-directory-saas-netsuite-tutorial/quick-start-netsuite.png
[6]: ./media/active-directory-saas-netsuite-tutorial/config-sso.png
[7]: ./media/active-directory-saas-netsuite-tutorial/sso-netsuite.png
[8]: ./media/active-directory-saas-netsuite-tutorial/sso-config-netsuite.png
[9]: ./media/active-directory-saas-netsuite-tutorial/config-sso-netsuite.png
[10]: ./media/active-directory-saas-netsuite-tutorial/ns-setup.png
[11]: ./media/active-directory-saas-netsuite-tutorial/ns-integration.png
[12]: ./media/active-directory-saas-netsuite-tutorial/ns-saml.png
[13]: ./media/active-directory-saas-netsuite-tutorial/ns-saml-setup.png
[14]: ./media/active-directory-saas-netsuite-tutorial/ns-sso-confirm.png
[15]: ./media/active-directory-saas-netsuite-tutorial/sso-email.png
[16]: ./media/active-directory-saas-netsuite-tutorial/ns-sso-setup.png
[17]: ./media/active-directory-saas-netsuite-tutorial/ns-attributes.png
[18]: ./media/active-directory-saas-netsuite-tutorial/ns-add-attribute.png
[19]: ./media/active-directory-saas-netsuite-tutorial/ns-add-account.png
[20]: ./media/active-directory-saas-netsuite-tutorial/ns-account-id.png
[21]: ./media/active-directory-saas-netsuite-tutorial/ns-save-saml.png
[22]: ./media/active-directory-saas-netsuite-tutorial/ns-manage-roles.png
[23]: ./media/active-directory-saas-netsuite-tutorial/ns-new-role.png
[24]: ./media/active-directory-saas-netsuite-tutorial/ns-sso.png
[25]: ./media/active-directory-saas-netsuite-tutorial/ns-manage-users.png
[26]: ./media/active-directory-saas-netsuite-tutorial/ns-edit-user.png
[27]: ./media/active-directory-saas-netsuite-tutorial/ns-add-role.png
[28]: ./media/active-directory-saas-netsuite-tutorial/netsuite-provisioning.png
[29]: ./media/active-directory-saas-netsuite-tutorial/ns-creds.png
[30]: ./media/active-directory-saas-netsuite-tutorial/ns-confirm.png
[31]: ./media/active-directory-saas-netsuite-tutorial/assign-users.png
[32]: ./media/active-directory-saas-netsuite-tutorial/assign-confirm.png

<!---HONumber=67-->