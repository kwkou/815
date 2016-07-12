<properties
   pageTitle="管理 Azure SQL 数据仓库中的计算能力 (REST) | Azure"
   description="用于管理计算能力的 PowerShell 任务。通过调整 DWU 缩放计算资源。或者，暂停和恢复计算资源来节省成本。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/14/2016"
   wacn.date="06/13/2016"/>

# 管理 Azure SQL 数据仓库中的计算能力 (REST)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-compute/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)


通过扩大计算资源和内存来提升性能，从而满足工作负荷不断变化的需求。通过在非高峰时段缩减资源或同时暂停计算来节省成本。

此任务集合使用 Powershell 实现：

- 缩放计算
- 暂停计算
- 恢复计算

若要了解相关信息，请参阅[管理计算能力概述][]。

<a name="scale-performance-bk"></a>
<a name="scale-compute-bk"></a>

## 缩放计算能力

[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../includes/sql-data-warehouse-scale-dwus-description)]

若要更改 DWU，请使用[创建或更新数据库][] REST API。以下示例将托管在服务器 MyServer 上的数据库 MySQLDW 的服务级别目标设置为 DW1000。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

```
PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/ResourceGroup1/providers/Microsoft.Sql/servers/MyServer/databases/MySQLDW?api-version=2014-04-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    "properties": {
        "requestedServiceObjectiveName": DW1000
    }
}
```

<a name="pause-compute-bk"></a>

## 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../includes/sql-data-warehouse-pause-description)]

若要暂停数据库，请使用[暂停数据库][] REST API。以下示例将暂停 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

```
POST https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/ResourceGroup1/providers/Microsoft.Sql/servers/Server01/databases/Database02/pause?api-version=2014-04-01-preview HTTP/1.1
```

<a name="resume-compute-bk"></a>

## 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../includes/sql-data-warehouse-resume-description)]

若要启动数据库，请使用[恢复数据库][] REST API。以下示例将启动 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

```
POST https://management.chinacloudapi.cn/subscriptions{subscription-id}/resourceGroups/ResourceGroup1/providers/Microsoft.Sql/servers/Server01/databases/Database02/resume?api-version=2014-04-01-preview HTTP/1.1
```

<a name="next-steps-bk"></a>

## 后续步骤

有关其他管理任务，请参阅[管理概述][]。

<!--Image references-->

<!--Article references-->
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/
[管理计算能力概述]: /documentation/articles/sql-data-warehouse-overview-manage-compute/

<!--MSDN references-->
[暂停数据库]: https://msdn.microsoft.com/zh-cn/library/azure/mt718817.aspx
[恢复数据库]: https://msdn.microsoft.com/zh-cn/library/azure/mt718820.aspx
[创建或更新数据库]: https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx

<!--Other Web references-->

[Azure portal]: http://manage.windowsazure.cn/

<!---HONumber=Mooncake_0606_2016-->