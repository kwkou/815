<properties 
	pageTitle="Azure AD Connect Health 操作。" 
	description="这是“Azure AD Connect Health”页，其中描述了部署 Azure AD Connect Health 后能够执行的其他操作。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/14/2015"
	wacn.date="11/02/2015"/>

# Azure AD Connect Health 操作

以下主题描述可以通过 Azure AD Connect Health 执行的各种操作。

## 启用电子邮件通知
你可以对 Azure AD Connect Health Service 进行配置，以便在生成的警报指示联合身份验证服务基础结构运行不正常时发送电子邮件通知。生成警报时，以及警报被标记为已解决时，将会发生这种情况。请按以下说明配置电子邮件通知。请注意，默认情况下禁用电子邮件通知。


### 启用 Azure AD Connect Health 电子邮件通知的步骤

1. 打开需要接收电子邮件通知的场的“警报”边栏选项卡。
2. 单击操作栏中的“通知设置”按钮。
3. 将“电子邮件通知”开关设置为“打开”。
4. 选择相应的复选框，将所有全局租户管理员配置为能够接收电子邮件通知。
5. 如果你希望通过其他电子邮件地址接收电子邮件通知，则可在“其他电子邮件收件人”框中进行指定。若要从该列表中删除电子邮件地址，可右键单击相应的条目，然后选择“删除”。
6. 若要完成所做的更改，请单击“保存”。只有在选择“保存”后，所有更改才会生效。






## 从 Azure AD Connect Health Service 中删除服务器

在某些情况下，你可能想要从被监视的服务器中删除某个服务器。请按以下说明进行操作，从 Azure AD Connect Health Service 中删除服务器。

删除服务器时，请注意以下事项：

- 此操作会导致再也无法从该服务器收集任何数据。将从监视服务中删除此服务器。执行此操作之后，你将无法查看该服务器的新警报、监视数据或使用情况分析数据。
- 此操作不会从服务器中卸载或删除 Health 代理。如果你在执行此步骤之前未卸载 Health 代理，则可能会在与 Health 代理相关的服务器上看到错误事件。
- 此操作不会删除已从该服务器上收集的数据。那些数据将按 Microsoft Azure 数据保留策略删除。 
- 执行此操作后，如果你希望再次监视同一服务器，则需卸载该服务器上的 Health 代理，然后重新安装。 


	### 从 Azure AD Connect Health Service 中删除服务器的步骤
<ol>
1. 通过选择要删除的服务器名称，从“服务器列表”边栏选项卡中打开“服务器”边栏选项卡。 
2. 在“服务器”边栏选项卡中，单击操作栏中的“删除”按钮。
3. 在确认框中键入服务器名称，以便确认删除服务器的操作。
4. 单击“删除”按钮。







## 从 Azure AD Connect Health Service 中删除服务实例

在某些情况下，你可能想要删除服务实例。请按以下说明进行操作，从 Azure AD Connect Health Service 中删除服务实例。

删除服务实例时，请注意以下事项：

- 此操作将从监视服务中删除当前的服务实例。 
- 此操作不会从任何服务器中卸载或删除已作为此服务器实例的一部分进行监视的 Health 代理。如果你在执行此步骤之前未卸载 Health 代理，则可能会在与 Health 代理相关的服务器上看到错误事件。 
- 此服务实例的所有数据将按 Microsoft Azure 数据保留策略删除。 
- 执行此操作后，如果你想要开始监视此服务，请卸载需要进行监视的所有服务器上的 Health 代理，然后重新安装。执行此操作后，如果你希望再次监视同一服务器，则需卸载该服务器上的 Health 代理，然后重新安装。







	### 从 Azure AD Connect Health Service 中删除服务实例的步骤
<ol>
1. 通过选择要删除的服务标识符（场名称），从“服务列表”边栏选项卡中打开“服务”边栏选项卡。 
2. 在“服务器”边栏选项卡中，单击操作栏中的“删除”按钮。
3. 在确认框中键入服务名称（例如 sts.contoso.com）进行确认。 
4. 单击“删除”按钮。


若要通过使用情况分析功能收集数据并进行分析，必须向 Azure AD Connect Health 代理提供 AD FS 审核日志中的信息。默认情况下未启用这些日志。这仅适用于 AD FS 联合服务器。不需在 AD FS 代理服务器或 Web 应用程序代理服务器上启用审核。请通过以下步骤启用 AD FS 审核并查找 AD FS 审核日志。

#### 启用 AD FS 2.0 审核的步骤

1. 单击“开始”，指向“程序”，然后指向“管理工具”，再单击“本地安全策略”。
2. 导航到“安全设置\\本地策略\\用户权限管理”文件夹，然后双击“生成安全审核”。
3. 在“本地安全设置”选项卡上，验证是否列出了 AD FS 2.0 服务帐户。如果该帐户不存在，请单击“添加用户或组”并将其添加到列表中，然后单击“确定”。
4. 使用提升的权限打开命令提示符，然后运行以下命令以启用审核。<code>auditpol.exe /set /subcategory:"生成的应用程序" /failure:enable /success:enable</code>
5. 关闭本地安全策略，然后打开“管理”管理单元。若要打开“管理”管理单元，请单击“开始”，指向“程序”，然后指向“管理工具”，再单击“AD FS 2.0 管理”。
6. 在“操作”窗格中，单击“编辑联合身份验证服务属性”。
7. 在“联合身份验证服务属性”对话框中，单击“事件”选项卡。
8. 选择“成功审核”和“失败审核”复选框。
9. 单击**“确定”**。

#### 在 Windows Server 2012 R2 上启用 AD FS 审核的步骤

1. 通过在“开始”屏幕上打开“服务器管理器”或在桌面的任务栏中打开“服务器管理器”的方式打开“本地安全策略”，然后单击“工具/本地安全策略”。
2. 导航到“安全设置\\本地策略\\用户权限分配”文件夹，然后双击“生成安全审核”。
3. 在“本地安全设置”选项卡上，验证是否列出了 AD FS 服务帐户。如果该帐户不存在，请单击“添加用户或组”并将其添加到列表中，然后单击“确定”。
4. 使用提升的权限打开命令提示符，然后运行以下命令以启用审核：<code>auditpol.exe /set /subcategory:"生成的应用程序" /failure:enable /success:enable。</code>
5. 关闭“本地安全策略”，然后打开“AD FS 管理”管理单元（在“服务器管理器”中，单击“工具”，然后选择“AD FS 管理”）。
6. 在“操作”窗格中，单击“编辑联合身份验证服务属性”。
7. 在“联合身份验证服务属性”对话框中，单击“事件”选项卡。
8. 选择“成功审核”和“失败审核”复选框，然后单击“确定”。






#### 查找 AD FS 审核日志的步骤


1. 打开“事件查看器”。
2. 转到“Windows 日志”，然后选择“安全”。
3. 在右侧单击“筛选当前日志”。
4. 在“事件源”下选择“AD FS 审核”。

![AD FS 审核日志](./media/active-directory-aadconnect-health-requirements/adfsaudit.png)

> [AZURE.WARNING]如果你的组策略正在禁用 AD FS 审核，则 Azure AD Connect Health 代理将不能收集信息。请确保没有可能正在禁用审核的组策略。


## 相关链接

* [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health)
* [适用于 AD FS 的 Azure AD Connect Health 代理安装](/documentation/articles/active-directory-aadconnect-health-agent-install-adfs)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs)
* [Azure AD Connect Health 常见问题](/documentation/articles/active-directory-aadconnect-health-faq)

<!---HONumber=76-->