<properties
   pageTitle="管理 Azure SQL 数据仓库中的计算能力（Azure 门户预览）| Azure"
   description="用于管理计算能力的 Azure 门户预览任务。通过调整 DWU 缩放计算资源。或者，暂停和恢复计算资源来节省成本。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="08/22/2016"
   wacn.date="10/17/2016"
   ms.author="barbkess;sonyama"/>  


# 管理 Azure SQL 数据仓库中的计算能力（Azure 门户）

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)


通过扩大计算资源和内存来提升性能，从而满足工作负荷不断变化的需求。通过在非高峰时段缩减资源或同时暂停计算来节省成本。

此任务集合使用 Azure 门户预览实现：

- 缩放计算
- 暂停计算
- 恢复计算

有关详细信息，请参阅 [管理计算概述][]。

<a name="scale-performance-bk"></a>
<a name="scale-compute-bk"></a>

## 缩放计算能力

[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../../includes/sql-data-warehouse-scale-dwus-description.md)]

更改计算资源：

1. 打开 [Azure 门户预览][]，打开你的数据库，然后单击“缩放”。

    ![单击“缩放”][1]

1. 在“缩放”边栏选项卡上，向左或向右移动滑块，以更改 DWU 设置。

    ![移动滑块][2]

1. 单击“保存”。此时会显示确认消息。单击“是”以确认或“否”以取消。

    ![点击“保存”(Save)][3]

<a name="pause-compute-bk"></a>

## 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../../includes/sql-data-warehouse-pause-description.md)]

暂停数据库：

1. 打开 [Azure 门户预览][]，并打开你的数据库。请注意，状态为“联机”。

    ![联机状态][6]  


1. 若要挂起计算和内存资源，请单击“暂停”，随后将显示确认消息。单击“是”以确认或“否”以取消。

    ![确认暂停][7]  


1. 当 SQL 数据仓库正在启动数据库时，状态为“正在暂停”。
2. 当状态为“已暂停”时，则暂停操作完成，将停止对用户收取 DWU 费用。

    ![暂停状态][4]  


<a name="resume-compute-bk"></a>

## 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../../includes/sql-data-warehouse-resume-description.md)] 
恢复数据库：

1. 打开 [Azure 门户预览][]，并打开你的数据库。请注意，状态为“已暂停”。

    ![暂停数据库][4]  


1. 若要恢复数据库，请单击“启动”，随后将显示确认消息。单击“是”确认，单击“否”取消。

    ![确认恢复][5]  


1. 当 SQL 数据仓库正在启动数据库时，状态为“正在恢复”。
2. 当状态为“联机”时，该数据库已就绪。

    ![联机状态][6]  


<a name="next-steps-bk"></a>

## 后续步骤
有关详细信息，请参阅[管理概述][]。

<!--Image references-->
[1]: ./media/sql-data-warehouse-manage-compute-portal/click-scale.png
[2]: ./media/sql-data-warehouse-manage-compute-portal/move-slider.png
[3]: ./media/sql-data-warehouse-manage-compute-portal/click-save.png
[4]: ./media/sql-data-warehouse-manage-compute-portal/resume-database.png
[5]: ./media/sql-data-warehouse-manage-compute-portal/resume-confirm.png
[6]: ./media/sql-data-warehouse-manage-compute-portal/pause-database.png
[7]: ./media/sql-data-warehouse-manage-compute-portal/pause-confirm.png

<!--Article references-->
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/
[Manage compute power overview]: /documentation/articles/sql-data-warehouse-manage-compute-overview/

<!--MSDN references-->


<!--Other Web references-->

[Azure 门户预览]: http://portal.azure.cn/

<!---HONumber=Mooncake_1010_2016-->