
<properties
	pageTitle="使用特性创建高级规则| Windows Azure"
	description="有关管理安全组以及如何使用这些组来管理对资源的访问的高级操作说明。"
	services="active-directory"
	documentationCenter=""
	authors="femila"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory" 
	ms.date="07/15/2015" 
	wacn.date="08/29/2015"/>


# 使用特性创建高级规则
Azure 管理门户使你可以灵活地设置更高级的规则，以便为组启用动态成员资格。

**创建高级规则** 在 Azure 管理门户中，在组的“配置”选项卡上，选择“高级规则”单选按钮，然后在提供的文本框中键入你的高级规则。可使用以下信息创建高级规则。

## 构造高级规则的主体
可为组的动态成员资格创建的高级规则实质上是一种二元表达式，由三部分组成，将返回 true 或 false 这两种结果。这三部分为：

- 左边参数
- 二元运算符
- 右边常量 

完整的高级规则类似于：(leftParameter binaryOperator "RightConstant")，其中整个二元表达式需使用左括号和右括号，右侧常量需使用双引号，而左侧参数的语法为 user.property。高级规则可包含多个二元表达式，以 -and、-or 和 -not 逻辑运算符隔开表达式。以下是构造正确的高级规则示例：

- (user.department -eq "Sales") -or (user.department -eq "Marketing") 
- (user.department -eq "Sales") -and -not (user.jobTitle -contains "SDE") 

有关受支持参数和表达式规则运算符的完整列表，请参阅以下部分。

高级规则的主体的总长度不能超过 255 个字符。
> [AZURE.NOTE]字符串和正则表达式的操作区分大小写。还可执行 Null 检查，将 $null 用作常量（例如 user.department -eq $null）。应使用 ' 字符对包含引号 " 的字符串进行转义，如 user.department -eq "Sa`"les"。

##支持的表达式规则运算符
下表列出了所有支持的表达式规则运算符及其将在高级规则主体中使用的语法：

| 运算符 | 语法 |
|-----------------|----------------|
| 不等于 | -ne |
| 等于 | -eq |
| 开头不是 | -notStartsWith |
| 开头为 | -startsWith |
| 不包含 | -notContains |
| 包含 | -contains |
| 不匹配 | -notMatch |
| 匹配 | -match |


| 查询分析错误 | 错误用法 | 已更正的用法 |
|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 错误：特性不受支持。 | (user.invalidProperty -eq "Value") | (user.department -eq "value") 属性应匹配以上列表中的支持的属性之一。 |
| 错误：运算符在特性上不受支持。 | (user.accountEnabled -contains true) | (user.accountEnabled-eq true) 属性的类型是布尔类型。使用以上列表中的布尔类型支持的运算符（-eq 或-ne）。 |
| 错误：查询编译错误。 | (user.department -eq "Sales") -and (user.department -eq "Marketing")(user.userPrincipalName -match "*@domain.ext") | (user.department -eq "Sales") -and (user.department -eq "Marketing") 逻辑运算符应匹配以上列表中支持的属性之一。(user.userPrincipalName -match ".*@domain.ext")or(user.userPrincipalName -match "@domain.ext$") 正则表达式中的错误。 |
| 错误：二元表达式的格式不正确。 | (user.department –eq “Sales”) (user.department -eq "Sales")(user.department-eq"Sales") | (user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain") 查询具有多个错误。括号的位置不正确。 |
| 错误：设置动态成员资格期间发生未知错误。 | (user.accountEnabled -eq "True" AND user.userPrincipalName -contains "alias@domain") | (user.accountEnabled -eq true) -and (user.userPrincipalName -contains "alias@domain") 查询具有多个错误。括号的位置不正确。 |

##支持的参数
以下是可在高级规则中使用的所有用户属性：

**布尔类型的属性**

允许的运算符

* -eq


* -ne


| 属性 | 允许的值 | 用法 |
|----------------|-----------------|--------------------------------|
| accountEnabled | true false | user.accountEnabled -eq true) |
| dirSyncEnabled | true false null | (user.dirSyncEnabled -eq true) |

**字符串类型的属性**

允许的运算符

* -eq


* -ne


* -notStartsWith


* -StartsWith


* -contains


* -notContains


* -match


* -notMatch

| 属性 | 允许的值 | 用法 |
|----------------------------|-------------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| city | 任何字符串值或 $null。 | (user.city -eq "value") |
| country | 任何字符串值或 $null。 | (user.country -eq "value") |
| department | 任何字符串值或 $null。 | (user.department -eq "value") |
| displayName | 任何字符串值。 | (user.displayName -eq "value") |
| facsimileTelephoneNumber | 任何字符串值或 $null。 | (user.facsimileTelephoneNumber -eq "value") |
| givenName | 任何字符串值或 $null。 | (user.givenName -eq "value") |
| jobTitle | 任何字符串值或 $null。 | (user.jobTitle -eq "value") |
| mail | 任何字符串值或 $null。用户的 SMTP 地址。 | (user.mail -eq "value") |
| mailNickName | 任何字符串值。用户的邮件别名。 | (user.mailNickName -eq "value") |
| mobile | 任何字符串值或 $null。 | (user.mobile -eq "value") |
| objectId | 用户对象的 GUID | (user.objectId -eq "1111111-1111-1111-1111-111111111111") |
| passwordPolicies | None DisableStrongPassword DisablePasswordExpiration DisablePasswordExpiration, DisableStrongPassword | (user.passwordPolicies -eq "DisableStrongPassword") |
| physicalDeliveryOfficeName | 任何字符串值或 $null。 | (user.physicalDeliveryOfficeName -eq "value") |
| postalCode | 任何字符串值或 $null。 | (user.postalCode -eq "value") |
| preferredLanguage | ISO 639-1 代码 | (user.preferredLanguage -eq "en-US") |
| sipProxyAddress | 任何字符串值或 $null。 | (user.sipProxyAddress -eq "value") |
| state | 任何字符串值或 $null。 | (user.state -eq "value") |
| streetAddress | 任何字符串值或 $null。 | (user.streetAddress -eq "value") |
| surname | 任何字符串值或 $null。 | (user.surname -eq "value") |
| telephoneNumber | 任何字符串值或 $null。 | (user.telephoneNumber -eq "value") |
| usageLocation | 两个字母的国家/地区代码 | (user.usageLocation -eq "US") |
| userPrincipalName | 任何字符串值。 | (user.userPrincipalName -eq "alias@domain") |
| userType | 成员来宾 $null | (user.userType -eq "Member") |

**字符串集合类型的属性**

允许的运算符

* -contains


* -notContains

| 属性 | 允许的值 | 用法 |
|----------------|---------------------------------------|------------------------------------------------------|
| otherMails | 任何字符串值 | (user.otherMails -contains "alias@domain") |
| proxyAddresses | SMTP：alias@domain smtp：alias@domain | (user.proxyAddresses -contains "SMTP: alias@domain") |

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [组的动态成员资格疑难解答](/documentation/articles/active-directory-accessmanagement-troubleshooting)

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->