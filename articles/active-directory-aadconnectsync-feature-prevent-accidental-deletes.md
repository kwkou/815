<properties
   pageTitle="Azure AD Connect 同步：防止意外删除 | Azure"
   description="本主题说明 Azure AD Connect 中的防止意外删除功能。"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="StevenPo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="03/07/2016"
   wacn.date="04/28/2016"/>

# Azure AD Connect 同步：防止意外删除
本主题说明 Azure AD Connect 中的防止意外删除功能。

安装 Azure AD Connect 时，会按默认启用防止意外删除功能，并将其配置为不允许超过 500 个删除项目的导出。此功能旨在防止配置及本地目录发生意外更改，因为这会影响大量的用户和其他对象。

会遇到这种情况的常见情景包括：

- 在取消选择整个 [OU](/documentation/articles/active-directory-aadconnectsync-configure-filtering/#organizational-unitbased-filtering) 或[域](/documentation/articles/active-directory-aadconnectsync-configure-filtering/#domain-based-filtering)的情况下更改为[筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/)。
- 已删除 OU 中的所有对象。
- 已重命名某个 OU，因此其中的所有对象被视为超出同步范围。

可以使用 PowerShell 的 `Enable-ADSyncExportDeletionThreshold` 进行更改的默认值是 500 个对象。应将此值配置为符合组织的大小。由于同步计划程序每隔 30 分钟运行一次，因此该值是 30 分钟内看到的删除数目。

启用此功能后，如果暂存了太多的要导出到 Azure AD 的删除项目，就不会继续导出，并且你会收到一封内容如下所示的电子邮件：

![有关防止意外删除的电子邮件](./media/active-directory-aadconnectsync-feature-prevent-accidental-deletes/email.png)

> 你好（技术联系人）。标识同步服务在（时间）检测到删除数目超过了为（组织名称）配置的删除阈值。在此次标识同步运行期间，总共已发送（数目）个对象进行删除。这达到或超过了配置的删除阈值，即（数目）个对象。在继续之前，我们需要你确认应该处理这些删除。有关此电子邮件中所列错误的详细信息，请参阅“防止意外删除”。

如果这是意外情况，请进行调查，并采取纠正措施。若要查看哪些对象即将被删除，请执行以下操作：

1. 从“开始”菜单启动“同步服务”。
2. 转到“连接器”。
3. 选择 **Azure Active Directory** 类型的连接器。
4. 在右侧的“操作”下，选择“搜索连接器空间”。
5. 在“范围”下的弹出框中选择“连接断开起始时间”，并选择过去的一个时间。单击“搜索”。此时将提供所有即将被删除的对象视图。单击每个项可以获取有关该对象的更多信息。你也可以单击“列设置”，以添加要在网格中显示的其他属性。

![搜索连接器空间](./media/active-directory-aadconnectsync-feature-prevent-accidental-deletes/searchcs.png)

如果想要查看所有删除项，请执行以下操作：

1. 若要暂时禁用此保护并允许删除这些项，请运行 PowerShell cmdlet：`Disable-ADSyncExportDeletionThreshold`。当系统要求提供凭据时，请提供 Azure AD 全局管理员帐户和密码。
![凭据](./media/active-directory-aadconnectsync-feature-prevent-accidental-deletes/credentials.png)
2. 如果 Azure Active Directory 连接器仍被选中，请选择“运行”操作，再选择“导出”。
3. 若要重新启用保护，请运行 PowerShell cmdlet：`Enable-ADSyncExportDeletionThreshold`

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0606_2016-->