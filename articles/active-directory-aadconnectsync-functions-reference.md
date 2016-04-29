<properties
	pageTitle="Azure AD Connect Sync：函数引用"
	description="在 Azure AD Connect Sync 中引用声明性设置表达式。"
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
&nbsp;&nbsp;&nbsp;&nbsp; [IsGuid](#isguid) &nbsp;&nbsp;&nbsp;&nbsp; [IsNull](#isnull) &nbsp;&nbsp;&nbsp;&nbsp; [IsNullOrEmpty](#isnullorempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsNumeric](#isnumeric) &nbsp;&nbsp;&nbsp;&nbsp; [IsPresent](#ispresent) &nbsp;&nbsp;&nbsp;&nbsp; [IsString](#isstring)

**数学：**

[BitAnd](#bitand) &nbsp;&nbsp;&nbsp;&nbsp; [BitOr](#bitor) &nbsp;&nbsp;&nbsp;&nbsp; [RandomNum](#randomnum)

**多值**

[Contains](#contains) &nbsp;&nbsp;&nbsp;&nbsp; [Count](#count) &nbsp;&nbsp;&nbsp;&nbsp; [Item](#item) &nbsp;&nbsp;&nbsp;&nbsp; [ItemOrNull](#itemornull) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [RemoveDuplicates](#removeduplicates) &nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)

**程序流：**

[Error](#error) &nbsp;&nbsp;&nbsp;&nbsp; [IIF](#iif) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)


**文本**

[GUID](#guid) &nbsp;&nbsp;&nbsp;&nbsp; [InStr](#instr) &nbsp;&nbsp;&nbsp;&nbsp; [InStrRev](#instrrev) &nbsp;&nbsp;&nbsp;&nbsp; [LCase](#lcase) &nbsp;&nbsp;&nbsp;&nbsp; [Left](#left) &nbsp;&nbsp;&nbsp;&nbsp; [Len](#len) &nbsp;&nbsp;&nbsp;&nbsp; [LTrim](#ltrim) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp;&nbsp;&nbsp; [PadLeft](#padleft) &nbsp;&nbsp;&nbsp;&nbsp; [PadRight](#padright) &nbsp;&nbsp;&nbsp;&nbsp; [PCase](#pcase) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [ReplaceChars](#replacechars) &nbsp;&nbsp;&nbsp;&nbsp; [Right](#right) &nbsp;&nbsp;&nbsp;&nbsp; [RTrim](rtrim) &nbsp;&nbsp;&nbsp;&nbsp; [Trim](#trim) &nbsp;&nbsp;&nbsp;&nbsp; [UCase](#ucase) &nbsp;&nbsp;&nbsp;&nbsp; [Word](#word)

----------
### <a name="bitand"></a>BitAnd

**说明：** <br>BitAnd 函数设置值的指定位。
 
**语法：**<br> `num BitAnd(num value1, num value2)`

- value1、value2：应该 AND 在一起的数字值
 
**备注：**<br>此函数将两个参数转换为二进制表示形式，并将位设置为：
 
- 0 - 如果*掩码*和*标志*中相应位的其中一个或两个均为 0
- 1 - 如果两个相应位均为 1。 

换而言之，除了当两个参数的相应位均为 1 时之外，所有情况下均返回 0。
 
**示例：**<br> `BitAnd(&HF, &HF7)`<br> 返回 7，因为十六进制“F”AND“F7”的计算结果为此值。
 
----------
### <a name="bitor"></a>BitOr

**说明：** <br>BitOr 函数设置值的指定位。

**语法：** <br> `num BitOr(num value1, num value2)`

- value1、value2：应该 OR 在一起的数字值 

**备注：** <br>此函数将两个参数转换为二进制表示形式，并且当掩码和标志中相应位的其中一个或两个均为 1 时，将位设置为 1，当两个相应位均为 0 时，设置为 0。<br> 换而言之，除了当两个参数的相应位均为 0 时之外，所有情况下均返回 1。

----------
### <a name="cbool"></a>CBool

**说明：**<br>CBool 函数基于求值的表达式返回布尔值

**语法：** <br> `bool CBool(exp Expression)`

**备注：**<br>如果表达式的计算结果为非零值，则 CBool 返回 True，否则则返回 False。


**示例：**<br> `CBool([attrib1] = [attrib2])` <br>

如果两个属性具有相同的值，则返回 True。
 
 


----------
### <a name="cdate"></a>CDate

**说明：**<br>CDate 函数通过字符串返回 UTC DateTime。DateTime 不是 Sync 中的原生属性类型，但被某些函数使用。

**语法：**<br> `dt CDate(str value)`

- Value：具有日期、时间和可选时区的字符串

**备注：**<br>返回的字符串始终是 UTC 格式。

**示例：**<br> `CDate([employeeStartTime])` <br>基于员工的开始时间返回 DateTime

`CDate("2013-01-10 4:00 PM -8")` <br> 返回表示“2013-01-11 12:00 AM”的 DateTime
 



----------
### <a name="cguid"></a>CGuid

**说明：**<br>CGuid 函数将 GUID 的字符串表示转换为其二进制表示形式。
 
**语法：**<br> `bin CGuid(str GUID)GUID`

- 采用这种模式设置格式的字符串：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}
 



----------
### <a name="contains"></a>Contains

**说明：**<br>Contains 函数寻找多值属性内的字符串
 
**语法：**<br> `num Contains (mvstring attribute, str search)` - 区分大小写<br> `num Contains (mvstring attribute, str search, enum Casetype)`<br> `num Contains (mvref attribute, str search)` - 区分大小写

- attribute：要搜索的多值属性。<br>
- search：在属性中查找的字符串。<br>
- Casetype：不区分大小写或区分大小写。<br>

返回找到字符串的多值属性中的索引。如果未找到字符串，则返回 0。
 

**备注：**<br>对于多值字符串属性，搜索会在值中查找子字符串。<br> 对于引用属性，搜索的字符串必须与视为匹配的值完全匹配。
 
**示例：**<br> `IIF(Contains([proxyAddresses],”SMTP:”)>0,[proxyAddresses],Error(“No primary SMTP address found.”))`<br>如果 proxyAddress 属性具有主电子邮件地址（由大写“SMTP:”表示)，则返回 proxyAddress 属性，否则返回错误。
 
 


----------
### <a name="convertfrombase64"></a>ConvertFromBase64

**说明：**<br>ConvertFromBase64 函数将指定的 base64 编码值转换为规则的字符串。
 
**语法：**<br> `str ConvertFromBase64(str source)` - 假定采用 Unicode 编码 <br> `str ConvertFromBase64(str source, enum Encoding)`

- source：Base64 编码的字符串<br>
- Encoding：Unicode、ASCII、UTF8

**示例**<br> `ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`<br> `ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

这两个示例均返回 "*Hello world!*"
 
 


----------
### <a name="convertfromutf8hex"></a>ConvertFromUTF8Hex

**说明：**<br>ConvertFromUTF8Hex 函数将指定的 UTF8 Hex 编码值转换为字符串。
 
**语法：**<br> `str ConvertFromUTF8Hex(str source)`

- source：UTF8 2 字节编码的字符串
 
**备注：**<br>该结果中此函数和 ConvertFromBase64 (,UTF8) 之间的差异对 DN 属性是友好的。<br> 此格式被 Azure Active Directory 用作 DN。
 
**示例：**<br> `ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`<br>返回 "*Hello world!*"
 
 


----------
### <a name="converttobase64"></a>ConvertToBase64

**说明：** <br>ConvertToBase64 函数将字符串转换为 Unicode base64 字符串。<br> 将整数数组的值转换为其等效字符串表示形式，该表示形式使用 base 64 数字编码。

**语法：** <br> `str ConvertToBase64(str source)`
 
**示例：** <br> `ConvertToBase64("Hello world!")` <br> 返回 "SABlAGwAbABvACAAdwBvAHIAbABkACEA"
 
 


----------
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex

**说明：**<br>ConvertToUTF8Hex 函数将字符串转换为 UTF8 Hex 编码的值。
 
**语法：**<br> `str ConvertToUTF8Hex(str source)`
 
**备注：**<br>此函数的输出格式被 Azure Active Directory 用作 DN 属性。
 
**示例：** <br> `ConvertToUTF8Hex("Hello world!")` <br>返回 48656C6C6F20776F726C6421
 
 


----------
### <a name="count"></a>计数

**说明：**<br>Count 函数返回多值属性中的元素数量
 
**语法：** <br> `num Count(mvstr attribute)`
 



----------
### <a name="cnum"></a>CNum

**说明：** <br>CNum 函数使用字符串并返回数值数据类型。
 
**语法：** <br> `num CNum(str value)`
 



----------
### <a name="cref"></a>CRef

**说明：** <br>将字符串转换为引用属性
 
**语法：** <br> `ref CRef(str value)`
 
**示例：** <br> `CRef(“CN=LC Services,CN=Microsoft,CN=lcspool01, CN=Pools,CN=RTC Service,” & %Forest.LDAP%)`
 
 


----------
### <a name="cstr"></a>CStr

**说明：** <br>CStr 函数转换为字符串数据类型。
 
**语法：** <br> `str CStr(num value)` <br> `str CStr(ref value)` <br> `str CStr(bool value)` <br>

- value：可以是数字值、引用属性或布尔值。 
 
**示例：** <br> `CStr([dn]) <br>`可能返回 “cn=Joe,dc=contoso,dc=com”
 
 


----------
### <a name="dateadd"></a>DateAdd

**说明：** <br>返回包含指定时间间隔已添加到其中的日期的日期。
 
**语法：** <br> `dt DateAdd(str interval, num value, dt date)`

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
 
**示例：** <br> `DateAdd("m", 3, CDate("2001-01-01"))` <br>添加 3 个月，并返回表示“2001-04-01”的 DateTime
 
 


----------
### <a name="datefromnum"></a>DateFromNum

**说明：** <br>DateFromNum 函数将 AD 的日期格式的值转换为 DateTime 类型。
 
**语法：** <br> `dt DateFromNum(num value)`
 
**示例：** <br> `DateFromNum([lastLogonTimestamp])` <br> `DateFromNum(129699324000000000)` <br>返回表示 2012-01-01 23:00:00 的 DateTime
 



----------
### <a name="dncomponent"></a>DNComponent

**说明：** <br>DNComponent 函数返回从左边起的指定 DN 组件的值。
 
**语法：** <br> `str DNComponent(ref dn, num ComponentNumber)`

- dn：要解释的引用属性
- ComponentNumber：要返回的 DN 中的组件
 
**示例：** <br> `DNComponent([dn],1)` <br> 如果 dn 为“cn=Joe,ou=…”，则返回 Joe
 



----------
### <a name="dncomponentrev"></a>DNComponentRev

**说明：** <br>DNComponentRev 函数返回从右边起（末尾）的指定 DN 组件的值。
 
**语法：** <br> `str DNComponentRev(ref dn, num ComponentNumber)` <br> `str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

- dn：要解释的引用属性
- ComponentNumber - 要返回的 DN 中的组件
- Options：DC – 忽略具有“dc=”的所有组件
 
**示例：** <br> `If dn is “cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com” then DNComponentRev([dn],3)` <br> `DNComponentRev([dn],1,”DC”)` <br>两者都返回 US。
 
 


----------
### <a name="error"></a>错误

**说明：** <br>Error 函数用于返回自定义错误。
 
**语法：** <br> `void Error(str ErrorMessage)`
 
**示例：** <br> `IIF(IsPresent([accountName]),[accountName],Error(“AccountName is required”))` <br>如果属性 accountName 不存在，则对象上引发错误。
 
 


----------
### <a name="escapedncomponent"></a>EscapeDNComponent

**说明：** <br>EscapeDNComponent 函数使用 DN 的一个组件，并对其进行转义，以便它可以在 LDAP 中表示。
 
**语法：** <br> `str EscapeDNComponent(str value)`
 
**示例：** <br> `EscapeDNComponent(“cn=” & [displayName]) & “,” & %ForestLDAP%` <br>即使 displayName 属性具有必须在 LDAP 中转义的字符，请确保可以在 LDAP 目录中创建对象。
 
 


----------
### <a name="formatdatetime"></a>FormatDateTime

**说明：** <br>FormatDateTime 函数用于为具有指定格式的字符串设置 DateTime 格式
 
**语法：** <br> `str FormatDateTime(dt value, str format)`

- value：DateTime 格式的值 <br>
- format：表示要转换为的格式的字符串。
 
**备注：** <br>格式的可能值可以在此处找到：[用户定义的日期/时间格式（格式函数）](http://msdn2.microsoft.com/library/73ctwf33(VS.90).aspx)
 
**示例：** <br>
 
`FormatDateTime(CDate(“12/25/2007”),”yyyy-mm-dd”)` <br> 结果是“2007-12-25”。

`FormatDateTime(DateFromNum([pwdLastSet]),”yyyyMMddHHmmss.0Z”)` <br> 结果可能是“20140905081453.0Z”
 
 


----------
### <a name="guid"></a>GUID

**说明：** <br>函数 GUID 生成新的随机 GUID
 
**语法：** <br> `str GUID()`
 



----------
### <a name="iif"></a>IIF

**说明：** <br>IIF 函数基于指定的条件返回一组可能值中的其中一个值。
 
**语法：** <br> `var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

- condition：计算结果可能为 true 或 false 的任何值或表达式。
- valueIfTrue：如果条件计算结果为 true，则返回的值。
- valueIfFalse：如果条件计算结果为 false，则返回的值。

**示例：** <br> `IIF([employeeType]=“Intern”,”t-“&[alias],[alias])` <br>如果用户是实习生，则返回用户的别名，同时将“t-”添加到其开头，否则按原样返回用户的别名。
 
 


----------
### <a name="instr"></a>InStr

**说明：** <br>InStr 函数查找字符串中第一次出现的子字符串
 
**语法：** <br>
 
`num InStr(str stringcheck, str stringmatch)` <br> `num InStr(str stringcheck, str stringmatch, num start)` <br> `num InStr(str stringcheck, str stringmatch, num start , enum compare)`

- stringcheck：要搜索的字符串 <br>
- stringmatch：要查找的字符串 <br>
- start：查找子字符串的起始位置 <br>
- compare：vbTextCompare 或 vbBinaryCompare
 
**备注：** <br>返回其中已找到子字符串的位置，如果未找到，则返回 0。

**示例：** <br> `InStr("The quick brown fox","quick")` <br>计算结果为 5

`InStr("repEated","e",3,vbBinaryCompare)` <br>计算结果为 7
 
 


----------
### <a name="instrrev"></a>InStrRev

**说明：** <br>InStrRev 函数查找字符串中最后一次出现的子字符串
 
**语法：** <br> `num InstrRev(str stringcheck, str stringmatch)` <br> `num InstrRev(str stringcheck, str stringmatch, num start)` <br> `num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

- stringcheck：要搜索的字符串 <br>
- stringmatch：要查找的字符串 <br>
- start：查找子字符串的起始位置 <br>
- compare：vbTextCompare 或 vbBinaryCompare

**备注：** <br>返回其中已找到子字符串的位置，如果未找到，则返回 0。

**示例：** <br> `InStrRev("abbcdbbbef","bb")` <br>返回 7
 
 


----------
### <a name="isbitset"></a>IsBitSet

**说明：** <br>函数 IsBitSet 测试是否设置了位
 
**语法：** <br> `bool IsBitSet(num value, num flag)`

- value：计算的数字值。标志：表示具有要计算的位的数字值
 
**示例：** <br> `IsBitSet(&HF,4)` <br>返回 True，因为位“4”在十六进制值“F”中设置
 



----------
### <a name="isdate"></a>IsDate

**说明：** <br>如果表达式可以计算为 DateTime 类型，则 IsDate 函数计算结果为 True。
 
**语法：** <br> `bool IsDate(var Expression)`
 
**备注：** <br>用来确定 CDate() 是否会成功。
 



----------
### <a name="isempty"></a>IsEmpty

**说明：** <br>如果属性是出现在 CS 或 MV 中，但计算结果为空字符串，则 IsEmpty 函数计算结果为 True。
 
**语法：** <br> `bool IsEmpty(var Expression)`
 



----------
### <a name="isguid"></a>IsGuid

**说明：** <br>如果字符串可以转换为 GUID，则 IsGuid 函数计算结果为 true。
 
**语法：** <br> `bool IsGuid(str GUID)`
 
**备注：** <br>GUID 定义为遵循以下其中一种模式的字符串：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

用来确定 CGuid() 是否会成功。
 
**示例：** <br> `IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)` <br>如果 StrAttribute 具有 GUID 格式，则返回二进制表示形式，否则返回 Null。
 
 


----------
### <a name="isnull"></a>IsNull

**说明：** <br>如果表达式的计算结果为 Null，则 IsNull 函数返回 true。
 
**语法：** <br> `bool IsNull(var Expression)`
 
**备注：** <br>对于属性，Null 表示缺少属性。
 
**示例：** <br> `IsNull([displayName])` <br>如果属性没有在 CS 或 MV 中出现，则返回 True。
 
 


----------
### <a name="bitand"></a>IsNullOrEmpty

**说明：** <br>如果表达式为 null 或空字符串，则 IsNullOrEmpty 函数返回 true。
 
**语法：** <br> `bool IsNullOrEmpty(var Expression)`
 
**备注：** <br>对于属性，如果属性不存在，或存在但为空字符串，此语法计算结果则为 True。<br> 此函数的逆函数被命名为 IsPresent。
 
**示例：** <br> `IsNull([displayName])` <br>如果属性在 CS 或 MV 中没有出现或为空字符串，则返回 True。
 
 


----------
### <a name="isnullorempty"></a>IsNumeric

**说明：** <br>IsNumeric 函数返回布尔值，该值指示表达式是否可以计算为数字类型。
 
**语法：** <br> `bool IsNumeric(var Expression)`
 
**备注：** <br>用来确定 CNum() 是否会成功地分析表达式。




----------
### <a name="isstring"></a>IsString

**说明：** <br>如果表达式可以计算为字符串类型，则 IsString 函数计算结果为 True。
 
**语法：** <br> `bool IsString(var expression)`
 
**备注：** <br>用来确定 CStr() 是否会成功地分析表达式。
 



----------
### <a name="ispresent"></a>IsPresent

**说明：** <br>如果表达式的计算结果为字符串，该字符串不是 Null 且不为空，则 IsPresent 函数返回 true。
 
**语法：** <br> `bool IsPresent(var expression)`
 
**备注：** <br>此函数的逆函数被命名为 IsNullOrEmpty。
 
**示例：** <br>
 
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`
  



----------
### <a name="item"></a>项目

**说明：** <br>Item 函数返回多值字符串/属性中的一个项。
 
**语法：** <br> `var Item(mvstr attribute, num index)`

- attribute：多值属性 <br>
- index：对多值字符串中某个项的索引。
 
**备注：** <br>Item 函数与 Contains 函数一起使用很有用，因为后者函数会返回对多值属性中某个项的索引。

如果索引超出界限，则引发错误。
 
**示例：** <br> `Mid(Item([proxyAddress],Contains([proxyAddress], ”SMTP:”)),6)` <br>返回主电子邮件地址。
 
 


----------
### <a name="itemornull"></a>ItemOrNull

**说明：** <br>ItemOrNull 函数返回多值字符串/属性中的一个项。
 
**语法：** <br> `var ItemOrNull(mvstr attribute, num index)`

- attribute：多值属性 <br>
- index：对多值字符串中某个项的索引。
 
**备注：** <br>ItemOrNull 函数与 Contains 函数一起使用很有用，因为后者函数会返回对多值属性中某个项的索引。

如果索引超出界限，则返回 Null 值。
 



----------
### <a name="join"></a>Join

**说明：** <br>Join 函数使用多值字符串，并返回每个项之间插入指定分隔符的单值字符串。
 
**语法：** <br> `str Join(mvstr attribute)` <br> `str Join(mvstr attribute, str Delimiter)`

- attribute：包含要联接的字符串的多值属性。 <br>
- delimiter：任意字符串，用于分隔返回的字符串中的子字符串。如果省略，则使用空格字符（“ ”）。如果分隔符为零长度字符串（“”）或零，则列表中的所有项都不使用分隔符连接。
 
**备注**<br>Join 和 Split 函数之间没有奇偶校验。Join 函数使用字符串数组，并使用分隔符字符串将它们联接起来，以返回单个字符串。Split 函数使用字符串并以分隔符分隔，以返回字符串数组。但是，主要区别是 Join 可以使用任何分隔符字符串将字符串连接起来，而 Split 仅可以使用单个字符分隔符分隔字符串。
 
**示例：** <br> `Join([proxyAddresses],”,”)` <br>可能返回：“SMTP:john.doe@contoso.com,smtp:jd@contoso.com”
 
 


----------
### <a name="lcase"></a>LCase

**说明：** <br>LCase 函数将字符串中的所有字符都转换为小写。
 
**语法：** <br> `str LCase(str value)`
 
**示例：** <br> `LCase(“TeSt”)` <br>返回 “test”。
 
 


----------
### <a name="left"></a>Left

**说明：** <br>Left 函数从字符串左侧起返回指定的字符数。
 
**语法：** <br> `str Left(str string, num NumChars)`

- string：从中返回字符的字符串 <br>
- NumChar：标识从字符串开头（左侧）起返回的字符数的数字
 
**备注：** <br>包含字符串中第一个 numChar 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的字符数比 numChar 中指定的数量少，则返回与该字符串相同的字符串（即：包含参数 1 中的所有字符）。
 
**示例：** <br> `Left(“John Doe”, 3)` <br>返回 “Joh”。
 



----------
### <a name="len"></a>Len

**说明：** <br>Len 函数返回字符串中的字符数。
 
**语法：** <br> `num Len(str value)`
 
**示例：** <br> `Len(“John Doe”)` <br>返回 8
 



----------
### <a name="ltrim"></a>LTrim

**说明：** <br>LTrim 函数从字符串中删除前导空格。
 
**语法：** <br> `str LTrim(str value)`
 
**示例：** <br> `LTrim(“ Test ”)` <br>返回 “Test”
 
 


----------
### <a name="mid"></a>Mid

**说明：** <br>Mid 函数从字符串指定位置起返回指定的字符数。
 
**语法：** <br> `str Mid(str string, num start, num NumChars)`

- string：从中返回字符的字符串 <br>
- start：标识从中返回字符的字符串中起始位置的数字
- NumChar：标识从字符串中的位置返回的字符数的数字
 

**备注：** <br>从字符串中的开始位置开始返回 numChar 字符。<br> 包含字符串中开始位置的 numChar 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果 start > 字符串的长度，则返回输入字符串。
- 如果 start <= 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串中开始位置没有保留的 numChar 字符，则会返回尽可能多可以返回的字符。
 
**示例：** <br>
 
`Mid(“John Doe”, 3, 5)` <br>返回 “hn Do”。

`Mid(“John Doe”, 6, 999)` <br>返回 “Doe”。
 
 


----------
### <a name="now"></a>Now

**说明：** <br>Now 函数根据计算机的系统日期和时间返回指定当前日期和时间的 DateTime。
 
**语法：** <br> `dt Now()`
 



----------
### <a name="numfromdate"></a>NumFromDate

**说明：** <br>NumFromDate 函数返回 AD 的日期格式的日期。
 
**语法：** <br> `num NumFromDate(dt value)`
 

**示例：** <br> `NumFromDate(CDate("2012-01-01 23:00:00"))` <br>返回 129699324000000000
 
 


----------
### <a name="padleft"></a>PadLeft

**说明：** <br>PadLeft 函数使用提供的填充字符将字符串从左侧填充到指定长度。
 
**语法：** <br> `str PadLeft(str string, num length, str padCharacter)`

- string：要填充的字符串。 <br>
- length：表示所需字符串长度的整数。 <br>
- padCharacter：包含用作填充字符的单个字符的字符串
 


----------
### 备注 
 
- 如果字符串的长度小于 length，则 padCharacter 会重复追加到字符串的开头（左侧），直到其长度等于 length。
- PadCharacter 可以是空格字符，但不能为 null 值。
- 如果字符串的长度等于或大于 length，则返回不变的字符串。
- 如果字符串的长度大于或等于 length，则返回与 string 相同的字符串。
- 如果字符串的长度小于 length，则返回具有所需长度的新字符串，其中包含用 padCharacter 填充的字符串。
- 如果字符串为 null，该函数则返回空字符串。

**示例：** <br> `PadLeft(“User”, 10, “0”)` <br>返回 “000000User”。
 
 


----------
### <a name="padright"></a>PadRight

**说明：** <br>PadRight 函数使用提供的填充字符将字符串从右侧填充到指定长度。
 
**语法：** <br> `str PadRight(str string, num length, str padCharacter)`

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
 

**示例：** <br> `PadRight(“User”, 10, “0”)` <br>返回 “User000000”。
 
 


----------
### <a name="pcase"></a>PCase

**说明：** <br>PCase 函数将字符串中每个空格分隔词的第一个字符转换为大写形式，并将所有其他字符都转换为小写形式。
 
**语法：** <br> `String PCase(string)`
 
**示例：** <br> `PCase(“TEsT”)` <br>返回 “Test”。
 
 


----------
### <a name="randomnum"></a>RandomNum

**说明：** <br>RandomNum 函数返回指定间隔之间的随机数字。
 
**语法：** <br> `num RandomNum(num start, num end)`

- start：标识要生成的随机值的下限的数字 <br>
- end：标识要生成的随机值的上限的数字
 
**示例：** <br> `Random(100,999)` <br>返回 734。
 
 


----------
### <a name="removeduplicates"></a>RemoveDuplicates

**说明：** <br>RemoveDuplicates 函数使用多值字符串，并确保每个值都是唯一值。
 
**语法：** <br> `mvstr RemoveDuplicates(mvstr attribute)`
 
**示例：** <br> `RemoveDuplicates([proxyAddresses])` <br>返回净化的 proxyAddress 属性，其中所有重复值已被删除。
 
 


----------
### <a name="replace"></a>Replace

**说明：** <br>Replace 函数将所有出现的某一字符串替换为另一个字符串。
 
**语法：** <br> `str Replace(str string, str OldValue, str NewValue)`

- string：替换其中的值的字符串。 <br>
- OldValue：要搜索和替换的字符串。 <br>
- NewValue：要替换的字符串。
 

**备注：** <br>该函数可以识别以下特殊 moniker:

- \\n – 新行 
- \\r – 回车符
- \\t – 选项卡
 

**示例：** <br>
 
`Replace([address],”\r\n”,”, “)` <br>将 CRLF 替换为逗号和空格，可能导致出现“One Microsoft Way, Redmond, WA, USA”
 
 


----------
### <a name="replacechars"></a>ReplaceChars

**说明：** <br>ReplaceChars 函数替换 ReplacePattern 字符串中找到的所有出现的字符。

**语法：** <br> `str ReplaceChars(str string, str ReplacePattern)`

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
 

**示例：** <br>'%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o'

`ReplaceChars(”Räksmörgås”,%ReplaceString%)` <br> 返回 Raksmorgas

`ReplaceChars(“O’Neil”,%ReplaceString%)` <br> 返回 “ONeil”，定义要删除单次勾选。
 



----------
### <a name="right"></a>Right

**说明：** <br>Right 函数从字符串右侧（末尾）起返回指定的字符数。
 
**语法：** <br> `str Right(str string, num NumChars)`

- string：从中返回字符的字符串 
- NumChar：标识从字符串末尾（右侧）起返回的字符数的数字
 
**备注：** <br>NumChar 字符从字符串的最后位置返回。

包含字符串中最后的 numChar 字符的字符串：

- 如果 numChar = 0，则返回空字符串。
- 如果 numChar < 0，则返回输入字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的字符数比 NumChar 中指定的数量少，则返回与该字符串相同的字符串。

**示例：** <br> `Right(“John Doe”, 3)` <br>返回 “Doe”。
 



----------
### <a name="rtrim"></a>RTrim

**说明：** <br>RTrim 函数从字符串中删除尾随空格。
 
**语法：** <br> `str RTrim(str value)`

**示例：** <br> `RTrim(“ Test ”)` <br>返回 “Test”。




----------
### <a name="split"></a>拆分

**说明：** <br>Split 函数使用采用分隔符分隔的字符串，并使其成为多值字符串。
 

**语法：** <br> `mvstr Split(str value, str delimiter)` <br? `mvstr Split(str value, str delimiter, num limit)`

- value：用分隔符字符来分隔的字符串。
- delimiter：用作分隔符的单个字符。 
- limit：将返回的最大数目的值。
 
**示例：** <br> `Split(“SMTP:john.doe@contoso.com,smtp:jd@contoso.com”,”,”)` <br>返回多值字符串，其中两个元素对 proxyAddress 属性有用
 



----------
### <a name="StringFromGuid"></a>StringFromGuid

**说明：** <br>StringFromGuid 函数使用二进制 GUID，并将其转换为字符串
 
**语法：** <br> `str StringFromGuid(bin GUID)`
 



----------
### <a name="stringfromsid"></a>StringFromSid

**说明：** <br>StringFromSid 函数将字节数组或包含安全标识符的多值字节数组转换为字符串或多值字符串。
 
**语法：** <br> `str StringFromSid(bin ObjectSID)` <br> `mvstr StringFromSid(mvbin ObjectSID)`
 



----------
### <a name="switch"></a>Switch

**说明：** <br>Switch 函数用于基于计算的条件返回单个值。

**语法：** <br> `var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

- expr：想要计算结果的变体表达式。 
- value：当相应表达式为 True 时要返回的值。
 
**备注：** <br>Switch 函数参数列表包含表达式和值对。表达式从左到右计算结果，并返回与计算结果为 True 的第一个表达式相关联的值。如果没有正确配对部件，则会发生运行时错误。

例如，如果 expr1 为 True，则 Switch 返回 value1。如果 expr-1 为 False，但 expr-2 为 True，则 Switch 返回 value-2，依此类推。

如果表达式均不为 True，第一个 True 表达式中相应值为 Null，则 Switch 返回零。

Switch 会对所有表达式计算结果，即使它只返回其中一个结果。为此，你应监视非预期的负面影响。例如，如果任何表达式的计算结果导致除数为零的错误，则会出现错误。

值还可以是将返回自定义字符串的错误函数。

**示例：** <br> `Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error(“Unknown city”))` <br>返回某些主要城市所使用的语言，否则返回错误。




----------
### <a name="trim"></a>Trim

**说明：** <br>Trim 函数从字符串中删除前导空格和尾随空格。
 
**语法：** <br> `str Trim(str value)` <br> `mvstr Trim(mvstr value)`
 
**示例：** <br> `Trim(“ Test ”)` <br>返回 “Test”。

`Trim([proxyAddresses])` <br>删除 proxyAddress 属性中每个值的前导空格和尾随空格。




----------
### <a name="ucase"></a>UCase

**说明：** <br>UCase 函数将字符串中的所有字符都转换为大写形式。

**语法：** <br> `str UCase(str string)`
 
**示例：** <br> `UCase(“TeSt”)` <br>返回 “TEST”。
 
 


----------
### <a name="word"></a>Word

**说明：** <br>基于描述要使用的分隔符与要返回的单词数的参数，Word 函数返回字符串中包含的单词。
 
**语法：** <br> `str Word(str string, num WordNumber, str delimiters)`

- string：从中返回单词的字符串。
- WordNumber：标识应返回单词数的数字。 
- delimiter：表示应该用于标识单词的分隔符的字符串
 
**备注：** <br>字符串中的字符由分隔符中其中一个字符分隔的每个字符串被标识为单词：

- 如果数字 < 1，则返回空字符串。
- 如果字符串为 null，则返回空字符串。

如果字符串包含的单词少于应返回数字或字符串不包含由分隔符标识的任何单词，则返回空字符串。
 

**示例：** <br> `Word(“The quick brown fox”,3,” “)` <br>返回 “brown”。

`Word(“This,string!has&many seperators”,3,”,!&#”)` <br>会返回 “has”


## 其他资源

* [了解声明性设置表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions)
* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=Mooncake_0411_2016-->