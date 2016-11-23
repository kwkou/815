<properties
	pageTitle="Azure SQL 数据库用户手册 - 第一部分 | Azure"
	description="Azure SQL 数据库用户手册 - 第一部分 | Azure"
	services=""
	documentationCenter=""
	authors="Lei Zhang"
	manager=""
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date=""
	wacn.date="10/19/2016"/>

#Azure SQL 数据库用户手册

- [Azure SQL 数据库用户手册 - 第二部分](/documentation/articles/azure-sql-database-user-manual-part-2/)

 
##<a id="overview"></a>1. 总体介绍

###<a id="what-is-azure-sql-database"></a>1.1 什么是 Azure SQL 数据库  

在传统 IDC 环境里，如果要使用 SQL Server 数据库服务，首先需要安装操作系统，然后再安装和配置 SQL Server 服务。这样需要管理的组件有：  

1. Windows Server 操作系统  

2. SQL Server 数据库服务  

用户不仅需要维护数据库，还要维护数据库底层的操作系统和运行等。管理复杂，且成本较高。

Azure 提供数据库即服务 (Database-as-a-Service)，包括 Azure SQL 数据库和 MySQL Database on Azure。用户收到的是一个数据库连接字符串。注意：用户无需管理该字符串后面的操作系统、数据库服务等。  

此外，这个数据库连接字符串背后的数据库服务，本身可提供 99.99% 服务级别协议和数据库备份功能，降低了用户管理数据库的成本。

> [注意事项] SQL Azure 是 Azure SQL 数据库名称的前身 ，为了描述方便有些地方可能仍会沿用，请注意它和 Azure SQL 数据库是同一个产品。  

###<a id="difference-between-on-premise-and-azure-sql-database"></a>1.2 Azure SQL 数据库与传统 SQL Server 虚拟机的区别
Azure SQL 数据库与传统 SQL Server 2008, 2012, 2014 虚拟机主要有以下几方面区别：  

<table width="100%" border="1" cellspacing="0" cellpadding="0">
	<tr>
		<td>内容</td>
		<td>SQL Server 虚拟机</td>
		<td>Azure SQL 数据库</td>
	</tr>
	<tr>
		<td><b>应用迁移</b></td>
		<td></td>
		<td></td>
	</tr>
	<tr>
		<td>迁移现有应用</td>
		<td>快速</td>
		<td>中等</td>
	</tr>
	<tr>
		<td>迁移新的应用</td>
		<td>中等</td>
		<td>快速</td>
	</tr>
	<tr>
		<td><b>管理成本</b></td>
		<td></td>
		<td></td>
	</tr>
	<tr>
		<td>管理操作系统</td>
		<td>是</td>
		<td>否</td>
	</tr>
	<tr>
		<td>虚拟机高可用</td>
		<td>需要手工设置</td>
		<td>N/A</td>
	</tr>
	<tr>
		<td>数据库高可用</td>
		<td>需要手工设置(比如 SQL Server Always-On，SQL Server replication 等)</td>
		<td>99.99% 服务级别协议</td>
	</tr>
	<tr>
		<td>使用成本</td>
		<td>中等</td>
		<td>低</td>
	</tr>
	<tr>
		<td><b>扩展(Scale)</b></td>
		<td></td>
		<td></td>
	</tr>
	<tr>
		<td>向上扩展 (Scale-Up)</td>
		<td>D14 (16Core / 112G)</td>
		<td>P11</td>
	</tr>
	<tr>
		<td>横向扩展 (Scale-Out)</td>
		<td>需要手工设置 (比如 SQL Server Always-On，SQL Server replication 等)</td>
		<td>需要用户自己设计 Scale-Out</td>
	</tr>
	<tr>
		<td>数据库最大容量</td>
		<td>32 TB (D14 16Core / 112G)</td>
		<td>1 TB</td>
	</tr>
	<tr>
		<td><b>管理平台</b></td>
		<td></td>
		<td></td>
	</tr>
	<tr>
		<td>操作系统和虚拟机</td>
		<td>用户完全控制</td>
		<td>N/A</td>
	</tr>
	<tr>
		<td>SQL Server 组件兼容性</td>
		<td>对 SQL Server 产品全部支持，包括 SSIS, SSAS, SSRS</td>
		<td>只提供数据库引擎</td>
	</tr>
	<tr>
		<td><b>其他</b></td>
		<td></td>
		<td></td>
	</tr>
	<tr>
		<td>服务级别</td>
		<td>服务器，实例，数据库</td>
		<td>数据库</td>
	</tr>
	<tr>
		<td>兼容性</td>
		<td>完全兼容</td>
		<td>部分兼容</td>
	</tr>
</table>

####<a id="migrate-application"></a>1.2.1 应用迁移
#####<a id="migrate-exist-application"></a>1.2.1.1 迁移现有应用
由于 SQL Server 虚拟机完全兼容 SQL Server 2008, 2012 和 2014，所以从迁移现有应用来说，使用 SQL Server 虚拟机是最快速的。

#####<a id="migrate-new-application"></a>1.2.1.2 迁移新的应用
如果要迁移一个新的应用程序，使用 SQL Server 虚拟机需要安装操作系统，配置数据库用户名、密码、防火墙规则等，操作过程复杂。  

而使用 Azure SQL 数据库，在创建数据库时只需做若干配置，就可以直接使用这个数据库服务。底层的数据库备份、高可用设计，Azure SQL 数据库都已配置好，用户无需再进行其他配置。  

####<a id="manage-cost"></a>1.2.2 管理成本

#####<a id="manage-operation-system"></a>1.2.2.1 管理操作系统

在 Azure 环境下，采用 SQL Server 虚拟机，需要同时管理操作系统和 SQL Server 数据库服务。  

而采用 Azure SQL 数据库，用户接收到的只是一个数据库连接字符串，无需管理该字符串背后的操作系统、数据库服务等。 

#####<a id="vm-high-availability"></a>1.2.2.2 虚拟机高可用

在 Azure 环境下，采用 SQL Server 虚拟机，用户需要手动设置虚拟机的高可用：即将 2 台或者 2 台以上的虚拟机，部署到同一个可用性集中。  

而采用 Azure SQL 数据库，用户无需关心虚拟机的高可用。  

#####<a id="database-high-availability"></a>1.2.2.3 数据库高可用  

在 Azure 环境下，采用 SQL Server 虚拟机，用户需要手动设置数据库的高可用：比如采用 SQL Server Always-On，或者 SQL Server replication，即让 SQL Server 事务在两个 SQL Server 虚拟机之间保持同步。  

采用 Azure SQL 数据库，数据库本身就提供高可用的能力。Azure SQL 数据库提供 99.99% 服务级别协议服务承诺。  

#####<a id="cost"></a>1.2.2.4 使用成本

在 Azure 环境下，采用 SQL Server 虚拟机，考虑的成本因素主要有：  

1.	至少 2 台 SQL Server 虚拟机，并设置在同一个高可用性集下  

2.	虚拟机开机时的计算成本  

3.	SQL Server License 费用  

4.	虚拟机磁盘的存储费用  

SQL Server 虚拟机所需成本需综合考虑上述四个因素。

> [ 注意事项 ] 采用 Azure SQL 数据库，无需考虑上述四个成本因素。用户只需按 Azure SQL 数据库的实际使用量来付费。从成本上来说，Azure SQL 数据库比 SQL Server 虚拟机价格便宜。 

####<a id="scale"></a>1.2.3 扩展

#####<a id="scale-up"></a>1.2.3.1 向上扩展

所谓向上扩展，就是一台虚拟机，从 1 个核 (Core) 升级到 2 Core，4 Core, 8 Core 或者是 16 Core。  

单个节点向上扩展是有限的。受限于现有的 CPU 制造技术，大量的计算资源无法都堆积到 1 台 300 Core 甚至 400 Core 的计算节点上。  

在 Azure 环境下，采用 SQL Server 虚拟机，单个节点的最大计算单元为 D14(16 Core / 112 GB) (或者是 DS14，也是 16 Core / 112 GB，区别是 DS14 支持 SSD 固态磁盘)。  

采用 Azure SQL 数据库，因为不存在虚拟机概念，所以无法升级虚拟机配置。但是 Azure SQL 数据库有一个 DTU 概念(后面会详细说明)。如果需提高 Azure SQL 数据库的性能，可以对 DTU 进行升级。  

#####<a id="scale-out"></a>1.2.3.2 横向扩展

所谓横向扩展，就是由 1 个计算节点，横向扩展到多个计算节点上并行计算，比如 50 个、100 个计算节点。比如一个互联网业务需要大量的计算资源，可以将这些计算需求由 100 台 4 Core 的计算节点进行并行计算。  

在 Azure 环境下采用 SQL Server 虚拟机，可以使用 SQL Server Always-On 或 SQL Server Replication 等技术，实现数据库读写分离和横向扩展。  

采用 Azure SQL 数据库，用户需要自己设计 Scale-Out。比如我们可以把数据库进行切片，分散到若干个 Azure SQL 数据库里（后面会详细说明）。  

####<a id="management-platform"></a>1.2.4 管理平台

#####<a id="manage-os-and-vm"></a>1.2.4.1 操作系统和虚拟机

在 Azure 环境下，采用 SQL Server 虚拟机，用户可以通过远程桌面连接，管理虚拟机和操作系统。  

采用 Azure SQL 数据库，用户不需要管理后台的虚拟机和操作系统。  

#####<a id="sql-server-components-compatibility"></a>1.2.4.2 SQL Server 组件兼容性

在 Azure 环境下，采用 SQL Server 虚拟机，用户可以安装和配置 SQL Server 全线产品，包括 SQL Server Data Engine, SQL Server Integration Service (SSIS), SQL Server Analysis Service (SSAS), SQL Server Reporting Service (SSRS)。  

采用 Azure SQL 数据库，用户只能使用数据库引擎服务，不包含 SSIS, SSAS, SSRS 等服务。  

####<a id="other-content"></a>1.2.5 其他内容

#####<a id="sla"></a>1.2.5.1 服务级别 

在 Azure 环境下，采用 SQL Server 虚拟机，有三个非常重要的概念：  

1.	服务器 (Server)，定义了 SQL Server 主机  

2.	实例 (Instance)，一个 SQL Server 主机下可以有多个实例 (instance)  

3.	数据库 (Database)，一个 SQL Server Instance 下可以有多个 SQL Server 数据库  

以上三点是使用 SQL Server 虚拟机时需要注意的内容。

采用 Azure SQL 数据库，用户只需要关心数据库 (Database)，不需要关心服务器 (Server) 和实例 (Instance)。请注意：  

(1)	Azure SQL 数据库中的服务器是虚拟的。我们可以在同一个服务器下，创建若干个数据库。这若干个数据库之间是资源隔离的，不会产生资源竞争。  

(2)	Azure SQL 数据库，不存在实例 (Instance) 这个概念  

(3)	用户只需关心数据库 (Database)  

#####<a id="compatibility"></a>1.2.5.2 兼容性

在 Azure 环境下，采用 SQL Server 虚拟机，数据库引擎与传统的 SQL Server 2008, 2012, 2014 完全兼容。所以迁移传统的SQL Server 应用时，使用 SQL Server 虚拟机是最快速、最便捷的。  

Azure SQL 数据库是 SQL Server 2014 的子集，兼容大部分的 SQL Server 引擎服务，但是有部分功能不支持。所以在将传统 SQL Server 应用迁移到 Azure SQL 数据库时，需要对数据库进行优化。  

###<a id="azure-sql-database-advantage"></a>1.3 Azure SQL 数据库优势

####<a id="save-management-cost"></a>1.3.1 降低管理成本

由于不存在 SQL Server 虚拟机，因此用户无需对虚拟机的操作系统升级、补丁、数据库备份等进行管理，从而降低了管理成本。  
                                                                       
####<a id="data-redundancy"></a>1.3.2 数据冗余 

采用 Azure SQL 数据库，数据文件在底层做了三重冗余。事务在这三重冗余中保持同步。  

####<a id="sla-ensure"></a>1.3.3 服务级别协议服务保障  

采用 Azure SQL 数据库，数据库本身就提供高可用的能力。Azure SQL 数据库提供 99.99% 服务级别协议服务承诺。  

####<a id="auto-log-backup"></a>1.3.4 自动日志备份  

采用 Azure SQL 数据库，数据库本身提供自动日志备份的功能。  

Azure SQL 数据库可自动备份，每周进行一次全备份，每天进行一次差异备份，每 5 分钟进行一次日志备份。  

1.	对于基本服务层的 Azure SQL 数据库，备份自动保留 7 天。基本服务层的 Azure SQL 数据库可以回滚到当前时间之前 7 天内的任意时间点。  

2.	对于标准服务层的 Azure SQL 数据库，备份自动保留 14 天。标准服务层的 Azure SQL 数据库可以回滚到当前时间之前 14 天内的任意时间点。  

3.	对于高级服务层的 Azure SQL 数据库，备份自动保留 35 天。高级服务层的 Azure SQL 数据库可以回滚到当前时间之前 35 天内的任意时间点。  

Azure SQL 数据库第一次创建完成后，系统会自动进行一次全备份。
  
####<a id="read-only-geo-replication"></a>1.3.5 跨数据中心只读副本  

国内的数据中心建设是成对的，比如 Azure 北京数据中心和 Azure 上海数据中心。这是充分考虑了异地容灾能力的结果。北京和上海数据中心之间有专线连接，该专线用于数据中心之间进行数据同步。  

最新的 Azure SQL 数据库，同时支持基本、标准和高级服务层，都可以创建跨数据中心标准地域复制 (Standard Geo-Replication)，最多支持4个只读副本。  

这样可以将 Azure 其中一个数据中心作为主站点，另一个数据中心作为只读站点，实现跨数据中心的读写分离。  

####<a id="geo-fail-over"></a>1.3.6 跨数据中心故障转移

 如果 Azure SQL 数据库主站点在 Azure 上海数据中心，只读站点在 Azure 北京数据中心。当上海数据中心发生故障时，可以手动转移故障，将原来的主站点 (上海) 和只读站点 (北京) 做切换。即将 Azure 北京数据中心作为主站点，Azure 上海数据中心作为只读站点，这样可保证业务不会因为上海数据中心的故障而造成业务宕机。  

####<a id="seamless-upgrade"></a>1.3.7 数据库无缝升级

Azure SQL 数据库支持无缝升级。  

Azure SQL 数据库有一个性能指标，叫做 DTU。有关 DTU 的详细信息，请参考 DTU 章节。 

在新项目上线前的开发测试阶段，可以设置比较小的 DTU，比如基本服务层。因为开发测试用户访问量不大，使用基本服务层比较节省成本。  

在项目上线之前，我们可以通过 Azure 管理界面，将 Azure SQL 数据库升级到标准服务层或者高级服务层，因为生产环境对于 DTU 的要求比较高。  

> [ 注意事项 ] 通过 Azure 管理界面对 Azure SQL 数据库进行升级时，服务不会宕机或者重启，整个升级过程对客户端是透明的。  

###<a id="azure-sql-database-service-tier"></a>1.4 Azure SQL 数据库服务层

Azure SQL 数据库服务层分为三种，基本服务层、 标准服务层和高级服务层。具体请参考以下表格：  

![服务层][1]
 
1.	基本服务层只有一种性能级别  

2.	标准服务层分为四种性能级别：S0, S1, S2, S3  

3.	高级服务层分为六种性能级别：P1, P2, P4, P6, P11, P15
 
####<a id="basic-tier"></a>1.4.1 基本服务层

1.	基本服务层，DTU 为 5  

2.	数据库最大容量为 2 GB，只包含数据库文件，不包含日志文件  

3.	Max concurrent workers 为 30  

4.	Max concurrent logins 为 30  

5.	Max current sessions 为 300  

6.	备份自动保留 7 天。基本服务层的 Azure SQL 数据库可以回滚到当前时间之前7天内的任意时间点。  

7.	Azure SQL 数据库提供自动备份，每周一次全备份，每天一次差异备份，每 5 分钟一次日志备份  

####<a id="standard-tier"></a>1.4.2 标准服务层  

1.	标准服务层，分为四个不同的性能级别，S0, S1, S2, S3  

2.	数据库最大容量为 250 GB，只包含数据库文件，不包含日志文件  

3.	Max concurrent workers 根据不同的性能级别，为 60, 90, 120, 200  

4.	Max concurrent logins 根据不同的性能级别，为 60, 90, 120, 200  

5.	Max current sessions 根据不同的性能级别，为 600, 900, 1200, 2400  

6.	备份自动保留 14 天。标准服务层的 Azure SQL 数据库可以回滚到当前时间之前 14 天内的任意时间点。  

7.	Azure SQL 数据库提供自动备份，每周一次全备份，每天一次差异备份，每 5 分钟一次日志备份  

####<a id="premium-tier"></a>1.4.3 高级服务层  

1.	高级服务层，分为六个不同的性能级别，P1, P2, P4, P6, P11, P15

2.	P1 – P6 数据库最大容量为 500 G，只包含数据库文件，不包含日志文件  

3.	P11, P15 数据库最大容量为 1 TB  

4.	Max concurrent workers 根据不同的性能级别，提供不同的性能指标  

5.	Max concurrent logins 根据不同的性能级别，提供不同的性能指标  

6.	Max concurrent sessions 根据不同的性能级别，提供不同的性能指标  

7.	备份自动保留 35 天。高级服务层的 Azure SQL 数据库可以回滚到当前时间之前 35 天内任意时间点。  

8.	Azure SQL 数据库提供自动备份，每周一次全备份，每天一次差异备份，每 5 分钟一次日志备份  

###<a id="azure-sql-database-concept"></a>1.5 Azure SQL 数据库相关知识介绍

####<a id="azure-sql-database-concept-dtu"></a>1.5.1 DTU  

在 Azure SQL 数据库里，有一个非常重要的性能指标，叫做 DTU。  

数据库事务单位 (DTU) 是 SQL 数据库中的度量单位，表示数据库基于实际度量（数据库事务）的相对性能。我们执行了针对联机事务处理 (OLTP) 请求通常需要执行的一组操作，然后度量了在全部加载的条件下每秒可以完成的事务量。 
   
简单理解 DTU，就是 DTU 数值越大，则该 SQL Azure Database 性能越好。所以高级服务层的 SQL Azure Database 性能最好，标准服务层次之，最后才是基本服务层。

举个例子，我们在 1.4 节中可以观察到，基本服务层的 DTU 为 5，而 标准 S3 的 DTU 为 100。则 标准 S3 的数据库性能比基本服务层的性能高 20 倍。  

总结来说，基本服务层适合于开发测试，或者并发性能比较小的情况。 标准服务层适合于一般的业务场景。 高级服务层适合并发请求比较大的情况。

高级服务层的 P2 级别的 Azure SQL 数据库，性能要略好于 2 台 A7 (8 Core / 56 GB) SQL VM 设置 Always-On。

> [ 注意事项 ] 如果要迁移现有的 SQL Server 数据库，可使用第三方工具 [Azure SQL 数据库 DTU 计算器](http://dtucalculator.azurewebsites.net/)对数据库在 Azure SQL 数据库中可能需要的性能级别和服务层进行估算。 

如果把现有的 SQL Server VM 数据库迁移到 SQL Azure 平台，可以采用 Azure SQL 数据库 DTU 计算器，运行 Azure PowerShell 收集性能计数器，然后上传到 Azure SQL 数据库 DTU 计算器，进行计算和评估。  

####<a id="azure-sql-database-concept-v12"></a>1.5.2 V12  

在 Azure SQL 数据库里，有一个非常重要的特性叫做 SQL Database V12。  

创建新的 Azure SQL 数据库时，系统会提示用户是否需要创建 Azure SQL 数据库 V12 版本，相比之前的 V11 版本，V12 版本的优势如下：  

#####<a id="improve-compatibility"></a>1.5.2.1 兼容性提高  

 1.2.5.1 小节中介绍了Azure SQL 数据库是 SQL Server 2014 的子集，兼容大部分的 SQL Server 引擎服务。使用 Azure SQL 数据库 V12 版本，兼容以下 SQL Server 特性：  

1.	[支持 JSON](https://msdn.microsoft.com/zh-cn/library/dn921897.aspx)  

2.	[Windows 函数](https://msdn.microsoft.com/zh-cn/library/ms189798.aspx)以及[OVER](https://msdn.microsoft.com/zh-cn/library/ms189461.aspx)  

3.	[XML indexes](https://msdn.microsoft.com/zh-cn/library/bb934097.aspx) 和 [selective XML indexes](https://msdn.microsoft.com/zh-cn/library/jj670104.aspx)  

4.	[更改跟踪](https://msdn.microsoft.com/zh-cn/library/bb933875.aspx)  

5.	[INTO 子句](https://msdn.microsoft.com/zh-cn/library/ms188029.aspx)  

6.	[全文搜索](https://msdn.microsoft.com/zh-cn/library/ms142571.aspx)  

7.	[ALTER DATABASE SCOPED CONFIGURATION (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx)  
  
#####<a id="improve-performance-level"></a>1.5.2.2 性能级别提高 

在 V12 版本里，Azure SQL 数据库所有服务层 (基本, 标准, 高级) 的 DTU 性能都提高了 25%，且不会增加任何成本。除此以外，V12 还有其他新的特性，包括：  

1.	支持[列存储索引](https://msdn.microsoft.com/zh-cn/library/gg492153.aspx)  

2.	支持对已经进行表分区的数据，进行 [truncate](https://msdn.microsoft.com/zh-cn/library/ms177570.aspx) 操作  

3.	支持动态管理视图 (Dynamic Management Views, [DMVs](https://msdn.microsoft.com/zh-cn/library/ms188754.aspx))，监控和调试数据库性能  

#####<a id="better-support-for-cloud-saas-providers"></a>1.5.2.3 更好的支持云 SaaS 供应商

在 V12 版本里，新增了标准的 S3，并且提供了弹性数据库池，使用弹性数据库池可以：  

1.	若干个 Azure SQL 数据库共享 DTU，以降低使用成本  

2.	使用弹性数据库作业，管理数据库

#####<a id="improve-security"></a>1.5.2.4 安全性增加  

在 V12 版本里，支持以下安全特性： 

1.	[行级安全性(RLS)](https://msdn.microsoft.com/zh-cn/library/dn765131.aspx)  

2.	[动态数据屏蔽](/documentation/articles/sql-database-dynamic-data-masking-get-started/)  

3.	[包含的数据库用户](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)  

4.	使用GRANT, DENY, REVOKE 管理[应用程序角色](https://msdn.microsoft.com/zh-cn/library/ms190998.aspx)  

5.	[透明数据加密 (TDE)](https://msdn.microsoft.com/zh-cn/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx)  

6.	[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)  

7.	[始终加密](https://msdn.microsoft.com/zh-cn/library/mt163865.aspx) (预览)  

#####<a id="recommend-user-upgrade-to-v12"></a>1.5.2.5 建议用户升级 Azure SQL 数据库 V12  

1.	Azure SQL 数据库 V12 支持的特性比 V11 更多  

2.	V12 会持续增加新的特性  

3.	很多其他新的特性会先增加到 Azure SQL 数据库 V12，然后再增加到 Microsoft SQL Server  

#####<a id="how-to-check-azure-sql-database-version"></a>1.5.2.6 如何查看当前的 Azure SQL 数据库版本  

1.	登陆 Azure 管理界面  

2.	查看 Azure SQL 数据库的图标  

3.	如果显示的是 ![v12][2] 则当前的数据库版本是 V12  

4.	如果显示的是 ![old][3] 则当前的数据库版本是老版本  

5.	另外一种方法是连接到 Azure SQL 数据库后，可以执行 `SELECT @@version`，查看当前的数据库版本  

6.	如果显示的是 12.0.2000.10 ，则当前版本是 V12  

7.	如果显示的是 11.0.9228.18 ，则当前版本是 V11

> [ 注意事项 ] V12 数据库只能托管在 V12 服务器 (注意 Azure SQL 数据库中的服务器是虚拟的 )。 V12 服务器只能托管 V12 数据库。 如果你的 Azure SQL 数据库不是 V12 版本，需要把当前数据库所在的服务器 (注意 Azure SQL 数据库中的服务器是虚拟的 ) 升级到 V12 版本。  

###<a id="azure-sql-database-considerations"></a>1.6 使用 Azure SQL 数据库注意事项  

####<a id="azure-sql-database-max-capacity"></a>1.6.1 数据库最大容量  

我们在 1.4 小节中可以观察到 Azure SQL 数据库限制了数据库最大容量：  

1.	基本服务层最大容量为 2 GB  

2.	标准服务层最大容量为 250 GB  

3.	高级服务层 P1 - P6 数据库最大容量为 500 GB  

4.	高级服务层 P11, P15 数据库最大容量为 1 TB  

####<a id="reduce-concurrent-requests"></a>1.6.2 并发请求减少  

我们在 1.4 节中可以观察到 Azure SQL 数据库不仅仅限制了数据库的最大容量，而且限制了 Max concurrent workers，Max concurrent logins，Max concurrent sessions。如下表所示：  
![服务层级和性能][4]
 
上图可以以标准服务层的 S3 为例。  

1.	Max concurrent Workers 为 200  

2.	Max concurrent Logins 为 200  

3.	Max concurrent sessions 为 2400

使用 Visual Studio Test Agent 或者 Load Runner 等压力测试软件，对 Azure SQL 数据库进行压力测试时，如果并发数小于等于 200，Azure SQL 数据库正常提供服务。  

但是如果并发用户数大于 200，在进行压力测试时，会产生 Server Error 500 错误。因为 Azure SQL 数据库对 Max concurrent 进行了限制。

结合到业务场景中，需要减少业务系统对 Azure SQL 数据库的并发。举个例子，某个企业在线培训系统的后台数据库采用 Azure SQL 数据库，在选择 Azure SQL 数据库服务层的时候，需考虑以下几点：  

1.	Max Storage，只包含数据库文件，不包含日志文件  

2.	DTU  

3.	Max concurrent Workers  

4.	Max concurrent Logins  

5.	Max concurrent sessions  

业务高峰时，最大并发请求为 400。从上表中发现，至少需要选择高级服务层的 P2 (Max concurrent Worker 和 Max concurrent Logins 都是 400)，这样才能满足数据库的最大并发请求。

或者，可以把经常需要查询的内容保存到缓存中 ( 比如保存到 Azure Redis 缓存中 )，以降低对 Azure SQL 数据库的并发请求。

####<a id="azure-sql-database-partitioning"></a>1.6.3 数据库切片  

很多大型的业务系统采用单一 Azure SQL 数据库，会遇到各种各样的问题。举个例子：  

1.	A 用户的业务系统，数据库采用 Azure SQL 数据库，保存全国用户的信息数据 

2.	该业务系统每个月的数据库容量增长 40 GB

3.	该业务系统最大的并发请求为 1000  

4.	对 DTU 的需求也非常大

针对上述现状，客户如果选择单一的 Azure SQL 数据库，只能选择高级服务层的 P6 (Max concurrent 为 1600)，价格较贵。  

而且考虑到业务是逐渐增长的，后期还可能将数据库升级到高级服务层的 P15。如果业务需求再增加，可能 P15 级别的数据库也无法满足业务压力。单个节点向上扩展是有限的。  

所以如果把所有的业务数据保存到一个数据库中，这个数据库就会产生性能瓶颈。如下图：  

![单个数据库性能瓶颈][5]
 
所以可以把数据表进行切片，下面介绍横向切片 (Horizontal partitioning)。  

![数据切片表][6]
 
上图把原本保存在同一个数据表中的数据，按照所在城市，拆分到不同数据库的数据表中。这样做的目的有以下几点:  

(1)	减少数据库的最大容量  

(2)	减少对同一张表频繁读写而产生的性能瓶颈，即降低 DTU  

(3)	减少数据库并发，即降低 Max concurrent

另外在进行数据库切片的时候，需注意上图中 DB\_A, DB\_B, DB\_C, DB\_D 这四个数据库是分离的。如果要引用其他相对静态的数据，比如上海市区县编码，全国省份编码，产品编码等表，需要考虑将上述三个表的内容，在四个 Azure SQL 数据库 (DB\_A, DB\_B, DB\_C, DB\_D)中进行冗余，避免出现跨数据库的表查询。

数据库表切片后，还要考虑对于不同的数据库，要设置不同的连接字符串。如下图： 

![设置不同的连接字符串][7]
 
####<a id="single-database-server-max-dtu"></a>1.6.4 单个数据库服务器限制最大 DTU  

上述内容介绍了 Azure SQL 数据库中的服务器是虚拟的，可在同一个 Azure SQL 数据库服务器下创建若干多个 Database，这若干个数据库之间是资源隔离的，不会产生资源竞争。但是 Azure 限制了单个 Azure SQL 数据库服务器的最大 DTU 为 45000。  

也就是说，可以在同一个 Azure SQL 数据库服务器里创建若干多个 Database，但是所有 Database 的 DTU 总和不能超过 45000。  

举个例子，Azure SQL 数据库高级服务层的 P4，DTU 是 500。  

而单个 Azure SQL 数据库服务器的最大 DTU 为 45000。  

那可以在一个 Azure SQL 数据库服务器下，最多创建 90 个 P4 级别的数据库 (45000 / 500 = 90)。  

当新创建第 91 个数据库时，Azure 会提示错误，因为这个 Azure SQL 数据库服务器的 DTU 已被用完。

####<a id="single-database-server-max-databases"></a>1.6.5 单个数据库服务器最多 5000 个 Database  

如标题所示，在单个数据库服务器下最多创建 5000 个 Database。  

举个例子，单个 Azure SQL 数据库服务器最大 DTU 为 45000。  

Azure SQL 数据库基本服务层的 DTU 是 5。  

在一个 Azure SQL 数据库服务器下最多创建 5000 个基本服务层级别的数据库。请注意：不是 9000 个 Database (45000 / 5 = 9000)。  

####<a id="azure-sql-database-backup"></a>1.6.6 数据库备份  

1.	基本服务层的 Azure SQL 数据库，备份自动保留 7 天  

2.	标准服务层下，备份自动保留 14 天  

3.	高级服务层下，备份自动保留 35 天

某些情况下，需要把数据备份的保留时间延长，这时候就需要结合 Azure 自动化功能，把 Azure SQL 数据库备份文件保存到 Azure 存储里。  

###<a id="azure-sql-database-limits"></a>1.7 Azure SQL 数据库的限制  

####<a id="azure-sql-database-default-time-zone"></a>1.7.1 默认时区为 UTC 时区  

在开发应用程序时，经常使用北京时间 (即 UTC+8 时区)。  

Azure SQL 数据库的时区默认为 UTC 时区，且无法进行修改和配置。这时候就需要修改 T-SQL 语句，将 UTC 时区修改为 UTC+8 时区。
   
####<a id="not-support-for-sql-agent"></a>1.7.2 暂不支持 SQL Agent  

在某些场景下，会使用 SQL Agent 执行 SQL Job。比如：  

1.	在凌晨 12 点计算库存变化  

2.	对数据库进行全备份，将备份文件保存到虚拟机本地磁盘 E 盘  

目前国内由世纪互联运营的 Microsoft Azure，暂时不支持使用 Azure SQL Agent。  

如果用户想在 Azure SQL 数据库执行 SQL Agent，需要结合 Azure 自动化功能。  

###<a id="pricing"></a>1.8 价格

有关 Azure SQL 数据库的价格，请参考[此处](/pricing/details/sql-database/)。  

请注意: Azure SQL 数据库按照不同的服务层来收费，即按照基本、标准和高级服务层来收费，不收取存储及事务费用。  


<!--image reference-->
[1]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-1.png
[2]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-2.png
[3]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-3.png
[4]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-4.png
[5]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-5.png
[6]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-6.png
[7]: ./media/azure-sql-database-user-manual-part-1/azure-sql-database-user-manual-7.png

