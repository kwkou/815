<properties
   pageTitle="Azure Linux 虚拟机上的数据库的备份 | Azure"
   description="本文介绍如何在 Azure Linux 虚拟机上备份 MySQL、Redis、MongoDB 数据库"
   services="open-source"
   documentationCenter=""
   authors=""
   manager=""
   editor=""/>

<tags
   ms.service="open-source-website"  
   ms.date=""
   wacn.date="06/14/2016"/>

#Azure Linux 虚拟机上的数据库的备份

##目录
- [备份 MySQL](#backup-mysql)  
- [Mysqldump](#mysqldump)  
- [主从复制](#primary-secondary)  
- [Xtrabackup](#xtrabackup)  
- [备份 Redis](#backup-redis)  
- [备份 MongoDB](#backup-mongodb)  




##<a id="backup-mysql"></a>备份 MySQL

有许多种备份 MySQL 数据库的方式。如果您使用的是 Azure 提供的 MySQL 服务，您可以参考 [Azure MySQL tutorial](/documentation/articles/mysql-database-get-started/) 进行备份

如果您是在 Azure 的 Linux 虚拟机上自行搭建的 MySQL 数据库的话，这篇文档就非常适合您了。我们会介绍三种 MySQL 备份的方式：mysqldump, 主从复制, 以及 xtrabackup. 

如果您还没有 Azure 下的 LINUX 虚拟机，请参考 [Azure Linux VM tutorial](/documentation/articles/virtual-machines-linux-tutorial-portal-rm/). 创建 LINUX 虚拟机。  
连接到您的 LINUX 虚拟机。如果这是您第一次使用 Azure 的 LINUX 虚拟机，请参考
 [Azure Linux VM tutorial](/documentation/articles/virtual-machines-linux-tutorial-portal-rm/) 连接到虚拟机。  
我们假设您已经在 Azure 的 Linux 虚拟机上安装好了 MySQL 数据库服务，接下来开始备份过程。

##<a id="mysqldump"></a>Mysqldump
Mysqldump 组件使用逻辑备份，产生一系列可执行的 SQL 语句，重现原始数据库对象定义和表数据。它备份一个或多个 MySQL 数据库，或者把备份传输到另一台 MySQL 数据库服务器。它能生成 CSV，XML 或其他格式的输出。  

Mysqldump 要求对备份的表至少有查询的权限，如果--single-transaction 选项没有使用的话则要求有锁表的权限。正如 mysqldump --help 中的介绍那样，特定的选项可能会要求其他的一些权限。要加载一个备份文件，您必须有权限去执行备份文件里包含的 SQL 语句，比如说创建对象的语句，您必须有创建这些对象的权限。

1.通常有三种使用 mysqldump 的方式—备份一组表；备份一组数据库；备份整个数据库服务器—正如下面所示：

	shell> mysqldump [options] db_name [tbl_name ...]  
	shell> mysqldump [options] --databases db_name ...  
	shell> mysqldump [options] --all-databases  

比如有一个数据库叫做 shop, 它下面有一个表名叫 price, 下面的命令即把 price 表备份到 /tmp/price.sql  

	$mysqldump –uroot –ppassword shop price > /tmp/price.sql

2.**mysqldump** 非常适合‘复制’数据库：即把数据从一台数据库服务器拷贝到另一台数据库服务器（必须确保能连接到远端的数据库服务器 ）  

	$mysqldump -uroot –ppassword --opt shop |mysql -uroot –ppassword --host=remote_mysqlserver -C shop  

上面的命令把本地的 shop 数据库数据拷贝到远端的 shop 数据库。当然前提是远端 MySQL 数据库服务器有一个叫做 shop 的数据库。

3.对于 InnoDB 表来说，**mysqldump** 能够提供在线备份：   

	$mysqldump –uroot -ppassword --all-databases --master-data --single-transaction > all_databases.sql

4.刷新binary log  

	$mysqldump –uroot –ppassword --all-databases --flush-logs --master-data=2 > all_databases.sql  

--master-data 和 --single-transaction 选项能同时使用，对使用 InnoDB 引擎的表来说。  
--flush-logs 选项会在开始备份前，刷新 MySQL 服务器的日志文件。

5.恢复数据。很简单：  

	$mysql –uroot –ppassword  < /tmp/price.sql

6.更多关于 mysqldump 的细节，可执行 mysqldump --help 或者参考 [MySQL 官网文档](https://dev.mysql.com/doc/refman/5.5/en/mysqldump.html)。 



##<a id="primary-secondary"></a>主从复制

主从复制通常指将主服务器上的数据及时同步到从服务器上，保持主从服务器数据的一致。在 Azure 上，建议将主从服务器放置于同一个子网下，或者位于同一个数据中心，比如区域都位于中国东部或者中国北部，以降低网络延迟。在创建虚拟机时选择区域。  

Azure 有提供虚拟网络的服务，可以让不同的虚拟机处于同一个子网下。请参考[虚拟网络链接](/documentation/articles/virtual-networks-create-vnet-classic-portal/)创建虚拟网络。  

>[Azure.Note]**必须**在创建虚拟机时指定虚拟网络。创建虚拟机后，不能将它加入虚拟网络。如果不想让这些服务器处于同一个子网下，则请在选择区域时保持一致。


不同的 LINUX 发行版在配置主从复制时有少许的不同。请根据您使用的 LINUX 版本选择下面对应的步骤：

**Redhat base Linux**: ( 以CentOS 7.0, 64-bit system, MySQL Server 5.6(yum install) 为例)  

1.在主从机器上都打开3306端口。请参考链接[创建终结点](/documentation/articles/virtual-machines-set-up-endpoints/)

2.连接到主服务器。编辑 /etc/my.cnf, 在 [mysqld] 下添加如下内容

	server-id	= 1  
	log_bin	= /var/lib/mysql/mysql-bin.log

3.连接到从服务器. 编辑 /etc/my.cnf, 在 [mysqld] 下添加如下内容

	server-id	= 2
	log_bin	= /var/lib/mysql/mysql-bin.log

4.去到主服务器，启动 mysql 服务

	$sudo service mysqld start

5.创建用于复制的用户。请根据实际情况，使用真实的从服务器 IP 地址，用户名，密码代替下面相应部分。 

	$mysql -uroot -p<password>
	grant replication slave on *.* to 'repluser'@'slave ip' identified by 'password';  
	flush privileges;
	exit

6.重启 mysql 服务并查看主服务器状态

	$sudo service mysqld restart
	$mysql -uroot -p<password>
	show master status;

‘show master status’ 命令返回结果类似下图，其中的 ’File’ 和 ’Position’ 信息我们稍后需要用到。  
![show master statu][1]
 

7.去到从服务器，启动 mysql 服务  

	$sudo service mysqld start

8.注意,下面部分请根据实际情况填写。 ’mysql master ip’ 是主服务器的IP地址，‘repluser’ 和 ’password’ 是在主服务器创建的用于复制的用户名和密码，’master log bin’ 和 ’pos’ 是在主服务器上 ’show master status’ 得到的 ’File’ 和 ’Position’.  

	$ mysql -uroot -p<password>  
	change master to master_host=’mysql master ip’, master_user=’repluser’, master_password=’password’, master_log_file=’master log bin’, master_log_pos=pos;
	start slave;
	show slave status\G  

show slave status\G 用来查看主从复制状态，得到结果如下图  
![主从复制状态][2]

查看 “Slave\_IO\_Running” 和 “Slave\_SQL\_Running”, 如果都是 “Yes”, 一般来讲意味着主从复制成功运行。

9.我们可以在主服务器上做一些更新操作，然后去到从服务器查看数据是否同步了。去到主服务器执行如下命令  

	$ mysql -uroot -p<password>
	create database shop;
	use shop;
	create table people (id int(10), name varchar(20));
	insert into people values (1, ‘alex’);  

去到从服务器查看是否有数据库名为 shop，表名为 people.  

	$ mysql -uroot -p<password>
	use shop;
	select * from people;  

如果结果类似下图，表明主从复制正常。  

![主从复制正常][3]
 

**Ubuntu Linux**: ( 以 Ubuntu 14.04, 64-bit system, MySQL Server 5.5(apt-get install) 为例)

1.在主从机器上都打开3306端口。请参考链接 [创建终结点](/documentation/articles/virtual-machines-set-up-endpoints/)   

然后在主从服务器上编辑/etc/mysql/my.cnf  

	$sudo sed -i 's/^bind-address/#bind-address/' /etc/mysql/my.cnf
	$sudo sed -i 's/^#server-id/server-id/' /etc/mysql/my.cnf
	$sudo sed -i 's/#log_bin/log_bin/' /etc/mysql/my.cnf  

2.在从服务器上编辑 /etc/mysql/my.cnf  

	server-id	= 2  

3.去到主服务器，启动 mysql 服务  

	$sudo service mysql start  

4.创建用于复制的用户。请根据实际情况，使用真实的从服务器 IP 地址，用户名，密码代替下面相应部分  

	$mysql -uroot -p<password>
	grant replication slave on *.* to 'repluser'@'slave ip' identified by 'password';  
	flush privileges;
	exit  

5.重启 mysql 服务并查看主服务器状态  

	$sudo service mysql restart
	$mysql -uroot -p<password>
	show master status;  

‘show master status’ 命令返回结果类似下图，其中的 ’File’ 和 ’Position’ 信息我们稍后会用到  
![show master status][4]
 
6.去到从服务器，启动 mysql 服务  

	$sudo service mysql start  

7.注意下面部分请根据实际情况填写。’mysql master ip’ 是主服务器的IP地址，‘repluser’ 和 ’password’ 是在主服务器创建的用于复制的用户名和密码，’master log bin’ 和 ’pos’ 是在主服务器上 ’show master status’ 得到的 ’File’ 和 ’Position’  

	$ mysql -uroot -p<password>
	change master to master_host=’mysql master ip’, master_user=’repluser’, master_password=’password’, master_log_file=’master log bin’, master_log_pos=pos;
	start slave;
	show slave status\G  

show slave status\G 用来查看主从复制状态，得到结果如下图  
![主从复制状态][5]

 
查看 “Slave\_IO\_Running” 和 “Slave\_SQL\_Running”, 如果都是 “Yes”, 一般来讲意味着主从复制成功运行.  

8.我们可以在主服务器上做一些更新操作，然后去到从服务器查看数据是否同步了。去到主服务器执行如下命令  

	$ mysql -uroot -p<password>
	create database shop;
	use shop;
	create table people (id int(10), name varchar(20));
	insert into people values (1, ‘alex’);  

9.去到从服务器查看是否有数据库名为 shop，表名为 people  

	$ mysql -uroot -p<password>
	use shop;
	select * from people;  

如果结果类似下图，表明主从复制正常. 
 
![主从复制正常][6]
 

**SUSE Linux**: ( 以 SLES 12, 64-bit system, MySQL Server 5.6(rpm install) 为例)

1.在主从机器上都打开3306端口。请参考链接 [创建终结点](/documentation/articles/virtual-machines-set-up-endpoints/) 

然后连接到主服务器。编辑 /etc/my.cnf, 在 [mysqld]下添加如下内容  

	server-id	= 1
	log_bin	= /var/lib/mysql/mysql-bin.log  

2.连接到从服务器. 编辑 /etc/my.cnf, 在 [mysqld]下添加如下内容  

	server-id	= 2
	log_bin	= /var/lib/mysql/mysql-bin.log  

3.去到主服务器，启动 mysql 服务  

	$sudo service mysql start  

4.创建用于复制的用户。请根据实际情况，使用真实的从服务器 IP 地址，用户名，密码代替下面相应部分.  
 
	$mysql -uroot -p<password>
	grant replication slave on *.* to 'repluser'@'slave ip' identified by 'password';  
	flush privileges;
	exit

5.重启 mysql 服务并查看主服务器状态  

	$sudo service mysql restart
	$mysql -uroot -p<password>
	show master status;

‘show master status’ 命令返回结果类似下图，其中的 ’File’ 和 ’Position’ 信息我们稍后需要用到  
![show master status][7]
 
6.去到从服务器，启动 mysql 服务  

	$sudo service mysql start  

7.注意下面部分请根据实际情况填写。’mysql master ip’ 是主服务器的IP地址，‘repluser’ 和 ’password’ 是在主服务器创建的用于复制的用户名和密码，’master log bin’ 和 ’pos’ 是在主服务器上 ’show master status’ 得到的 ’File’ 和 ’Position’  

	$ mysql -uroot -p<password>
	change master to master_host=’mysql master ip’, master_user=’repluser’, master_password=’password’, master_log_file=’master log bin’, master_log_pos=pos;
	start slave;
	show slave status\G

show slave status\G 用来查看主从复制状态，得到结果如下图  
![主从复制状态][8]
 
查看 “Slave_IO_Running” 和 “Slave_SQL_Running”, 如果都是 “Yes”, 一般来讲意味着主从复制成功运行.  

8.我们可以在主服务器上做一些更新操作，然后去到从服务器查看数据是否同步了。去到主服务器执行如下命令  

	$ mysql -uroot -p<password>
	create database shop;
	use shop;
	create table people (id int(10), name varchar(20));
	insert into people values (1, ‘alex’);  

9.去到从服务器查看是否有数据库名为 shop，表名为 people  

	$ mysql -uroot -p<password>
	use shop;
	select * from people;  

如果结果类似下图，表明主从复制正常.  

![主从复制正常][9]
 

##<a id="xtrabackup"></a>Xtrabackup

Percona XtraBackup 是开源的热备份工具—在备份的时候，不会锁住 MySQL 数据库的表  

它能备份 InnoDB, XtraDB, and MyISAM 表，支持 MySQL 5.1 [1], 5.5, 5.6 and 5.7    

不管您的 MySQL 服务器是 24x7 高负载，还是只有少量交易的环境，Percona XtraBackup 设计是用来“绝不阻挠生产环境下的数据库服务器的表现”。

请选择对应的 xtrabackup 版本和 MySQL 服务器版本.  

**Redhat base Linux**: ( 以 CentOS 7.0, 64-bit system, xtrabackup 2.3.2, MySQL Server 5.6 为例)

1.安装 xtrabackup, 授予权限

	$sudo wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.3.2/binary/redhat/7/x86_64/Percona-XtraBackup-2.3.2-r306a2e0-el7-x86_64-bundle.tar
	$sudo tar -xf Percona-XtraBackup-2.3.2-r306a2e0-el7-x86_64-bundle.tar
	$sudo yum install perl-DBD-MySQL.x86_64
	$sudo yum install rsync
	$sudo rpm -ivh percona-xtrabackup-2.3.2-1.el7.x86_64.rpm
	$sudo mysql -uroot -ppassword
	mysql> CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 's3cret';
	mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
	mysql> FLUSH PRIVILEGES;  

2.全量备份和恢复示例

2.1  全量备份  

	$sudo innobackupex --defaults-file=/etc/my.cnf --user='bkpuser' --password='s3cret' /tmp/backup --no-timestamp
	$sudo innobackupex --apply-log /tmp/backup  

2.2  全量恢复  

	$sudo service mysqld stop        
	$sudo cp -r /var/lib/mysql /backup/
	$sudo rm -rf /var/lib/mysql/*
	$sudo innobackupex --copy-back /tmp/backup
	$sudo chown -R mysql:mysql /var/lib/mysql
	$sudo service mysqld restart  

如果 /var/lib/mysql 非空的话，innobackupex --copy-back 命令会失败。因此我们需要先把这些文件移走。

3.增量备份和恢复示例  

3.1 增量备份  

	$innobackupex --defaults-file=/etc/my.cnf --user='bkpuser' --password='s3cret' /tmp/backup --no-timestamp
	$sudo innobackupex --defaults-file=/etc/my.cnf --user='bkpuser' --password='s3cret' --incremental /tmp/inc1 --incremental-basedir=/tmp/backup --no-timestamp
	$sudo innobackupex --defaults-file=/etc/my.cnf --user='bkpuser' --password='s3cret' --incremental /tmp/inc2 --incremental-basedir=/tmp/inc1 --no-timestamp  

您可以继续做第三次增量备份，只是需要注意 --incremental-basedir 应该指向第二次备份的目录 /tmp/inc2  

	$sudo innobackupex --defaults-file=/etc/my.cnf --user='bkpuser' --password='s3cret' --incremental /tmp/inc3 --incremental-basedir=/tmp/inc2 --no-timestamp

3.2  增量备份恢复  

	$ sudo innobackupex --apply-log --redo-only /tmp/backup
	$sudo innobackupex --apply-log --redo-only /tmp/backup --incremental-dir=/tmp/inc1
	$sudo innobackupex --apply-log --redo-only /tmp/backup --incremental-dir=/tmp/inc2
	$sudo innobackupex --apply-log  /tmp/backup --incremental-dir=/tmp/inc3
	$sudo innobackupex --apply-log  /tmp/backup
	$sudo service mysqld stop        
	$sudo cp -r /var/lib/mysql /backup/
	$sudo rm -rf /var/lib/mysql/*
	$sudo innobackupex --copy-back /tmp/backup
	$sudo chown -R mysql:mysql /var/lib/mysql
	$sudo service mysqld restart




**Ubuntu Linux**: ( 以Ubuntu 14.04, 64-bit system, Xtrabackup 2.2.13, MySQL Server 5.6 为例)

备份和恢复过程和上面的 Redhat base Linux 备份恢复过程非常相似，只是在安装时有少许不同，后面的授权，备份和恢复都是一样的。  

1.安装  

	$sudo wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.2.13/binary/debian/trusty/x86_64/percona-xtrabackup-22_2.2.13-1.trusty_amd64.deb
	$sudo dpkg -i percona-xtrabackup-22_2.2.13-1.trusty_amd64.deb  

2.请参考上面的备份恢复过程。注意 <font color='red'>--defaults-file=/etc/mysql/my.cnf , 不是 /etc/my.cnf</font>


##<a id="backup-redis"></a>备份 Redis

请参考文档“[在Azure Linux虚拟机上配置Redis集群.docx](/documentation/articles/open-source-azure-virtual-machines-linux-configure-redis-cluster/)” 中关于复制和集群的部分。


##<a id="backup-mongodb"></a>备份 MongoDB
请参考文档 “[在Azure Linux虚拟机上管理配置MongoDB集群.docx](/documentation/articles/open-source-azure-virtual-machines-manage-mongodb-cluster/)” 中复制和分片的部分。  

亦可使用mongodb tools.具体请参考[官网](#https://docs.mongodb.com/manual/administration/backup/)


<!--image reference -->
[1]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-1.png  
[2]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-2.png  
[3]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-3.png
[4]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-4.png  
[5]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-5.png  
[6]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-6.png  
[7]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-7.png  
[8]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-8.png  
[9]: ./media/open-source-azure-virtual-machine-linux-backup-databases/open-source-virtual-machine-linux-backup-databases-9.png