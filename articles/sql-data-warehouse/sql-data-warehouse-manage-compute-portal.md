<properties
    pageTitle="管理 Azure SQL 数据仓库中的计算能力（Azure 预览门户）| Azure"
    description="用于管理计算能力的 Azure 预览门户任务。通过调整 DWU 缩放计算资源。或者，暂停和恢复计算资源来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="233b0da5-4abd-4d1d-9586-4ccc5f50b071"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="03/20/2017"
    ms.author="barbkess" />  


# 管理 Azure SQL 数据仓库中的计算能力（Azure 门户）

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

## 缩放计算能力

[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../../includes/sql-data-warehouse-scale-dwus-description.md)]

更改计算资源：

1. 打开 [Azure 门户][Azure portal]，打开你的数据库，然后单击“缩放”。

    ![单击“缩放”][1]
2. 在“缩放”边栏选项卡上，向左或向右移动滑块，以更改 DWU 设置。

    ![移动滑块][2]

3. 单击“保存”。此时会显示确认消息。单击“是”以确认或“否”以取消。

    ![点击“保存”(Save)][3]

## <a name="pause-compute"></a><a name="pause-compute-bk"></a> 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../../includes/sql-data-warehouse-pause-description.md)]

暂停数据库：

1. 打开 [Azure 门户][Azure portal]，并打开你的数据库。请注意，状态为“联机”。

    ![联机状态][6]  


2. 若要挂起计算和内存资源，请单击“暂停”，随后将显示确认消息。单击“是”以确认或“否”以取消。

    ![确认暂停][7]  


3. 当 SQL 数据仓库正在启动数据库时，状态为“正在暂停”。
4. 当状态为“已暂停”时，则暂停操作完成，将停止对用户收取 DWU 费用。

    ![暂停状态][4]  


## <a name="resume-compute-bk"></a> 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../../includes/sql-data-warehouse-resume-description.md)] 
恢复数据库：

1. 打开 [Azure 门户][Azure portal]，并打开数据库。请注意，状态为“已暂停”。

    ![暂停数据库][4]  


2. 若要恢复数据库，请单击“启动”，随后将显示确认消息。单击“是”确认，单击“否”取消。

    ![确认恢复][5]  


3. 当 SQL 数据仓库正在启动数据库时，状态为“正在恢复”。
4. 当状态为“联机”时，该数据库已就绪。

    ![联机状态][6]  

## <a name="next-steps-bk"></a>后续步骤
有关详细信息，请参阅[管理概述][Management overview]。

<!--Image references-->
[1]: ./media/sql-data-warehouse-manage-compute-portal/click-scale.png
[2]: ./media/sql-data-warehouse-manage-compute-portal/move-slider.png
[3]: ./media/sql-data-warehouse-manage-compute-portal/click-save.png
[4]: ./media/sql-data-warehouse-manage-compute-portal/resume-database.png
[5]: ./media/sql-data-warehouse-manage-compute-portal/resume-confirm.png
[6]: ./media/sql-data-warehouse-manage-compute-portal/pause-database.png
[7]: ./media/sql-data-warehouse-manage-compute-portal/pause-confirm.png

<!--Article references-->
[Management overview]: /documentation/articles/sql-data-warehouse-overview-manage/
[Manage compute overview]: /documentation/articles/sql-data-warehouse-manage-compute-overview/

<!--MSDN references-->


<!--Other Web references-->

[Azure portal]: http://portal.azure.cn/

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update meta properties,wording update-->