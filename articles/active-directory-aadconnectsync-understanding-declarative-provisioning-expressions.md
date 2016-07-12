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
	ms.date="04/14/2016"
	wacn.date="06/14/2016"/>


# Azure AD Connect 同步：了解声明性预配表达式

Azure AD Connect 同步构建在声明性预配的基础之上。该声明性预配是在 Forefront Identity Manager 2010 中首次引入的。利用该设置，你可以在无需编写编译代码的情况下，实现完整的标识集成业务逻辑。

声明性设置的一个重要组成部分是属性流中使用的表达式语言。所用的语言是 Microsoft® Visual Basic® for Applications (VBA) 的子集。Microsoft Office 中使用了这种语言，具有 VBScript 经验的用户都认识该语言。声明性设置表达式语言只使用函数，不属于结构化语言；它不提供任何方法或语句。函数将嵌套在表达式程序流中。

有关详细信息，请参阅[欢迎使用适用于 Office 2013 的 Visual Basic 应用程序语言参考](https://msdn.microsoft.com/library/gg264383.aspx)。

属性属于强类型。函数只接受正确类型的属性。它也区分大小写。函数名称和属性名称都必须具有正确的大小写，否则会引发错误

## 语言定义和标识符

- 函数名称后跟加括号的参数：FunctionName(argument 1,argument N)。
- 属性采用方括号标识，如 [attributeName]。
- 参数通过百分比符号标识：%ParameterName%
- 字符串常量放在引号中：例如"Contoso"（注意：必须使用直引号 ""，而不能使用弯引号“”）
- 数字值表示不带引号，并且应为十进制。十六进制值带有前缀 &H。例如98052、&HFF
- 表示布尔值的常量： True、 False。
- 内置常量仅采用其名称表示：NULL、CRLF、IgnoreThisFlow

### 函数
声明性预配使用许多函数来实现转换属性值的可能性。这些函数可以嵌套，因此，一个函数的结果将传递到另一个函数。

函数还可以处理多值属性。在这种情况下，函数将处理各个值，并向每个值应用同一个函数。例如，`Trim([proxyAddresses])` 将对 proxyAddress 属性中的每个值执行 Trim。

有关函数的完整列表，请参阅[函数参考](/documentation/articles/active-directory-aadconnectsync-functions-reference/)。

### Parameters

参数由连接器定义，或者由系统管理员使用 PowerShell 来设置。参数通常包含因系统不同而各异的值，例如用户所在域的名称。这些参数可在属性流中使用。

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

使用用户所在域的 netbios 名称填充 Metaverse 属性域的示例：

`domain` <- `%Domain.Netbios%`

### 运算符

可以使用以下运算符：

- **比较**：<、<=、<>、=、>、>=
- **数学**：+、-、*、-
- **字符串**：&（连接）
- **逻辑**：&&（和）、||（或）
- **计算顺序**：( )

运算符从左到右进行求值，并具有相同的求值优先级。也就是说，*（乘号）不会在 -（减号）之前求值。2*(5+3) 与 2*5+3 不同。如果从左到右的求值顺序不适当，可以使用括号 () 来更改求值顺序。

## 常见方案

### 属性的长度

字符串属性在默认情况下设置为可编制索引，并且最大长度为 448 个字符。如果你正在使用其中可能包含更多字符的字符串属性，则请确保属性流中包括以下内容：

`attributeName` <- `Left([attributeName],448)`

### 更改 userPrincipalSuffix

Active Directory 中的 userPrincipalName 属性并非始终被用户知晓，并且可能不适合作为登录 ID。Azure AD Connect 同步安装向导允许选择不同的属性，例如 mail。但在某些情况下，必须计算该属性。例如：公司 Contoso 具有两个 Azure AD 目录，一个用于生产，另一个用于测试。他们希望测试租户中的用户只能更改登录 ID 中的后缀。

`userPrincipalName` <- `Word([userPrincipalName],1,"@") & "@contosotest.com"`

在此表达式中，我们首先使用左侧所有内容 @-sign (Word)，并与固定字符串连接。

### 将多值转换为单值

Active Directory 中的某些属性在架构中是多值，不过它们在 Active Directory 用户和计算机中看上去是单值。一个示例就是说明属性。

`description` <- `IIF(IsNullOrEmpty([description]),NULL,Left(Trim(Item([description],1)),448))`

在此表达式中，如果属性具有值，我们会使用属性中的第一项 (Item)，删除前导空格和尾随空格 (Trim)，然后保留字符串中的前 448 个字符（左）。

## 高级概念

### NULL 和 IgnoreThisFlow

对于入站同步规则，应始终使用常量 **NULL**。这表示流程没有要提供的值，并且另一个规则可以提供值。如果没有规则提供值，则会删除 metaverse 属性。

对于出站同步规则，有两个不同的常量可以使用：NULL 和 IgnoreThisFlow。两个常量均指示属性流没有要提供的内容，但不同之处是当其他规则都没有要提供的任何内容时会发生的情况。如果已连接目录中存在现有值，NULL 则会在删除它的属性上暂存删除，而 IgnoreThisFlow 则会保留现有值。

### ImportedValue

函数 ImportedValues 不同于其他所有函数，因为其属性名称必须放在引号内，而不是括在方括号中：ImportedValue(“proxyAddresses”)。

通常在同步期间，即使尚未导出或在导出过程中收到错误 (“top of the tower”)，属性也会使用预期值。入站同步会假定尚未到达已连接目录的属性最终会到达该目录。在某些情况下，仅同步由已连接目录已确认的值很重要，并且在这种情况下会使用函数 ImportedValue (“hologram and delta import tower”)。

可以从 Exchange 的现成同步规则 In from AD – User Common 中找到这种示例。其中，在混合 Exchange 中，如果已确认已成功导出由 Exchange Online 添加的值，则应该只同步该值：

`proxyAddresses` <- `RemoveDuplicates(Trim(ImportedValues("proxyAddresses")))`

有关函数的完整列表，请参阅 [Azure AD Connect Sync：函数引用](/documentation/articles/active-directory-aadconnectsync-functions-reference/)


## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis/)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)
 
<!--Image references-->

<!---HONumber=Mooncake_0606_2016-->