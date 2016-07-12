<properties
   pageTitle="安装适用于 SQL 数据仓库的 Visual Studio 和 SSDT | Azure"
   description="安装适用于 Azure SQL 数据仓库的 Visual Studio 和 SQL Server 开发工具 (SSDT)"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/13/2016"
   wacn.date="06/13/2016"/>

# 安装适用于 SQL 数据仓库的 Visual Studio 2015 和 SSDT

若要开发 SQL 数据仓库的应用程序，建议使用 Visual Studio 2015，并结合最新版本的 SQL Server Data Tools (SSDT)。也支持配合 SSDT 使用 Visual Studio 2013。

此外，若要从 Visual Studio 集成开发环境 (IDE) 运行查询，需要有**数据库工具的 Microsoft SQL Server 更新**。安装此扩展后，可以在对象资源管理器树中查看数据库对象，并对 SQL 数据仓库运行查询。

> [AZURE.NOTE] SQL 数据仓库尚不支持 Visual Studio 数据库项目。将来的版本会添加此功能。

## 步骤 1：安装 Visual Studio 2015

遵循以下链接来下载并安装 Visual Studio 2015。如果已安装 Visual Studio 2013 或 2015，可以跳到安装 SSDT 的步骤。

1. 从 Visual Studio Team Services [下载 Visual Studio 2015][]。
2. 按照 MSDN 上的[安装 Visual Studio][]指南安装，并选择默认配置。

## 步骤 2：安装 SSDT

若要安装适用于 Visual Studio 的 SSDT，只需遵循以下步骤从 Visual Studio 检查 SSDT 更新。

1. 单击“工具”>“扩展和更新”>“更新” 
2. 选择“产品更新”，然后查找“数据库工具的 Microsoft SQL Server 更新”

如果找不到更新，则表示已安装最新版本。若要确认是否已安装 SSDT，请单击“帮助”>“关于 Microsoft Visual Studio”，然后在列表中查找 SQL Server Data Tools。

## 后续步骤

安装最新版本的 SSDT 后，便可以[连接][]到你的数据库。

<!--Anchors-->

<!--Image references-->

<!--Arcticles-->
[连接]: /documentation/articles/sql-data-warehouse-get-started-connect/

<!--Other-->
[下载 Visual Studio 2015]: https://www.visualstudio.com/downloads/
[安装 Visual Studio]: https://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx

<!---HONumber=Mooncake_0606_2016-->