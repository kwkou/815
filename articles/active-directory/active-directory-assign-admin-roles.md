<properties
	pageTitle="在 Azure Active Directory 中分配管理员角色 | Azure"
	description="介绍 Azure Active Directory 提供的管理员角色，以及如何分配这些角色。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="06/29/2016"
	wacn.date="08/22/2016"/>

# 在 Azure Active Directory (Azure AD) 中分配管理员角色

使用 Azure Active Directory (Azure AD) 时，可以指定不同的管理员来执行不同的功能。这些管理员可以按角色访问 Azure 门户或 Azure 经典门户中的各种功能：创建或编辑用户、将管理角色分配给他人、重置用户密码、管理用户许可证以及管理域等。分配为管理员角色的用户在你的组织所订阅的所有云服务中拥有相同的权限，不管该角色是通过 Office 365 门户、Azure 经典门户还是用于 Windows PowerShell 的 Azure AD 模块分配的。

提供以下管理员角色：

- **计费管理员**：进行采购、管理订阅、管理支持票证并监视服务运行状况。

- **全局管理员**：有权访问所有管理功能。注册 Azure 帐户的人员将成为全局管理员。只有全局管理员才能分配其他管理员角色。你的公司中可以有多个全局管理员。

	> [AZURE.NOTE] 在 Microsoft 图形 API、Azure AD 图形 API 和 Azure AD PowerShell 中，此角色标识为“公司管理员”。

- **密码管理员**：重置密码、管理服务请求并监视服务运行状况。密码管理员只能为用户和其他密码管理员重置密码。

	> [AZURE.NOTE] 在 Microsoft 图形 API、Azure AD 图形 API 和 Azure AD PowerShell 中，此角色标识为“支持管理员”。

- **服务管理员**：管理服务请求并监视服务运行状况。

	> [AZURE.NOTE] 若要为用户分配服务管理员角色，全局管理员必须先在服务（例如 Exchange Online）中将管理权限分配给用户，然后再在 Azure 经典门户中将服务管理员角色分配给用户。

- **用户管理员**：重置密码、监视服务运行状况，并管理用户帐户、用户组和服务请求。用户管理管理员权限存在一些限制。例如，他们不能删除全局管理员或创建其他管理员。另外，他们也不能为计费管理员、全局管理员和服务管理员重置密码。

- **安全读取者**：能够以只读方式访问 Identity Protection Center、Privileged Identity Management、监视 Office 365 运行状况和 Office 365 防护中心的一些安全功能。

- **安全管理员**：具备**安全读取者**角色的所有只读权限，再加上下列相同服务的一些附加管理权限：Identity Protection Center、Privileged Identity Management、监视 Office 365 运行状况和 Office 365 防护中心。

## 管理员权限

### 计费管理员

有权执行的操作 | 无权执行的操作
------------- | -------------
<p>查看公司信息和用户信息</p><p>管理 Office 支持票证为 Office 产品</p><p>执行计费和采购操作</p> | <p>重置用户密码</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p>

### 全局管理员

有权执行的操作 | 无权执行的操作
------------- | -------------
<p>查看公司信息和用户信息</p><p>管理 Office 支持票证为 Office 产品</p><p>执行计费和采购操作</p><p>重置用户密码</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p><p>启用或禁用多重身份验证</p> | 不适用

### 密码管理员

有权执行的操作 | 无权执行的操作
------------- | -------------
<p>查看公司信息和用户信息</p><p>管理 Office 支持票证</p><p>重置用户密码</p> | <p>为 Office 产品执行计费和采购操作</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p>

### 服务管理员

有权执行的操作 | 无权执行的操作
------------- | -------------
<p>查看公司信息和用户信息</p><p>管理 Office 支持票证</p> | <p>重置用户密码</p><p>为 Office 产品执行计费和采购操作</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p>

### 用户管理员

有权执行的操作 | 无权执行的操作
------------- | -------------
<p>查看公司信息和用户信息</p><p>管理 Office 支持票证</p><p>重置用户密码，但有限制。他（她）不能为计费管理员、全局管理员和服务管理员重置密码。</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证，但有限制。他（她）不能删除全局管理员或创建其他管理员。</p> | <p>为 Office 产品执行计费和采购操作</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p><p>启用或禁用多重身份验证</p>

### 安全读取者

In | 有权执行的操作
------------- | -------------
Identity Protection Center | 读取安全功能的所有安全报告和设置信息<ul><li>反垃圾邮件<li>加密<li>数据丢失防护<li>防病毒<li>高级威胁防护<li>防钓鱼<li>邮件流规则
Privileged Identity Management | <p>以只读方式访问 Azure AD PIM 中所显示的一切信息：Azure AD 角色分配的策略和报告、安全审阅，以及在未来还可通过读取来访问 Azure AD 角色分配以外的方案的策略数据和报告。<p>**无法**注册 Azure AD PIM 或对它进行任何更改。担任此角色的人员可以在 PIM 的门户中或通过 PowerShell，为其他角色（例如，全局管理员或特权角色管理员）的候选用户激活角色。
<p>监视 Office 365 服务运行状况</p><p>Office 365 防护中心</p> | <ul><li>读取和管理警报<li>读取安全策略<li>读取威胁情报、执行 Cloud App Discovery，以及在搜索和调查时执行隔离<li>读取所有报告

### 安全管理员

In | 有权执行的操作
------------- | -------------
Identity Protection Center | <ul><li>安全读取者角色的所有权限。<li>此外，还能够执行除了重置密码以外的所有 IPC 操作。
Privileged Identity Management | <ul><li>安全读取者角色的所有权限。<li>**无法**管理 Azure AD 角色成员资格或设置。
<p>监视 Office 365 服务运行状况</p><p>Office 365 防护 | <ul><li>安全读取者角色的所有权限。<li>可以配置高级威胁防护功能中的所有设置（恶意软件和病毒保护、恶意 URL 配置、URL 跟踪等）。

## 有关全局管理员角色的详细信息

全局管理员有权访问所有管理功能。默认情况下，系统会将注册 Azure 订阅的人员指派为目录的全局管理员角色。只有全局管理员才能分配其他管理员角色。

## 分配或删除管理员角色

1. 在 Azure 经典门户中，单击“Active Directory”，然后单击你所在组织的目录的名称。

2. 在“用户”页上，单击你想要编辑的用户的显示名称。
3. 在“组织角色”列表中，选择要分配给此用户的管理员角色，或者选择“用户”（如果要删除现有的管理员角色）。
4. 在“备用电子邮件地址”框中键入一个电子邮件地址。此电子邮件地址用于接收重要通知（包括有关密码自助重置的通知），因此，不管该用户是否能够访问 Azure，都必须能够访问其电子邮件帐户。

5. 选择“允许”或“阻止”以指定是否允许用户登录并访问服务。

6. 从“使用位置”下拉列表中指定位置。
7. 完成后，单击“保存”。

## 后续步骤

- 若要了解有关如何更改 Azure 订阅管理员的详细信息，请参阅[如何添加或更改 Azure 管理员角色](/documentation/articles/billing-add-change-azure-subscription-administrator/)

- 若要了解有关如何在 Microsoft Azure 中控制资源访问的详细信息，请参阅[了解 Azure 中的资源访问权限](/documentation/articles/active-directory-understanding-resource-access/)

- 有关 Azure Active Directory 如何与你的 Azure 订阅相关联的详细信息，请参阅 [Azure 订阅与 Azure Active Directory 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory/)

- [管理用户](/documentation/articles/active-directory-create-users/)

- [管理密码](/documentation/articles/active-directory-manage-passwords/)

- [管理组](/documentation/articles/active-directory-manage-groups/)

<!---HONumber=Mooncake_0808_2016-->