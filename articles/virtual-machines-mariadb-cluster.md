<properties
	pageTitle="在 Azure 上运行 MariaDB (MySQL) 群集"
	description="在 Azure 虚拟机上创建 MariaDB + Galera MySQL 群集"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="sabbour"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/15/2015"
	wacn.date="06/08/2016"/>

# MariaDB (MySQL) 群集 - Azure 教程

我们正在创建的是 [MariaDB](https://mariadb.org/en/about/) 的多主机 [Galera](http://galeracluster.com/products/) 群集，这是 MySQL 的嵌入式替代版本，具有稳健性、可扩展性和可靠性，可在 Azure 虚拟机上的高度可用环境中使用。

## 体系结构概述

本主题中将执行下列步骤：

1. 创建包含 3 个节点的群集
2. 将数据磁盘与操作系统磁盘隔离开来
3. 在 RAID-0/条带化设置下创建数据磁盘，以提高 IOPS
4. 使用 Azure 负载平衡器，以使 3 个节点的负载保持平衡
5. 为了最大程度地减少重复工作，可创建一个包含 MariaDB+Galera 的 VM 映像，并将其用于创建其他群集 VM。

![体系结构](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Setup.png)

> [AZURE.NOTE]本主题将使用 [Azure CLI] 工具，所以请确保下载这些工具，并根据说明将其连接到你的 Azure 订阅。如果你需要对 Azure CLI 中可用命令的参考，可单击此链接以查看 [Azure CLI 命令参考]。你还将需要[创建用于身份验证的 SSH 密钥]，并记下 **.pem 文件的位置**。


## 创建模板

### 基础结构

1. 创建地缘组，以将资源保存在一起

		azure account affinity-group create mariadbcluster --location "China North" --label "MariaDB Cluster"
        
2. 创建虚拟网络

		azure network vnet create --address-space 10.0.0.0 --cidr 8 --subnet-name mariadb --subnet-start-ip 10.0.0.0 --subnet-cidr 24 --affinity-group mariadbcluster mariadbvnet

3. 创建存储帐户，以保存所有磁盘。请注意，不要将超过 40 个常用磁盘放置在同一存储帐户上，以免达到存储帐户的 20,000 IOPS 上限。在本例中，我们的磁盘数量远低于这个数字，所以为了简单起见，我们将所有磁盘都存储在同一帐户上。

		azure storage account create mariadbstorage --label mariadbstorage --affinity-group mariadbcluster

3. 查找 CentOS 7 虚拟机映像的名称

		azure vm image list | findstr CentOS        
该输出将如 `f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-70-20160329` 所示。在以下步骤中使用该名称。

4. 创建 VM 模板，将 **/path/to/key.pem** 替换为已生成的 .pem SSH 密钥的存储位置路径

		azure vm create --virtual-network-name mariadbvnet --subnet-names mariadb --blob-url "http://mariadbstorage.blob.core.chinacloudapi.cn/vhds/mariadbhatemplate-os.vhd"  --vm-size Medium --ssh 22 --ssh-cert "/path/to/key.pem" --no-ssh-password mariadbhatemplate f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-70-20160329 azureuser
        
5. 将 4 个 500GB 的数据磁盘附加到 VM，以便在 RAID 配置中使用

		FOR /L %d IN (1,1,4) DO azure vm disk attach-new mariadbhatemplate 512 http://mariadbstorage.blob.core.chinacloudapi.cn/vhds/mariadbhatemplate-data-%d.vhd        

6. 通过 SSH 登录到你在 **mariadbhatemplate.chinacloudapp.cn:22** 创建的模板 VM，并使用私钥进行连接。

### 软件

1. 获取根

        sudo su
        
2. 安装 RAID 支持：

     - 安装 mdadm
        
        		yum install mdadm
                
     - 创建具有 EXT4 文件系统的 RAID0/条带化配置
        
				mdadm --create --verbose /dev/md0 --level=stripe --raid-devices=4 /dev/sdc /dev/sdd /dev/sde /dev/sdf
				mdadm --detail --scan >> /etc/mdadm.conf
				mkfs -t ext4 /dev/md0
         
     - 创建装入点目录
         
				mkdir /mnt/data
                
     - 检索新创建的 RAID 设备的 UUID
        
				blkid | grep /dev/md0
    
     - 编辑 /etc/fstab
        
        		vi /etc/fstab
        
     - 将设备添加到此位置，以便在重新启动时自动安装，并将 UUID 替换为之前从 **blkid** 命令中获取的值
        
        		UUID=<UUID FROM PREVIOUS>   /mnt/data ext4   defaults,noatime   1 2
        	
     - 装载新分区
        
        		mount /mnt/data
            
3. 安装 MariaDB：
		
     - 创建 MariaDB.repo 文件： 
        
              	vi /etc/yum.repos.d/MariaDB.repo
        
     - 在其中填充以下内容
        
				[mariadb]
				name = MariaDB
				baseurl = http://yum.mariadb.org/10.0/centos7-amd64
				gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
				gpgcheck=1
    
     - 删除现有后缀和 mariadb-libs，以免产生冲突
    
    		yum remove postfix mariadb-libs-*
            
     - 安装包含 Galera 的 MariaDB
    
    		yum install MariaDB-Galera-server MariaDB-client galera
   
4. 将 MySQL 数据目录移动到 RAID 块设备

     - 将当前 MySQL 目录复制到其新位置，然后删除旧目录
    
    		cpccp -avr /var/lib/mysql /mnt/data  
    		rm -rf /var/lib/mysql
      
     - 相应地设置新目录上的权限
        
        	chochown -R mysql:mysql /mnt/data && chmod -R 755 /mnt/data/  
            
     - 创建一个符号链接，将旧目录指向 RAID 分区上的新位置
    
    		ln -s /mnt/data/mysql /var/lib/mysql   

5. [SELinux 会干扰群集操作](http://galeracluster.com/documentation-webpages/configuration.html#selinux)，所以在当前会话中有必要将其禁用（直到兼容版本出现）。编辑 `/etc/selinux/config` 以禁止其后续重新启动：
    	
	        setenforce 0
    
       然后编辑 `/etc/selinux/config` 以设置 `SELINUX=permissive`
       
6. 验证 MySQL 是否运行

    - 启动 MySQL
    
    		service mysql start
            
    - 保护 MySQL 安装、设置根密码、删除匿名用户，禁用远程根登录并删除测试数据库
    		
            mysql_smysql_secure_installation
            
    - 在数据库上创建一个用户，以便在群集操作中使用，也可以选择在应用程序中使用
   
			mysql -mysql -u root -p
			GRANT ALL PRIVILEGES ON *.* TO 'cluster'@'%' IDENTIFIED BY 'p@ssw0rd' WITH GRANT OPTION; FLUSH PRIVILEGES;
            exit
   
   - 停止 MySQL
   
			service mysql stop
            
7. 创建配置占位符

	- 编辑 MySQL 配置，以便为群集设置创建一个占位符。现在不要替换 **`<Vairables>`** 或取消注释。当我们通过此模板创建 VM 后，才需执行此操作。
	
			vi /etc/my.cnf.d/server.cnf
			
	- 编辑 **[galera]** 部分，并将其清空
	
	- 编辑 **[mariadb]** 部分
	
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so 
            binlog_format=ROW
            wsrep_sst_method=rsync
			bind-address=0.0.0.0 # When set to 0.0.0.0, the server listens to remote connections
			default_storage_engine=InnoDB
            innodb_autoinc_lock_mode=2
			
            wsrep_sst_auth=cluster:p@ssw0rd # CHANGE: Username and password you created for the SST cluster MySQL user
            #wsrep_cluster_name='mariadbcluster' # CHANGE: Uncomment and set your desired cluster name
            #wsrep_cluster_address="gcomm://mariadb1,mariadb2,mariadb3" # CHANGE: Uncomment and Add all your servers
            #wsrep_node_address='<ServerIP>' # CHANGE: Uncomment and set IP address of this server
			#wsrep_node_name='<NodeName>' # CHANGE: Uncomment and set the node name of this server

8. 打开防火墙上的所需端口（在 CentOS 7 上使用 FirewallD）
	
	- MySQL：`firewall-cmd --zone=public --add-port=3306/tcp --permanent`
    - GALERA：`firewall-cmd --zone=public --add-port=4567/tcp --permanent`
    - GALERA IST：`firewall-cmd --zone=public --add-port=4568/tcp --permanent`
    - RSYNC：`firewall-cmd --zone=public --add-port=4444/tcp --permanent`
    - 重新加载防火墙：`firewall-cmd --reload`
	
9.  优化系统性能。有关更多详细信息，请参阅本文中的[性能优化策略]。

	- 再次编辑 MySQL 配置文件
	
			vi /etc/my.cnf.d/server.cnf
		
	- 编辑 **[mariadb]** 部分，后接以下内容
	
	> [AZURE.NOTE]建议 **innodb\_buffer\_pool\_size** 占用 VM 的 70% 内存。对于 RAM 为 3.5GB 的 Medium Azure VM，这里已设置为 2.45GB。
	
	        innodb_buffer_pool_size = 2508M # The buffer pool contains buffered data and the index. This is usually set to 70% of physical memory.
            innodb_log_file_size = 512M #  Redo logs ensure that write operations are fast, reliable, and recoverable after a crash
            max_connections = 5000 # A larger value will give the server more time to recycle idled connections
            innodb_file_per_table = 1 # Speed up the table space transmission and optimize the debris management performance
            innodb_log_buffer_size = 128M # The log buffer allows transactions to run without having to flush the log to disk before the transactions commit
            innodb_flush_log_at_trx_commit = 2 # The setting of 2 enables the most data integrity and is suitable for Master in MySQL cluster
			query_cache_size = 0
			
10. 停止 MySQL 并禁用 MySQL 服务在启动时运行，以免在添加新节点时导致群集混乱，然后取消设置机器。

		service mysql stop
        chkconfig mysql off
		waagent -deprovision	
		
11. 通过经典管理门户捕获 VM。（目前，[Azure CLI 工具中的问题 #1268] 描述的事实是，Azure CLI 工具所捕获的映像并没有捕获所附加的数据磁盘。）

	- 通过经典管理门户关闭机器
    - 单击“捕获”并将映像名称指定为 **mariadb-galera-image**，然后提供描述并选中“我已运行 waagent”。![捕获虚拟机](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Capture.png)![捕获虚拟机](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Capture2.PNG)
	
## 创建群集
 
使用刚才创建的模板创建 3 个 VM，然后配置并启动群集。

1. 从之前创建的 **mariadb-galera-image** 映像创建第一个 CentOS 7 VM，设置虚拟网络名称为 **mariadbvnet**、子网为 **mariadb**、虚拟机大小为“中等”，传入云服务名称为 **mariadbha**（或者你希望通过 mariadbha.chinacloudapp.cn 访问的任何名称），将此虚拟机的名称设置为 **mariadb1**、用户名设置为 **azureuser**，启用 SSH 访问并传送 SSH 证书 .pem 文件，并将 **/path/to/key.pem** 替换为已生成的 .pem SSH 密钥的存储位置路径。

	> [WACOM.NOTE]为清楚起见，以下命令拆开显示在多行内，但每个都应作为一整行进行输入。

		azure vm create 
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium 
		--ssh-cert "/path/to/key.pem" 
		--no-ssh-password 
		--ssh 22 
		--vm-name mariadb1 
		mariadbha mariadb-galera-image azureuser 

2. 通过_连接_到当前创建的 **mariadbha** 云服务、更改 **VM 名称**，并将 **SSH 端口**更改为不会与同一云服务中的其他 VM 产生冲突的独特端口，创建另外 2 个虚拟机。

		azure vm create		
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium
		--ssh-cert "/path/to/key.pem"
		--no-ssh-password
		--ssh 23
		--vm-name mariadb2
        --connect mariadbha mariadb-galera-image azureuser
以及对于 MariaDB3

		azure vm create
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium
		--ssh-cert "/path/to/key.pem"
		--no-ssh-password
		--ssh 24
		--vm-name mariadb3
        --connect mariadbha mariadb-galera-image azureuser
		
3. 你需获取 3 个 VM 各自的内部 IP 地址，才能执行后续步骤：

	![获取 IP 地址](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/IP.png)

4. 通过 SSH 登录到 3 个 VM，并编辑每个虚拟机的配置文件

		sudo vi /etc/my.cnf.d/server.cnf
		
	删除开始处的 **#**，取消对 **`wsrep_cluster_name`** 和 **`wsrep_cluster_address`** 的注释，并验证其是否为所需内容。此外，将 **`wsrep_node_address`** 中的 **`<ServerIP>`** 和 **`wsrep_node_name`** 中的 **`<NodeName>`** 分别替换为 VM 的 IP 地址和名称，然后同样取消这些行的注释。
	
5. 启动 MariaDB1 上的群集，并允许其在启动时运行

		sudo service mysql bootstrap
        chkconfig mysql on
	
6. 在 MariaDB2 和 MariaDB3 上启动 MySQL，并允许其在启动时运行

		sudo service mysql start
        chkconfig mysql on
		
## 对群集进行负载平衡
在创建聚集 VM 时，你将其添加到了名为 **clusteravset** 的可用性集中，以确保其放置在不同的容错和更新域上，且 Azure 不会同时在所有虚拟机上执行维护。此配置符合该 Azure 服务级别协议 (SLA) 将支持的要求。

现在，使用 Azure 负载平衡器，以平衡 3 个节点之间的请求。

在使用 Azure CLI 的机器上运行以下命令。命令参数结构是：`azure vm endpoint create-multiple <MachineName> <PublicPort>:<VMPort>:<Protocol>:<EnableDirectServerReturn>:<Load Balanced Set Name>:<ProbeProtocol>:<ProbePort>`

>[AZURE.NOTE] 以下的命令只适用于 Azure CLI 0.8，更新的版本的 endpoints-config 已经变更为 `<public-port>:<local-port>:<protocol>:<idle-timeout>:<direct-server-return>:<probe-protocol>:<probe-port>:<probe-path>:<probe-interval>:<probe-timeout>:<load-balanced-set-name>:<internal-load-balancer-name>:<load-balancer-distribution>`

	azure vm endpoint create-multiple mariadb1 3306:3306:tcp:false:MySQL:tcp:3306
    azure vm endpoint create-multiple mariadb2 3306:3306:tcp:false:MySQL:tcp:3306
    azure vm endpoint create-multiple mariadb3 3306:3306:tcp:false:MySQL:tcp:3306
	
最后，由于 CLI 将负载平衡器的探测时间间隔设置为 15 秒（可能有点太长了），可以在经典管理门户中的“终结点”下更改任一 VM 的该时间间隔

![编辑终结点](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Endpoint.PNG)

然后单击“重新配置负载平衡集”，并转到下一步

![重新配置负载平衡集](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Endpoint2.PNG)

然后将“探测时间间隔”设置为 5 秒，并保存

![更改探测时间间隔](./media/virtual-machines-linux-classic-mariadb-mysql-cluster/Endpoint3.PNG)

## 验证群集

繁琐的工作已经完成。现在应该可以在 `mariadbha.chinacloudapp.cn:3306` 访问群集，这将触发负载平衡器并在 3 个 VM 之间顺利、高效地路由请求。

使用偏好的 MySQL 客户端进行连接，或直接从任一 VM 进行连接，以验证此群集是否正常运行。

	 mysql -u cluster -h mariadbha.chinacloudapp.cn -p
	 
然后，创建新的数据库并在其中填充一些数据

	CREATE DATABASE TestDB;
    USE TestDB;
    CREATE TABLE TestTable (id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, value VARCHAR(255));
	INSERT INTO TestTable (value)  VALUES ('Value1');
	INSERT INTO TestTable (value)  VALUES ('Value2');
    SELECT * FROM TestTable;
	
将导致以下表格

	+----+--------+
	| id | value  |
	+----+--------+
	|  1 | Value1 |
	|  4 | Value2 |
	+----+--------+
	2 rows in set (0.00 sec)	
	
<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

在本文中，你在运行 CentOS 7 的 Azure 虚拟机上创建了包含 3 个节点的 MariaDB + Galera 高度可用群集。VM 已通过 Azure 负载平衡器实现了负载平衡。

你可能希望找到[另一种方式在 Linux 上对 MySQL 进行集群]以及探究如何[优化和测试 Azure Linux VM 上的 MySQL 性能]。

<!--Anchors-->
[Architecture overview]: #architecture-overview
[Creating the template]: #creating-the-template
[Creating the cluster]: #creating-the-cluster
[Load balancing the cluster]: #load-balancing-the-cluster
[Validating the cluster]: #validating-the-cluster
[Next steps]: #next-steps

<!--Image references-->

<!--Link references-->
[Galera]: http://galeracluster.com/products/
[MariaDBs]: https://mariadb.org/en/about/
[Azure CLI]: /zh-cn/documentation/articles/xplat-cli-install/
[Azure CLI 命令参考]: /documentation/articles/virtual-machines-command-line-tools/
[创建用于身份验证的 SSH 密钥]: http://www.jeff.wilcox.name/2013/06/secure-linux-vms-with-ssh-certificates/
[性能优化策略]: /zh-cn/documentation/articles/virtual-machines-linux-classic-optimize-mysql/
[优化和测试 Azure Linux VM 上的 MySQL 性能]: /zh-cn/documentation/articles/virtual-machines-linux-classic-optimize-mysql/
[Azure CLI 工具中的问题 #1268]: https://github.com/Azure/azure-xplat-cli/issues/1268
[另一种方式在 Linux 上对 MySQL 进行集群]: /zh-cn/documentation/articles/virtual-machines-linux-classic-mysql-cluster/

<!---HONumber=67-->