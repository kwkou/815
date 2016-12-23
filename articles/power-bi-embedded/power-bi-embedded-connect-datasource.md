<properties
   pageTitle="Microsoft Power BI Embedded - 连接到数据源"
   description="Power BI Embedded, 连接到数据源"
   services="power-bi-embedded"
   documentationCenter=""
   authors="mgblythe"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="powerbi"
   ms.date="07/05/2016"
   ms.author="mblythe"
   wacn.date=""/>  


# 连接到数据源

使用 **Power BI Embedded**，可以在自己的应用中嵌入报表。当在应用中嵌入 Power BI 报表时，可以通过**导入**数据的副本或通过使用 **DirectQuery** **直接连接**到数据源将该报表连接到基础数据。

以下是使用**导入**和 **DirectQuery** 之间的区别。

|导入 | DirectQuery
|---|---
|将表、列*和数据*导入或复制到报表的数据集。若要查看对基础数据所做的更改，必须刷新，或重新导入完整的当前数据集。|仅会将*表和列*导入或复制到报表的数据集中。始终可以查看最新的数据。
目前，通过 Power BI Embedded，可以将 DirectQuery 与云数据源一起使用，但不能与本地数据源一起使用。

## 使用 DirectQuery 的优点

使用 **DirectQuery** 具有两个主要优点：

   -	**DirectQuery** 允许对超大型数据集生成可视化元素，否则首先导入所有数据将不可行。
   -	基础数据更改可能需要刷新数据，对于某些报表，显示当前数据可能需要大型数据传输，从而使重新导入数据变得不可行。与此相反，**DirectQuery** 报表始终使用当前数据。

## DirectQuery 的限制

   使用 **DirectQuery** 存在一些限制：

   -	所有表必须都来自单一数据库。
   -	如果查询过于复杂，则会出错。若要修复此错误，则必须重构查询，降低其复杂程度。如果查询必须非常复杂，建议导入数据，而不要使用 **DirectQuery**。
   -	关系筛选仅限于单向，而不是双向。
   -	不能更改某列的数据类型。
   -	默认情况下，将对度量值中允许使用的 DAX 表达式加以限制。请参阅 [DirectQuery 和度量值](#measures)。

<a name="measures"/>  

## DirectQuery 和度量值

为了确保发送到基础数据源的查询具有可接受的性能，针对度量值施加了一些限制。使用 **Power BI Desktop** 时，高级用户可以通过选择“文件”>“选项和设置”>“选项”来选择绕过此限制。在“选项”对话框中，选择“DirectQuery”，然后选择“允许 DirectQuery 模式下的度量值不受限制”选项。选中该选项后，可以使用对度量值有效的任何 DAX 表达式。但是用户必须知道，在导入数据时性能非常好的某些表达式在 **DirectQuery** 模式下可能会导致对后端源的查询非常缓慢。

## 另请参阅
- [Microsoft Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)
- [Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)

<!---HONumber=Mooncake_1010_2016-->