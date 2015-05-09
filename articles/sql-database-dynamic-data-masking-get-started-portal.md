<properties 
   pageTitle="SQL Database 动态数据屏蔽（Azure 门户）入门" 
   description="如何开始在 Azure 门户中使用 SQL Database 动态数据屏蔽" 
   services="sql-database" 
   documentationCenter="" 
   authors="nadavhelfman" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="03/25/2015"
   wacn.date="04/13/2015"
   />

# SQL Database 动态数据屏蔽（Azure 门户）入门

<!--
> [AZURE.SELECTOR]
- [Dynamic Data Masking - Azure Preview portal](/documentation/articles/sql-database-dynamic-data-masking-get-started)
-->

## 概述

SQL Database 动态数据屏蔽通过向无特权用户屏蔽敏感数据来控制此类数据的透漏。在 Azure SQL Database V12 版本的基本、标准和高级服务层中，动态数据屏蔽以预览版提供。

动态数据屏蔽允许客户指定在对应用层产生最小影响的前提下可以透露的敏感数据量，从而帮助防止未经授权的用户访问敏感数据。它是一种基于策略的安全功能，会在针对指定的数据库字段运行查询后返回的结果集中隐藏敏感数据，同时保持数据库中的数据不变。

例如，呼叫中心支持人员可以根据呼叫者的身份证号或信用卡号的多个数字来识别其身份，但这些数据项不应完全透露给支持人员。开发人员可以定义要应用到每个查询结果的屏蔽规则，该规则可以屏蔽结果集中任何身份证号或信用卡号的除最后四位数以外的其他所有数字。另举一例，通过使用适当的数据屏蔽来保护个人身份信息 (PII) 数据，开发人员一方面可以查询生产环境以进行故障排除，同时又不违反法规遵从性要求。

## SQL Database 动态数据屏蔽基础知识

可以在 Azure 门户中设置动态数据屏蔽策略，并使用应用程序或访问数据库的其他客户端所用的已启用安全性的连接字符串完成设置。


### 动态数据屏蔽权限

Azure 数据库管理员、服务器管理员或安全主管角色可以配置动态数据屏蔽。

### 动态数据屏蔽策略

* **特权登录名** - 可在 SQL 查询结果中获取非屏蔽数据的一组登录名。
  
* **屏蔽规则** - 一组规则，定义将要屏蔽的指定字段，以及要使用的屏蔽函数。可以使用数据库表名称和列名称或使用别名定义指定的字段。

* **屏蔽依据** - 可按源或目标屏蔽。可以通过标识**表**名称和**列**名称在源级别配置屏蔽，或者通过标识查询中使用的**别名**在结果级别配置屏蔽。如果你熟悉数据库的数据体系结构并想要限制所有查询结果的透露，你可能会偏向于使用源屏蔽规则。如果你想要限制查询结果的透露，且不分析数据库数据体系结构或者可能来自不同源的字段，则你可以添加结果屏蔽规则。  
  
* **扩展限制** - 限制敏感数据的透露，但可能会对某些应用程序的功能产生负面影响。

>[AZURE.TIP] 使用**"扩展限制"**选项可以限制直接访问数据库并运行 SQL 查询以进行故障排除的开发人员的访问权限。

* **屏蔽函数** - 一组方法，用于控制不同情况下的数据透露。

| 屏蔽函数 | 屏蔽逻辑 |
|----------|---------------|
| **默认** |**根据指定字段的数据类型完全屏蔽**<br/><br/>• 对于字符串数据类型（nchar、ntext、nvarchar），将使用 XXXXXXXX；如果字段大小小于 8 个字符，则使用更少的 X。<br/>• 对于数字数据类型（bigint、bit、decimal、int、money、numeric、smallint、smallmoney、tinyint、float、real），将使用零值。<br/>• 对于日期/时间数据类型（date、datetime2、datetime、datetimeoffset、smalldatetime、time），将使用当前时间。<br/>• 对于 SQL 变量，将使用当前类型的默认值。<br/>• 对于 XML，将使用文档 <masked/>。<br/>• 对于特殊数据类型（timestamp、table、hierarchyid、GUID、binary、image、varbinary 空间类型），将使用空值。
| **信用卡** |**公开指定字段的最后四位数，并添加一个信用卡格式的常量字符串作为前缀的屏蔽方法**。<br/><br/>XXXX-XXXX-XXXX-1234|
| **身份证号** |**公开指定字段的最后两位数，并添加一个美国身份证号格式的常量字符串作为前缀的屏蔽方法**。<br/><br/>XXX-XX-XX12 |
| **电子邮件** | **公开第一个字母和域，并使用一个电子邮件地址格式的常量字符串作为前缀的屏蔽方法**。<br/><br/>aXX@XXXX.com |
| **随机数** | **根据选定边界和实际数据类型生成随机数的屏蔽方法**。如果指定的边界相等，则屏蔽函数将是常数。<br/><br/>![Navigation pane][Image1] |
| **自定义文本** | **公开第一个和最后一个字母，并在中间添加一个自定义填充字符串的屏蔽方法**。<br/>前缀[填充字符]后缀<br/><br/>![Navigation pane][Image2] |

  
<a name="Anchor1"></a>
### 已启用安全性的连接字符串

当你设置动态数据屏蔽时，Azure 将会提供数据库的已启用安全性的连接字符串。只会根据动态数据屏蔽策略屏蔽使用此连接字符串的客户端的敏感数据。你还需要更新现有的客户端（例如：应用程序），以使用新的连接字符串格式。

* 原始连接字符串格式：<*服务器名称*>.database.chinacloudapi.cn
* 已启用安全性的连接字符串：<*服务器名称*>.database.**secure**.chinacloudapi.cn

你还可以将"已启用安全性的访问"设置从"可选"更改为"必需"，这可确保用户无法使用原始连接字符串访问数据库和忽略动态数据屏蔽策略。在使用特定的客户端（例如，开发阶段的应用程序或 SSMS）体验动态数据屏蔽时，可以选择"可选"。对于生产环境，请选择"必需"。<br/><br/>

![Navigation pane][Image3]<br/><br/>

## 使用 Azure 门户为数据库设置动态数据屏蔽

1. 启动 Azure 门户 ([https://manage.windowsazure.cn](https://manage.windowsazure.cn))。

2. 单击要屏蔽的数据库，然后单击"审核和安全性"选项卡。

3. 在"动态数据屏蔽"下，单击"已启用"以启用动态数据屏蔽功能。  

4. 键入有权访问非屏蔽敏感数据的特权登录名。

	>[AZURE.TIP] 要使应用程序层可以向应用程序特权用户显示敏感数据，请添加用于查询的应用程序登录名和数据库。强烈建议在此列表中包含最少量的登录名，以最大程度地降低透露敏感数据的风险。

	![Navigation pane][Image4]

5. 在页面底部的菜单栏中，单击"添加屏蔽"打开屏蔽规则配置窗口。

6. 选择"屏蔽依据"以指定是要在源级别还是目标级别进行屏蔽。可以通过标识**表**名称和**列**名称在源级别配置屏蔽，或者通过标识查询中使用的**别名**在结果级别配置屏蔽。如果你熟悉数据库的数据体系结构并想要限制所有查询结果的透露，你可能会偏向于使用源屏蔽规则。如果你想要限制查询结果的透露，且不分析数据库数据体系结构或者可能来自不同源的字段，则你可以添加结果屏蔽规则。

7. 键入"表名称"和"列名称"或"别名"，以定义要屏蔽的指定字段。

8. 从敏感数据屏蔽类别列表中选择"屏蔽函数"。

	![Navigation pane][Image5] 
 	
9. 在数据屏蔽规则窗口中单击"更新"，以更新动态数据屏蔽策略中的屏蔽规则集。

10. 请考虑选择"使用扩展限制"，这可以限制通过即席查询透露敏感数据。

11. 单击"保存"以保存新的或更新的屏蔽规则。

## 使用 REST API 为数据库设置动态数据屏蔽

请参阅[对 Azure SQL 数据库的操作](https://msdn.microsoft.com/zh-CN/library/dn505719.aspx)。

[Image1]: ./media/sql-database-dynamic-data-masking-get-started-portal/1_DDM_Random_number.png
[Image2]: ./media/sql-database-dynamic-data-masking-get-started-portal/2_DDM_Custom_text.png
[Image3]: ./media/sql-database-dynamic-data-masking-get-started-portal/3_DDM_Current_Preview.png
[Image4]: ./media/sql-database-dynamic-data-masking-get-started-portal/4_DMM_Policy_Classic_Portal.png
[Image5]: ./media/sql-database-dynamic-data-masking-get-started-portal/5_DDM_Add_Masking_Rule_Classic_Portal.png

<!--HONumber=51-->