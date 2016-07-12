<properties 
	pageTitle="用于 SQL 数据库的非 1433 端口 | Azure"
	description="从 ADO.NET 到 Azure SQL 数据库 V12 的客户端连接有时会绕过代理直接与数据库交互。除 1433 以外的端口变得非常重要。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jhubbard"
	editor="" />


<tags 
	ms.service="sql-database" 
	ms.date="04/25/2016" 
	wacn.date="05/23/2016"/>


# 用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口


本主题介绍 Azure SQL 数据库 V12 对于使用 ADO.NET 4.5 或更高版本的客户端连接行为所带来的变化。


## SQL 数据库 V11：端口 1433


当客户端程序使用 ADO.NET 4.5 来连接并查询 SQL 数据库 V11 时，内部顺序如下：


1. ADO.NET 尝试连接到 SQL 数据库。

2. ADO.NET 使用端口 1433 来调用中间件模块，中间件将连接到 SQL 数据库。

3. SQL 数据库将其响应发回给中间件，中间件将响应转发给 ADO.NET 端口 1433。


**术语：**我们使用*代理 路由*来描述 ADO.NET 与 SQL 数据库交互的上述顺序。如果没有涉及到中间件，我们说使用的是*直接路由*。


## SQL 数据库 V12：内部与外部


对于现 V12 的连接，我们必须询问你的客户端程序是在 Azure 云边界*外部*还是*内部*运行。以下小节讨论了两种常见案例。


#### *外部：*客户端在台式机上运行


端口 1433 是托管 SQL 数据库客户端应用程序的台式机上唯一必须打开的端口。


#### *内部：*客户端在 Azure 上运行


如果你的客户端在 Azure 云边界内部运行，则它使用我们所谓的*直接路由*来与 SQL 数据库服务器交互。建立连接后，客户端与数据库之间的进一步交互不涉及到任何中间件代理。


顺序如下：


1. ADO.NET 4.5（或更高版本）发起与 Azure 云的简短交互，并接收动态识别的端口号。
 - 动态识别的端口号范围为 11000-11999 或 14000-14999。

2. 然后，ADO.NET 不通过任何中间件直接连接到 SQL 数据库服务器。

3. 查询直接发送到数据库，结果直接返回到客户端。


请确保 Azure 客户端计算机上 11000-11999 和 14000-14999 的端口范围已保留给 ADO.NET 4.5 客户端与 SQL 数据库 V12 之间的交互。

- 具体而言，范围中的端口必须没有其他任何出站阻塞器。

- 在 Azure VM 上，**高级安全 Windows 防火墙**控制端口设置。
 - 你可以使用[防火墙的用户界面](http://msdn.microsoft.com/zh-cn/library/cc646023.aspx)为指定 **TCP** 协议以及语法类似于 **11000-11999** 的端口范围添加规则。


## 版本澄清


本部分澄清引用产品版本的 Moniker。此外还列出了产品之间的版本配对。


#### ADO.NET


- ADO.NET 4.0 支持 TDS 7.3 协议，但不支持 7.4。
- ADO.NET 4.5 和更高版本支持 TDS 7.4 协议。


#### SQL 数据库 V11 和 V12


本主题重点说明 SQL 数据库 V11 和 V12 之间的客户端连接差异。


*注意：*Transact-SQL 语句 `SELECT @@version;` 返回一个以数字（例如“11”或“12”）开头的值，这些值与 SQL 数据库 V11 和 V12 的版本名称相匹配。


## 相关链接


- ADO.NET 4.6 已于 2015 年 7 月 20 日发布。可以在[这里](http://blogs.msdn.com/b/dotnet/archive/2015/07/20/announcing-net-framework-4-6.aspx)访问 .NET 团队的博客通告。


- ADO.NET 4.5 已于 2012 年 8 月 15 日发布。可以在[这里](http://blogs.msdn.com/b/dotnet/archive/2012/08/15/announcing-the-release-of-net-framework-4-5-rtm-product-and-source-code.aspx)访问 .NET 团队的博客通告。
 - 可以在[这里](http://blogs.msdn.com/b/dotnet/archive/2013/06/26/announcing-the-net-framework-4-5-1-preview.aspx)访问有关 ADO.NET 4.5.1 的博客文章。


- [TDS 协议版本列表](http://www.freetds.org/userguide/tdshistory.htm)


- [连接到 SQL 数据库：链接、最佳实践和设计准则](/documentation/articles/sql-database-connect-central-recommendations/)


- [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)

<!---HONumber=Mooncake_0307_2016-->