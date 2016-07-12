<properties
	pageTitle="Configuring Oracle GoldenGate in VMs | Azure"
	description="Step through a tutorial for setting up and implementing Oracle GoldenGate on Azure Windows Server VMs for high availability and disaster recovery."
	services="virtual-machines-windows"
	authors="rickstercdn"
	manager="timlt"
	documentationCenter=""
	tags="azure-service-management"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="06/22/2015"
	wacn.date="08/29/2015"/>

#配置适用于 Azure 的 Oracle GoldenGate
本教程演示如何为 Azure 虚拟机环境设置 Oracle GoldenGate，以实现高可用性和灾难恢复。本教程重点介绍非 RAC Oracle 数据库的[双向复制](http://docs.oracle.com/goldengate/1212/gg-winux/GWUAD/wu_about_gg.htm)，并要求两个站点都处于活动状态。

Oracle GoldenGate 支持数据分发和数据集成。它允许你通过 Oracle-Oracle 复制配置来设置数据分发和数据同步解决方案，并提供了灵活的高可用性解决方案。Oracle GoldenGate 补强了 Oracle 数据防护，它的复制功能可以实现企业范围的信息分发和零停机时间的升级与迁移。有关详细信息，请参阅[将 Oracle GoldenGate 与 Oracle 数据防护配合使用](http://docs.oracle.com/cd/E11882_01/server.112/e17157/unplanned.htm)。

Oracle GoldenGate 包含以下主要组件：Extract、Data Pump、Replicat、Trails（或提取文件）、Checkpoints、Manager 和 Collector。若要在两个站点之间进行双向复制，你需要在两个站点上创建并启动所有组件。有关 Oracle GoldenGate 体系结构的详细信息，请参阅 [Oracle GoldenGate 指南](http://docs.oracle.com/goldengate/1212/gg-winux/index.html)。

本教程假设你对 Oracle 数据库高可用性和灾难恢复的概念以及 [Oracle GoldenGate](http://docs.oracle.com/goldengate/1212/gg-winux/index.html) 具有理论和实践知识。有关详细信息，请参阅 [Oracle 网站](http://www.oracle.com/technetwork/database/features/availability/index.html)。

此外，本教程假设你满足以下先决条件：

- 你已经查看 [Oracle 虚拟机映像 - 其他注意事项](/documentation/articles/virtual-machines-windows-classic-oracle-considerations/)主题中的“高可用性和灾难恢复注意事项”部分。请注意，Azure 支持独立的 Oracle 数据库实例，但目前不支持 Oracle 真正应用程序群集 (Oracle RAC)。

- 你已从 [Oracle 下载](http://www.oracle.com/us/downloads/index.html)网站下载了 Oracle GoldenGate 软件。你已选择产品包“Oracle Fusion Middleware – Data Integration”。然后，你已选择适用于 Oracle 11g 数据库的 Oracle GoldenGate on Oracle v11.2.1 Media Pack for Microsoft Windows x64（64 位）。接下来，请下载 Oracle GoldenGate V11.2.1.0.3 for Oracle 11g 64bit on Windows 2008（64 位）。

- 你已使用 Windows Server 上的 Oracle 企业版映像中提供的平台在 Azure 中创建了两个虚拟机 (VM)。有关信息，请参阅 [Azure 虚拟机](/documentation/services/virtual-machines/)。确保这些虚拟机位于[相同的云服务](/documentation/articles/virtual-machines-windows-load-balance/)和相同的[虚拟网络](/documentation/services/networking/)中，以确保它们可以通过持久性专用 IP 地址相互访问。

- 已在 Azure 经典管理门户中将站点 A 的虚拟机名称设置为“MachineGG1”，将站点 B 的虚拟机名称设置为“MachineGG2”。

- 已在站点 A 和站点 B 上分别创建测试数据库“TestGG1”和“TestGG2”。

- 以管理员组成员或 **ORA_DBA** 组成员的身份登录到 Windows 服务器。

在本教程中，您将：

1. 在站点 A 和站点 B 上安装数据库  

	1. 执行初始数据加载
	
2. 准备站点 A 和站点 B 以进行数据库复制

3. 创建所有必要的对象以支持 DDL 复制

4. 在站点 A 和站点 B 上配置 GoldenGate Manager

5. 在站点 A 和站点 B 上创建 Extract Group 和 Data Pump 进程

	1. 在站点 A 上创建 Extract 和 Data Pump 进程

	2. 在站点 B 上创建 GoldenGate 检查点表

	3. 在站点 B 上添加 REPLICAT

	4. 在站点 B 上创建 Extract 和 Data Pump 进程

	5. 在站点 A 上创建 GoldenGate 检查点表

	6. 在站点 A 上添加 REPLICAT

	7. 在站点 A 和站点 B 上添加 trandata

	8. 在站点 A 上启动 Extract 和 Data Pump 进程

	9. 在站点 B 上启动 Extract 和 Data Pump 进程

	10. 在站点 A 上启动 REPLICAT 进程

	11. 在站点 B 上启动 REPLICAT 进程

6. 验证双向复制过程

>[AZURE.IMPORTANT]本教程已针对以下软件配置进行设置和测试：
>
>| | **站点 A 数据库** | **站点 B 数据库** |
>|------------------------|----------------------------------|----------------------------------|
>| **Oracle 版本** | Oracle11g 版本 2 – (11.2.0.1) | Oracle11g 版本 2 – (11.2.0.1) |
>| **计算机名称** | MachineGG1 | MachineGG2 |
>| **操作系统** | Windows 2008 R2 | Windows 2008 R2 |
>| **Oracle SID** | TESTGG1 | TESTGG2 |
>| **复制架构** | SCOTT | SCOTT |

对于 Oracle 数据库和 Oracle GoldenGate 的后续版本，你可能需要实施一些附加更改。有关最新的版本特定信息，请参阅 Oracle 网站上的 [Oracle GoldenGate](http://docs.oracle.com/goldengate/1212/gg-winux/index.html) 和 [Oracle 数据库](http://www.oracle.com/us/corporate/features/database-12c/index.html)文档。例如，对于 11.2.0.4 和更高版本的源数据库，DDL 捕获由日志挖掘服务器以异步方式执行，而不需要安装任何特殊的触发器、表或其他数据库对象。可以在不停止用户应用程序的情况下执行 Oracle GoldenGate 升级。当 Extract 处于集成模式并且存在版本低于 11.2.0.4 的 Oracle 11g 源数据库时，需要使用 DDL 触发器和支持对象。有关详细指导，请参阅[安装和配置适用于 Oracle 数据库的 Oracle GoldenGate](http://docs.oracle.com/goldengate/1212/gg-winux/GIORA.pdf)。

##1.在站点 A 和站点 B 上安装数据库
本部分介绍如何在站点 A 和站点 B 上执行数据库必备组件安装。你必须在两个站点上执行本部分所述的所有步骤：站点 A 和站点 B。

首先，通过经典管理门户与站点 A 和站点 B 建立远程桌面连接。打开 Windows 命令提示符，然后创建 Oracle GoldenGate 安装文件的主目录：

	mkdir C:\OracleGG

然后，在此文件夹中解压缩并安装 Oracle GoldenGate 软件。完成此步骤后，可以通过执行以下命令来启动 GoldenGate 软件命令解释器 (GGSCI)：

	C:\OracleGG.\ggsci

可以使用 [GGSCI](http://docs.oracle.com/goldengate/1212/gg-winux/GWUAD/wu_gettingstarted.htm) 来运行用于配置、控制和监视 Oracle GoldenGate 的多个命令。

接下来，运行以下命令，以从安装包创建所有子文件夹：

	GGSCI (Hostname) 1> CREATE SUBDIRS

运行以下命令退出 GGSCI 命令提示符：

	GGSCI (Hostname) 1> EXIT

然后，需要创建一个将由 Oracle GoldenGate Manager、Extract 和 Replication 进程使用的数据库用户。请注意，你可以为每个进程创建一个用户，也可以只配置一个通用用户。在本教程中，我们将创建一个名为 ggate 的用户。然后，我们将授予该用户必要的特权。请注意，你必须在站点 A 和站点 B 上执行以下操作。

使用 **SYSDBA** 以数据库管理员特权在站点 A 和站点 B 上打开 SQL*Plus 命令窗口，例如：

	Enter username: / as sysdba

然后运行：

	SQL> create tablespace ggs_data   datafile 'c:\OracleDatabase\oradata<DBNAME><DBNAME>ggs_data01.dbf' size 200m; 
	SQL> create user ggate identified by ggate default tablespace ggs_data  temporary tablespace temp;
	      grant connect, resource to ggate;
	      grant select any dictionary, select any table to ggate;
	      grant create table to ggate;
	      grant flashback any table to ggate;
	      grant execute on dbms_flashback to ggate;
	      grant execute on utl_file to ggate;
	      grant create any table to ggate;
	      grant insert any table to ggate;
	      grant update any table to ggate;
	      grant delete any table to ggate;
	      grant drop any table to ggate;

接下来，在站点 A 和站点 B 上的 %ORACLE_HOME%\\database 文件夹中找到 INITDatabaseSID.ORA 文件，并将以下数据库参数追加到 INITTEST.ora 中：

	UNDO_MANAGEMENT=AUTO
	UNDO_RETENTION=86400

有关所有 Oracle GoldenGate GGSCI 命令的完整列表，请参阅 [Oracle GoldenGate for Windows 参考](http://docs.oracle.com/goldengate/1212/gg-winux/GWURF/ggsci_commands.htm)。

### 执行初始数据加载

你可以通过以下几种方法在数据库中执行初始数据加载。例如，可以使用 [Oracle GoldenGate Direct Load](http://docs.oracle.com/goldengate/1212/gg-winux/GWUAD/wu_initsync.htm) 或普通的导出和导入实用工具，将表数据从站点 A 导出到站点 B。

为了演示 Oracle GoldenGate 复制过程，本教程演示了如何使用以下命令在站点 A 和站点 B 上创建一个表。

首先，请打开 SQL*Plus 命令窗口，然后运行以下命令，以在站点 A 和站点 B 数据库中创建一个库存表：
	
	create table scott.inventory
	(prod_id number,
	prod_category varchar2(20),
	qty_in_stock number,
	last_dml timestamp default systimestamp);

接下来，将约束添加到站点 A 和站点 B 数据库上的新建表：

	alter table scott.inventory add constraint pk_inventory primary key (prod_id) ;

然后，在站点 A 和站点 B 上向用户 ggate 授予对新库存表的所有特权：

	grant all on scott.inventory to ggate;

接下来，在新建的表上创建并启用数据库触发器 INVENTORY_CDR_TRG，以确保当用户不是 ggate 时，记录向新表中写入的所有事务。在站点 A 和站点 B 上执行此操作。

	CREATE OR REPLACE TRIGGER INVENTORY_CDR_TRG
	BEFORE UPDATE
	ON SCOTT.INVENTORY
	REFERENCING NEW AS New OLD AS Old
	FOR EACH ROW
	BEGIN
	IF SYS_CONTEXT ('USERENV', 'SESSION_USER') != 'GGATE'
	THEN
	:NEW.LAST_DML := SYSTIMESTAMP;
	END IF;
	END;
	/ 


##2.准备站点 A 和站点 B 以进行数据库复制
本部分说明如何准备站点 A 和站点 B 以进行数据库复制。必须在两个站点上执行本部分所述的所有步骤：站点 A 和站点 B。

首先，通过 Azure 经典管理门户与站点 A 和站点 B 建立远程桌面连接。使用 SQL*Plus 命令窗口将数据库切换到 archivelog 模式：
	
	sql>shutdown immediate 
	sql>startup mount 
	sql>alter database archivelog; 
	sql>alter database open;


然后，按如下所示启用最小[补充日志记录](http://docs.oracle.com/cd/E11882_01/server.112/e22490/logminer.htm)：

	SQL> ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;

接下来，准备好数据库，以支持 DDL（数据定义语言）复制：

	SQL> alter system set recyclebin=off scope=spfile;

然后，关闭再重启数据库：

	sql>shutdown immediate 
	sql>startup


##3.创建所有必要的对象以支持 DDL 复制
本部分列出了你需要使用哪些脚本来创建所有必要的对象以支持 DDL 复制。需要在站点 A 和站点 B 上运行本部分中指定的脚本。

打开 Windows 命令提示符并导航到 Oracle GoldenGate 文件夹，例如 C:\\OracleGG。在站点 A 和站点 B 上以数据库管理员特权启动 SQL*Plus 命令提示符，例如，使用 **SYSDBA**。

然后，运行以下脚本：
	
	SQL> @marker_setup.sql  
	Enter GoldenGate schema name: ggate
	SQL> @ddl_setup.sql  
	Enter GoldenGate schema name: ggate
	SQL> @role_setup.sql 
	Enter GoldenGate schema name: ggate
	SQL> grant ggs_ggsuser_role to ggate;
	 Grant succeeded.
	 SQL> @ddl_enable
	 Trigger altered.
	 SQL> @ddl_pin ggate


Oracle GoldenGate 工具要求使用表级登录名来支持 DDL（数据定义语言）。正因如此，请在表级别使用 ADD TRANDATA 命令启用补充日志记录。打开 Oracle GoldenGate 命令解释器窗口，登录数据库，然后运行 ADD TRANDATA 命令：

	GGSCI 5> DBLOGIN USERID ggate, PASSWORD ggate

	GGSCI(Hostname) 6> add trandata scott.inventory

##4.在站点 A 和站点 B 上配置 GoldenGate Manager
Oracle GoldenGate Manager 执行许多功能，例如，启动其他 GoldenGate 进程、跟踪日志文件管理和报告。

你需要在站点 A 和站点 B 上配置 Oracle GoldenGate Manager 进程。为此，请在站点 A 和站点 B 上执行以下步骤。

打开 Windows 命令窗口，然后启动 Oracle GoldenGate 命令解释器：

	cd C:\OracleGG\
	c:\OracleGG>ggsci
	Oracle GoldenGate Command Interpreter for Oracle
	Version 11.2.1.0.3 14400833 OGGCORE_11.2.1.0.3_PLATFORMS_120823.1258
	Windows x64 (optimized), Oracle 11g on Aug 23 2012 16:50:36
	Copyright (C) 1995, 2012, Oracle and/or its affiliates. All rights reserved.


将 GGSCI 会话记录到数据库，以便可以执行影响数据库的命令：

	GGSCI (HostName) 1> DBLOGIN USERID ggate, PASSWORD ggate
	Successfully logged into database.

显示系统上所有 Manager、Extract 和 Replicat 进程的状态与滞后时间（如果适用）：

	GGSCI (HostName) 2> info all
	Program     Status      Group       Lag           Time Since Chkpt
	MANAGER     STOPPED

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (HostName) 3> edit params mgr
	PORT 7809
	USERID ggate, PASSWORD ggate
	PURGEOLDEXTRACTS  C:\OracleGG\dirdat\ex, USECHECKPOINTS

显示系统上所有 Manager、Extract 和 Replicat 进程的状态与滞后时间（如果适用）：

	GGSCI (HostName) 46> info all
	Program     Status      Group       Lag           Time Since Chkpt
	MANAGER     STOPPED

将 GGSCI 会话记录到数据库，以便可以执行影响数据库的命令：

	GGSCI (HostName) 47> dblogin USERID ggate, PASSWORD ggate

	Successfully logged into database.

启动管理器进程：

	GGSCI (HostName) 48> start manager
	Manager started.

##5.在站点 A 和站点 B 上创建 Extract Group 和 Data Pump 进程

###在站点 A 上创建 Extract 和 Data Pump 进程

需要在站点 A 和站点 B 上创建 Extract 和 Data Pump 进程。通过经典管理门户与站点 A 和站点 B 建立远程桌面连接。打开 GGSCI 命令解释器窗口。在站点 A 上运行以下命令：

	GGSCI (MachineGG1) 14> add extract ext1 tranlog begin now
	EXTRACT added.
	GGSCI (MachineGG1) 4> add exttrail C:\OracleGG\dirdat\lt, extract ext1
	EXTTRAIL added.
	GGSCI (MachineGG1) 16> add extract dpump1 exttrailsource C:\OracleGG\dirdat\aa
	EXTRACT added.
	GGSCI (MachineGG1) 17> add rmttrail C:\OracleGG\dirdat\ab extract dpump1
	RMTTRAIL added.

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：
	GGSCI (MachineGG1) 18> edit params ext1
	EXTRACT ext1
	 USERID ggate, PASSWORD ggate
	 EXTTRAIL C:\OracleGG\dirdat\aa
	 TRANLOGOPTIONS EXCLUDEUSER ggate
	 TABLE scott.inventory,
	 GETBEFORECOLS (
	 ON UPDATE KEYINCLUDING (prod_category,qty_in_stock, last_dml),
	 ON DELETE KEYINCLUDING (prod_category,qty_in_stock, last_dml));

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (MachineGG1) 15> edit params dpump1
	EXTRACT dpump1
	 USERID ggate, PASSWORD ggate
	 RMTHOST ActiveGG2orcldb, MGRPORT 7809, TCPBUFSIZE 100000
	 RMTTRAIL C:\OracleGG\dirdat\ab
	 PASSTHRU
	 TABLE scott.inventory;

###在站点 B 上创建 GoldenGate 检查点表

接下来，需要在站点 B 上添加一个检查点表。为此，需要打开 GoldenGate 命令解释器窗口并运行：

	C:\OracleGG\ggsci
	GGSCI (MachineGG2) 1> DBLOGIN USERID ggate, PASSWORD ggate
	Successfully logged into database.

然后，将检查点表添加到数据库，其中 ggate 是所有者：
	
	GGSCI (MachineGG2) 2> ADD CHECKPOINTTABLE ggate.checkpointtable
	Successfully created checkpoint table ggate.checkpointtable.

将检查点表的名称添加到目标服务器（在本步骤中为站点 B）上的 GLOBALS 文件。在站点 B 上编辑 GLOBALS 文件：

	GGSCI (MachineGG2) 1> EDIT PARAMS ./GLOBALS

然后，将 CHECKPOINTTABLE 参数追加到现有的 GLOBALS 文件：

	GGSCHEMA ggate
	CHECKPOINTTABLE ggate.checkpointtable

最后，请保存并关闭 GLOBALS 参数文件。


###在站点 B 上添加 REPLICAT
本部分介绍如何在站点 B 上添加 REPLICAT 进程“REP2”。
 
在站点 B 上使用 ADD REPLICAT 命令创建 Replicat 组：

	GGSCI (MachineGG2) 37> add replicat rep2 exttrail C:\OracleGG\dirdatab, checkpointtable ggate.checkpointtable

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (MachineGG2) 10> edit params rep2
	REPLICAT rep2
	ASSUMETARGETDEFS
	USERID ggate, PASSWORD ggate
	DISCARDFILE C:\OracleGG\dirdat\discard.txt, append,megabytes 10
	MAP scott.inventory, TARGET scott.inventory;

###在站点 B 上创建 Extract 和 Data Pump 进程

本部分介绍如何在站点 B 上创建新的 Extract 进程“EXT2”和新的 Data Pump 进程“DPUMP2”：

	GGSCI (MachineGG2) 3> add extract ext2 tranlog begin now
	 EXTRACT added.
	GGSCI (MachineGG2) 4> add exttrail C:\OracleGG\dirdat\ac extract ext2
	 EXTTRAIL added.
	GGSCI (MachineGG2) 5> add extract dpump2 exttrailsource C:\OracleGG\dirdat\ac
	 EXTRACT added.
	GGSCI (MachineGG2) 6> add rmttrail C:\OracleGG\dirdat\ad extract dpump2
	 RMTTRAIL added.

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (MachineGG2) 31> edit params ext2
	EXTRACT ext2
	USERID ggate, PASSWORD ggate
	EXTTRAIL C:\OracleGG\dirdat\ac
	TRANLOGOPTIONS EXCLUDEUSER ggate
	TABLE scott.inventory,
	GETBEFORECOLS (
	ON UPDATE KEYINCLUDING (prod_category,qty_in_stock, last_dml),
	ON DELETE KEYINCLUDING (prod_category,qty_in_stock, last_dml));

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (MachineGG2) 32> edit params dpump2
	EXTRACT dpump2
	USERID ggate, PASSWORD ggate
	RMTHOST MachineGG1, MGRPORT 7809, TCPBUFSIZE 100000
	RMTTRAIL C:\OracleGG\dirdat\ad
	PASSTHRU
	TABLE scott.inventory;

###在站点 A 上创建 GoldenGate 检查点表

打开 Oracle GoldenGate 命令解释器窗口，然后创建一个检查点表：

	GGSCI (MachineGG1) 1> DBLOGIN USERID ggate, PASSWORD ggate
	Successfully logged into database.

	GGSCI (MachineGG1) 2> ADD CHECKPOINTTABLE ggate.checkpointtable

	Successfully created checkpoint table ggate.checkpointtable.

你还需要将检查点表的名称添加到站点 A 上的 GLOBALS 文件。

在站点 A 上打开 Oracle GoldenGate 命令解释器窗口，然后编辑 GLOBALS 文件：

	GGSCI (MachineGG1) 1> EDIT PARAMS ./GLOBALS
	Add the CHECKPOINTTABLE parameter to the existing GLOBALS file as follows:
	GGSCHEMA ggate
	CHECKPOINTTABLE ggate.checkpointtable

保存并关闭 GLOBALS 参数文件。

###在站点 A 上添加 REPLICAT

本部分介绍如何在站点 A 上添加 REPLICAT 进程“REP1”。

以下命令将使用某个线索的名称以及关联的 checkpointtable 创建 Replicat 组 rep1。

	GGSCI (MachineGG1) 21> add replicat rep1 exttrail C:\OracleGG\dirdat\ad,checkpointtable ggate.checkpointtable
	 REPLICAT added.

使用 EDIT PARAMS 命令打开参数文件，然后追加以下信息：

	GGSCI (MachineGG1) 10> edit params rep1
	REPLICAT rep1
	ASSUMETARGETDEFS
	USERID ggate, PASSWORD ggate
	DISCARDFILE C:\OracleGG\dirdat\discard.txt, append, megabytes 10
	MAP scott.inventory, TARGET scott.inventory;

### 在站点 A 和站点 B 上添加 trandata

使用 ADD TRANDATA 命令在表级别启用补充日志记录。打开 Oracle GoldenGate 命令解释器窗口，登录数据库，然后运行 ADD TRANDATA 命令。

与 MachineGG1 建立远程桌面连接，打开 Oracle GoldenGate 命令解释器，然后运行：

	GGSCI (MachineGG1) 11> dblogin userid ggate password ggate
	 Successfully logged into database.
	GGSCI (MachineGG1) 12> add trandata scott.inventory cols (prod_category,qty_in_stock, last_dml)
	GGSCI (MachineGG1) 13> info trandata scott.inventory
	Logging of supplemental redo log data is enabled for table SCOTT.INVENTORY.
	Columns supplementally logged for table SCOTT.INVENTORY: PROD_ID, PROD_CATEGORY, QTY_IN_STOCK, LAST_DML.
		
与 MachineGG2 建立远程桌面连接，打开 Oracle GoldenGate 命令解释器，然后运行：

	GGSCI (MachineGG2) 18> dblogin userid ggate password ggate
	 Successfully logged into database.
	GGSCI (MachineGG2) 14> add trandata scott.inventory cols (prod_category,qty_in_stock, last_dml)
	Logging of supplemental redo data enabled for table SCOTT.INVENTORY.

显示有关表级补充日志记录的状态的信息：

	GGSCI (MachineGG2) 15> info trandata scott.inventory
	Logging of supplemental redo log data is enabled for table SCOTT.INVENTORY.
	Columns supplementally logged for table SCOTT.INVENTORY: PROD_ID, PROD_CATEGORY, QTY_IN_STOCK, LAST_DML.

###在站点 A 和站点 B 上添加 trandata

使用 ADD TRANDATA 命令在表级别启用补充日志记录。打开 Oracle GoldenGate 命令解释器窗口，登录数据库，然后运行 ADD TRANDATA 命令。

与 MachineGG1 建立远程桌面连接，打开 Oracle GoldenGate 命令解释器，然后运行：

	GGSCI (MachineGG1) 11> dblogin userid ggate password ggate
	 Successfully logged into database.
	GGSCI (MachineGG1) 12> add trandata scott.inventory cols (prod_category,qty_in_stock, last_dml)
	GGSCI (MachineGG1) 13> info trandata scott.inventory
	Logging of supplemental redo log data is enabled for table SCOTT.INVENTORY.
	Columns supplementally logged for table SCOTT.INVENTORY: PROD_ID, PROD_CATEGORY, QTY_IN_STOCK, LAST_DML.

与 MachineGG2 建立远程桌面连接，打开 Oracle GoldenGate 命令解释器，然后运行：

	GGSCI (MachineGG2) 18> dblogin userid ggate password ggate
	 Successfully logged into database.
	GGSCI (MachineGG2) 14> add trandata scott.inventory cols (prod_category,qty_in_stock, last_dml)
	Logging of supplemental redo data enabled for table SCOTT.INVENTORY.
	Display information about the state of table-level supplemental logging:
	GGSCI (MachineGG2) 15> info trandata scott.inventory
	Logging of supplemental redo log data is enabled for table SCOTT.INVENTORY.
	Columns supplementally logged for table SCOTT.INVENTORY: PROD_ID, PROD_CATEGORY, QTY_IN_STOCK, LAST_DML.

###在站点 A 上启动 Extract 和 Data Pump 进程

在站点 A 上启动 Extract 进程 ext1：

	GGSCI (MachineGG1) 31> start extract ext1
	Sending START request to MANAGER …
	EXTRACT EXT1 starting

在站点 A 上启动 Data Pump 进程 dpump1：

	GGSCI (MachineGG1) 23> start extract dpump1
	Sending START request to MANAGER …
	EXTRACT DPUMP1 starting
显示有关 Extract 组 ext1 的信息：GGSCI (MachineGG1) 32> info extract ext1 EXTRACT EXT1 Last Started 2013-11-25 08:03 Status RUNNING Checkpoint Lag 00:00:00 (updated 00:00:02 ago) Log Read Checkpoint Oracle Redo Logs 2013-11-25 08:03:18 Seqno 6, RBA 3230720 SCN 0.1074371 (1074371)

显示系统上所有 Manager、Extract 和 Replicat 进程的状态与滞后时间（如果适用）：

	GGSCI (MachineGG1) 16> info all
	Program     Status      Group       Lag at Chkpt  Time Since Chkpt
	
	MANAGER     RUNNING
	EXTRACT     RUNNING     DPUMP1      00:00:00      00:46:33
	EXTRACT     RUNNING     EXT1        00:00:00      00:00:04

###在站点 B 上启动 Extract 和 Data Pump 进程

在站点 B 上启动 Extract 进程 ext2：

	GGSCI (MachineGG2) 22> start extract ext2
	Sending START request to MANAGER …
	EXTRACT EXT2 starting

在站点 B 上启动 Data Pump 进程 dpump2：

	GGSCI (MachineGG2) 23> start extract dpump2
	Sending START request to MANAGER …
	EXTRACT DPUMP2 starting

显示系统上所有 Manager、Extract 和 Replicat 进程的状态与滞后时间（如果适用）：

	GGSCI (ActiveGG2orcldb) 6> info all
	Program     Status      Group       Lag at Chkpt  Time Since Chkpt
	
	MANAGER     RUNNING
	EXTRACT     RUNNING     DPUMP2      00:00:00      136:13:33
	EXTRACT     RUNNING     EXT2        00:00:00      00:00:04

### 在站点 A 上启动 REPLICAT 进程

本部分介绍如何在站点 A 上启动 REPLICAT 进程“REP1”。

在站点 A 上启动 Replicat 进程：

	GGSCI (MachineGG1) 38> start replicat rep1
	Sending START request to MANAGER …
	REPLICAT REP1 starting

显示 Replicat 组的状态：

	GGSCI (MachineGG1) 39> status replicat rep1
	 REPLICAT REP1: RUNNING

###在站点 B 上启动 REPLICAT 进程

本部分介绍如何在站点 B 上启动 REPLICAT 进程“REP2”。

在站点 B 上启动 Replicat 进程：

	GGSCI (MachineGG2) 26> start replicat rep2
	Sending START request to MANAGER …
	REPLICAT REP2 starting

显示 Replicat 组的状态：

	GGSCI (MachineGG2) 27> status replicat rep2
	REPLICAT REP2: RUNNING

##6.验证双向复制过程

若要验证 Oracle GoldenGate 配置，请在站点 A 上的数据库中插入一行。与站点 A 建立远程桌面连接。打开 SQL*Plus 命令窗口并运行：SQL> select name from v$database;
	
	NAME
	———
	TESTGG
	
	SQL> insert into inventory values  (100,’TV’,100,sysdate);
	
	1 row created.
	
	SQL> commit;
	
	Commit complete.

然后，检查该行是否已复制到站点 B。为此，请与站点 B 建立远程桌面连接。打开 SQL Plus 窗口并运行：

	SQL> select name from v$database;
	
	NAME
	———
	TESTGG
	
	SQL> select * from inventory;
	
	PROD_ID PROD_CATEGORY QTY_IN_STOCK LAST_DML
	———- ——————– ———— ———
	100 TV 100 22-MAR-13

在站点 B 上插入一条新记录：

	SQL> insert into inventory  values  (101,’DVD’,10,sysdate);
	1 row created.
	
	SQL> commit;
	
	Commit complete.

与站点 A 建立远程桌面连接，然后检查是否已发生复制：

	SQL> select * from inventory;
	
	PROD_ID PROD_CATEGORY QTY_IN_STOCK LAST_DML
	———- ——————– ———— ———
	100 TV 100 22-MAR-13
	101 DVD 10 22-MAR-13

<!---HONumber=67-->