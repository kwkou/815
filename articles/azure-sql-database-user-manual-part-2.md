<properties
	pageTitle="Azure SQL 数据库用户手册 - 第二部分 | Azure"
	description=""
	services=""
	documentationCenter=""
	authors="Lei Zhang"
	manager=""
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date=""
	wacn.date="09/13/2016"/>

#Azure SQL 数据库用户手册

- [Azure SQL 数据库用户手册 - 第一部分](/documentation/articles/azure-sql-database-user-manual-part-1)

##<a id="azure-sql-database-get-started"></a>4.开始使用 Azure SQL 数据库  

介绍如何使用 Azure SQL 数据库。  

###<a id="create-azure-sql-database-server"></a>4.1 创建 Azure SQL 数据库服务器  

Azure SQL 数据库中的服务器是虚拟的，可在同一个服务器下创建若干个数据库。这些数据库之间是资源隔离的，不会产生资源竞争。  

1. 首先登陆 [Azure 管理门户](https://manage.windowsazure.cn/)  

2. 依次选择 SQL 数据库，服务器，创建 SQL 数据库服务器  

	![创建 SQL 数据库服务器][8]
 
3. 在弹出的窗口里，输入登陆的用户名和密码。如下图：  

![输入登陆的用户名和密码][9]
 
上图中有 3 个需要关注的点:  

(1)	区域。中国东部表示 Azure 上海数据中心，中国北部表示 Azure 北京数据中心。  

(2)	允许 Azure 服务访问服务器。因为访问 Azure SQL 数据库资源需要设置 IP 白名单。  

勾选表示任何部署在 Azure 上的服务，都可以访问创建好的 Azure SQL 数据库。  

使用诸如 Web 应用， 云服务， Azure 虚拟机等其他服务，都可以访问创建好的 Azure SQL 数据库。  

但会产生一个问题: 其他客户创建的云资源，也可以访问现在创建好的 Azure SQL 数据库。  

假设以下场景：前端 Web 服务器采用虚拟机，后端采用 Azure SQL 数据库。不勾选该选项，将前端 Web 服务器的 IP 地址固定好，然后把这个 Web 服务器的公网 IP 地址加入到 Azure SQL 数据库的 IP 白名单里。这样只有我们的 Web 服务器的 IP 才可以访问 Azure SQL 数据库，从而提高了安全性。  

(3)	直接使用最新的 V12 服务。

4.	创建完毕后，Azure 会开始创建新的 Azure SQL 数据库服务器，服务器名称是随机的。创建完毕后，如下图：  

![创建完毕][10]
 
上图中可发现以下几点:  

(1)	新创建的服务器名称是随机的  

(2)	上图服务器已启用 V12 功能  

(3)	该服务器可用的 DTU 为 45000 

5.	请注意，我们只创建服务器，不创建数据库，不收取任何费用。  

6.	因为我们是首次使用 Azure SQL 数据库，所以先介绍创建服务器，再创建数据库。如果您熟悉 Azure SQL 数据库，也可以在创建数据库的同时创建服务器。  

###<a id="create-azure-sql-database"></a>4.2 创建 Azure SQL 数据库 

之前已成功创建好服务器，现在开始创建 Azure SQL 数据库。  

1.	点击“创建 SQL 数据库”  

2.	在弹出的界面中，输入如下信息:  

![输入信息][11]
 
(1)	上图设置数据库名称为 OrderDB  

(2)	性能级别选择 S0，10 个 DTU  

(3)	数据库最大容量为 250 GB  

(4)	排序规则选择默认值  

(5)	服务器选择上一节中创建的服务器。该 Server 总的 DTU 为 45000，创建完 S0 后，剩余的 DTU 为 45000 - 10 = 44990  

(6)	创数据库完毕后，如下图:  

![创数据库完毕][12]
 
> [ 注意事项 ]：创建完 Azure SQL 数据库，不管有没有客户端连接，都开始计费。  


(7)	继续创建另一个数据库，命名为 CRMDB。步骤同上。  

###<a id="connect-to-azure-sql-database"></a>4.3 连接 Azure SQL 数据库(重要性高)  

本节将介绍如何使用本地计算机的 SQL Server Management Studio(SSMS) 连接创建好的 Azure SQL 数据库。这里使用的是 SQL 2014 的 SSMS。  

1.点击下图的第一列名称，页面跳转：  

![页面跳转][13]
 
2.页面跳转。点击下图的“显示连接字符串”：  

![显示连接字符串][14]
 
3.在弹出的窗口中，查看到连接字符串。如下图：  

![查看连接字符串][15]
 
保存连接字符串内容到记事本上  

	ew79sank1x.database.chinacloudapi.cn,1433 

> [ 注意事项 ] 请注意: 上图的 ADO.NET 连接字符串，包含关键字 Encrypt=True，也就是可以通过 SSL 连接 Azure SQL 数据库，这就需要把本地的证书上传到 Azure。具体步骤请参考博客:  http://www.cnblogs.com/threestone/p/4001632.html 

4.打开本地计算机的 SSMS，输入上面的连接字符串，并输入在 4.1 小节创建的 Server Login Name 和 Password。  

![输入连接字符串][16]
 
5.点击连接，发现 SSMS 报错。在 4.1 小节中介绍过，访问 Azure SQL 数据库资源需要设置 IP 白名单。因为本机计算的出口 IP 地址没有设置在 IP 白名单里。  

6.

![error][17]
 

7.回到 Azure 管理门户，点击下图的“管理允许的 IP 地址”：  

![允许 IP 地址][18]
 
页面跳转，把当前计算机的 IP 地址加入到 IP 白名单里。如下图： 
 
![IP 白名单][19]
 
设置完 IP 白名单后，就可以通过 SSMS 直接连接到 Azure SQL 数据库。如下图：  

![连接到 Azure SQL 数据库][20]
 
###<a id="encrypt-connection"></a>4.4 加密连接  

从4.3 小节中了解到，可以查看到 Azure SQL 数据库的连接字符串，如下图:  

![连接字符串][21]
 
> [ 注意事项 ] 请注意: 上图的 ADO.NET 连接字符串，包含关键字 Encrypt=True，也就是可以通过 SSL 连接 Azure SQL 数据库，这就需要把本地的证书上传到 Azure。具体步骤请参考博客: http://www.cnblogs.com/threestone/p/4001632.html  

证书上传后，可以通过本地计算机的 SQL Server Management Studio (SSMS) 加密连接到 Azure SQL 数据库。  

1.打开本地计算机的SSMS，输入相应的连接字符串，然后点击下图的选项：  

![SSMS][22]
 
2.在下图中选择“连接属性”，然后点击“加密连接”。  

![加密连接][23]
 
这样就可以通过加密方式，连接到 Azure SQL 数据库。  

###<a id="start-to-use"></a>4.5 开始使用  

上一节已经成功连接到 Azure SQL 数据库，本节将介绍如何使用。  

####<a id="check-database-version"></a>4.5.1 查看数据库版本

在新建查询中，输入 `SELECT @@version`  

![新建查询][24]

 可以查看到这个数据库的版本是 12.0.2000，这个是 V12 版本。  

####<a id="check-database-time"></a>4.5.2 查看数据库时间
输入 `SELECT GetDate()`，可以看到 Azure SQL 数据库默认时间为 UTC 时间，且无法进行修改和配置。本地计算的时间是 UTC+8，即北京时间。  

![数据库时间][25]
 
####<a id="unicode"></a>4.5.3 Unicode  

执行以下语句：  

	create table dbo.ChnStudent
	(
		studentnumber int identity(1,1) not null,
		value nvarchar(100) not null,
	)
	Go

	insert into ChnStudent(value) values 
	('小张'),('小李'),(N'小张'),(N'小张')

	select * from dbo.ChnStudent

执行结果：  

![执行结果][26]
 
可以看到，插入 Unicode 时，需要在插入值之前加入大写 N，才不会出现乱码。  

###<a id="sql-server-vm-migrate-to-azure-sql-database"></a>4.6 将 SQL Server VM 迁移到 Azure SQL 数据库  

如果在托管机房 IDC 已使用 SQL Server 2008, 2012 虚拟机，现在想使用基于云端的数据库 Azure SQL 数据库，这时候就需要考虑迁移。

参考资料：  

[将 SQL Server 数据库迁移到云中的 SQL 数据库](/documentation/articles/sql-database-cloud-migrate)

这篇用户手册简单介绍迁移到 Azure SQL 数据库的方法，分为三种：(更多方法，请参考上面的连接)  

1.	推荐使用：SQL Server Data Tools for Visual Studio (SSDT)  

2.	导出应用层数据(Data Tier wizard)  

3.	SQL Azure Migration Wizard (SAMW)  

####<a id="migrate-consideration"></a>4.6.1 注意事项

之前章节介绍了 Azure SQL 数据库限制了 Max concurrent workers，Max concurrent logins，Max concurrent sessions。  

将本地 SQL Server 迁移到云端 Azure SQL 数据库时，需要在云端数据库插入大量的数据。为了加快迁移时间，提高迁移速度，建议在迁移前，提高 Azure SQL 数据库的数据库性能。 迁移完毕后，再降低数据库性能，以节省成本。  

####<a id="ssdt"></a>4.6.2 SSDT
参考资料：  

[使用 SQL Server Data Tools for Visual Studio 将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt) 

前提要求：  

1.	在本地计算机，安装 SQL Server2005, 2008, 2012,2014 数据库服务  

2.	在本地计算机，安装 Visual Studio 2012, 2013, 2015  

3.	在本地计算机，安装最新的 [SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)


使用 SSDT 有以下关键步骤:  

1.	使用 Visual Studio SQL Server Object Explorer，连接到本地计算机的 SQL Server 数据库
2.	创建新的项目
3.	修改兼容性设置为 Azure SQL 数据库 V12
4.	编译并修改不兼容的数据库内容
5.	将已修改好的数据库副本发布到本机计算机
6.	将副本数据库与源数据库进行比较
7.	将副本数据库的 Table Schema 发布到 Azure SQL 数据库 V12
8.	使用 BCP 工具，将行数据导入到 Azure SQL 数据库 V12

接下来介绍详细内容:  

1.以管理员身份，运行 Visual Studio 2013，然后点击 View, SQL Server Object Explorer，图略。  

2.连接到本地计算机的 SQL Server 数据库，然后点击 Create New Project

![Create New Project][27]

3.在弹出的窗口中，设置项目的本地磁盘路径，然后选择 Import Application-scoped object only。如下图：

![Import Application-scoped object only][28]
 
4.点击上图的 Start，将会开始导入数据库 T-SQL 脚本。如下图：  

![T-SQL 脚本][29]
 
5.在 Visual Studio 项目文件中，选择该项目，然后点击右键。在 Project Setting 页面里，选择 Target Platform 为 Azure SQL 数据库  V12。如下图:

![Target Platform][30]
 

6.然后点击项目，右键，点击 Build 。如下图：  

![Build][31]
 

7.在 Error List 中，查看不兼容的 T-SQL 语句。如下图：  

![不兼容的 T-SQL语句][32]
 
8.编译并修改不兼容的数据库内容  

![编译并修改][33]
 

9.再次编译并检查错误内容，直到项目不含任何报错信息。  

![再次编译并检查][34]
 
10.将已经修改好的数据库副本发布到本机计算机  

![发布其副本到本机计算机一][35]  
![发布其副本到本机计算机二][36]

11.在 SQL Server Object Explorer 里点击副本数据库，与源数据库比较  

![与源数据库比较一][37]  
![与源数据库比较二][38]
 

12.检查副本数据库与源数据库的差异。  

![与源数据库的差异][39]
 
可以观察到新的副本数据库只包含 Table Schema，不包含行数据

13.将项目发布到 Azure SQL 数据库 V12，即在云端创建 Table Schema。点击项目文件，右键 Publish

![Publish][40]
 

14.在弹出的窗口中，输入Azure SQL 数据库的用户名、密码，并输入之前的数据库名称。

![数据库的用户名、密码][41]  
![输入数据库名称][42] 


15.点击上图的 Publish，Visual Studio 就会把副本数据库的 Table Schema 发布到 Azure SQL 数据库 V12  

![Table Schema 发布][43] 
 

16.Table Schema 发布完以后，可查看到发布结果。  

![发布结果][44] 
 
17.最后可以通过 BCP 工具，把本地 SQL Serve r的表数据，插入到 Azure SQL 数据库的表中  


####<a id="export-data-tier-application"></a>4.6.3 Export Data Tier Application  

本节介绍如何使用 SQL Server Management Studio (SSMS)，直接把本地计算的 SQL Server 数据库，导出 BACPAC 文件到本地磁盘，把 BACPAC 文件上传到 Azure China Storage 存储账号，最后通过 BACPAC 导入到 Azure SQL 数据库里。

#####<a id="export-data-tier-application-considerations"></a>4.6.3.1 注意事项

因为 BACPAC 文件包含了数据库 Schema 和行数据，如果需要迁移非常大的数据库，则通过本地计算机，将 BACPAC 还原到 Azure SQL 数据库的速度会非常慢。建议按照以下方法操作:  

1.	在 Azure 云端创建一台预装了 SQL Server 2012 或 2014 的虚拟机  

2.	把本地计算的 BACPAC 文件，通过远程桌面连接，上传到 Azure 云端机器的本地磁盘，请注意不要保存到 D 盘，D 盘是临时盘，不能保存持久化数据  

3.	通过 Azure 云端机器的 SSMS，将 BACPAC 文件还原到 Azure SQL 数据库

#####<a id="export-data-tier-application-prerequisite"></a>4.6.3.2 前提要求

安装最新的 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx)  

#####<a id="export-data-tier-application-details"></a>4.6.3.3 详细内容  

1.通过 SSMS，把本地 SQL Server 数据库导，导出 BACPAC 文件到本地磁盘我们选择相应的数据库，邮件，Tasks，Export Data-tier Application。  

![导出 BACPAC 文件][45]
 
2.选择保存到本地磁盘。  

![保存到本地磁盘][46]
 
3.默认情况下，BACPAC 文件保存该 Database 下的所有对象。如果想选择迁移某些对象，可以点击下图的“高级”。 

![高级][47]
 
4.SSMS开始导出内容  

![开始导出][48]
 
5.如果导出过程中发生任何错误，请参考保存信息，并修改响应错误内容。  

![错误][49]
 
6.在 Azure 云端创建一台预装了 SQL Server 2012 或 2014 的虚拟机，步骤略。  

7.把本地计算的 BACPAC 文件，通过远程桌面连接，上传到 Azure 云端机器的本地磁盘，请注意不要保存到 D 盘，D 盘是临时盘，不能保存持久化数据  

8.通过 Azure 云端机器的 SSMS，连接到 Azure SQL 数据库。选择数据库，右键，点击导入数据层应用程序。如下图：  

![导入数据层应用程序][50]
 
9.输入新数据库名称，设置相应的数据库版本信息。  

![数据库版本信息][51]

10.为了加快迁移时间，可以在上图的数据库版本，选择“高级”级别的数据库版本，如下图：  

![选择“高级”级别的数据库版本][52]

11.等待导入 BACPAC 完毕  

![导入BACPAC完毕][53]
 
12.可以查看到导入成功  

![导入成功][54]
 
####<a id="sql-azure-migration-wizard"></a>4.6.4 SQL Azure Migration Wizard

SQL Azure Migration Wizard 迁移工具，可以协助用户将本地 SQL Server 数据库迁移到 Azure SQL 数据库。具体下载地址：http://sqlazuremw.codeplex.com/  
 
####<a id="other-migration-tool"></a>4.6.5 其他迁移工具

其他迁移工具，请参考[此文](/documentation/articles/sql-database-cloud-migrate) 
<!--
###<a id=""></a>4.7 使用外部表进行跨库查询  

以后再写  
-->
###<a id="restore-database"></a>4.7 数据库还原

Azure SQL 数据库不同的服务层，提供 7 天、14 天、35 天的数据库备份。可针对当前数据库进行数据库还原。 

1.点击需要还原的数据库名称  

![数据库名称][55]
 
2.点击仪表板，还原，如下图：  

![还原][56]
 
3.在弹出的窗口中，选择数据库还原点  

![选择数据库还原点][57]
 
上图中：  

(1)	数据库名称可以重命名  

(2)	请注意还原数据库的过程，其实是在同一个数据库服务器下，创建一个新的还原数据库。  

(3)	标准服务层的数据库，备份自动保留 14 天。可以回滚到当前时间之前14 天内的任意时间点。  

(4)	当天时间是 2016 年 5 月 31 日，当前的数据库服务层是标准。  

(5)	最早的还原点是 2016 年 5 月 17 日，即 14 天之前。

###<a id="monitoring"></a>4.8 监控  

Azure SQL 数据库提供了内置的监控功能，用户可以监控数据库的不同性能指标。包括：  

1.	CPU percentage 
2.	Data IO percentage 
3.	Database size percentage  
4.	DTU limit 
5.	DTU percentage  
6.	DTU used 
7.	In-Memory OLTP storage percent(Preview) 
8.	Log IO percentage 
9.	Sessions percentage 
10.	Workers percentage 
11.	成功的连接 
12.	存储空间 
13.	失败的连接  
14.	死锁 
15.	由防火墙阻止

1.可以点击需要监控的数据库名称，如下图：  

![数据库名称][58]
 
2.页面跳转，选择监控，如下图：  

![选择监控][59]
 
上图中，可以查看：  

(1)	选择右上角，查看过去 1 小时，24 小时，7 天，14 天的性能监控  

(2)	可以看到在过去 24 小时内，有一段时间的 DTU 达到了 100%，则表明在该时间范围内，数据库的性能达到瓶颈，需要考虑数据库升级  

(3)	如果默认的性能指标不满足监控需求，则可以点击上图的“添加度量值”按钮  

(4)	在弹出的窗口中，选择需要的度量值。如下图:  

![度量值][60]
 

###<a id="scale-up-service-tier-and-performance-level"></a>4.9 切换数据库服务层和性能级别

参考资料:
[更改 SQL 数据库的服务层和性能级别](/documentation/articles/sql-database-scale-up)  
 
之前介绍了 Azure SQL 数据库支持无缝升级。  

打个比方，即将上线的一个新项目在开发测试时，可以设置比较小的 DTU，比如基本服务层。因为开发测试用户访问量不大，使用基本服务层也比较节省成本。  

在项目上线之前，可以通过 Azure 管理界面将 Azure SQL 数据库升级到标准服务层或者是高级服务层。因为生产环境对 DTU 的要求会比较高。  

####<a id="scale-up-service-tier-and-performance-level-considerations"></a>4.9.1 注意事项

请注意，Azure SQL 数据库在切换过程中，是对当前数据库按照新的性能指标，创建了一个新的副本，然后把客户端从当前数据库，切换到新的副本数据库。  

在切换过程中数据不会丢失，但是会出现数据库连接被关闭的情况，所以有一些事务会回滚 (roll back)。这个过程时间长短不定，但平均时间小于 4 秒，超过 99% 的情况下小于 30 秒。极少数情况下，数据库连接关闭时，大量的事务处于提交过程中，切换过程可能花费较长时间。  

Azure SQL 数据库的切换时间，取决于数据库容量和数据库的性能指标。举例来说，一个容量为 250 GB 的标准服务层的数据库，切换到其他标准服务层的数据库，整个过程需花费 6 个小时。同样容量为 250 GB 的高级服务层的数据库，切换到其他高级服务层的数据库，整个过程需花费 3 个小时。  

####<a id="evaluate-before-scale-up"></a>4.9.2 切换前的评估

1.	评估数据库容量大小  

	举个例子，高级服务层的数据库，最大容量为 500 GB。当切换到标准服务层的时候，因为标准服务层最大容量为250 GB，小于 500 GB，会出现切换失败的情况。

2.	评估自动备份时间  

	基本服务层的 Azure SQL 数据库，备份自动保留 7 天  
	标准服务层下，备份自动保留 14 天  
	高级服务层下，备份自动保留 35 天  
	在切换之前，需要进行相应的评估。  

3.	在对数据库性能降级 (基本服务层, 标准服务层, 高级服务层之间切换)时，首先需要删除跨数据中心只读副本

4.	评估 Max Concurrent  

	不同服务层的 Azure SQL 数据库，提供不同的 Max concurrent workers，Max concurrent logins，Max concurrent sessions。  

####<a id="start-scale-up"></a>4.9.3 开始切换  

1.点击需要切换的数据库名称  

![数据库名称][61]
 
2.选择不同的服务层，性能级别和最大数据库容量。如下图红色部分：  

![容量大小][62]
 
3.点击上图的保存后，Azure SQL 数据库开始切换。可以在界面上看到“切换开始”。如下图：  

![切换开始][63]
 
4.在数据库切换过程中，客户端仍可以正常连接到这个数据库  

###<a id="standard-geo-replication"></a>4.10 跨数据中心标准地域复制(Standard Geo-Replication)  

本节内容摘自博客： http://www.cnblogs.com/threestone/p/5481911.html  
 
####<a id="standard-geo-replication-instruction"></a>4.10.1 说明  

Azure SQL 数据库提供不同等级的、跨数据中心的异地冗余功能。  

1.	基本服务层模式：不提供异地冗余能力  

2.	标准服务层模式：提供跨数据中心的异地冗余数据库。但这个冗余数据库是冷备份，无法提供读取操作  

3.	高级服务层模式：提供只读跨数据中心的异地冗余数据库。这个冗余数据库只能提供读操作

如果用户的 Azure SQL 数据库需要较低的 DTU，但需要跨数据中心的异地冗余能力，则需把 SQL Azure 的性能升级到高级服务层级别，只有 P 级别只读异地冗余的数据库。  

现在最新的 SQL Azure，同时支持基本服务层, 标准服务层和高级服务层三个级别，都可以创建跨数据中心标准地域复制 (Standard Geo-Replication)，最多支持4个只读副本。

而且 SQL Azure Standard Geo-Replication 支持故障转移。  

 SQL Azure 主站点在 Azure 上海数据中心，只读站点在 Azure 北京数据中心。上海数据中心发生故障时，可以手动转移故障，将原来的主站点(上海)和只读站点(北京)做切换，即将 Azure 北京数据中心作为主站点，Azure 上海数据中心作为只读站点。这样可保证业务不会因为上海数据中心发生故障，造成业务宕机。

灾难恢复演练(Disaster Recovery Drills)  

请注意，SQL Azure 故障转移和数据有关，是破坏性方法，所以需要周期性的测试故障转移工作流，确保应用程序的一致性和稳定性，这个过程称为灾难恢复演练(Disaster Recovery Drills)。可按如下方法测试数据库灾难恢复演练：关闭跨数据中心标准地域复制(Standard Geo-Replication)。  

注意，当关闭跨数据中心标准地域复制 (Standard Geo-Replication) 时，在主站点已经提交的事务，如果没有在备份节点提交，这些事务会丢失。由于可能会产生数据丢失的风险，因此不推荐在生产环境里实施灾难恢复演练 (DR Drills)。建议在主站点数据中心创建一个测试数据库，然后对这个数据库实施灾难恢复演练。  

####<a id="standard-geo-replication-demo"></a>4.10.2 演示

接下来介绍演示内容，演示中有几个关键步骤：

1.注意这里牵涉到 Azure Resource Group 的概念，默认情况下，Azure 上海数据中心的 ResourceGroupName 为 Default-SQL-ChinaEast  

2.Azure 北京数据中心的 ResourceGroupName 为 Default-SQL-ChinaNorth  

3.在 Azure 上海数据中心创建 SQL Azure Server，在北京数据中心创建 SQL Azure Server  

4.在上海数据中心创建数据库 TestDB  

5.运行 Azure PowerShell，在北京数据中心创建只读数据库  

6.验证上海数据中心的数据库是可读写，北京数据库中心的数据库是只读  

7.运行 Azure PowerShell，设置故障转移 (Failover)。设置完毕后，北京数据中心可读写，上海数据中心只读。

接下来开始演示内容：  

1.在 Azure 上海数据中心创建 SQL Azure Server，在北京数据中心创建 SQL Azure Server  

![创建数据库][64]
 
创建完毕后，Azure 上海数据中心 SQL Azure Server：  

	hfgmi3msar.database.chinacloudapi.cn,1433

Azure 上海北京数据中心 SQL Azure Server：  

	dbcljcn986.database.chinacloudapi.cn,1433

2.在上海数据中心创建数据库 TestDB  

![创建数据库 TestDB 一][65]  

![创建数据库 TestDB 二][66]
 
执行完毕后，上海站点是主站点，SQL Azure 数据可读写。北京站点是只读站点，SQL Azure 数据只读。

3.Azure PowerShell 步骤如下  

	#弹出界面输入用户名密码
	Add-AzureRmAccount -EnvironmentName AzureChinaCloud

	#设置当前订阅名称
	Select-AzureRmSubscription –SubscriptionName "Internal Billing" |  Select-AzureRmSubscription

	Get-AzureRmResourceGroup | Get-AzureRmSqlServer

	#通过Management Portal ，在上海创建新的Server: hfgmi3msar，新的Database: LeiDB
	#通过Management Portal，在北京创建新的Server：dbcljcn986，但是不创建新的Database

	#执行下面的脚本，在北京创建只读库
	$database1 = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaEast" –ServerName "hfgmi3msar"

	$secondaryLink = $database1 | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaNorth" –PartnerServerName "dbcljcn986" -AllowConnections "All"

	#Shanghai读写的连接字符串
	#hfgmi3msar.database.chinacloudapi.cn,1433

	#Beijing只读的连接字符串
	#dbcljcn986.database.chinacloudapi.cn,1433

	#Failover, 北京Database，变成读写，上海Database只读
	#上海Server: hfgmi3msar 只读
	$database_beijing = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaNorth" –ServerName "dbcljcn986" 

	$database_beijing | Set-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaEast" -Failover


	#Failover, 上海Database读写，北京Database只读
	$database_shanghai = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaEast" –ServerName "hfgmi3msar" 

	$database_shanghai | Set-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaNorth" -Failover

4.上面的 PowerShell 分为几部分：  

5.当执行以下脚本时，会在备份站点 Azure 北京数据中心创建只读数据库  

	#执行下面的脚本，在北京创建只读库
	$database1 = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaEast" –ServerName "hfgmi3msar"

	$secondaryLink = $database1 | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaNorth" –PartnerServerName "dbcljcn986" -AllowConnections "All"
 
![创建只读数据库][67]  


6.当执行以下脚本的时候，主站点会变成 Azure 北京数据中心，备份站点为 Azure 上海数据中心  

	#Failover, 北京Database，变成读写，上海Database只读
	#上海Server: hfgmi3msar 只读
	$database_beijing = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaNorth" –ServerName "dbcljcn986" 

	$database_beijing | Set-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaEast" -Failover

执行结果，如下图：  

![执行结果][68]
 
7.当再次执行以下脚本时，Azure 上海站点重新变成主站点，Azure 北京站点变成备份站点。  

	#Failover, 上海Database读写，北京Database只读
	$database_shanghai = Get-AzureRmSqlDatabase –DatabaseName "TestDB" –ResourceGroupName "Default-SQL-ChinaEast" –ServerName "hfgmi3msar" 

	$database_shanghai | Set-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "Default-SQL-ChinaNorth" -Failover

<!--image reference-->
[8]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-8.png
[9]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-9.png
[10]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-10.png
[11]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-11.png
[12]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-12.png
[13]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-13.png
[14]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-14.png
[15]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-15.png
[16]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-16.png
[17]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-17.png
[18]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-18.png
[19]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-19.png
[20]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-20.png
[21]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-21.png
[22]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-22.png
[23]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-23.png
[24]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-24.png
[25]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-25.png
[26]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-26.png
[27]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-27.png
[28]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-28.png
[29]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-29.png
[30]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-30.png
[31]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-31.png
[32]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-32.png
[33]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-33.png
[34]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-34.png
[35]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-35.png
[36]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-36.png
[37]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-37.png
[38]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-38.png
[39]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-39.png
[40]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-40.png
[41]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-41.png
[42]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-42.png
[43]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-43.png
[44]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-44.png
[45]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-45.png
[46]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-46.png
[47]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-47.png
[48]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-48.png
[49]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-49.png
[50]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-50.png
[51]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-51.png
[52]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-52.png
[53]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-53.png
[54]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-54.png
[55]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-55.png
[56]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-56.png
[57]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-57.png
[58]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-58.png
[59]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-59.png
[60]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-60.png
[61]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-61.png
[62]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-62.png
[63]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-63.png
[64]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-64.png
[65]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-65.png
[66]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-66.png
[67]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-67.png
[68]: ./media/azure-sql-database-user-manual-part-2/azure-sql-database-user-manual-68.png
