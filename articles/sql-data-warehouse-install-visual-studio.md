<properties
   pageTitle="安装适用于 SQL 数据仓库的 Visual Studio 和/或 SSDT | Windows Azure"
   description="安装适用于 Azure SQL 数据仓库的 Visual Studio 和/或 SSDT 开发工具"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="twounder"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="10/21/2015"
   wacn.date="01/20/2016"/>

# 安装适用于 SQL 数据仓库的 Visual Studio 2015 和/或 SSDT

若要开发 SQL 数据仓库的应用程序，建议使用 Visual Studio 2013 或更高版本，并结合最新版本的 SQL Server Data Tools (SSDT)。

若要从 Visual Studio 集成开发环境 (IDE) 运行查询，只需安装 SSDT。这将会安装 Visual Studio IDE 和 SSDT，以便你可以使用 SQL Server 对象资源管理器连接到 Azure SQL 服务器。然后，你便可以针对 SQL 数据仓库数据库查看和运行查询。


## 步骤 1：下载并安装 Visual Studio

如果你选择安装 Visual Studio，便可以将 Visual Studio 2013 或 Visual Studio 2015 与 SQL 数据仓库配合使用。如果你已安装 Visual Studio 2013 或 2015，请跳到步骤 2 以安装 SSDT。

若要安装 Visual Studio 2015，请执行以下操作：

1. 从 Visual Studio Team Services [下载 Visual Studio 2015](https://www.visualstudio.com/downloads)。 
2. 按照 MSDN 上的[安装 Visual Studio](https://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx)指南安装，并选择默认配置。

## 步骤 2：下载并安装最新的 SQL Server Data Tools (SSDT) 

无论是否已安装 Visual Studio，都需要有最新版本的 SQL Server Data Tools (SSDT) 才能支持 SQL 数据仓库。

若要安装最新版本的 SSDT，请执行以下操作：

1. [下载 SQL Server Data Tools Preview](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)（适用于 Visual Studio 2013 或 2015）。
2. 按照下载站点上的说明安装。

## 后续步骤

安装最新版本的 SSDT 后，便可以[连接](/documentation/articles/sql-data-warehouse-get-started-connect)到你的数据库。

<!--Anchors-->

<!--Image references-->

<!---HONumber=Mooncake_1207_2015-->
