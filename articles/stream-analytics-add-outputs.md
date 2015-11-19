<properties 
	pageTitle="添加输出 | Windows Azure" 
	description="添加输出的学习路径段。"
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="09/29/2015" 
	wacn.date="11/12/2015"/>

# 添加输出

Azure 流分析作业可以连接到一个或多个输出，其定义了一个到现有数据接收器的连接。当你的流分析作业处理并转换传入数据时，输出事件的流将被写入到你的作业输出中。

流分析输出可用于实时仪表板或警报的源、触发下游工作流，或者仅仅为以后批量处理存档数据。流分析可以与多个 Azure 服务进行一流集成，在这里进行了详细记录。

若要向你的流分析作业添加输出：

1. 在 Azure 门户中，单击“输出”，然后在流分析作业中单击“添加输出”。

    ![添加输出](./media/stream-analytics-add-outputs/1-stream-analytics-add-outputs.png)

2. 指定输出的类型：

    ![选择输出源](./media/stream-analytics-add-outputs/2-stream-analytics-add-outputs.png)

3. 在“输出别名”框中为该输出提供一个友好名称。此名称以后会用于你的作业查询以引用该输出。
    
    填充所需连接属性的其余部分以连接到你的输出。这些字段根据输出类型而变化，在此处进行了详细定义。

    ![添加属性](./media/stream-analytics-add-outputs/3-stream-analytics-add-outputs.png)

4. 根据输出类型，你可能需要指定序列化或格式化数据的方式。此处记录了每个输出类型的特定序列化设置。

    填充所需连接属性的其余部分以连接到你的数据源。这些字段根据输入类型和源类型而变化，在[此处](/documentation/articles/stream-analytics-create-a-job)进行了详细定义。

    ![数据序列化设置](./media/stream-analytics-add-outputs/4-stream-analytics-add-outputs.png)

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!---HONumber=79-->