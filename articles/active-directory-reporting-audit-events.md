<properties 
   pageTitle="Azure Active Directory 审核报告事件 - Azure 教程" 
   description="可从 Azure Active Directory 查看和下载的已审核事件" 
   services="active-directory" 
   documentationCenter="" 
   authors="kenhoff" 
   manager="mbaldwin" 
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity" 
   ms.date="04/13/2015"
   wacn.date="05/15/2015"
   ms.author="kenhoff"/>

# Azure Active Directory 审核报告事件

Azure Active Directory 审核报告可帮助客户识别其 Azure Active Directory 中发生的特权操作。特权操作包括提升更改（例如，创建角色或密码重置）、更改策略配置（例如密码策略）或更改目录配置（例如，对域联合身份验证设置的更改）。报告提供了事件名称的审核记录、操作执行者、更改影响的目标资源，以及日期和时间 (UTC)。客户可通过 [Azure 管理门户](https://manage.windowsazure.cn/)检索其 Azure Active Directory 的审核事件列表。  

## 每个审核事件随附的属性

| 属性	| 说明								|
| ------	| ------								|		
| 日期和时间	| 审核事件发生的日期和时间			|
| 参与者		| 执行操作的用户或服务主体		|
| 操作	| 执行的操作						|
| 目标	| 执行的操作所针对的用户或服务主体	|

## 审核报告事件列表


| 事件 				| 事件说明																				|
| ------------------------------	| -------																					|
| 添加用户				| 已将用户添加到目录。																		|
| 删除用户				| 已从目录中删除用户。																		|
| 设置许可证属性		| 已设置目录中用户的许可证属性。															|
| 重置用户密码			| 已重置目录中用户的密码。																|
| 更改用户密码			| 已更改目录中用户的密码。																|
| 更改用户许可证			| 已更改分配给目录中用户的许可证。															|
| 更新用户				| 已更新目录中的用户。																		|
| 将角色成员添加到角色		| 已将用户添加到目录角色。																		|
| 从角色中删除角色成员		| 已从目录角色中删除用户。																	|
| 设置公司联系信息	| 已设置公司级联系人首选项。这包括营销电子邮件地址，以及有关 Microsoft Online Services 的技术通知。			|
| 将合作伙伴添加到公司		| 已在目录中添加合作伙伴。																		|
| 从公司中删除合作伙伴		| 已从目录中删除合作伙伴。																		|
| 添加服务主体			| 已在目录中添加服务主体。																	|
| 删除服务主体		| 已从目录中删除服务主体。																|
| 添加服务主体凭据	| 已将凭据添加到服务主体。																	|
| 删除服务主体凭据	| 已从服务主体中删除凭据。																	|
| 将域添加到公司		| 已在目录中添加域。																		|
| 从公司中删除域		| 已从目录中删除域。																		|
| 更新域				| 已更新目录中的域。																		|
| 设置域身份验证		| 已更改公司的默认域设置																|
| 设置域中的联合身份验证设置	| 已更新域的联合身份验证设置。																	|
| 验证域				| 已验证目录中的域。																		|
| 验证电子邮件验证域		| 已使用电子邮件验证来验证目录中的域。															|
| 添加委托条目			| 已在目录中添加委托条目。																	|
| 设置委托条目			| 更新目录中的委托条目。																	|
| 删除委托条目		| 已从目录中删除委托条目。																|
| 设置公司的 DirSyncEnabled 标志	| 已设置用来为 Azure AD Sync 启用目录的属性。															|
| 设置密码策略			| 已设置用户密码的长度和字符约束。															|
| 设置公司信息		| 已更新公司级信息。有关详细信息，请参阅 [Get-MsolCompanyInformation](https://msdn.microsoft.com/zh-CN/library/azure/dn194126.aspx)。	|
| 设置强制更改用户密码	| 已设置强制用户在登录时更改其密码的属性。													|


### "更新用户"审核事件中包含的用户属性

"更新用户"审核事件包含有关更新了哪些用户属性的附加信息。对于每个属性，将包含以前的值和新值。

| 属性 				| 说明																			|
| ---------------------------------	| ---------																			|
| AccountEnabled			| 允许用户登录。																|
| AssignedLicense			| 分配给用户的所有许可证。														|
| AssignedPlan				| 向用户分配许可证后生成的服务计划。												|
| LicenseAssignmentDetail		| 有关分配给用户的许可证的详细信息。例如，如果涉及到基于组的许可，则这包括授予许可证的组。		|
| Mobile				| 用户的手机号码。																	|
| OtherMail				| 用户的备用电子邮件地址。																|
| OtherMobile				| 用户的备用手机号码。																|
| StrongAuthenticationMethod		| 用户为多重身份验证配置的验证方法列表，例如语音呼叫、短信或移动应用发送的验证码。	|
| StrongAuthenticationRequirement	| 是否为此用户强制实施、启用或禁用多重身份验证。										|
| StrongAuthenticationUserDetails	| 用户用于多重身份验证和密码重置验证的电话号码、备用电话号码和电子邮件地址。			|
| TelephoneNumber			| 用户的电话号码。																	|



<!--HONumber=53-->