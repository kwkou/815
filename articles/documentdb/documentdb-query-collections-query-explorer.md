<properties
    pageTitle="DocumentDB 门户工具：查询浏览器 | Azure"
    description="了解 DocumentDB 查询浏览器，它是 Azure 门户预览中的一个 SQL 查询编辑器，用于编写 SQL 查询并针对 DocumentDB 集合运行这些查询。"
    keywords="编写 SQL 查询, SQL 查询编辑器"
    services="documentdb"
    author="kirillg"
    manager="jhubbard"
    editor="monicar"
    documentationcenter="" />
<tags
    ms.assetid="ac378240-b11f-4522-ae9f-09da3a6f9c16"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/10/2017"
    wacn.date="05/31/2017"
    ms.author="kirillg"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="5de8124569da23dc7272b768b2ae5d3607a387dc"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="write-edit-and-run-sql-queries-for-azure-documentdb-using-query-explorer-in-the-azure-portal"></a>在 Azure 门户预览中使用查询浏览器对 DocumentDB 编写、编辑和运行 SQL 查询
本文概述了 [DocumentDB](/home/features/documentdb/) 查询浏览器，该查询浏览器是一个 Azure 门户预览工具，可用于针对 [DocumentDB 集合](/documentation/articles/documentdb-create-collection/)编写、编辑和运行 SQL 查询。

1. 在 [Azure 门户预览](https://portal.azure.cn)的左侧导航栏中，单击![DocumentDB 图标](./media/documentdb-query-collections-query-explorer/nosql-documentdb-portal-icon.png) **DocumentDB**。 

    如果 **DocumentDB** 不可见，请单击底部的“更多服务”，然后单击 ![DocumentDB 图标](./media/documentdb-query-collections-query-explorer/nosql-documentdb-portal-icon.png) **DocumentDB**。
2. 在资源菜单中，单击“查询资源管理器” 。 
   
    ![Azure 门户预览的屏幕截图，其中突出显示了查询资源管理器](./media/documentdb-query-collections-query-explorer/queryexplorercommand.png)
3. 在“查询浏览器”边栏选项卡中，从下拉列表中选择要查询的**数据库**和**集合**，然后键入要运行的查询。 
   
    “数据库”和“集合”下拉列表会根据启动查询浏览器的上下文进行预填充。 
   
    提供了 `SELECT TOP 100 * FROM c` 的默认查询。  可以接受默认查询，也可以使用 [SQL 查询速查表](/documentation/articles/documentdb-sql-query-cheat-sheet/)或 [SQL 查询和 SQL 语法](/documentation/articles/documentdb-sql-query/)文章中所述的 SQL 查询语言构造自己的查询。
   
    单击“运行查询”  查看结果。
   
    ![在查询资源管理器（一个 SQL 查询编辑器）中编写 SQL 查询的屏幕截图](./media/documentdb-query-collections-query-explorer/queryexplorerinitial.png)
4. “结果”  边栏选项卡将显示查询的输出。 
   
    ![在查询资源管理器中编写 SQL 查询的结果的屏幕截图](./media/documentdb-query-collections-query-explorer/queryresults1.png)

## <a name="work-with-results"></a>处理结果
默认情况下，查询资源管理器会返回 100 条一组的结果。  如果查询生成了 100 多条结果，只需使用“下一页”和“上一页”命令即可浏览结果集。

![查询资源管理器分页支持的屏幕截图](./media/documentdb-query-collections-query-explorer/queryresultspagination.png)

对于成功的查询，“信息”窗格将包含指标，如请求费用、查询进行的往返数、当前显示的结果集，以及是否有更多结果（如上文所述，可通过“下一页”命令进行访问）。

![查询资源管理器查询信息的屏幕截图](./media/documentdb-query-collections-query-explorer/queryinformation.png)

## <a name="use-multiple-queries"></a>使用多个查询
如果正在使用多个查询，并且想要在它们之间快速切换，则可以在“查询浏览器”边栏选项卡的查询文本框中输入所有查询，然后突出显示想要运行的查询，再单击“运行查询”以查看结果。

![在查询浏览器（一个 SQL 查询浏览器）中编写多个 SQL 查询并突出显示和运行单个查询的屏幕截图](./media/documentdb-query-collections-query-explorer/queryexplorerhighlightandrun.png)

## <a name="add-queries-from-a-file-into-the-sql-query-editor"></a>将查询从文件添加到 SQL 查询编辑器
可以使用“加载文件”  命令加载现有文件的内容。

![显示如何使用“加载文件”将 SQL 查询从文件加载到查询资源管理器的屏幕截图](./media/documentdb-query-collections-query-explorer/loadqueryfile.png)

## <a name="troubleshoot"></a>故障排除
如果查询完成但有错误，查询资源管理器将显示可以帮助进行故障排除工作的错误列表。

![查询资源管理器查询错误的屏幕截图](./media/documentdb-query-collections-query-explorer/queryerror.png)

## <a name="run-documentdb-api-sql-queries-outside-the-portal"></a>运行门户外部的 DocumentDB API SQL 查询
Azure 门户预览中的查询浏览器只是一种对 DocumentDB 运行 SQL 查询的方式。 还可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 或[客户端 SDK](/documentation/articles/documentdb-sdk-dotnet/) 运行 SQL 查询。 有关使用这些其他方法的详细信息，请参阅[执行 SQL 查询](/documentation/articles/documentdb-sql-query/#ExecutingSqlQueries/)

## <a name="next-steps"></a>后续步骤
若要详细了解查询浏览器中支持的 DocumentDB API SQL 语法，请参阅 [SQL 查询和 SQL 语法](/documentation/articles/documentdb-sql-query/)一文或打印 [SQL 查询速查表](/documentation/articles/documentdb-sql-query-cheat-sheet/)。
还可以尝试使用 [查询板块](https://www.documentdb.com/sql/demo) ，在其中可以使用示例数据集联机测试查询。

<!---Update_Description: wording update -->