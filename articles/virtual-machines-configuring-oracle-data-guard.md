<properties
	pageTitle="Configuring Oracle Data Guard in VMs | Azure"
	description="Step through a tutorial for setting up and implementing Oracle Data Guard on Azure virtual machines for high availability and disaster recovery."
	services="virtual-machines-windows"
	authors="rickstercdn"
	manager="timlt"
	documentationCenter=""
	tags="azure-service-management"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="06/22/2015"
	wacn.date="08/29/2015"/>

#为 Azure 配置 Oracle Data Guard

本教程将演示如何在 Azure 虚拟机环境中设置和实施 Oracle Data Guard，实现高可用性和灾难恢复。本教程重点介绍对非 RAC Oracle Database 的单向复制。

Oracle Data Guard 支持对 Oracle Database 的数据保护和灾难恢复。它是针对整个的 Oracle Database 提供灾难恢复、数据保护和高可用性的简单、高性能、嵌入式解决方案。

本教程假定你已具有 Oracle Database 的高可用性和灾难恢复概念的理论和实践知识。有关信息，请参阅 [Oracle 网站](http://www.oracle.com/technetwork/database/features/availability/index.html)以及[Oracle Data Guard 概念和管理指南](https://docs.oracle.com/cd/E11882_01/server.112/e41134/toc.htm)。

此外，本教程假定你已经实现了以下先决条件：

- 已查看 [Oracle 虚拟机映像 - 其他注意事项](/documentation/articles/virtual-machines-windows-classic-oracle-considerations/)主题中的高可用性和灾难恢复注意事项部分。请注意，Azure 支持独立 Oracle Database 实例，但当前暂不支持 Oracle 实时应用集群 (Oracle RAC)。

- 已使用 Windows Server 上 Oracle Enterprise Edition 映像所提供的相同平台在 Azure 中创建了两个虚拟机 (VM)。有关信息，请参阅[在 Azure 中创建 Oracle Database 12 c 虚拟机](/documentation/articles/virtual-machines-windows-create-oracle-weblogic-server-12c/)和 [Azure 虚拟机](http://www.windowsazure.cn/documentation/services/virtual-machines/)。确保这两个虚拟机在[相同的云服务](/documentation/articles/virtual-machines-windows-load-balance/)和相同的[虚拟网络](/documentation/services/networking/)中，以确保它们可以通过永久性专用 IP 地址相互进行访问。此外，建议将 VM 放在相同的[可用性集](/documentation/articles/virtual-machines-windows-manage-availability/)中，以便允许 Azure 将它们放入不同的容错域和升级域中。请注意，Oracle Data Guard 仅在 Oracle Database Enterprise Edition 中可用。每个虚拟机必须具有至少 2 GB 的内存和 5 GB 的磁盘空间。有关平台提供的 VM 大小的最新信息，请参阅 [Azure 的虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。如果自己的 VM 需要附加磁盘卷，则可以附加更多磁盘。有关信息，请参阅[如何将数据磁盘附加到虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)。

- 已将名称为“Machine1”的虚拟机设为主虚拟机，而名称为“Machine2”的虚拟机则设为 Azure 经典管理门户中的备用 VM。

- 已设置“ORACLE\_HOME”环境变量以指向主虚拟机和备用虚拟机中的同一个 oracle 根安装路径，如 `C:\OracleDatabase\product\11.2.0\dbhome_1\database`。

- 作为“管理员”组或“ORA\_DBA”组的成员登录到自己的 Windows 服务器。

在本教程中，你将：

实现物理备用数据库环境

1. 创建主数据库

2. 为备用数据库的创建准备主数据库

	1. 启用强制日志记录

	2. 创建密码文件

	3. 配置备用重做日志

	4. 启用存档

	5. 设置主数据库初始化参数

创建物理备用数据库

1. 为备用数据库准备初始化参数文件

2. 配置侦听器和 tnsnames 以支持主虚拟机和备用虚拟机上的数据库

	1. 在这两个服务器中配置 listener.ora 以保存两个数据库的条目

	2. 在主虚拟机和备用虚拟机上配置 tnsnames.ora 以保存主数据库和备用数据库的条目

	3. 启动侦听器并针对两个服务检查两个虚拟机上的 tnsping。

3. 启动 nomount 状态中的备用实例

4. 使用 RMAN 克隆数据库并创建备用数据库

5. 在托管的恢复模式下启动物理备用数据库

6. 验证物理备用数据库

> [AZURE.IMPORTANT]本教程中已安装程序并针对以下硬件和软件配置进行了测试：
>
>| | **主数据库** | **备用数据库** |
>|----------------------|-------------------------------------------|-------------------------------------------|
>| **Oracle 版本** | Oracle11g 企业版本 (11.2.0.4.0) | Oracle11g 企业版本 (11.2.0.4.0) |
>| **虚拟机名称** | Machine1 | Machine2 |
>| **操作系统** | Windows 2008 R2 | Windows 2008 R2 |
>| **Oracle SID** | TEST | TEST\_STBY |
>| **内存** | 最小值 2 GB | 最小值 2 GB |
>| **磁盘空间** | 最小值 5 GB | 最小值 5 GB |

对于后续版本的 Oracle Database 和 Oracle Data Guard，可能需要实现一些其他更改。有关最新版本的特定信息，请参阅 Oracle 网站上的 [Data Guard](http://www.oracle.com/technetwork/database/features/availability/data-guard-documentation-152848.html) 和 [Oracle Database](http://www.oracle.com/us/corporate/features/database-12c/index.html) 文档。

##实现物理备用数据库环境
### 1\.创建主数据库

- 在主虚拟机中创建主数据库“TEST”。有关信息，请参阅“创建和配置 Oracle Database”。
- 在 SQL*Plus 命令提示符中使用 SYSDBA 角色作为 SYS 用户连接到自己的数据库，并运行以下语句以查看数据库的名称：

		SQL> select name from v$database;
		
		The result will display like the following:
		
		NAME
		---------
		TEST
- 然后，从 dba\_data\_files 系统视图查询数据库文件的名称：

		SQL> select file_name from dba_data_files; 
		FILE_NAME 
		------------------------------------------------------------------------------- 
		C:\ <YourLocalFolder>\TEST\USERS01.DBF
		C:\ <YourLocalFolder>\TEST\UNDOTBS01.DBF
		C:\ <YourLocalFolder>\TEST\SYSAUX01.DBF
		C:<YourLocalFolder>\TEST\SYSTEM01.DBF
		C:<YourLocalFolder>\TEST\EXAMPLE01.DBF

### 2\.为备用数据库的创建准备主数据库

创建备用数据库之前，建议你确保已正确配置主数据库。以下是需要执行的步骤列表：

1. 启用强制日志记录
2. 创建密码文件
3. 配置备用重做日志
4. 启用存档
5. 设置主数据库初始化参数

#### 启用强制日志记录

为了实现备用数据库，需要在主数据库中启用“强制日志记录”。此选项可确保即使事件中已完成“nologging”操作，也将优先进行强制日志记录并将所有操作记录到重做日志中。因此，我们需确保记录主数据库中的所有内容并复制到备用服务器中，包括主数据库中的所有操作。运行 alter database 语句以启用强制日志记录：

	SQL> ALTER DATABASE FORCE LOGGING;

	Database altered.

#### 创建密码文件

为了能够将主服务器中的存档日志转移并应用到备用服务器，主服务器和备用服务器上的 sys 密码必须完全相同。这正是在主数据库上创建密码文件并将其复制到备用服务器的原因。

>[AZURE.IMPORTANT]使用 Oracle Database 12c 时，存在新用户“SYSDG”，可将此用户用于管理 Oracle Data Guard。有关详细信息，请参阅 [Oracle Database 12c 版本中的更改](http://docs.oracle.com/database/121/UNXAR/release_changes.htm#UNXAR404)。

此外，请确保 Machine1 中已定义了 ORACLE\_HOME 环境。如果尚未定义，请使用“环境变量”对话框将其定义为环境变量。要访问此对话框，双击“控制面板”中的“系统图标”启动“系统”实用工具；然后单击“高级”选项卡并选择“环境变量”。单击“系统变量”下的“新建”按钮设置环境变量。设置环境变量后，关闭现有的 Windows 命令提示符并打开新的提示符。

运行以下语句以切换到 Oracle\_Home 目录，例如 C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\database。

	cd %ORACLE_HOME%\database

然后，使用密码文件创建实用工具创建密码文件 [ORAPWD](http://docs.oracle.com/cd/B28359_01/server.111/b28310/dba007.htm)。在 Machine1 的同一个 Windows 命令提示符中，将密码值设置为“SYS”来运行以下命令：

	ORAPWD FILE=PWDTEST.ora PASSWORD=password FORCE=y

此命令创建一个名为 PWDTEST.ora 的密码文件，该文件位于 ORACLE\_HOME\\database 目录中。应手动将此文件复制到 Machine2 的 %ORACLE\_HOME%\\database 目录中。

#### 配置备用重做日志

然后，需要配置备用重做日志，以便主虚拟机成为备用虚拟机时，可以正确地接收重做日志。在此处预先创建这些日志还可在备用服务器上自动创建备用重做日志。请务必按与联机重做日志相同的大小配置备用重做日志 (SRL)。当前备用重做日志文件的大小必须与当前主数据库联机重做日志文件的大小完全匹配。

在 Machine1 SQL*PLUS 命令提示符中运行以下语句。V$ 日志文件是一个包含关于重做日志文件信息的系统视图。

	SQL> select * from v$logfile;
	GROUP# STATUS  TYPE    MEMBER                                                       IS_
	---------- ------- ------- ------------------------------------------------------------ ---
	3         ONLINE  C:<YourLocalFolder>\TEST\REDO03.LOG               NO
	2         ONLINE  C:<YourLocalFolder>\TEST\REDO02.LOG               NO
	1         ONLINE  C:<YourLocalFolder>\TEST\REDO01.LOG               NO
	Next, query the v$log system view, displays log file information from the control file. 
	SQL> select bytes from v$log; 
	BYTES
	----------
	52428800
	52428800
	52428800


请注意，52428800 是 50 MB。

然后，在 SQL*Plus 窗口中，运行以下语句以将新的备用重做日志文件组添加到备用数据库中，并指定使用 GROUP 子句标识组的编号。使用组编号可以简化管理备用重做日志文件组操作：

	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 4 'C:<YourLocalFolder>\TEST\REDO04.LOG' SIZE 50M;
	Database altered.
	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 5 'C:<YourLocalFolder>\TEST\REDO05.LOG' SIZE 50M;
	Database altered.
	SQL> ALTER DATABASE ADD STANDBY LOGFILE GROUP 6 'C:<YourLocalFolder>\TEST\REDO06.LOG' SIZE 50M;
	Database altered.

接下来，运行以下系统视图查看有关重做日志文件的信息列表。此操作还将验证已创建的备用重做日志文件组：

	SQL> select * from v$logfile;
	GROUP# STATUS  TYPE MEMBER IS_
	---------- ------- ------- --------------------------------------------- ---
	3         ONLINE C:<YourLocalFolder>\TEST\REDO03.LOG NO
	2         ONLINE C:<YourLocalFolder>\TEST\REDO02.LOG NO
	1         ONLINE C:<YourLocalFolder>\TEST\REDO01.LOG NO
	4         STANDBY C:<YourLocalFolder>\TEST\REDO04.LOG
	5         STANDBY C:<YourLocalFolder>\TEST\REDO05.LOG NO
	6         STANDBY C:<YourLocalFolder>\TEST\REDO06.LOG NO
	6 rows selected.

#### 启用存档

然后，运行以下语句以将主数据库设置为 ARCHIVELOG 模式并启用自动存档，从而启用存档。可以通过装入数据库然后执行 archivelog 命令来启用存档日志模式。

首先，以 sysdba 身份登录。在 Windows 命令提示符中，运行：

	sqlplus /nolog
	
	connect / as sysdba

然后，在 SQL*Plus 命令提示符中关闭数据库：

	SQL> shutdown immediate;
	Database closed.
	Database dismounted.
	ORACLE instance shut down.

然后，执行 startup mount 命令装入数据库。这可确保 Oracle 将指定数据库与实例相关联。

	SQL> startup mount;
	ORACLE instance started.
	Total System Global Area 1503199232 bytes
	Fixed Size                  2281416 bytes
	Variable Size             922746936 bytes
	Database Buffers          570425344 bytes
	Redo Buffers                7745536 bytes
	Database mounted.

然后，运行：

	SQL> alter database archivelog; 
	Database altered.

然后，运行 Alter database 语句及 Open 子句使数据库可供正常使用：

	SQL> alter database open;
	
	Database altered.

#### 设置主数据库初始化参数

要配置 Data Guard，首先需要在常规 pfile（文本初始化参数文件）上创建并配置备用参数。当 pfile 准备就绪后，你需要将其转换为服务器参数文件 (SPFILE)。

可以使用 INIT.ORA 文件中的参数控制 Data Guard 环境。在学习本教程时，你需要更新主数据库 INIT.ORA，以便可以同时容纳这两种角色：主数据库或备用数据库。

	SQL> create pfile from spfile;
	File created.

接下来，你需要编辑 pfile 以添加备用参数。要执行此操作，请打开 %ORACLE\_HOME%\\database 位置上的 INITTEST.ORA 文件。接下来，将以下语句追加到 INITTEST.ora 文件中。请注意，INIT.ORA 文件的命名约定是 INIT<YourDatabaseName>.ORA。
	
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


前面的语句块包括三个重要的安装程序项：- **LOG\_ARCHIVE\_CONFIG...：**使用此语句定义唯一的数据库 ID。- **LOG\_ARCHIVE\_DEST\_1...：**使用此语句定义本地存档文件夹位置。我们建议为数据库存档需求创建一个新目录，并显式使用此语句指定本地存档位置，而不是使用 Oracle 的默认文件夹 %ORACLE\_HOME%\\database\\archive。- **LOG\_ARCHIVE\_DEST\_2 ....LGWR ASYNC...：**定义异步日志写入程序进程 (LGWR) 以收集事务重做数据，并将其传递到备用目标。在这里，DB\_UNIQUE\_NAME 指定目标备用服务器上数据库的唯一名称。

新的参数文件准备就绪后，你需要使用它创建 spfile。

首先，关闭数据库：

	SQL> shutdown immediate;
	
	Database closed.
	
	Database dismounted.
	
	ORACLE instance shut down.

接下来，按以下方式运行 startup nomount 命令：

	SQL> startup nomount pfile='c:\OracleDatabase\product\11.2.0\dbhome_1\database\initTEST.ora';
	ORACLE instance started.
	Total System Global Area 1503199232 bytes
	Fixed Size                  2281416 bytes
	Variable Size             922746936 bytes
	Database Buffers          570425344 bytes
	Redo Buffers                7745536 bytes

现在，创建 spfile：

	SQL>create spfile frompfile='c:\OracleDatabase\product\11.2.0\dbhome\_1\database\initTEST.ora';

	File created.

然后，关闭数据库：

	SQL> shutdown immediate;
	
	ORA-01507: database not mounted

接下来，使用 startup 命令启动一个实例：

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
本部分重点介绍必须在 Machine2 中执行以准备物理备用数据库的步骤。

首先，需要通过 Azure 经典管理门户远程桌面连接到 Machine2。

然后，在备用服务器 (Machine2) 上，创建备用数据库所有的必要文件夹，如 C:\\<YourLocalFolder>\\TEST。在学习本教程时，请确保文件夹结构匹配 Machine1 中的文件夹结构，以保留所有必要文件，例如 controlfile、datafiles、redologfiles、udump、bdump 和 cdump 文件。此外，在 Machine2 中定义 ORACLE\_HOME 和 ORACLE\_BASE 环境变量。如果尚未定义，请使用“环境变量”对话框将它们定义为环境变量。要访问此对话框，双击“控制面板”中的“系统图标”启动“系统”实用工具；然后单击“高级”选项卡并选择“环境变量”。单击“系统变量”下的“新建”按钮设置环境变量。设置环境变量后，则需要关闭现有的 Windows 命令提示符并打开新的提示符以查看更改。

接下来，执行以下步骤：

1. 为备用数据库准备初始化参数文件

2. 配置侦听器和 tnsnames 以支持主虚拟机和备用虚拟机上的数据库

	1. 在这两个服务器中配置 listener.ora 以保存两个数据库的条目

	2. 在主虚拟机和备用虚拟机上配置 tnsnames.ora 以保存主数据库和备用数据库的条目

	3. 启动侦听器并针对两个服务检查两个虚拟机上的 tnsping。

3. 启动 nomount 状态中的备用实例

4. 使用 RMAN 克隆数据库并创建备用数据库

5. 在托管的恢复模式下启动物理备用数据库

6. 验证物理备用数据库

### 1\.为备用数据库准备初始化参数文件

本部分将介绍如何为备用数据库准备初始化参数文件。要执行此操作，首先需手动从 Machine1 将 INITTEST.ORA 文件复制到 Machine2 中。应该可以在两个虚拟机的 %ORACLE\_HOME%\\database 文件夹中看到 INITTEST.ORA 文件。然后，修改 Machine2 中的 INITTEST.ora 文件以将其设置为按照以下指定的备用角色：
	
	db_name='TEST'
	db_unique_name='TEST_STBY'
	db_create_file_dest='c:\OracleDatabase\oradata\test_stby’
	db_file_name_convert=’TEST’,’TEST_STBY’
	log_file_name_convert='TEST','TEST_STBY'
	
	
	job_queue_processes=10
	LOG_ARCHIVE_CONFIG='DG_CONFIG=(TEST,TEST_STBY)'
	LOG_ARCHIVE_DEST_1='LOCATION=c:\OracleDatabase\TEST_STBY\archives VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=’TEST'
	LOG_ARCHIVE_DEST_2='SERVICE=TEST LGWR ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) 
	LOG_ARCHIVE_DEST_STATE_1='ENABLE'
	LOG_ARCHIVE_DEST_STATE_2='ENABLE'
	LOG_ARCHIVE_FORMAT='%t_%s_%r.arc'
	LOG_ARCHIVE_MAX_PROCESSES=30


前面的语句块中包含两个重要的安装程序项：

-	***.LOG\_ARCHIVE\_DEST\_1：**需要在 Machine2 中手动创建 c:\\OracleDatabase\\TEST\_STBY\\archives 文件夹。
-	***.LOG\_ARCHIVE\_DEST\_2：**这是一个可选步骤。进行设置，因为主计算机处于维护和备用计算机成为主数据库时可能会需要它。

然后，你需要启动备用实例。在备用数据库服务器上，在 Windows 命令提示符中输入以下命令，通过创建新的 Windows 服务来创建 Oracle 实例：

	oradim -NEW -SID TEST\_STBY -STARTMODE MANUAL

请注意，“Oradim”命令将创建一个 Oracle 实例，但并不会启动该实例。可以在 C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\BIN 目录中找到它。

##配置侦听器和 tnsnames 以支持主虚拟机和备用虚拟机上的数据库
创建备用数据库之前，你需要确保自己配置中的主要数据库和备用数据库可以互相通信。要执行此操作，需要手动或使用网络配置实用工具 NETCA 来配置侦听器和 TNSNames。当你使用恢复管理器实用工具 (RMAN) 时，这是必需的任务。

### 在这两个服务器中配置 listener.ora 以保存两个数据库的条目

远程桌面连接到 Machine1 并按以下指定编辑 listener.ora 文件。编辑 listener.ora 文件时，请始终确保左括号和右括号在同一列对齐。listener.ora 文件位于以下文件夹：c:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\NETWORK\\ADMIN\\。

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

接下来，远程桌面连接到 Machine2 并如下所示编辑 listener.ora 文件：# listener.ora 网络配置文件：C:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\network\\admin\\listener.ora
	
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


### 在主虚拟机和备用虚拟机上配置 tnsnames.ora 以保存主数据库和备用数据库的条目

远程桌面连接到 Machine1 并按以下所述编辑 tnsnames.ora 文件。tnsnames.ora 文件位于以下文件夹中：c:\\OracleDatabase\\product\\11.2.0\\dbhome\_1\\NETWORK\\ADMIN\\。

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

远程桌面连接到 Machine2 并如下所示编辑 tnsnames.ora 文件：
	
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


### 启动侦听器并针对两个服务检查两个虚拟机上的 tnsping。

打开主要虚拟机和备用虚拟机中新的 Windows 命令提示符并运行以下语句：

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


##启动 nomount 状态中的备用实例
需要设置环境以支持备用虚拟机 (MACHINE2) 中的备用数据库。

首先，从主计算机 (Machine1) 中将密码文件手动复制到备用计算机 (Machine2) 中。这是必需的，因为两台计算机上的“sys”密码必须完全相同。

然后，在 Machine2 中打开 Windows 命令提示符并设置环境变量以指向备用数据库，如下所示：

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
恢复管理器实用工具 (RMAN) 可用于获取主数据库的任何备份副本，以创建物理备用数据库。

通过为目标（主数据库，Machine1）实例和辅助（备用数据库，Machine2）实例指定完整的连接字符串，远程桌面连接到备用虚拟机 (MACHINE2) 并运行 RMAN 实用工具。

>[AZURE.IMPORTANT]请勿使用操作系统身份验证，因为备用服务器计算机中尚无任何数据库。

	C:\> RMAN TARGET sys/password@test AUXILIARY sys/password@test_STBY
	
	RMAN>DUPLICATE TARGET DATABASE
	  FOR STANDBY
	  FROM ACTIVE DATABASE
	  DORECOVER
	    NOFILENAMECHECK;

##在托管的恢复模式下启动物理备用数据库
本教程将演示如何创建物理备用数据库。有关创建逻辑备用数据库的信息，请参阅 Oracle 文档。

打开 SQL*Plus 命令提示符并启用备用虚拟机或服务器 (MACHINE2) 上的 Data Guard，如下所示：

	SHUTDOWN IMMEDIATE;
	STARTUP MOUNT;
	ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;

打开处于“装入”模式的备用数据库时，存档日志传送将继续，并且托管的恢复过程会继续备用数据库中的日志应用。这可确保备用数据库与主数据库同步。请注意，在此期间无法出于报告目的访问备用数据库。

打开处于“只读”模式的备用数据库时，存档日志传送将继续。但托管的恢复过程将停止。这将导致备用数据库变得越来越过时，直到托管的恢复过程恢复。你可以在此期间出于报告目的访问备用数据库，但数据可能不会反映最新的更改。

通常情况下，我们建议将备用数据库保留为“装入”模式，从而保持备用数据库中数据为最新，以防主数据库发生故障。但是，也可以出于报告目的将备用数据库保留为“只读”模式，具体取决于应用程序的要求。以下步骤将演示如何使用 SQL*Plus 启用只读模式的 Data Guard：

	SHUTDOWN IMMEDIATE;
	STARTUP MOUNT;
	ALTER DATABASE OPEN READ ONLY;


##验证物理备用数据库
本节将演示如何以管理员身份验证高可用性配置。

打开 SQL*Plus 命令提示符窗口并检查备用虚拟机 (Machine2) 上的已存档重做日志：

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

打开 SQL*Plus 命令提示符窗口并交换主计算机 (Machine1) 上的 logfiles：

	SQL> alter system switch logfile; 
	System altered.
	
	SQL> archive log list
	Database log mode              Archive Mode
	Automatic archival             Enabled
	Archive destination            C:\OracleDatabase\archive
	Oldest online log sequence     69
	Next log sequence to archive   71
	Current log sequence           71

检查备用虚拟机 (Machine2) 上的已存档重做日志：

	SQL> SELECT SEQUENCE#, FIRST_TIME, NEXT_TIME, APPLIED FROM V$ARCHIVED_LOG ORDER BY SEQUENCE#;
	
	SEQUENCE# FIRST_TIM NEXT_TIM APPLIED
	----------------  ---------------  --------------- ------------
	45                    23-FEB-14   23-FEB-14   YES
	46                    23-FEB-14   23-FEB-14   YES
	47                    23-FEB-14   23-FEB-14   YES
	48                    23-FEB-14   23-FEB-14   YES
	
	49                    23-FEB-14   23-FEB-14   YES
	50                    23-FEB-14   23-FEB-14   IN-MEMORY

检查备用虚拟机 (Machine2) 上的任何缺陷：

	SQL> SELECT * FROM V$ARCHIVE_GAP;
	no rows selected.

另一种验证方法是，故障转移到备用数据库，然后测试是否可以故障回复到主数据库。要激活备用数据库作为主数据库，请使用以下语句：

	SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE FINISH;
	SQL> ALTER DATABASE ACTIVATE STANDBY DATABASE;

如果尚未在原始主数据库中启用闪回原，则建议你删除原始主数据库，然后重新创建备用数据库。

我们建议你在主数据库和备用数据库上启用闪回数据库。发生故障转移时，主数据库可以闪回到故障转移前的时间，并将其快速转换为备用数据库。

<!---HONumber=67-->