<properties
   pageTitle="使用 Redgate 的 Data Platform Studio 将数据载入 SQL 数据仓库 | Azure"
   description="了解如何将 Redgate 的 Data Platform Studio 用于数据仓库方案。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="twounder"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/13/2016"
   wacn.date="11/15/2016"
   ms.author="mausher;barbkess"/>  



# 使用 Redgate Data Platform Studio 加载数据

> [AZURE.SELECTOR]
- [Redgate](/documentation/articles/sql-data-warehouse-load-with-redgate/)
- [PolyBase](/documentation/articles/sql-data-warehouse-get-started-load-with-polybase/)
- [BCP](/documentation/articles/sql-data-warehouse-load-with-bcp/)

本教程介绍如何使用 [Redgate 的 Data Platform Studio](http://www.red-gate.com/products/azure-development/data-platform-studio/) (DPS) 将数据从本地 SQL Server 迁移到 Azure SQL 数据仓库。Data Platform Studio 应用了适当的兼容性修补程序和优化措施，可以快速启动 SQL 数据仓库操作。

> [AZURE.NOTE] [Redgate](http://www.red-gate.com) 是 Microsoft 的长期合作伙伴，提供各种 SQL Server 工具。Data Platform Studio 中的此功能免费提供，可作商业和非商业用途。

## 开始之前
### 创建或标识资源

在开始此教程之前，用户需具备：

- **本地 SQL Server 数据库**：需要导入到 SQL 数据仓库中的数据必须来自本地 SQL Server（2008R2 或更高版本）。Data Platform Studio 不能直接从 Azure SQL 数据库或文本文件导入数据。
- **Azure 存储帐户**：Data Platform Studio 先将数据暂存在 Azure Blob 存储中，然后再将其加载到 SQL 数据仓库。存储帐户必须使用“Resource Manager”部署模型（默认）而非“经典”部署模型。如果还没有存储帐户，请学习如何创建一个存储帐户。
- **SQL 数据仓库**：本教程将数据从本地 SQL Server 移到 SQL 数据仓库，因此用户需要有一个联机数据仓库。如果还没有数据仓库，请学习如何创建 Azure SQL 数据仓库。

> [AZURE.NOTE] 如果在同一区域创建存储帐户和数据仓库，则可提高性能。

## 步骤 1：使用 Azure 帐户登录到 Data Platform Studio
打开 Web 浏览器，导航到 [Data Platform Studio](https://www.dataplatformstudio.com/) 网站。使用 Azure 帐户登录，该帐户也用于创建过存储帐户和数据仓库。如果电子邮件地址与工作/学校帐户和 Microsoft 帐户均有关联，请务必选择能够访问资源的帐户。

> [AZURE.NOTE] 如果这是用户第一次使用 Data Platform Studio，系统会要求用户授予管理其 Azure 资源所需的应用程序权限。

## 步骤 2：启动导入向导
在 DPS 主屏幕中选择“导入到 Azure SQL 数据仓库”链接，启动导入向导。

![][1]  


## 步骤 3：安装 Data Platform Studio 网关
若要连接到本地 SQL Server 数据库，需安装 DPS 网关。该网关是一个客户端代理，用于访问本地环境、提取数据，以及将数据上载到存储帐户。用户的数据不会通过 Redgate 的服务器。安装该网关的步骤：

1.	单击“创建网关”链接
2. 使用提供的安装程序下载和安装该网关

![][2]  


> [AZURE.NOTE] 该网关可以安装在能够通过网络访问源 SQL Server 数据库的任何计算机上。它可以使用当前用户的凭据通过 Windows 身份验证访问 SQL Server 数据库。

安装以后，网关状态变为“已连接”，此时可选择“下一步”。

## 步骤 4：标识源数据库
在“输入服务器名称”文本框中，输入托管数据库的服务器的名称，然后选择“下一步”。然后，从下拉菜单中选择要从其中导入数据的数据库。

![][3]  


DPS 会检查所选数据库中是否存在要导入的表。DPS 默认导入数据库中的所有表。展开“所有表”链接即可选择或取消选择表。选择“下一步”按钮继续操作。

## 步骤 5：选择要暂存数据的存储帐户
DPS 会提示用户输入暂存数据的位置。从订阅中选择一个现有的存储帐户，然后选择“下一步”。

> [AZURE.NOTE] DPS 会在所选存储帐户中创建新的 blob 容器，并且每次导入都会使用不同的文件夹。

![][4]  


## 步骤 6：选择数据仓库
接下来，选择一个要将数据导入到其中的联机 [Azure SQL 数据仓库](http://aka.ms/sqldw)数据库。选择数据库以后，需输入连接到数据库所需的凭据，然后选择“下一步”。

![][5]  


> [AZURE.NOTE] DPS 将源数据表合并到数据仓库中。如果表名称相同，导致 DPS 必须覆盖数据仓库中的现有表，DPS 会警告用户。用户可以选择删除数据仓库中的任何现有对象，只需在导入之前勾选“删除所有现有对象”即可。

## 步骤 7：导入数据
DPS 会确认用户是否要导入数据。直接单击“开始导入”按钮，开始数据导入过程。

![][6]  


DPS 会以可视化方式显示从本地 SQL Server 提取和上载数据的进度，以及将数据导入 SQL 数据仓库的进度。

![][7]  


导入完成后，DPS 会显示数据导入情况摘要，以及所做更改的报告，其中包含已执行的兼容性修补程序。

![][8]  


## 后续步骤
若要浏览 SQL 数据仓库中的数据，请先查看以下内容：

- [查询 Azure SQL 数据仓库 (Visual Studio)][]

若要详细了解 Redgate 的 Data Platform Studio，请：

- [访问 DPS 主页](http://www.dataplatformstudio.com/)
- [观看 Channel9 上的 DPS 演示](https://channel9.msdn.com/Blogs/cloud-with-a-silver-lining/Loading-data-into-Azure-SQL-Datawarehouse-with-Redgate-Data-Platform-Studio)

若只需大致了解如何通过其他方式在 SQL 数据仓库中迁移和加载数据，请参阅：

- [将解决方案迁移到 SQL 数据仓库][]
- [将数据载入 Azure SQL 数据仓库](/documentation/articles/sql-data-warehouse-overview-load/)

如需更多的开发技巧，请参阅 [SQL 数据仓库开发概述](/documentation/articles/sql-data-warehouse-overview-develop/)。

<!--Image references-->

[1]: ./media/sql-data-warehouse-redgate/2016-10-05_15-59-56.png
[2]: ./media/sql-data-warehouse-redgate/2016-10-05_11-16-07.png
[3]: ./media/sql-data-warehouse-redgate/2016-10-05_11-17-46.png
[4]: ./media/sql-data-warehouse-redgate/2016-10-05_11-20-41.png
[5]: ./media/sql-data-warehouse-redgate/2016-10-05_11-31-24.png
[6]: ./media/sql-data-warehouse-redgate/2016-10-05_11-32-20.png
[7]: ./media/sql-data-warehouse-redgate/2016-10-05_11-49-53.png
[8]: ./media/sql-data-warehouse-redgate/2016-10-05_12-57-10.png

<!--Article references-->

[查询 Azure SQL 数据仓库 (Visual Studio)]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[使用 Power BI 可视化数据]: /documentation/articles/sql-data-warehouse-get-started-visualize-with-power-bi/
[将解决方案迁移到 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-overview-migrate/
[Load data into Azure SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-overview-load/
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-develop/

<!---HONumber=Mooncake_1024_2016-->