<properties
pageTitle="将 Tez UI 与基于 Windows 的 HDInsight 配合使用 | Azure"
description="了解如何使用 Tez UI 调试基于 Windows 的 HDInsight 上的 Tez 作业。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="05/03/2016"
	wacn.date="06/29/2016"/>

# 使用 Tez UI 调试基于 Windows 的 HDInsight 上的 Tez 作业

Tez UI 是一个网页，可用于了解和调试在基于 Windows 的 HDInsight 群集上将 Tez 用作执行引擎的作业。利用 Tez UI，你可以将作业显示为包含已连接项目的图形，深入了解每个项目并检索统计信息和日志记录信息。

##先决条件

* 基于 Windows 的 HDInsight 群集。有关创建新群集的步骤，请参阅[开始使用基于 Windows 的 HDInsight](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/)。

    > [AZURE.IMPORTANT] Tez UI 只在 2016 年 2 月 8 日以后创建的基于 Windows 的 HDInsight 群集上提供。

* 基于 Windows 的远程桌面客户端。

##了解 Tez

Tez 是 Hadoop 中的一种可扩展数据处理框架，其处理速度比传统的 MapReduce 处理要快。对于基于 Windows 的 HDInsight 群集来说，它是一个可以为 Hive 启用的可选引擎，启用时只需在 Hive 查询中使用以下命令即可：

    set hive.execution.engine=tez;

将工作提交到 Tez 时，Tez 会创建一个有向无环图 (DAG)，用于描述作业所需操作的执行顺序。单独的操作称为顶点，每个顶点执行完整作业的一部分。实际执行顶点所描述的工作称为完成任务，任务可以分布在群集的多个节点中。

###了解 Tez UI

Tez UI 是一个网页，提供正在使用 Tez 运行的或此前使用 Tez 运行过的进程的信息。它允许你查看 Tez 生成的 DAG 及其在群集中的分布情况、计数器（例如任务和顶点所用内存）以及错误信息。它可以为以下情况提供有用的信息：

* 监视长时间运行的进程、查看映射的进度以及精简任务。

* 分析成功进程或失败进程的历史数据，了解处理过程的改进方式或其失败的原因。

##生成 DAG

Tez UI 包含数据的前提是使用 Tez 引擎的作业当前正在运行或过去曾经运行过。简单的 Hive 查询通常在不使用 Tez 的情况下即可进行解析，但是，更复杂的需要进行筛选、分组、排序、联接等操作的查询则通常需要使用 Tez。

请使用以下步骤来运行 Hive 查询，该查询将通过 Tez 来执行。

1. 在 Web 浏览器中导航到 https://CLUSTERNAME.azurehdinsight.cn ，其中 __CLUSTERNAME__ 是 HDInsight 群集的名称。

2. 从页面顶部的菜单中选择“Hive 编辑器”。此时会显示包含以下示例性查询的页面。

        Select * from hivesampletable

    擦除示例性查询，将其替换为以下内容。

        set hive.execution.engine=tez;
        select market, state, country from hivesampletable where deviceplatform='Android' group by market, country, state;

3. 选择“提交”按钮。页面底部的“作业会话”部分将显示查询的状态。一旦状态更改为“已完成”，即可选择“查看详细信息”链接来查看结果。“作业输出”应如下所示：
        
        en-GB   Hessen      Germany
        en-GB   Kingston    Jamaica
        en-GB   Nairobi Area    Kenya

##使用 Tez UI

> [AZURE.NOTE] Tez UI 只能从群集头节点的桌面使用，因此必须使用远程桌面来连接到头节点。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，选择你的 HDInsight 群集。在 HDInsight 页的底部，单击“配置”。此时谁显示“配置”页

2. 在“配置”页中，选择“连接”连接到群集头节点。出现提示时，使用群集远程桌面用户的用户名和密码来验证连接。

    > [AZURE.NOTE] 如果尚未启用远程桌面连接，请提供用户名、密码和到期日期，然后选择“启用”来启用远程桌面。一旦启用，即可使用前面的步骤进行连接。

3. 连接后，在远程桌面上打开 Internet Explorer，在浏览器的右上角选择齿轮图标，然后选择“兼容性视图设置”。

4. 在“兼容性视图设置”底部，清除“在兼容性视图中显示 Intranet 站点”和“使用 Microsoft 兼容性列表”所对应的复选框，然后选择“关闭”。

5. 在 Internet Explorer 中浏览到 http://headnodehost:8188/tezui/#/。此时会显示 Tez UI

    ![Tez UI](./media/hdinsight-debug-tez-ui/tezui.png)

    在 Tez UI 加载以后，你会看到群集中当前正在运行或过去曾经运行过的 DAG 的列表。默认视图包括“DAG 名称”、“ID”、“提交者”、“状态”、“开始时间”、“结束时间”、“持续时间”、“应用程序 ID”和“队列”。使用页面右侧的齿轮图标可以添加更多列。

    如果只有一个条目，则该条目是针对你在前一部分运行的查询的。如果有多个条目，你可以在 DAG 上方的字段中输入搜索条件，然后按 __Enter__ 进行搜索。

4. 选择最新的 DAG 条目的“DAG 名称”。此时会显示有关 DAG 的信息，以及用于下载 JSON Zip 文件（其中包含有关 DAG 的信息）的选项。

    ![DAG 详细信息](./media/hdinsight-debug-tez-ui/dagdetails.png)

5. “DAG 详细信息”上方是多个可用于显示 DAG 相关信息的链接。

    * __DAG 计数器__显示为此 DAG 的计数器信息。
    
    * __图形视图__显示此 DAG 的图形表示方式。
    
    * __所有顶点__显示此 DAG 中顶点的列表。
    
    * __所有任务__显示此 DAG 中所有顶点的任务的列表。
    
    * __所有 TaskAttempts__ 显示尝试针对此 DAG 运行任务的信息。
    
    > [AZURE.NOTE] 如果滚动“顶点”、“任务”和“TaskAttempts”的列显示，你会注意到存在查看“计数器”的链接，以及每个行的“查看或下载日志”链接。

    如果作业失败，“DAG 详细信息”会显示状态“已失败”，同时会显示有关已失败任务的信息链接。DAG 详细信息下方会显示诊断信息。

7. 选择“图形视图”。此视图显示 DAG 的图形表示方式。将鼠标放在视图中的单个顶点上即可显示其信息。

    ![图形视图](./media/hdinsight-debug-tez-ui/dagdiagram.png)

8. 单击某个顶点会加载该项的“顶点详细信息”。单击“映射 1”顶点可显示该项的详细信息。选择“确认”进行导航确认。

    ![顶点详细信息](./media/hdinsight-debug-tez-ui/vertexdetails.png)

9. 请注意，此时页面顶部会存在与顶点和任务相关的链接。

    > [AZURE.NOTE] 你还可以通过以下方式访问此页：回到“DAG 详细信息”，选择“顶点详细信息”，然后选择“映射 1”顶点。

    * __顶点计数器__显示此顶点的计数器信息。
    
    * __任务__显示此顶点的任务。
    
    * __任务尝试__显示尝试针对此顶点运行任务的信息。
    
    * __源和接收器__显示此顶点的数据源和接收器。

    > [AZURE.NOTE] 与前一菜单一样，你可以滚动“任务”、“任务尝试”、“源和接收器”的列显示，以便显示每个项的详细信息链接。

10. 选择“任务”，然后选择名为“00\_000000”的项。此时会针对此任务显示“任务详细信息”。在此屏幕中，你可以查看“任务计数器”和“任务尝试”。

    ![任务详细信息](./media/hdinsight-debug-tez-ui/taskdetails.png)

##后续步骤

现在，你已了解如何使用 Tez 视图，因此可以详细了解如何[使用 Hive on HDInsight](/documentation/articles/hdinsight-use-hive/)。

有关 Tez 的更多详细的技术信息，请参阅 [Hortonworks 的 Tez 页](http://hortonworks.com/hadoop/tez/)。
<!---HONumber=Mooncake_0405_2016-->