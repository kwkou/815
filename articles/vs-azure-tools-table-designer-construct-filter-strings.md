<properties
   pageTitle="为表设计器构造筛选字符串 | Azure"
   description="为表设计器构造筛选字符串"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />
<tags
   ms.service="storage"
   ms.date="04/18/2016"
   wacn.date="05/23/2016" />

# 为表设计器构造筛选字符串

## 概述

若要筛选 Visual Studio **表设计器**中显示在 Azure 表中的数据，可以构造一个筛选器字符串并将其输入到筛选器字段中。筛选器字符串语法由 WCF 数据服务进行定义，与 SQL WHERE 子句类似，但通过 HTTP 请求发送给表服务。**表设计器**会为你处理正确的编码以便筛选所需的属性值，你只需要在筛选器字段中输入属性名、比较运算符、条件值以及可选的布尔运算符。无需像构造通过[存储空间服务 REST API 参考](https://msdn.microsoft.com/zh-cn/library/dd179355.aspx)来查询表的 URL 那样包括 $filter 查询选项。

WCF 数据服务基于[开放数据协议](http://go.microsoft.com/fwlink/p/?LinkId=214805) (OData)。有关筛选系统查询选项 (**$filter**) 的详细信息，请参阅 [OData URI 约定规范](http://go.microsoft.com/fwlink/p/?LinkId=214806)。

## 比较运算符

所有属性类型都支持以下逻辑运算符：

|逻辑运算符|说明|示例筛选器字符串|
|---|---|---|
|eq|等于|City eq 'Redmond'|
|gt|大于|Price gt 20|
|ge|大于或等于|Price ge 10|
|lt|小于|Price lt 20|
|le|小于或等于|Price le 100|
|ne|不等于|City ne 'London'|
|和|And|Price le 200 and Price gt 3.5|
|或|或|Price le 3.5 or Price gt 200|
|not|Not|not isAvailable|

构造筛选器字符串时，以下规则非常重要：

- 使用逻辑运算符可将属性与值进行比较。请注意不能比较属性和动态值；表达式的一侧必须是常量。

- 筛选器字符串的所有部分都区分大小写。

- 常量值的数据类型必须与属性的类型相同，这样筛选器才能返回有效的结果。有关支持的属性类型的详细信息，请参阅 [了解表服务数据模型](https://msdn.microsoft.com/zh-cn/library/dd179338.aspx)。

## 针对字符串属性进行筛选

当对字符串属性进行筛选时，用单引号将字符串常量括起来。

以下示例对 **PartitionKey** 和 **RowKey** 属性进行筛选；也可以将其他非键属性添加到筛选器字符串中：

    PartitionKey eq 'Partition1' and RowKey eq '00001'

可以用圆括号将每个筛选器表达式括起来，但这不是必需的：

    (PartitionKey eq 'Partition1') and (RowKey eq '00001')

请注意，表服务不支持通配符查询，并且表设计器中也不支持这些查询。但是，可以通过对所需前缀使用比较运算符来执行前缀匹配。下面的示例返回 LastName 属性以字母“A”开头的实体：

    LastName ge 'A' and LastName lt 'B'

## 针对数值属性进行筛选

若要对整数或浮点数进行筛选，请指定不带引号的数字。

此示例将返回 Age 属性值大于 30 的所有实体：

    Age gt 30

此示例将返回 AmountDue 属性值小于或等于 100.25 的所有实体：

    AmountDue le 100.25

## 针对布尔值属性进行筛选

若要对布尔值进行筛选，请指定 **true** 或 **false**（不带引号）。

以下示例将返回 IsActive 属性设置为 **true** 的所有实体：

    IsActive eq true

你也可以在不使用逻辑运算符的情况下编写此筛选器表达式。在以下示例中，表服务还将返回 IsActive 为 **true** 的所有实体：

    IsActive

若要返回 IsActive 为 false 的所有实体，可以使用 not 运算符：

    not IsActive

## 针对日期时间属性进行筛选

若要对日期时间值进行筛选，请指定 **datetime** 关键字，后接单引号括起来的日期/时间常量。日期/时间常量必须采用组合的 UTC 格式，如 [设置 DateTime 属性值格式](https://msdn.microsoft.com/zh-cn/library/azure/dd894027.aspx)中所述。

以下示例将返回 CustomerSince 属性等于 2008-07-10 的实体：

    CustomerSince eq datetime'2008-07-10T00:00:00Z'

<!---HONumber=Mooncake_0503_2016-->