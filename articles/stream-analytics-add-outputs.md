<properties 
	pageTitle="如何配置流分析作业的数据输出 | Azure" 
	description="配置流分析作业的输出 | 学习路径段。"
	keywords="数据输出、数据移动"
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>

# 如何配置流分析作业的数据输出

Azure 流分析作业可以连接到一个或多个数据输出，这些数据输出定义了一个到现有数据接收器的连接。当你的流分析作业处理并转换传入数据时，数据输出事件的流将被写入到你的作业输出中。

流分析数据输出可用于实时仪表板或警报的源、触发数据移动工作流，或者仅仅为以后批量处理存档数据。流分析可以与多个 Azure 服务进行一流集成，在这里进行了详细记录。

若要向你的流分析作业添加输出：

1. 在 Azure 门户中，单击“输出”，然后在流分析作业中单击“添加输出”。

    ![添加输出](./media/stream-analytics-add-outputs/1-stream-analytics-add-outputs.png)

2. 指定输出的类型：

    ![选择数据移动类型](./media/stream-analytics-add-outputs/2-stream-analytics-add-outputs.png)

3. 在“输出别名”框中为该输出提供一个友好的名称。此名称以后会用于你的作业查询以引用该输出。
    
    填充所需连接属性的其余部分以连接到你的输出。这些字段根据输出类型而变化，在此处进行了详细定义。

    ![添加数据输出属性](./media/stream-analytics-add-outputs/3-stream-analytics-add-outputs.png)

4. 根据输出类型，你可能需要指定序列化或格式化数据的方式。此处记录了每个输出类型的特定序列化设置。

    填充所需连接属性的其余部分以连接到你的数据源。这些字段根据输入类型和源类型而变化，[此处](/documentation/articles/stream-analytics-create-a-job/)进行了详细定义。

    ![将数据输出添加到事件中心](./media/stream-analytics-add-outputs/4-stream-analytics-add-outputs.png)

> [Azure.Note] 必须先存在添加到作业的输出元素，然后才能启动作业并开始事件的流动。例如，如果你使用 Blob 存储作为输出，该作业将不会自动创建存储帐户。在启动 ASA 作业之前，需要由用户创建该存储帐户。

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!---HONumber=Mooncake_0314_2016-->