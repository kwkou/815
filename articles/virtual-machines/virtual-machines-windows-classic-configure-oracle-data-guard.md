<properties
	pageTitle="在 VM 中配置 Oracle 数据防护 | Azure"
	description="有关如何在 Azure 虚拟机中设置和实施 Oracle 数据防护以实现高可用性和灾难恢复的分步教程。"
	services="virtual-machines-windows"
	authors="rickstercdn"
	manager="timlt"
	documentationCenter=""
	tags="azure-service-management"/>  

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="infrastructure-services"
	ms.date="09/06/2016"
	wacn.date="10/24/2016"
	ms.author="rclaus" />  


#为 Azure 配置 Oracle 数据防护


本教程介绍如何在 Azure 虚拟机环境中设置和实施 Oracle 数据防护，以实现高可用性和灾难恢复。本教程着重于非 RAC Oracle 数据库的单向复制。

Oracle 数据防护支持对 Oracle 数据库实施数据保护和灾难恢复。它是一种简单、高性能、即时的解决方案，可针对整个 Oracle 数据库实现灾难恢复、数据保护和高可用性。

本教程假设用户具有 Oracle 数据库高可用性和灾难恢复概念方面的理论和实践知识。有关信息，请参阅 [Oracle 网站](http://www.oracle.com/technetwork/database/features/availability/index.html)和 [Oracle Data Guard Concepts and Administration Guide](https://docs.oracle.com/cd/E11882_01/server.112/e41134/toc.htm)（Oracle 数据防护概念和管理指南）。

此外，本教程假设你满足以下先决条件：

- Azure 支持独立的 Oracle 数据库实例，但目前不支持 Oracle 真正应用程序群集 (Oracle RAC)。


- 已在 Azure 中使用由平台提供的同一 Oracle 企业版映像创建了两个虚拟机 (VM)。确保这些虚拟机位于[相同的云服务](/documentation/articles/virtual-machines-windows-load-balance/)和相同的虚拟网络中，可以通过持久性专用 IP 地址相互访问。另外，建议将 VM 放在同一[可用性集](/documentation/articles/virtual-machines-windows-manage-availability/)中，以便 Azure 将它们置于不同的容错域和升级域中。Oracle 数据防护仅适用于 Oracle 数据库企业版。每个虚拟机必须至少具有 2 GB 内存和 5 GB 磁盘空间。有关由平台提供的 VM 大小的最新信息，请参阅 [Virtual Machine Sizes for Azure](/documentation/articles/virtual-machines-windows-sizes/)（Azure 的虚拟机大小）。如果需要为 VM 提供更多的磁盘卷，可以附加更多的磁盘。有关信息，请参阅 [How to Attach a Data Disk to a Virtual Machine](/documentation/articles/virtual-machines-windows-classic-attach-disk/)（如何将数据磁盘附加到虚拟机）。



- 已在 Azure 经典管理门户中将主 VM 的虚拟机名称设置为“Machine1”，将备用 VM 的虚拟机名称设置为“Machine2”。

- 已将 **ORACLE\_HOME**环境变量设置为指向主虚拟机和备用虚拟机中的同一个 Oracle 根安装路径，例如 `C:\OracleDatabase\product\11.2.0\dbhome_1\database`。

- 以“管理员”组成员或 **ORA\_DBA** 组成员的身份登录到 Windows 服务器。

在本教程中，您将：

实施物理备用数据库环境

1. 创建主数据库

2. 准备用于创建备用数据库的主数据库

	1. 启用强制日志记录

	2. 创建密码文件

	3. 配置备用重做日志

	4. 启用存档

	5. 设置主数据库初始化参数

创建物理备用数据库

1. 为备用数据库准备初始化参数文件

2. 配置侦听器和 tnsnames，支持主计算机和备用计算机上的数据库

	1. 在两个服务器上配置 listener.ora，用于保存两个数据库的条目

	2. 在主虚拟机和备用虚拟机上配置 tnsnames.ora，用于保存主数据库和备用数据库的条目。

	3. 启动侦听器，并检查是否可以从两个虚拟机 tnsping 到两个服务。

3. 在 nomount 状态下启动备用实例

4. 使用 RMAN 克隆数据库并创建备用数据库

5. 在托管恢复模式下启动物理备用数据库

6. 验证物理备用数据库

> [AZURE.IMPORTANT] 本教程已针对以下硬件和软件配置进行设置和测试：
>
>| | **主数据库** | **备用数据库** |
>|----------------------|-------------------------------------------|-------------------------------------------|
>| **Oracle 版本** | Oracle11g 企业版 (11.2.0.4.0) | Oracle11g 企业版 (11.2.0.4.0) |
>| **计算机名称** | Machine1 | Machine2 |
>| **操作系统** | Windows 2008 R2 | Windows 2008 R2 |
>| **Oracle SID** | TEST | TEST\_STBY |
>| **内存** | 至少 2 GB | 至少 2 GB |
>| **磁盘空间** | 至少 5 GB | 至少 5 GB |

对于 Oracle 数据库和 Oracle 数据防护的后续版本，可能需要实施一些附加更改。有关最新的版本特定信息，请参阅 Oracle 网站上的[数据防护](http://www.oracle.com/technetwork/database/features/availability/data-guard-documentation-152848.html)和 [Oracle 数据库](http://www.oracle.com/us/corporate/features/database-12c/index.html)文档。

##实施物理备用数据库环境
### 1\.创建主数据库

- 在主虚拟机中创建主数据库“TEST”。有关信息，请参阅“创建和配置 Oracle 数据库”。
- 若要查看数据库的名称，请在 SQL*Plus 命令提示符下，以具有 SYSDBA 角色的 SYS 用户身份连接到数据库，然后运行以下语句：

		SQL> select name from v$database;

		The result will display like the following:

		NAME
		---------
		TEST
- 接下来，请从 dba\_data\_files 系统视图查询数据库文件的名称：

		SQL> select file_name from dba_data_files;
		FILE_NAME
		-------------------------------------------------------------------------------
		C:\ <YourLocalFolder>\TEST\USERS01.DBF
		C:\ <YourLocalFolder>\TEST\UNDOTBS01.DBF
		C:\ <YourLocalFolder>\TEST\SYSAUX01.DBF
		C:\<YourLocalFolder>\TEST\SYSTEM01.DBF
		C:\<YourLocalFolder>\TEST\EXAMPLE01.DBF

### 2\.准备用于创建备用数据库的主数据库

在创建备用数据库之前，建议用户确保正确配置主数据库。下面是需执行的步骤的列表：

1. 启用强制日志记录
2. 创建密码文件
3. 配置备用重做日志
4. 启用存档
5. 设置主数据库初始化参数

#### 启用强制日志记录

为了实现备用数据库，需在主数据库中启用“强制日志记录”。此选项可确保即使已完成“nologging”操作，也会优先处理强制日志记录，并将所有操作记录到重做日志。因此，请确保记录主数据库中的所有内容，且在复制到备用数据库时包括主数据库中的所有操作。运行 alter database 语句以启用强制日志记录：

	SQL> ALTER DATABASE FORCE LOGGING;

	Database altered.

#### 创建密码文件

若要将存储日志从主服务器传递并应用到备用服务器，主服务器和备用服务器上的 sys 密码必须完全相同。正因如此，需要在主数据库中创建一个密码文件，并将其复制到备用服务器。

>[AZURE.IMPORTANT] 使用 Oracle 数据库 12c 时，可以使用新用户 **SYSDG** 来管理 Oracle 数据防护。有关详细信息，请参阅 [Changes in Oracle Database 12c Release](http://docs.oracle.com/database/121/UNXAR/release_changes.htm#UNXAR404)（Oracle 数据库 12c 版本中的更改）。

此外，请确保已在 Machine1 中定义 ORACLE\_HOME 环境。如果未定义，请使用“环境变量”对话框将它定义为环境变量。若要访问此对话框，请双击控制面板中的“系统”图标启动“系统”实用工具，然后单击“高级”选项卡并选择“环境变量”。若要设置环境变量，请单击“系统变量”下的“新建”按钮。设置环境变量后，请关闭现有的 Windows 命令提示符并打开一个新的。

运行以下语句以切换到 Oracle\_Home 目录，例如 C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\database。

	cd %ORACLE_HOME%\database

然后，使用密码文件创建实用工具 [ORAPWD](http://docs.oracle.com/cd/B28359_01/server.111/b28310/dba007.htm) 创建密码文件。在 Machine1 中的同一个 Windows 命令提示符下，通过将密码值设置为 **SYS** 的密码来运行以下命令：

	ORAPWD FILE=PWDTEST.ora PASSWORD=password FORCE=y

此命令将在 ORACLE\_HOME\\database 目录中创建名为 PWDTEST.ora 的密码文件。应手动将此文件复制到 Machine2 上的 %ORACLE\_HOME%\\database 目录中。

#### 配置备用重做日志

然后，需要配置备用重做日志，使主服务器在变为备用服务器后可以正确接收重做日志。另外，在此处预先创建重做日志后，备用服务器上会自动创建备用重做日志。必须为备用重做日志 (SRL) 配置与联机重做日志相同的大小。当前备用重做日志文件的大小必须与当前主数据库联机重做日志文件的大小完全匹配。

在 Machine1 中的 SQL*PLUS 命令提示符下运行以下语句。v$logfile 是一个系统视图，其中包含有关重做日志文件的信息。

	SQL> select * from v$logfile;
	GROUP# STATUS  TYPE    MEMBER                                                       IS_
	---------- ------- ------- ------------------------------------------------------------ ---
	3         ONLINE  C:\<YourLocalFolder>\TEST\REDO03.LOG               NO
	2         ONLINE  C:\<YourLocalFolder>\TEST\REDO02.LOG               NO
	1         ONLINE  C:\<YourLocalFolder>\TEST\REDO01.LOG               NO
	Next, query the v$log system view, displays log file information from the control file.
	SQL> select bytes from v$log;
	BYTES
	----------
	52428800
	52428800
	52428800


请注意，52428800 即 50 MB。

然后，在 SQL*Plus 窗口中运行以下语句，将新的备用重做日志文件组添加到备用数据库，并使用 GROUP 子句指定用于标识该组的编号。使用组编号可以更轻松地管理备用重做日志文件组：

	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 4 'C:\<YourLocalFolder>\TEST\REDO04.LOG' SIZE 50M;
	Database altered.
	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 5 'C:\<YourLocalFolder>\TEST\REDO05.LOG' SIZE 50M;
	Database altered.
	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 6 'C:\<YourLocalFolder>\TEST\REDO06.LOG' SIZE 50M;
	Database altered.

接下来，运行以下系统视图，列出有关重做日志文件的信息。此操作还会验证是否已创建备用重做日志文件组：

	SQL> select * from v$logfile;
	GROUP# STATUS  TYPE MEMBER IS_
	---------- ------- ------- --------------------------------------------- ---
	3         ONLINE C:\<YourLocalFolder>\TEST\REDO03.LOG NO
	2         ONLINE C:\<YourLocalFolder>\TEST\REDO02.LOG NO
	1         ONLINE C:\<YourLocalFolder>\TEST\REDO01.LOG NO
	4         STANDBY C:\<YourLocalFolder>\TEST\REDO04.LOG
	5         STANDBY C:\<YourLocalFolder>\TEST\REDO05.LOG NO
	6         STANDBY C:\<YourLocalFolder>\TEST\REDO06.LOG NO
	6 rows selected.

#### 启用存档

然后，通过运行以下语句启用存档，将主数据库置于 ARCHIVELOG 模式，并启用自动存档。可以通过装入数据库，然后执行 archivelog 命令来启用存档日志模式。

首先，以 sysdba 身份登录。在 Windows 命令提示符下，运行以下命令：

	sqlplus /nolog

	connect / as sysdba

然后，在 SQL*Plus 命令提示符下关闭数据库：

	SQL> shutdown immediate;
	Database closed.
	Database dismounted.
	ORACLE instance shut down.

然后，执行 startup mount 命令装入数据库。这可确保 Oracle 将实例与指定数据库相关联。

	SQL> startup mount;
	ORACLE instance started.
	Total System Global Area 1503199232 bytes
	Fixed Size                  2281416 bytes
	Variable Size             922746936 bytes
	Database Buffers          570425344 bytes
	Redo Buffers                7745536 bytes
	Database mounted.

然后，运行以下命令：

	SQL> alter database archivelog;
	Database altered.

然后，结合 Open 子句运行 Alter database 语句，使数据库可供正常使用：

	SQL> alter database open;

	Database altered.

#### 设置主数据库初始化参数

若要配置数据防护，首先需要在正则 pfile（文本初始化参数文件）中创建并配置备用参数。Pfile 准备就绪后，需要将其转换为服务器参数文件 (SPFILE)。

可以使用 INIT.ORA 文件中的参数控制数据防护环境。在学习本教程时，需要更新主数据库 INIT.ORA，使其能够保存两个角色：主角色和备用角色。

	SQL> create pfile from spfile;
	File created.

接下来，需要编辑 pfile 以添加备用参数。为此，请打开 %ORACLE\_HOME%\\database 中的 INITTEST.ORA 文件。接下来，将以下语句追加到 INITTEST.ora 文件中。INIT.ORA 文件的命名约定为 INIT<数据库名称>.ORA。

	db_name='TEST'
	db_unique_name='TEST'
	LOG_ARCHIVE_CONFIG='DG_CONFIG=(TEST,TEST_STBY)'
	LOG_ARCHIVE_DEST_1= 'LOCATION=C:\OracleDatabase\archive   VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=TEST'
	LOG_ARCHIVE_DEST_2= 'SERVICE=TEST_STBY LGWR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=TEST_STBY'
	LOG_ARCHIVE_DEST_STATE_1=ENABLE
	LOG_ARCHIVE_DEST_STATE_2=ENABLE
	REMOTE_LOGIN_PASSWORDFILE=EXCLUSIVE
	LOG_ARCHIVE_FORMAT=%t_%s_%r.arc
	LOG_ARCHIVE_MAX_PROCESSES=30
	# Standby role parameters --------------------------------------------------------------------
	fal_server=TEST_STBY
	fal_client=TEST
	standby_file_management=auto
	db_file_name_convert='TEST_STBY','TEST'
	log_file_name_convert='TEST_STBY','TEST'
	# ---------------------------------------------------------------------------------------------


前面的语句块包含三个重要设置项：
-	**LOG\_ARCHIVE\_CONFIG...：**使用此语句定义唯一的数据库 ID。
-	**LOG\_ARCHIVE\_DEST\_1...：**使用此语句定义本地存档文件夹位置。建议根据数据库存档需要创建一个新目录，并显式使用此语句指定本地存档位置，而不要使用 Oracle 的默认文件夹 %ORACLE\_HOME%\\database\\archive。
-	**LOG\_ARCHIVE\_DEST\_2 ....LGWR ASYNC...：**定义一个异步日志写入器进程 (LGWR)，收集重做事务数据并将其传输到备用目标。此处的 DB\_UNIQUE\_NAME 指定目标备用服务器上的数据库的唯一名称。

新参数文件准备就绪后，需要根据它创建 spfile。

首先，请关闭数据库：

	SQL> shutdown immediate;

	Database closed.

	Database dismounted.

	ORACLE instance shut down.

接下来请运行 startup nomount 命令，如下所示：

	SQL> startup nomount pfile='c:\OracleDatabase\product\11.2.0\dbhome_1\database\initTEST.ora';
	ORACLE instance started.
	Total System Global Area 1503199232 bytes
	Fixed Size                  2281416 bytes
	Variable Size             922746936 bytes
	Database Buffers          570425344 bytes
	Redo Buffers                7745536 bytes

现在，请创建 spfile：

	SQL>create spfile frompfile='c:\OracleDatabase\product\11.2.0\dbhome\_1\database\initTEST.ora';

	File created.

然后，请关闭数据库：

	SQL> shutdown immediate;

	ORA-01507: database not mounted

再然后，使用 startup 命令启动一个实例：

	SQL> startup;
	ORACLE instance started.
	Total System Global Area 1503199232 bytes
	Fixed Size                  2281416 bytes
	Variable Size             922746936 bytes
	Database Buffers          570425344 bytes
	Redo Buffers                7745536 bytes
	Database mounted.
	Database opened.

##创建物理备用数据库
本部分重点介绍准备物理备用数据库时必须在 Machine2 中执行的步骤。

首先，需要通过 Azure 经典管理门户从远程桌面连接到 Machine2。

然后，在备用服务器 (Machine2） 上，为备用数据库创建所有必要的文件夹，例如 C:\\<本地文件夹>\\TEST。在学习本教程时，请确保该文件夹结构与 Machine1 上的文件夹结构匹配，并保留所有必要的文件，例如 controlfile、datafiles、redologfiles、udump、bdump 和 cdump 文件。此外，请在 Machine2 中定义 ORACLE\_HOME 和 ORACLE\_BASE 环境变量。如果未定义，请使用“环境变量”对话框将其定义为环境变量。若要访问此对话框，请双击控制面板中的“系统”图标启动“系统”实用工具，然后单击“高级”选项卡并选择“环境变量”。若要设置环境变量，请单击“系统变量”下的“新建”按钮。设置环境变量后，需要关闭现有的 Windows 命令提示符并打开一个新的命令提示符以查看所做的更改。

然后，执行以下步骤：

1. 为备用数据库准备初始化参数文件

2. 配置侦听器和 tnsnames，支持主计算机和备用计算机上的数据库

	1. 在两个服务器上配置 listener.ora，用于保存两个数据库的条目

	2. 在主虚拟机和备用虚拟机上配置 tnsnames.ora，用于保存主数据库和备用数据库的条目

	3. 启动侦听器，并检查是否可以从两个虚拟机 tnsping 到两个服务。

3. 在 nomount 状态下启动备用实例

4. 使用 RMAN 克隆数据库并创建备用数据库

5. 在托管恢复模式下启动物理备用数据库

6. 验证物理备用数据库

### 1\.为备用数据库准备初始化参数文件

本部分介绍如何为备用数据库准备初始化参数文件。为此，请先手动将 INITTEST.ORA 文件从 Machine1 复制到 Machine2。应该可以在两个计算机上看到 %ORACLE\_HOME%\\database 文件夹中的 INITTEST.ORA 文件。然后，在 Machine2 中修改 INITTEST.ora 文件，根据下面的指定为备用角色设置该文件：

	db_name='TEST'
	db_unique_name='TEST_STBY'
	db_create_file_dest='c:\OracleDatabase\oradata\test_stby'
	db_file_name_convert='TEST','TEST_STBY'
	log_file_name_convert='TEST','TEST_STBY'


	job_queue_processes=10
	LOG_ARCHIVE_CONFIG='DG_CONFIG=(TEST,TEST_STBY)'
	LOG_ARCHIVE_DEST_1='LOCATION=c:\OracleDatabase\TEST_STBY\archives VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME='TEST'
	LOG_ARCHIVE_DEST_2='SERVICE=TEST LGWR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE)
	LOG_ARCHIVE_DEST_STATE_1='ENABLE'
	LOG_ARCHIVE_DEST_STATE_2='ENABLE'
	LOG_ARCHIVE_FORMAT='%t_%s_%r.arc'
	LOG_ARCHIVE_MAX_PROCESSES=30


前面的语句块包含两个重要设置项：

-	***.LOG\_ARCHIVE\_DEST\_1：**需要在 Machine2 中手动创建 c:\\OracleDatabase\\TEST\_STBY\\archives 文件夹。
-	***.LOG\_ARCHIVE\_DEST\_2：**此为可选步骤。设置此项的目的是，当主计算机处于维护模式并且备用计算机成为主数据库时，可能需要用到此项设置。

然后，需要启动备用实例。在备用数据库服务器上的 Windows 命令提示符下输入以下命令，通过创建 Windows 服务来创建 Oracle 实例：

	oradim -NEW -SID TEST\_STBY -STARTMODE MANUAL

**Oradim** 命令将创建 Oracle 实例，但不会启动它。可以在 C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\BIN 目录中找到该实例。

##配置侦听器和 tnsnames，支持主计算机和备用计算机上的数据库
在创建备用数据库之前，需确保配置中的主数据库和备用数据库可以互相通信。为此，需要手动或使用网络配置实用工具 NETCA 来配置侦听器和 TNSNames。使用恢复管理器实用工具 (RMAN) 时，必须完成此任务。

### 在两个服务器上配置 listener.ora，用于保存两个数据库的条目

通过远程桌面连接到 Machine1，并根据下面的指定编辑 listener.ora 文件。编辑 listener.ora 文件时，请始终确保左括号和右括号在同一列中对齐。可在以下文件夹中找到 listener.ora 文件：c:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\NETWORK\\ADMIN\\。

	# listener.ora Network Configuration File: C:\OracleDatabase\product\11.2.0\dbhome_1\network\admin\listener.ora

	# Generated by Oracle configuration tools.

	SID_LIST_LISTENER =
	  (SID_LIST =
	    (SID_DESC =
	      (SID_NAME = test)
	      (ORACLE_HOME = C:\OracleDatabase\product\11.2.0\dbhome_1)
	      (PROGRAM = extproc)
	      (ENVS = "EXTPROC_DLLS=ONLY:C:\OracleDatabase\product\11.2.0\dbhome_1\bin\oraclr11.dll")
	    )
	  )

	LISTENER =
	  (DESCRIPTION_LIST =
	    (DESCRIPTION =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE1)(PORT = 1521))
	      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
	    )
	  )

接下来，通过远程桌面连接到 Machine2，并按如下所示编辑 listener.ora 文件：
	# listener.ora Network Configuration File: C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\network\\admin\\listener.ora

	# Generated by Oracle configuration tools.

	SID_LIST_LISTENER =
	  (SID_LIST =
	    (SID_DESC =
	      (SID_NAME = test_stby)
	      (ORACLE_HOME = C:\OracleDatabase\product\11.2.0\dbhome_1)
	      (PROGRAM = extproc)
	      (ENVS = "EXTPROC_DLLS=ONLY:C:\OracleDatabase\product\11.2.0\dbhome_1\bin\oraclr11.dll")
	    )
	  )

	LISTENER =
	  (DESCRIPTION_LIST =
	    (DESCRIPTION =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE2)(PORT = 1521))
	      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
	    )
	  )


### 在主虚拟机和备用虚拟机上配置 tnsnames.ora，用于保存主数据库和备用数据库的条目

通过远程桌面连接到 Machine1，并根据下面的指定编辑 tnsnames.ora 文件。可在以下文件夹中找到 tnsnames.ora 文件：c:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\NETWORK\\ADMIN\\。

	TEST =
	  (DESCRIPTION =
	    (ADDRESS_LIST =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE1)(PORT = 1521))
	    )
	    (CONNECT_DATA =
	      (SERVICE_NAME = test)
	    )
	  )

	TEST_STBY =
	  (DESCRIPTION =
	    (ADDRESS_LIST =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE2)(PORT = 1521))
	    )
	    (CONNECT_DATA =
	      (SERVICE_NAME = test_stby)
	    )
	  )

通过远程桌面连接到 Machine2，并按如下所示编辑 tnsnames.ora 文件：

	TEST =
	  (DESCRIPTION =
	    (ADDRESS_LIST =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE1)(PORT = 1521))
	    )
	    (CONNECT_DATA =
	      (SERVICE_NAME = test)
	    )
	  )

	TEST_STBY =
	  (DESCRIPTION =
	    (ADDRESS_LIST =
	      (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE2)(PORT = 1521))
	    )
	    (CONNECT_DATA =
	      (SERVICE_NAME = test_stby)
	    )
	  )


### 启动侦听器，并检查是否可以从两个虚拟机 tnsping 到两个服务。

在主虚拟机和备用虚拟机中打开新的 Windows 命令提示符并运行以下语句：

	C:\Users\DBAdmin>tnsping test

	TNS Ping Utility for 64-bit Windows: Version 11.2.0.1.0 - Production on 14-NOV-2013 06:29:08
	Copyright (c) 1997, 2010, Oracle.  All rights reserved.
	Used parameter files:
	C:\OracleDatabase\product\11.2.0\dbhome_1\network\admin\sqlnet.ora
	Used TNSNAMES adapter to resolve the alias
	Attempting to contact (DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE1)(PORT = 1521))) (CONNECT_DATA = (SER
	VICE_NAME = test)))
	OK (0 msec)


	C:\Users\DBAdmin>tnsping test_stby

	TNS Ping Utility for 64-bit Windows: Version 11.2.0.1.0 - Production on 14-NOV-2013 06:29:16
	Copyright (c) 1997, 2010, Oracle.  All rights reserved.
	Used parameter files:
	C:\OracleDatabase\product\11.2.0\dbhome_1\network\admin\sqlnet.ora
	Used TNSNAMES adapter to resolve the alias
	Attempting to contact (DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = MACHINE2)(PORT = 1521))) (CONNECT_DATA = (SER
	VICE_NAME = test_stby)))
	OK (260 msec)


##在 nomount 状态下启动备用实例
设置环境，以支持备用虚拟机 (MACHINE2) 上的备用数据库。

首先，将密码文件从主计算机 (Machine1) 手动复制到备用计算机 (Machine2)。必须执行此操作，因为两个计算机上的 **sys** 密码必须相同。

然后，在 Machine2 中打开 Windows 命令提示符，并将环境变量设置为指向备用数据库，如下所示：

	SET ORACLE_HOME=C:\OracleDatabase\product\11.2.0\dbhome_1
	SET ORACLE_SID=TEST_STBY

接下来，在 nomount 状态下启动备用数据库，然后生成 spfile。

启动数据库：

	SQL>shutdown immediate;

	SQL>startup nomount
	ORACLE instance started.

	Total System Global Area  747417600 bytes
	Fixed Size                  2179496 bytes
	Variable Size             473960024 bytes
	Database Buffers          264241152 bytes
	Redo Buffers                7036928 bytes


##使用 RMAN 克隆数据库并创建备用数据库
可以使用恢复管理器实用工具 (RMAN) 生成主数据库的任何备份副本以创建物理备用数据库。

通过远程桌面连接到备用虚拟机 (MACHINE2)，并通过为 TARGET（主数据库，Machine1）和 AUXILLARY（备用数据库，Machine2）实例指定完整连接字符串来运行 RMAN 实用工具。

>[AZURE.IMPORTANT] 由于备用服务器计算机中尚无数据库，因此请不要使用操作系统身份验证。

	C:\> RMAN TARGET sys/password@test AUXILIARY sys/password@test_STBY

	RMAN>DUPLICATE TARGET DATABASE
	  FOR STANDBY
	  FROM ACTIVE DATABASE
	  DORECOVER
	    NOFILENAMECHECK;

##在托管恢复模式下启动物理备用数据库
本教程介绍如何创建物理备用数据库。有关创建逻辑备用数据库的信息，请参阅 Oracle 文档。

打开 SQL*Plus 命令提示符，在备用虚拟机或服务器 (MACHINE2) 上启用数据防护，如下所示：

	SHUTDOWN IMMEDIATE;
	STARTUP MOUNT;
	ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;

在 **MOUNT** 模式下打开备用数据库时，将继续传递存档日志，并且托管恢复进程会继续在备用数据库上应用日志。这可以确保备用数据库与主数据库保持同步更新。请注意，在此期间，无法为了报告目的访问备用数据库。

以 **READ ONLY** 模式打开备用数据库时，将继续传送存档日志。但是，托管恢复进程将会停止。这将导致备用数据库越来越过时，直到托管恢复进程恢复。在此期间，可以为了报告目的访问备用数据库，但数据可能不会反映最新的更改。

一般情况下，建议让备用数据库保持 **MOUNT** 模式，使其中的数据在主数据库发生故障时能够保持最新。但是，也可以为了报告目的，根据应用程序的要求让备用数据库保持 **READ ONLY** 模式。以下步骤演示如何使用 SQL*Plus 在只读模式下启用数据防护：

	SHUTDOWN IMMEDIATE;
	STARTUP MOUNT;
	ALTER DATABASE OPEN READ ONLY;


##验证物理备用数据库
本部分介绍如何以管理员身份验证高可用性配置。

打开 SQL*Plus 命令提示窗口，并检查备用虚拟机 (Machine2) 上的存档重做日志：

	SQL> show parameters db_unique_name;

	NAME                                TYPE       VALUE
	------------------------------------ ----------- ------------------------------
	db_unique_name                      string     TEST_STBY

	SQL> SELECT NAME FROM V$DATABASE

	SQL> SELECT SEQUENCE#, FIRST_TIME, NEXT_TIME, APPLIED FROM V$ARCHIVED_LOG ORDER BY SEQUENCE#;

	SEQUENCE# FIRST_TIM NEXT_TIM APPLIED
	----------------  ---------------  --------------- ------------
	45                    23-FEB-14   23-FEB-14   YES
	45                    23-FEB-14   23-FEB-14   NO
	46                    23-FEB-14   23-FEB-14   NO
	46                    23-FEB-14   23-FEB-14   YES
	47                    23-FEB-14   23-FEB-14   NO
	47                    23-FEB-14   23-FEB-14   NO

打开 SQL*Plus 命令提示窗口，并在主计算机 (Machine1) 上切换日志文件：

	SQL> alter system switch logfile;
	System altered.

	SQL> archive log list
	Database log mode              Archive Mode
	Automatic archival             Enabled
	Archive destination            C:\OracleDatabase\archive
	Oldest online log sequence     69
	Next log sequence to archive   71
	Current log sequence           71

检查备用虚拟机 (Machine2) 上的存档重做日志：

	SQL> SELECT SEQUENCE#, FIRST_TIME, NEXT_TIME, APPLIED FROM V$ARCHIVED_LOG ORDER BY SEQUENCE#;

	SEQUENCE# FIRST_TIM NEXT_TIM APPLIED
	----------------  ---------------  --------------- ------------
	45                    23-FEB-14   23-FEB-14   YES
	46                    23-FEB-14   23-FEB-14   YES
	47                    23-FEB-14   23-FEB-14   YES
	48                    23-FEB-14   23-FEB-14   YES

	49                    23-FEB-14   23-FEB-14   YES
	50                    23-FEB-14   23-FEB-14   IN-MEMORY

检查备用虚拟机 (Machine2) 上是否存在任何差异：

	SQL> SELECT * FROM V$ARCHIVE_GAP;
	no rows selected.

另一种验证方法是故障转移到备用数据库，然后测试它是否可以故障回复到主数据库。若要将备用数据库激活为主数据库，请使用以下语句：

	SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE FINISH;
	SQL> ALTER DATABASE ACTIVATE STANDBY DATABASE;

如果未在原始主数据库上启用闪回，建议删除原始主数据库，然后将它重新创建为备用数据库。

建议在主数据库和备用数据库上启用闪回数据库。发生故障转移时，主数据库可以闪回到故障转移前的时间点，并快速转换为备用数据库。

<!---HONumber=Mooncake_1017_2016-->