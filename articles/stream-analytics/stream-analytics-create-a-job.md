<properties
    pageTitle="如何为流分析创建数据分析处理作业 | Azure"
    description="为流分析创建数据分析处理作业 | 学习路径段。"
    keywords="数据分析处理"
    documentationcenter=""
    services="stream-analytics"
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="e825fbcf-69e9-443f-b402-3b7a4568f415"
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
    ms.openlocfilehash="927e67ad38f3fb9306a6b878951f800f5767e833"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="how-to-create-a-data-analytics-processing-job-for-stream-analytics"></a>如何为流分析创建数据分析处理作业
在 Azure 流分析中的最上层资源是一个流分析作业。  它包含一个或多个输入数据源、一个表达数据转换的查询以及一个或多个结果写入的输出目标。 用户可以利用所有这些元素，针对流式数据方案进行数据分析处理。

若要开始使用流分析，请创建一个新的流分析作业。  请注意，该操作直到作业启动后才对计费产生影响。

1. 登录在线 [Azure 经典管理门户](http://manage.windowsazure.cn)或 [Azure 门户](https://portal.azure.cn/)。
2. 在门户中：单击**新建**，然后单击**数据服务**或**数据分析**（具体取决于门户），然后依次单击**Azure 流分析**或**流分析**和**快速创建**。

    ![数据分析处理作业向导](./media/stream-analytics-create-a-job/1-stream-analytics-create-a-job.png)  

    ![创建数据分析处理作业](./media/stream-analytics-create-a-job/4-stream-analytics-create-a-job.png)  

3. 指定流分析作业所需的配置。

    * 在**作业名称**框中，输入一个名称以标识该流分析作业。对**作业名称**进行验证后，作业名称框中会出现一个绿色的复选标记。**作业名称**只能包含字母数字字符和字符**-**，且长度必须在 3 到 63 个字符之间。
    * 使用 Azure 门户中的**区域**或**位置**来指定要运行作业的地理位置。
    * 如果使用 Azure 门户，请选择或创建存储帐户以用作**区域监视存储帐户**。该存储帐户用来存储针对该区域内运行的所有流分析作业的监控数据。
    * 如果使用 Azure 门户，请指定新的或现有的**资源组**以保存应用程序的相关资源。

4. 当新的流分析作业选项配置完成后，请单击**创建流分析作业**。创建流分析作业需要几分钟时间。要查看状态，你可以在通知中心监视进度。

    ![数据分析处理作业通知中心](./media/stream-analytics-create-a-job/2-stream-analytics-create-a-job.png)  

    ![Azure 门户数据分析处理作业创建作业](./media/stream-analytics-create-a-job/5-stream-analytics-create-a-job.png)  

5.  新作业在显示时的状态为**已创建**。请注意，**启动**按钮被禁用。你必须先配置作业输入、查询和输出，然后才能启动作业。

    ![数据分析处理作业作业状态](./media/stream-analytics-create-a-job/3-stream-analytics-create-a-job.png)  

    ![Azure 门户数据分析处理作业作业状态](./media/stream-analytics-create-a-job/6-stream-analytics-create-a-job.png)  

## <a name="get-help"></a>获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>后续步骤
* [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!--Update_Description:update meta properties;wording update;-->