<properties
    pageTitle="如何配置流分析作业的数据输出 | Azure"
    description="配置流分析作业的输出 | 学习路径段。"
    keywords="数据输出、数据移动"
    documentationcenter=""
    services="stream-analytics"
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="3bbea3da-bfce-4af1-a15e-d4b23874034f"
    ms.service="stream-analytics"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="03/28/2017"
    wacn.date="05/15/2017"
    ms.author="jeffstok"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="37f2c79d7f2660a74b6fbeeab38aa13cbb06359e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="how-to-configure-data-outputs-for-stream-analytics-jobs"></a>如何配置流分析作业的数据输出
Azure 流分析作业可以连接到一个或多个数据输出，这些数据输出定义了一个到现有数据接收器的连接。 流分析作业处理并转换传入数据时，数据输出事件流会写入作业输出。

流分析数据输出可用于对实时仪表板或警报追源、触发数据移动工作流，或仅为以后的批量处理存档数据。 流分析可以与多个 Azure 服务进行一流集成，在这里进行了详细记录。

若要向流分析作业添加输出：

1. 在 Azure 经典管理门户中，单击“输出”，然后在流分析作业中单击“添加输出”。

    ![添加输出](./media/stream-analytics-add-outputs/1-stream-analytics-add-outputs.png)  

    在 Azure 门户中，单击流分析作业中的“输出”  磁贴。

    ![Azure 门户添加输出](./media/stream-analytics-add-outputs/5-stream-analytics-add-outputs.png)
2. 指定输出的类型：

    ![选择数据移动类型](./media/stream-analytics-add-outputs/2-stream-analytics-add-outputs.png)  

    ![Azure 门户选择数据移动类型](./media/stream-analytics-add-outputs/6-stream-analytics-add-outputs.png)

3. 在“输出别名”  框中为该输出提供一个友好的名称。 此名称以后会用于作业查询以引用该输出。  

    填充所需连接属性的其余部分以连接到输出。  这些字段根据输出类型而变化，在此处进行了详细定义。  

    ![添加数据输出属性](./media/stream-analytics-add-outputs/3-stream-analytics-add-outputs.png)  
4. 根据输出类型，可能需要指定序列化或格式化数据的方式。 此处记录了每个输出类型的特定序列化设置。

    填充所需连接属性的其余部分以连接到数据源。 这些字段根据输入类型和源类型而变化，[此处](/documentation/articles/stream-analytics-create-a-job/)进行了详细定义。  

    ![将数据输出添加到事件中心](./media/stream-analytics-add-outputs/4-stream-analytics-add-outputs.png)  

    ![Azure 门户将数据输出添加到事件中心](./media/stream-analytics-add-outputs/7-stream-analytics-add-outputs.png)  

> [AZURE.NOTE]
> 必须先存在添加到作业的输出元素，然后才能启动作业并开始事件的流动。 例如，如果使用 Blob 存储作为输出，该作业将不会自动创建存储帐户。 在启动 ASA 作业之前，需要由用户创建该存储帐户。
> 
> 

## <a name="get-help"></a>获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>后续步骤
* [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!--Update_Description:update meta properties; wording update -->