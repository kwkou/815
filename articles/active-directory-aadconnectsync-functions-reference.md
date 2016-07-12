<properties
	pageTitle="Azure AD Connect 同步：函数参考 | Azure"
	description="在 Azure AD Connect 同步中引用声明性设置表达式。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="StevenPo"
	editor=""/>

<tags 
	ms.service="active-directory"
	ms.date="03/07/2016"
	wacn.date="04/28/2016"/>


# Azure AD Connect 同步：函数参考


在 Azure Active Directory Sync 中，函数用于在同步期间操作属性值。  
函数的语法使用以下格式表示：  
`<output type> FunctionName(<input type> <position name>, ..)`

如果函数被重载并接受多个语法，则会列出所有的有效语法。  
该函数为强类型函数，并会验证传递的类型是否匹配记录的类型。  
如果类型不匹配，则会引发错误。

类型使用以下语法表示：

- **bin** – 二进制
- **bool** – 布尔值
- **dt** – UTC 日期/时间
- **enum** – 已知常量的枚举
- **exp** – 表达式，计算结果预计为布尔值
- **mvbin** – 多值二进制
- **mvstr** – 多值引用
- **num** – 数值
- **ref** – 单值引用
- **str** – 单值字符串
- **var** – （几乎）任何其他类型的变体
- **void** – 不返回值



## 函数引用

----------
**转换：**

[CBool](#cbool) &nbsp;&nbsp;&nbsp;&nbsp; [CDate](#cdate) &nbsp;&nbsp;&nbsp;&nbsp; [CGuid](#cguid) &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromBase64](#convertfrombase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToBase64](#converttobase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromUTF8Hex](#convertfromutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToUTF8Hex](#converttoutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [CNum](#cnum) &nbsp;&nbsp;&nbsp;&nbsp; [CRef](#cref) &nbsp;&nbsp;&nbsp;&nbsp; [CStr](#cstr) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromGuid](#StringFromGuid) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromSid](#stringfromsid)

**日期/时间：**

[DateAdd](#dateadd) &nbsp;&nbsp;&nbsp;&nbsp; [DateFromNum](#datefromnum) &nbsp;&nbsp;&nbsp;&nbsp; [FormatDateTime](#formatdatetime) &nbsp;&nbsp;&nbsp;&nbsp; [Now](#now) &nbsp;&nbsp;&nbsp;&nbsp; [NumFromDate](#numfromdate)

**Directory**

[DNComponent](#dncomponent) &nbsp;&nbsp;&nbsp;&nbsp; [DNComponentRev](#dncomponentrev) &nbsp;&nbsp;&nbsp;&nbsp; [EscapeDNComponent](#escapedncomponent)

**计算：**

[IsBitSet](#isbitset) &nbsp;&nbsp;&nbsp;&nbsp; [IsDate](#isdate) &nbsp;&nbsp;&nbsp;&nbsp; [IsEmpty](#isempty)
&nbsp;&nbsp;&nbsp;&nbsp; [IsGuid](#isguid) &nbsp;&nbsp;&nbsp;&nbsp; [IsNull](#isnull) &nbsp;&nbsp;&nbsp;&nbsp; [IsNullOrEmpty](#isnullorempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsNumeric](#isnumeric)  &nbsp;&nbsp;&nbsp;&nbsp; [IsPresent](#ispresent) &nbsp;&nbsp;&nbsp;&nbsp; [IsString](#isstring)

**数学：**

[BitAnd](#bitand) &nbsp;&nbsp;&nbsp;&nbsp; [BitOr](#bitor) &nbsp;&nbsp;&nbsp;&nbsp; [RandomNum](#randomnum)

**多值**

[Contains](#contains) &nbsp;&nbsp;&nbsp;&nbsp; [Count](#count) &nbsp;&nbsp;&nbsp;&nbsp; [Item](#item) &nbsp;&nbsp;&nbsp;&nbsp; [ItemOrNull](#itemornull) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [RemoveDuplicates](#removeduplicates) &nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)

**程序流：**

[Error](#error) &nbsp;&nbsp;&nbsp;&nbsp; [IIF](#iif) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)


**文本**

[GUID](#guid) &nbsp;&nbsp;&nbsp;&nbsp; [InStr](#instr) &nbsp;&nbsp;&nbsp;&nbsp; [InStrRev](#instrrev) &nbsp;&nbsp;&nbsp;&nbsp; [LCase](#lcase) &nbsp;&nbsp;&nbsp;&nbsp; [Left](#left) &nbsp;&nbsp;&nbsp;&nbsp; [Len](#len) &nbsp;&nbsp;&nbsp;&nbsp; [LTrim](#ltrim) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp;&nbsp;&nbsp; [PadLeft](#padleft) &nbsp;&nbsp;&nbsp;&nbsp; [PadRight](#padright) &nbsp;&nbsp;&nbsp;&nbsp; [PCase](#pcase) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [ReplaceChars](#replacechars) &nbsp;&nbsp;&nbsp;&nbsp; [Right](#right) &nbsp;&nbsp;&nbsp;&nbsp; [RTrim](rtrim) &nbsp;&nbsp;&nbsp;&nbsp; [Trim](#trim) &nbsp;&nbsp;&nbsp;&nbsp; [UCase](#ucase) &nbsp;&nbsp;&nbsp;&nbsp; [Word](#word)

----------
### BitAnd

**说明：**  
BitAnd 函数设置值的指定位。

**语法：**  
`num BitAnd(num value1, num value2)`

- value1、value2：应该 AND 在一起的数字值

**备注：**  
此函数将两个参数转换为二进制表示形式，并将位设置为：

- 0 - 如果*掩码*和*标志*中相应位的其中一个或两个均为 0
- 1 - 如果两个相应位均为 1。

换而言之，除了当两个参数的相应位均为 1 时之外，所有情况下均返回 0。

**示例：**  
`BitAnd(&HF, &HF7)`  
返回 7，因为十六进制“F”AND“F7”的计算结果为此值。

----------
### BitOr

**说明：**  
BitOr 函数设置值的指定位。

**语法：**  
`num BitOr(num value1, num value2)`

- value1、value2：应该 OR 在一起的数字值

**备注：**  
此函数将两个参数转换为二进制表示形式，并且当掩码和标志中相应位的其中一个或两个均为 1 时，将位设置为 1，当两个相应位均为 0 时，设置为 0。换而言之，除了当两个参数的相应位均为 0 时之外，所有情况下均返回 1。

----------
### CBool

**说明：**  
CBool 函数基于求值的表达式返回布尔值

**语法：**  
`bool CBool(exp Expression)`

**备注：**  
如果表达式的计算结果为非零值，则 CBool 返回 True，否则则返回 False。

**示例：**  
`CBool([attrib1] = [attrib2])`

如果两个属性具有相同的值，则返回 True。

----------
### CDate

**说明：**  
CDate 函数通过字符串返回 UTC DateTime。DateTime 不是 Sync 中的原生属性类型，但被某些函数使用。

**语法：**  
`dt CDate(str value)`

- Value：具有日期、时间和可选时区的字符串

**备注：**  
返回的字符串始终是 UTC 格式。

**示例：**  
`CDate([employeeStartTime])`  
基于员工的开始时间返回 DateTime

`CDate("2013-01-10 4:00 PM -8")`  
返回表示“2013-01-11 12:00 AM”的 DateTime

----------
### CGuid

**说明：**  
CGuid 函数将 GUID 的字符串表示转换为其二进制表示形式。

**语法：**  
`bin CGuid(str GUID)`

- 采用这种模式设置格式的字符串：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

----------
### Contains

**说明：**  
Contains 函数寻找多值属性内的字符串

**语法：**  
`num Contains (mvstring attribute, str search)` - 区分大小写  
`num Contains (mvstring attribute, str search, enum Casetype)`  
`num Contains (mvref attribute, str search)` - 区分大小写

- attribute：要搜索的多值属性。<br>
- search：在属性中查找的字符串。<br>
- Casetype：不区分大小写或区分大小写。<br>

返回找到字符串的多值属性中的索引。如果未找到字符串，则返回 0。

**备注：**  
对于多值字符串属性，搜索会在值中查找子字符串。  
对于引用属性，搜索的字符串必须与视为匹配的值完全匹配。

**示例：**  
`IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))`  
如果 proxyAddresses 属性具有主电子邮件地址（由大写“SMTP:”表示），则返回 proxyAddress 属性，否则返回错误。

----------
### ConvertFromBase64

**说明：**  
ConvertFromBase64 函数将指定的 base64 编码值转换为规则的字符串。

**语法：**  
`str ConvertFromBase64(str source)` - 假定采用 Unicode 编码 <br> 
`str ConvertFromBase64(str source, enum Encoding)`

- source：Base64 编码的字符串  
- Encoding：Unicode、ASCII、UTF8

**示例**  
`ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`  
`ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

这两个示例均返回 "Hello world!"

----------
### ConvertFromUTF8Hex

**说明：**  
ConvertFromUTF8Hex 函数将指定的 UTF8 Hex 编码值转换为字符串。

**语法：**  
`str ConvertFromUTF8Hex(str source)`

- source：UTF8 2 字节编码的字符串

**备注：**  
该结果中此函数和 ConvertFromBase64 (,UTF8) 之间的差异对 DN 属性是友好的。  
此格式被 Azure Active Directory 用作 DN。

**示例：**  
`ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`  
返回 "Hello world!"

----------
### ConvertToBase64

**说明：**  
ConvertToBase64 函数将字符串转换为 Unicode base64 字符串。  
将整数数组的值转换为其等效字符串表示形式，该表示形式使用 base 64 数字编码。

**语法：**  
`str ConvertToBase64(str source)`

**示例：**  
`ConvertToBase64("Hello world!")`  
返回 "SABlAGwAbABvACAAdwBvAHIAbABkACEA"

----------
### ConvertToUTF8Hex

**说明：**  
ConvertToUTF8Hex 函数将字符串转换为 UTF8 Hex 编码的值。

**语法：**  
`str ConvertToUTF8Hex(str source)`

**备注：**  
此函数的输出格式被 Azure Active Directory 用作 DN 属性。

**示例：**  
`ConvertToUTF8Hex("Hello world!")`  
返回 48656C6C6F20776F726C6421

----------
### 计数

**说明：**  
Count 函数返回多值属性中的元素数量

**语法：**  
`num Count(mvstr attribute)`

----------
### CNum

**说明：**  
CNum 函数使用字符串并返回数值数据类型。

**语法：**  
`num CNum(str value)`

----------
### CRef

**说明：**  
将字符串转换为引用属性

**语法：**  
`ref CRef(str value)`

**示例：**  
`CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

----------
### CStr

**说明：**  
CStr 函数转换为字符串数据类型。

**语法：**  
`str CStr(num value)`  
`str CStr(ref value)`  
`str CStr(bool value)`

- value：可以是数字值、引用属性或布尔值。

**示例：**  
`CStr([dn])`  
可能返回 “cn=Joe,dc=contoso,dc=com”

----------
### DateAdd

**说明：**  
返回包含指定时间间隔已添加到其中的日期的日期。

**语法：**  
`dt DateAdd(str interval, num value, dt date)`

- interval：字符串表达式，即你想要添加的时间间隔。字符串必须具有下列值之一：
 - yyyy Year
 - q Quarter
 - m Month
 - y Day of year
 - d Day
 - w Weekday
 - ww Week
 - h Hour
 - n Minute
 - s Second
- 值：你想要添加的单元数。它可以是正值（以获取将来的日期）或负值（以获取过去的日期）。
- 日期：表示间隔添加到其中的日期的 DateTime。

**示例：**  
`DateAdd("m", 3, CDate("2001-01-01"))`  
添加 3 个月，并返回表示“2001-04-01”的 DateTime

----------
### DateFromNum

**说明：**  
DateFromNum 函数将 AD 的日期格式的值转换为 DateTime 类型。

**语法：**  
`dt DateFromNum(num value)`

**示例：**  
`DateFromNum([lastLogonTimestamp])`  
`DateFromNum(129699324000000000)`  
返回表示 2012-01-01 23:00:00 的 DateTime

----------
### DNComponent

**说明：**  
DNComponent 函数返回从左边起的指定 DN 组件的值。

**语法：**  
`str DNComponent(ref dn, num ComponentNumber)`

- dn：要解释的引用属性
- ComponentNumber：要返回的 DN 中的组件

**示例：**  
`DNComponent([dn],1)`  
如果 dn 为“cn=Joe,ou=…”，则返回 Joe

----------
### DNComponentRev

**说明：**  
DNComponentRev 函数返回从右边起（末尾）的指定 DN 组件的值。

**语法：**  
`str DNComponentRev(ref dn, num ComponentNumber)`  
`str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

- dn：要解释的引用属性
- ComponentNumber - 要返回的 DN 中的组件
- Options：DC – 忽略具有“dc=”的所有组件

**示例：**  
如果 dn 为 "cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com"，则  
`DNComponentRev([dn],3)`  
`DNComponentRev([dn],1,"DC")`  
两者都返回 US。

----------
### 错误

**说明：**  
Error 函数用于返回自定义错误。

**语法：**  
`void Error(str ErrorMessage)`

**示例：**  
`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))`  
如果属性 accountName 不存在，则对象上引发错误。

----------
### EscapeDNComponent

**说明：**  
EscapeDNComponent 函数使用 DN 的一个组件，并对其进行转义，以便它可以在 LDAP 中表示。

**语法：**  
`str EscapeDNComponent(str value)`

**示例：**  
`EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)`  
即使 displayName 属性具有必须在 LDAP 中转义的字符，请确保可以在 LDAP 目录中创建对象。

----------
### FormatDateTime

**说明：**  
FormatDateTime 函数用于为具有指定格式的字符串设置 DateTime 格式

**语法：**  
`str FormatDateTime(dt value, str format)`

- value：DateTime 格式的值
- format：表示要转换为的格式的字符串。

**备注：**  
格式的可能值可以在此处找到：[用户定义的日期/时间格式（格式函数）](http://msdn2.microsoft.com/library/73ctwf33(VS.90).aspx)

**示例：**

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")`  
结果是“2007-12-25”。

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")`  
结果可能是“20140905081453.0Z”

----------
### GUID

**说明：**  
函数 GUID 生成新的随机 GUID

**语法：**  
`str GUID()`

----------
### IIF

**说明：**  
IIF 函数基于指定的条件返回一组可能值中的其中一个值。

**语法：**  
`var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

- condition：计算结果可能为 true 或 false 的任何值或表达式。
- valueIfTrue：如果条件计算结果为 true，则返回的值。
- valueIfFalse：如果条件计算结果为 false，则返回的值。

**示例：**  
`IIF([employeeType]="Intern","t-" & [alias],[alias])`  
如果用户是实习生，则返回用户的别名，同时将“t-”添加到其开头，否则按原样返回用户的别名。

----------
### InStr

**说明：**  
InStr 函数查找字符串中第一次出现的子字符串

**语法：**

`num InStr(str stringcheck, str stringmatch)`  
`num InStr(str stringcheck, str stringmatch, num start)`  
`num InStr(str stringcheck, str stringmatch, num start , enum compare)`

- stringcheck：要搜索的字符串
- stringmatch：要查找的字符串
- start：查找子字符串的起始位置
- compare：vbTextCompare 或 vbBinaryCompare

**备注：**  
返回其中已找到子字符串的位置，如果未找到，则返回 0。

**示例：**  
`InStr("The quick brown fox","quick")`  
计算结果为 5

`InStr("repEated","e",3,vbBinaryCompare)`  
计算结果为 7

----------
### InStrRev

**说明：**  
InStrRev 函数查找字符串中最后一次出现的子字符串

**语法：**  
`num InstrRev(str stringcheck, str stringmatch)`  
`num InstrRev(str stringcheck, str stringmatch, num start)`  
`num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

- stringcheck：要搜索的字符串
- stringmatch：要查找的字符串
- start：查找子字符串的起始位置
- compare：vbTextCompare 或 vbBinaryCompare

**备注：**  
返回其中已找到子字符串的位置，如果未找到，则返回 0。

**示例：**  
`InStrRev("abbcdbbbef","bb")`  
返回 7

----------
### IsBitSet

**说明：**  
函数 IsBitSet 测试是否设置了位

**语法：**  
`bool IsBitSet(num value, num flag)`

- value：计算的数字值。标志：表示具有要计算的位的数字值

**示例：**  
`IsBitSet(&HF,4)`  
返回 True，因为位“4”在十六进制值“F”中设置

----------
### IsDate

**说明：**  
如果表达式可以计算为 DateTime 类型，则 IsDate 函数计算结果为 True。

**语法：**  
`bool IsDate(var Expression)`

**备注：**  
用来确定 CDate() 是否会成功。

----------
### IsEmpty

**说明：**  
如果属性是出现在 CS 或 MV 中，但计算结果为空字符串，则 IsEmpty 函数计算结果为 True。

**语法：**  
`bool IsEmpty(var Expression)`

----------
### IsGuid

**说明：**  
如果字符串可以转换为 GUID，则 IsGuid 函数计算结果为 true。

**语法：**  
`bool IsGuid(str GUID)`

**备注：**  
GUID 定义为遵循以下其中一种模式的字符串：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

用来确定 CGuid() 是否会成功。

**示例：**  
`IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)`  
如果 StrAttribute 具有 GUID 格式，则返回二进制表示形式，否则返回 Null。

----------
### IsNull

**说明：**  
如果表达式的计算结果为 Null，则 IsNull 函数返回 true。

**语法：**  
`bool IsNull(var Expression)`

**备注：**  
对于属性，Null 表示缺少属性。

**示例：**  
`IsNull([displayName])`  
如果属性没有在 CS 或 MV 中出现，则返回 True。

----------
### IsNullOrEmpty

**说明：**  
如果表达式为 null 或空字符串，则 IsNullOrEmpty 函数返回 true。

**语法：**  
`bool IsNullOrEmpty(var Expression)`

**备注：**  
对于属性，如果属性不存在，或存在但为空字符串，此语法计算结果则为 True。<br>
此函数的逆函数被命名为 IsPresent。

**示例：**  
`IsNullOrEmpty([displayName])`  
如果属性在 CS 或 MV 中没有出现或为空字符串，则返回 True。

----------
### IsNumeric

**说明：**  
IsNumeric 函数返回布尔值，该值指示表达式是否可以计算为数字类型。

**语法：**  
`bool IsNumeric(var Expression)`

**备注：**  
用来确定 CNum() 是否会成功地分析表达式。

----------
### IsString

**说明：**  
如果表达式可以计算为字符串类型，则 IsString 函数计算结果为 True。

**语法：**  
`bool IsString(var expression)`

**备注：**  
用来确定 CStr() 是否会成功地分析表达式。

----------
### IsPresent

**说明：**  
如果表达式的计算结果为字符串，该字符串不是 Null 且不为空，则 IsPresent 函数返回 true。

**语法：**  
`bool IsPresent(var expression)`

**备注：**  
此函数的逆函数被命名为 IsNullOrEmpty。

**示例：**  
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

----------
### 项目

**说明：**  
Item 函数返回多值字符串/属性中的一个项。

**语法：**  
`var Item(mvstr attribute, num index)`

- attribute：多值属性
- index：对多值字符串中某个项的索引。

**备注：**  
Item 函数与 Contains 函数一起使用很有用，因为后者函数会返回对多值属性中某个项的索引。

如果索引超出界限，则引发错误。

**示例：**  
`Mid(Item([proxyAddress],Contains([proxyAddress], "SMTP:")),6)`  
返回主电子邮件地址。

----------
### ItemOrNull

**说明：**  
ItemOrNull 函数返回多值字符串/属性中的一个项。

**语法：**  
`var ItemOrNull(mvstr attribute, num index)`

- attribute：多值属性
- index：对多值字符串中某个项的索引。

**备注：**  
ItemOrNull 函数与 Contains 函数一起使用很有用，因为后者函数会返回对多值属性中某个项的索引。

如果索引超出界限，则返回 Null 值。

----------
### Join

**说明：**  
Join 函数使用多值字符串，并返回每个项之间插入指定分隔符的单值字符串。

**语法：**  
`str Join(mvstr attribute)`  
`str Join(mvstr attribute, str Delimiter)`

- attribute：包含要联接的字符串的多值属性。
- delimiter：任意字符串，用于分隔返回的字符串中的子字符串。如果省略，则使用空格字符（“ ”）。如果分隔符为零长度字符串（“”）或零，则列表中的所有项都不使用分隔符连接。

**备注：**  
Join 和 Split 函数之间没有奇偶校验。Join 函数使用字符串数组，并使用分隔符字符串将它们联接起来，以返回单个字符串。Split 函数使用字符串并以分隔符分隔，以返回字符串数组。但是，主要区别是 Join 可以使用任何分隔符字符串将字符串连接起来，而 Split 仅可以使用单个字符分隔符分隔字符串。

**示例：**  
`Join([proxyAddresses],",")`  
可能返回：“SMTP:john.doe@contoso.com,smtp:jd@contoso.com”

----------
### LCase

**说明：**  
LCase 函数将字符串中的所有字符都转换为小写。

**语法：**  
`str LCase(str value)`

**示例：**  
`LCase("TeSt")`  
返回“test”。

----------
### Left

**说明：**  
Left 函数从字符串左侧起返回指定的字符数。

**语法：**  
`str Left(str string, num NumChars)`

- string：从中返回字符的字符串
- NumChar：标识从字符串开头（左侧）起返回的字符数的数字

**备注：**  
包含字符串中第一个 numChars 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的字符数比 numChar 中指定的数量少，则返回与该字符串相同的字符串（即：包含参数 1 中的所有字符）。

**示例：**  
`Left("John Doe", 3)`  
返回 “Joh”。

----------
### Len

**说明：**  
Len 函数返回字符串中的字符数。

**语法：**  
`num Len(str value)`

**示例：**  
`Len("John Doe")`  
返回 8

----------
### LTrim

**说明：**  
LTrim 函数从字符串中删除前导空格。

**语法：**  
`str LTrim(str value)`

**示例：**  
`LTrim(" Test ")`  
返回 “Test”

----------
### Mid

**说明：**  
Mid 函数从字符串指定位置起返回指定的字符数。

**语法：**  
`str Mid(str string, num start, num NumChars)`

- string：从中返回字符的字符串
- start：标识从中返回字符的字符串中起始位置的数字
- NumChar：标识从字符串中的位置返回的字符数的数字

**备注：**  
从字符串中的开始位置开始返回 numChars 字符。  
包含字符串中开始位置的 numChar 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果 start > 字符串的长度，则返回输入字符串。
- 如果 start <= 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串中开始位置没有保留的 numChar 字符，则会返回尽可能多可以返回的字符。

**示例：**  
`Mid("John Doe", 3, 5)`  
返回 "hn Do"。

`Mid("John Doe", 6, 999)`  
返回 "Doe"

----------
### Now

**说明：**  
Now 函数根据计算机的系统日期和时间返回指定当前日期和时间的 DateTime。

**语法：**  
`dt Now()`

----------
### NumFromDate

**说明：**  
NumFromDate 函数返回 AD 的日期格式的日期。

**语法：**  
`num NumFromDate(dt value)`

**示例：**  
`NumFromDate(CDate("2012-01-01 23:00:00"))`  
返回 129699324000000000

----------
### PadLeft

**说明：**  
PadLeft 函数使用提供的填充字符将字符串从左侧填充到指定长度。

**语法：**  
`str PadLeft(str string, num length, str padCharacter)`

- string：要填充的字符串。
- length：表示所需字符串长度的整数。
- padCharacter：包含用作填充字符的单个字符的字符串

**备注：**

- 如果字符串的长度小于 length，则 padCharacter 会重复追加到字符串的开头（左侧），直到其长度等于 length。
- PadCharacter 可以是空格字符，但不能为 null 值。
- 如果字符串的长度等于或大于 length，则返回不变的字符串。
- 如果字符串的长度大于或等于 length，则返回与 string 相同的字符串。
- 如果字符串的长度小于 length，则返回具有所需长度的新字符串，其中包含用 padCharacter 填充的字符串。
- 如果字符串为 null，该函数则返回空字符串。

**示例：**  
`PadLeft("User", 10, "0")`  
返回 "000000User"。

----------
### PadRight

**说明：**  
PadRight 函数使用提供的填充字符将字符串从右侧填充到指定长度。

**语法：**  
`str PadRight(str string, num length, str padCharacter)`

- string：要填充的字符串。
- length：表示所需字符串长度的整数。
- padCharacter：包含用作填充字符的单个字符的字符串

**备注：**

- 如果字符串的长度小于 length，则 padCharacter 会重复追加到字符串的末尾（右侧），直到其长度等于 length。
- padCharacter 可以是空格字符，但不能为 null 值。
- 如果字符串的长度等于或大于 length，则返回不变的字符串。
- 如果字符串的长度大于或等于 length，则返回与 string 相同的字符串。
- 如果字符串的长度小于 length，则返回具有所需长度的新字符串，其中包含用 padCharacter 填充的字符串。
- 如果字符串为 null，该函数则返回空字符串。

**示例：**  
`PadRight("User", 10, "0")`  
返回 "User000000"。

----------
### PCase

**说明：**  
PCase 函数将字符串中每个空格分隔词的第一个字符转换为大写形式，并将所有其他字符都转换为小写形式。

**语法：**  
`String PCase(string)`

**示例：**  
`PCase("TEsT")`  
返回“Test”。

----------
### RandomNum

**说明：**  
RandomNum 函数返回指定间隔之间的随机数字。

**语法：**  
`num RandomNum(num start, num end)`

- start：标识要生成的随机值的下限的数字
- end：标识要生成的随机值的上限的数字

**示例：**  
`Random(100,999)`  
可以返回 734。

----------
### RemoveDuplicates

**说明：**  
RemoveDuplicates 函数使用多值字符串，并确保每个值都是唯一值。

**语法：**  
`mvstr RemoveDuplicates(mvstr attribute)`

**示例：**  
`RemoveDuplicates([proxyAddresses])`  
返回净化的 proxyAddress 属性，其中所有重复值已被删除。

----------
### Replace

**说明：**  
Replace 函数将所有出现的某一字符串替换为另一个字符串。

**语法：**  
`str Replace(str string, str OldValue, str NewValue)`

- string：替换其中的值的字符串。
- OldValue：要搜索和替换的字符串。
- NewValue：要替换的字符串。

**备注：**  
该函数可以识别以下特殊 moniker：

- \\n – 新行
- \\r – 回车符
- \\t – 选项卡

**示例：**  
`Replace([address],"\r\n",", ")`  
将 CRLF 替换为逗号和空格，可能导致出现“One Microsoft Way, Redmond, WA, USA”

----------
### ReplaceChars

**说明：**  
ReplaceChars 函数替换 ReplacePattern 字符串中找到的所有出现的字符。

**语法：**  
`str ReplaceChars(str string, str ReplacePattern)`

- string：替换其中值的字符串。
- ReplacePattern：包含具有要替换字符的字典的字符串。

格式为 {source1}:{target1},{source2}:{target2},{sourceN},{targetN}，其中源是要查找并确定要替换的目标字符串的字符。

**备注：**

- 该函数使用每次出现的定义的源，并使用目标替换它们。
- 源必须正好是一个 (unicode) 字符。
- 源不能为空或长度超过一个字符（分析错误）。
- 目标可以具有多个字符，例如 ö:oe、β:ss。
- 目标可以为空，该值指示应删除字符。
- 源区分大小写，并且必须是完全匹配。
- 逗号 (,) 和冒号 (:) 是保留的字符，不能使用此函数进行替换。
- 空格和 ReplacePattern 字符串中的其他空白字符被忽略。

**示例：**  
`%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)`  
返回 Raksmorgas

`ReplaceChars("O’Neil",%ReplaceString%)`  
返回 “ONeil”，定义要删除单次勾选。

----------
### Right

**说明：**  
Right 函数从字符串右侧（末尾）起返回指定的字符数。

**语法：**  
`str Right(str string, num NumChars)`

- string：从中返回字符的字符串
- NumChar：标识从字符串末尾（右侧）起返回的字符数的数字

**备注：**  
NumChars 字符从字符串的最后位置返回。

包含字符串中最后的 numChar 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的字符数比 NumChar 中指定的数量少，则返回与该字符串相同的字符串。

**示例：**  
`Right("John Doe", 3)`  
返回 "Doe"。

----------
### RTrim

**说明：**  
RTrim 函数从字符串中删除尾随空格。

**语法：**  
`str RTrim(str value)`

**示例：**  
`RTrim(" Test ")`  
返回“Test”。

----------
### 拆分

**说明：**  
Split 函数使用采用分隔符分隔的字符串，并使其成为多值字符串。

**语法：**  
`mvstr Split(str value, str delimiter)`  
`mvstr Split(str value, str delimiter, num limit)`

- value：用分隔符字符来分隔的字符串。
- delimiter：用作分隔符的单个字符。
- limit：将返回的最大数目的值。

**示例：**  
`Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")`  
返回多值字符串，其中 2 个元素对 proxyAddress 属性有用。

----------
### StringFromGuid

**说明：**  
StringFromGuid 函数使用二进制 GUID，并将其转换为字符串

**语法：**  
`str StringFromGuid(bin GUID)`

----------
### StringFromSid

**说明：**  
StringFromSid 函数将字节数组或包含安全标识符的多值字节数组转换为字符串或多值字符串。

**语法：**  
`str StringFromSid(bin ObjectSID)`  
`mvstr StringFromSid(mvbin ObjectSID)`

----------
### Switch

**说明：**  
Switch 函数用于基于计算的条件返回单个值。

**语法：**  
`var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

- expr：想要计算结果的变体表达式。
- value：当相应表达式为 True 时要返回的值。

**备注：**  
Switch 函数参数列表包含表达式和值对。表达式从左到右计算结果，并返回与计算结果为 True 的第一个表达式相关联的值。如果没有正确配对部件，则会发生运行时错误。

例如，如果 expr1 为 True，则 Switch 返回 value1。如果 expr-1 为 False，但 expr-2 为 True，则 Switch 返回 value-2，依此类推。

如果满足以下条件，Switch 将返回 Nothing：

- 没有任何表达式求值为 True。
- 第一个 True 表达式的相应值为 Null。

Switch 会对所有表达式计算结果，即使它只返回其中一个结果。为此，你应监视非预期的负面影响。例如，如果任何表达式的计算结果导致除数为零的错误，则会出现错误。

值还可以是将返回自定义字符串的错误函数。

**示例：**  
`Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))`  
返回某些主要城市所使用的语言，否则返回错误。

----------
### Trim

**说明：**  
Trim 函数从字符串中删除前导空格和尾随空格。

**语法：**  
`str Trim(str value)`  
`mvstr Trim(mvstr value)`

**示例：**  
`Trim(" Test ")`  
返回“Test”。

`Trim([proxyAddresses])`  
删除 proxyAddress 属性中每个值的前导空格和尾随空格。

----------
### UCase

**说明：**  
UCase 函数将字符串中的所有字符都转换为大写形式。

**语法：**  
`str UCase(str string)`

**示例：**  
`UCase("TeSt")`  
返回“TEST”。

----------
### Word

**说明：**  
基于描述要使用的分隔符与要返回的单词数的参数，Word 函数返回字符串中包含的单词。

**语法：**  
`str Word(str string, num WordNumber, str delimiters)`

- string：从中返回单词的字符串。
- WordNumber：标识应返回单词数的数字。
- delimiter：表示应该用于标识单词的分隔符的字符串

**备注：**  
字符串中的字符由分隔符中其中一个字符分隔的每个字符串被标识为单词：

- 如果数字 < 1，则返回空字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的单词少于应返回数字或字符串不包含由分隔符标识的任何单词，则返回空字符串。

**示例：**  
`Word("The quick brown fox",3," ")`  
返回 “brown”

`Word("This,string!has&many separators",3,",!&#")`  
返回 “has”

## 其他资源

* [了解声明性设置表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions/)
* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis/)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---HONumber=Mooncake_0606_2016-->