<properties 
	pageTitle="如何使用流分析编写查询 | Azure" 
	description="使用流分析编写查询和查询数据 | 学习路径段。"
	keywords="如何编写查询, 查询数据, 编写查询, 正在编写查询"
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>


# 如何使用流分析编写查询

针对 Azure 流分析中的流处理逻辑编写查询将作为一种“现有查询”来实施，该查询在作业启动前定义并在数据抵达作业时对数据执行。使用一种类似于 SQL 的查询语言来表示数据转换，这种语言大部分是 T-SQL 的一个子集，但增加了某些用于表示临时语义的语言扩展，例如 [Windowing](https://msdn.microsoft.com/zh-cn/library/azure/dn835019.aspx)。

## 编写查询： ##

1. 在 Azure 管理门户上的流分析作业中，单击“查询”。

    ![选择查询](./media/stream-analytics-write-queries/1-stream-analytics-write-queries.png)

2.	新的作业有一个查询模板来帮助你开始。查询模板执行一种“传递”查询，将来自输入的所有字段投射到输出。

    - 如果你已经为你的作业定义了至少一个输入和一个输出，则可以用你希望首先使用的输入和输出的别名代替占位符 "[YourOutputAlias]" 和 "[YourInputAlias]"。此外，你仍然可以在 Azure 门户中编写和测试你的查询而无需在作业上定义输入和输出。
    - 如果你希望执行除简单传递以外的更多处理，则可以编辑查询定义。要开始编写查询，请阅读[此处](/documentation/articles/stream-analytics-stream-analytics-query-patterns/)提供的某些常见查询模式。  
  
    ![查询数据窗口](./media/stream-analytics-write-queries/2-stream-analytics-write-queries.png)

## 若要验证查询数据是否正常工作，请执行以下操作： ##

可以通过在浏览器中使用一个或多个包含测试数据的本地 JSON 文件来运行查询，从而测试查询的行为。这不会启动作业，也不会对计费有任何影响。

1.	确保查询没有任何错误（否则“测试”按钮将被禁用），然后单击“测试”按钮。  

    ![查询数据测试](./media/stream-analytics-write-queries/3-stream-analytics-write-queries.png)

2.	系统将提示你为查询中引用的每个输入指定文件。在本示例中，模板查询保持原样，因此对话框提示提供一个名为 "yourinputalias" 的输入。

    ![测试数据查询](./media/stream-analytics-write-queries/4-stream-analytics-write-queries.png)

3.	浏览到一个测试文件。[github](https://github.com/Azure/azure-stream-analytics/tree/master/Sample 数据) 提供了几个样本文件，你还可以通过输入选项卡上的“样本数据”功能，从你自己的数据流输入获取样本数据。

    ![查询输入](./media/stream-analytics-write-queries/5-stream-analytics-write-queries.png)

4.	在关闭对话框后，你的查询将使用测试数据运行，并且你将在“查询”页的底部看到结果。

    ![查询摘要](./media/stream-analytics-write-queries/6-stream-analytics-write-queries.png)

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!---HONumber=Mooncake_0307_2016-->