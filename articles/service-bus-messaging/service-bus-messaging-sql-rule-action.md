<properties
    pageTitle="SQLRuleAction 语法参考 | Azure"
    description="有关 SQLRuleAction 语法的详细信息。"
    services="service-bus-messaging"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="" 
    ms.service="service-bus-messaging"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/22/2016"
    wacn.date="01/23/2017"
    ms.author="sethm" />  


# SQLRuleAction 语法

*SqlRuleAction* 是 SqlRuleAction 类的实例，表示使用基于 SQL 语言的语法编写的、针对 BrokeredMessage 执行的操作集。
  
本主题列出有关 SQL 规则操作语法的详细信息。
  
  
		<statements> ::=
		    <statement> [, ...n]  
  
  
  
  
		<statement> ::=
		    <action> [;]
		    Remarks
		    -------
		    Semicolon is optional.  
  
  
  
  
		<action> ::=
		    SET <property> = <expression>
		    REMOVE <property>  



		<expression> ::=
		    <constant>
		    | <function>
		    | <property>
		    | <expression> { + | - | * | / | % } <expression>
		    | { + | - } <expression>
		    | ( <expression> )
 


		<property> := 
		    [<scope> .] <property_name>
 
  
## 参数  
  
-   `<scope>` 是可选字符串，指示 `<property_name>` 的作用域。有效值为 `sys` 或 `user`。`sys` 值指示系统作用域，其中 `<property_name>` 是 BrokeredMessage 类的公共属性名称。`user` 指示用户作用域，其中 `<property_name>` 是 BrokeredMessage 类字典的键。如果未指定 `<scope>`，则 `user` 作用域是默认作用域。
  
### 备注  

访问不存在的系统属性的尝试是错误，访问不存在的用户属性的尝试不是错误。相反，不存在的用户属性在内部作为未知值进行求值。运算符求值期间会对未知值进行特殊处理。
  
## property\_name  
  
  
		<property_name> ::=  
		     <identifier>  
		     | <delimited_identifier>  
  
		<identifier> ::=  
		     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
  
  
### 参数  
 `<regular_identifier>` 是字符串，由以下正则表达式表示：
  
  
		[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
  
  
 这是指以字母开头且后跟一个或多个下划线/字母/数字的任何字符串。
  
 `[:IsLetter:]` 是指分类为 Unicode 字母的任何 Unicode 字符。如果 `c` 是 Unicode 字母，`System.Char.IsLetter(c)` 返回 `true`。
  
 `[:IsDigit:]` 是指分类为十进制数字的任何 Unicode 字符。如果 `c` 是 Unicode 数字，`System.Char.IsDigit(c)` 返回 `true`。
  
 `<regular_identifier>` 不能是保留关键字。
  
 `<delimited_identifier>` 是指使用左/右方括号 ([]) 括起来的任何字符串。右方括号以两个右方括号表示。下面是 `<delimited_identifier>` 的示例：
  
  
		[Property With Space]  
		[HR-EmployeeID]  
  
  
  
 `<quoted_identifier>` 是指使用双引号引起来的任何字符串。标识符中的双引号以两个双引号表示。不建议使用带引号的标识符，因为很容易将其与字符串常量混淆。如果可能，请使用分隔标识符。下面是 `<quoted_identifier>` 的示例：
  
  
		"Contoso & Northwind"  
  
  
## pattern  
  
  
		<pattern> ::=  
		      <expression>  
  
  
### 备注
  
 `<pattern>` 必须是作为字符串进行求值的表达式。它用作 LIKE 运算符的模式。它可以包含以下通配符：
  
-   `%`：具有零个或多个字符的任何字符串。
  
-   `_`：任何单个字符。
  
## escape\_char  
  
  
		<escape_char> ::=  
		      <expression>  
  
  
### 备注
  
 `<escape_char>` 必须是作为长度为 1 的字符串进行求值的表达式。它用作 LIKE 运算符的转义符。
  
 例如，`property LIKE 'ABC\%' ESCAPE '\'` 匹配 `ABC%`，而不匹配以 `ABC` 开头的字符串。
  
## constant  
  
  
		<constant> ::=  
		      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
  
  
### 参数  
  
-   `<integer_constant>` 是指不使用引号引起来且不包含小数点的数字字符串。这些值作为 `System.Int64` 在内部存储，并具有相同的作用域。
  
     下面是长整数常量的示例：
  
      
		1894  
		2  
      
  
-   `<decimal_constant>` 是指不使用引号引起来但包含小数点的数字字符串。这些值作为 `System.Double` 在内部存储，并具有相同的作用域/精度。
  
     在将来的版本中，此数字可能会作为其他数据类型存储，以支持精确数字语义，因此，不应依赖于基础数据类型是 `<decimal_constant>` 的 `System.Double` 这一事实。
  
     下面是十进制常量的示例：
  
      
		1894.1204  
		2.0  
      
  
-   `<approximate_number_constant>` 是指使用科学记数法书写的数字。这些值作为 `System.Double` 在内部存储，并具有相同的作用域/精度。下面是近似数常量的示例：
  
      
		101.5E5  
		0.5E-2  
      
  
## boolean\_constant  
  
  
		<boolean_constant> :=  
		      TRUE | FALSE  
  
  
### 备注
  
布尔常量由关键字 `TRUE` 或 `FALSE` 表示。这些值作为 `System.Boolean` 存储。
  
## string\_constant  
  
  
		<string_constant>  
  
  
### 备注
  
字符串常量使用单引号引起来，并包含任何有效的 Unicode 字符。字符串常量中嵌入的单引号以两个单引号表示。
  
## function  
  
  
		<function> :=  
		      newid() |  
		      property(name) | p(name)  
  
  
### 备注  

`newid()` 函数返回由 `System.Guid.NewGuid()` 方法生成的 **System.Guid**。
  
`property(name)` 函数返回 `name` 引用的属性的值。`name` 值可以是返回字符串值的任何有效表达式。
  
## 注意事项

- SET 用于创建新属性或更新现有属性的值。
- REMOVE 用于删除属性。
- 表达式类型和现有属性类型不同时，SET 会在可能的情况下执行隐式转换。
- 如果引用不存在的系统属性，操作会失败。
- 如果引用不存在的用户属性，操作不会失败。
- 不存在的用户属性在内部作为“未知”进行求值，对运算符进行求值时遵循与 SQLFilter 相同的语义。


<!---HONumber=Mooncake_0116_2017-->