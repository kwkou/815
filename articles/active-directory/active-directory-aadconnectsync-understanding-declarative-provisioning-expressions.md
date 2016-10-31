<properties
	pageTitle="Azure AD Connect 同步：了解声明性预配表达式 | Azure"
	description="说明声明性设置表达式"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	ms.author="markusvi;andkjell"
	wacn.date="10/11/2016"/>


# Azure AD Connect 同步：了解声明性预配表达式
Azure AD Connect 同步基于 Forefront Identity Manager 2010 中最先引入的声明式预配。使用该功能可以实现完整的标识集成业务逻辑，而无需编写已编译的代码。

声明性设置的一个重要组成部分是属性流中使用的表达式语言。所用的语言是 Microsoft® Visual Basic® for Applications (VBA) 的子集。Microsoft Office 中使用了这种语言，具有 VBScript 经验的用户都认识该语言。声明性预配表达式语言只使用函数，不属于结构化语言。它不提供任何方法或语句。函数嵌套在表达式程序流中。

有关详细信息，请参阅 [Welcome to the Visual Basic for Applications language reference for Office 2013](https://msdn.microsoft.com/zh-cn/library/gg264383.aspx)（欢迎使用适用于 Office 2013 的 Visual Basic 应用程序语言参考）。

属性属于强类型。函数只接受正确类型的属性。它还区分大小写。函数名称和属性名称都必须具有正确的大小写，否则会引发错误。

## 语言定义和标识符

- 函数名称后跟加括号的参数：FunctionName(argument 1, argument N)。
- 属性采用方括号标识，如 [attributeName]。
- 参数通过百分比符号标识：%ParameterName%
- 字符串常量放在引号中：例如 "Contoso"（注意：必须使用直引号 ""，而不能使用弯引号""）
- 数字值表示不带引号，并且应为十进制。十六进制值带有前缀 &H。例如，98052, &HFF
- 表示布尔值的常量： True、 False。
- 内置常量和文本仅使用其名称表示：NULL、CRLF、IgnoreThisFlow

### 函数
声明性预配使用许多函数来实现转换属性值的可能性。这些函数可以嵌套，因此，一个函数的结果将传递到另一个函数。

`Function1(Function2(Function3()))`

有关函数的完整列表，请参阅 [function reference](/documentation/articles/active-directory-aadconnectsync-functions-reference/)（函数参考）。

### Parameters
通过连接器或由管理员使用 PowerShell 定义参数。参数通常包含因系统不同而各异的值，例如用户所在域的名称。这些参数可在属性流中使用。

Active Directory 连接器为入站同步规则提供以下参数：

| 参数名称 | 注释 |
| --- | --- |
| Domain.Netbios | 当前正在导入的域的 Netbios 格式，例如 FABRIKAMSALES |
| Domain.FQDN | 当前正在导入的域的 FQDN 格式，例如 sales.fabrikam.com |
| Domain.LDAP | 当前正在导入的域的 LDAP 格式，例如 DC=sales,DC=fabrikam,DC=com |
| Forest.Netbios | 当前正在导入的林名称的 Netbios 格式，例如 FABRIKAMCORP |
| Forest.FQDN | 当前正在导入的林名称的 FQDN 格式，例如 fabrikam.com |
| Forest.LDAP | 当前正在导入的林名称的 LDAP 格式，例如 DC=fabrikam,DC=com |

系统提供以下参数用于获取当前正在运行的连接器的标识符：  
`Connector.ID`

以下示例使用用户所在域的 netbios 名称填充 Metaverse 属性域：  
`domain` <- `%Domain.Netbios%`

### 运算符
可以使用以下运算符：

- **比较**：<、<=、<>、=、>、>=
- **数学**：+、-、\*、-
- **字符串**：&（连接）
- **逻辑**：&&（和）、||（或）
- **计算顺序**：( )

运算符从左到右进行求值，并具有相同的求值优先级。也就是说，\*（乘号）不会在 -（减号）之前求值。2\*(5+3) 与 2\*5+3 不同。如果从左到右的求值顺序不适当，可以使用括号 () 来更改求值顺序。

## 多值属性
可对单值和多值属性运行函数。对于多值属性，函数将针对每个值运行，向每个值应用相同的函数。

例如：  
`Trim([proxyAddresses])` 对 proxyAddress 属性中的每个值执行 Trim。  
`Word([proxyAddresses],1,"@") & "@contoso.com"` 对于包含 @ 符号的每个值，将域替换为 @contoso.com。  
`IIF(InStr([proxyAddresses],"SIP:")=1,NULL,[proxyAddresses])` 查找 SIP 地址并从值中删除该地址。

## 后续步骤

- 在 [Understanding Declarative Provisioning](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning/)（了解声明性预配）中了解有关配置模型的详细信息。
- 以 [Understanding the default configuration](/documentation/articles/active-directory-aadconnectsync-understanding-default-configuration/)（了解默认配置）中了解如何现成地使用声明式预配。
- 在 [How to make a change to the default configuration](/documentation/articles/active-directory-aadconnectsync-change-the-configuration/)（如何对默认配置进行更改）中了解如何使用声明性预配进行实际更改。

**概述主题**

- [Azure AD Connect 同步：理解和自定义同步](/documentation/articles/active-directory-aadconnectsync-whatis/)
- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

**参考主题**

- [Azure AD Connect 同步：函数参考](/documentation/articles/active-directory-aadconnectsync-functions-reference/)

<!---HONumber=Mooncake_0926_2016-->
