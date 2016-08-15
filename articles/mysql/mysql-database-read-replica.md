<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,主从复制,只读实例,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-read-replica/)
- [English](/documentation/articles/mysql-database-enus-read-replica/)

# MySQL 主从复制和只读实例

MySQL Database on Azure支持用户用复制功能为MySQL实例创建从属实例。 所有在主实例上的改动都会复制到从属实例。 通过该功能，用户可以轻松实现弹性扩展，突破单个数据库实例的访问限制，从而降低运行负荷及增加高可用性。

  对于商业智能报告(BI)或数据仓库方案，通常用户希望对独立的只读实例（而非生产数据库的只读实例）运行业务报告查询。复制功能还可以用于在生产环境和开发环境之间迁移数据库。此外，用户还可以利用复制功能提高生产环境的可用性-容灾。在故障发生时，您可以提升只读实例取代失效的主实例，切换工作负载，从而保障业务的可用性和连续性。

当用户创建一个从属实例时，Azure首先会对主实例做一个备份，接着通过基于该备份创建一个新的只读实例作为从属实例。然后，Azure会不断地将主实例上的所有改动复制到从属实例上。


注意事项：

1. MySQL Database on Azure只支持为MySQL5.6的主实例创建从属实例， 不支持MySQL5.5。
2. 从属实例和主实例的MySQL版本必须一致。MySQL Database on Azure不支持不同MySQL版本间的复制。
3. 目前，MySQL Database on Azure只支持主实例和从属实例位于同一位置（数据中心）。
4. 目前，对于同一个主实例，MySQL Database on Azure最多只支持5个从属实例。
5. 为了保证主实例和从属实例间的数据一致，从属实例是只读实例。它上面的所有MySQL连接都是只读连接。在从属实例上，用户不能创建，更改，删除数据库和数据库账户。用户可以在主实例进行操作，系统自动同步到只读实例。


## 创建只读（从属）实例
1.	选择一已有实例，点击“复制”页。
2.	点击下方的添加从属实例， 填写从属服务器的名称，确认了解对于计费的影响，然后点击提交。
![添加从属实例][1]

注意：因为在创建从属实例的过程中，MySQL Database on Azure 会对主实例做备份， 请确保当时的主实例上没有需要长时间运行的查询或改动，从而避免备份失败。

## 监控从属实例复制状态
从属实例创建成功后，用户可以通过多种方法监控主实例和从属实例间的复制。

选择主实例，点击“复制”页。 该页面显示了它的所有从属实例及它们状态。

![监控复制][2]

选择从属实例, 点击“复制”页。该页面显示了该从属实例的复制状态和它所对应的主实例名称。
![复制][3]

## 在应用程序中实现读写分离
下面是应用端读写分离的Java样例程序:


	package test1;
	
	import java.sql.Connection;
	import java.sql.ResultSet;
	import java.sql.Statement;
	import java.util.Properties;
	
	import com.mysql.jdbc.Driver;
	import com.mysql.jdbc.ReplicationDriver;;
	
	public class ConnectionDemo {
	
	  public static void main(String[] args) throws Exception {
		
	    ReplicationDriver driver = new ReplicationDriver();
	    String url = "jdbc:mysql:replication://address=(protocol=tcp)(type=master)(host=masterhost)(port=3306)(user=masteruser),address=(protocol=tcp)(type=slave)(host=slavehost)(port=3306)(user=slaveuser)/yourdb";
	    Properties props = new Properties();    
	    props.put("password", "yourpassword");
	    try (Connection conn = driver.connect(url, props))
	    {
	    	// Perform read/write work on the master
	        conn.setReadOnly(false);
	        conn.setAutoCommit(false);
	        conn.createStatement().executeUpdate("update t1 set id = id+1;");
	        conn.commit();    
	
	        // Set up connection to slave;
	        conn.setReadOnly(true);
	        
	        // Now, do a query from a slave
	        try (Statement statement = conn.createStatement())
	    	{
	    		ResultSet res = statement.executeQuery("show tables");
	    		System.out.println("There are below tables:");
	    		while (res.next()) {
	    			String tblName = res.getString(1);
	    			System.out.println(tblName);
	    		}
	    	} 
	    }
	  }
	}

下面是应用端读写分离的PHP样例程序:

* 读写分离可以通过PHP MySQL原生驱动配合主从实例插件(PECL/mysqlnd_ms)来实现。这个插件在已配置好的从属实例上执行只读语句，其他所有的查询都执行在MySQL主实例上。具体内容可参考<a href="http://php.net/manual/zh/mysqlnd-ms.rwsplit.php" target="_blank">这里</a>。
* 用户定义的读写分离器可以请求内置的逻辑语句发送到特定位置，通过调用 mysqlnd\_ms\_is\_select()。
* 安装 PECL/mysqlnd_ms 请参考<a href="http://php.net/manual/zh/mysqlnd-ms.quickstart.configuration.php" target="_blank">这里</a>。
* 创建 PECL/mysqlnd_ms 插件配置文件如下所示:

		File mysqlnd_ms_plugin.ini:
		{
	    	"myapp": {
	       		"master": {
	            	"master_0": {
		                "host": "<your master host>",
		                "port": "<your master port>",
		                "user": "<your master username>",
		                "password": "<your master password>"
	            		}
	        	},
	        	"slave": {
	            	"slave_0": {
		                "host": "<your slave host>",
		                "port": "<your slave port>",
		                "user": "<your slave username>",
		                "password": "<your slave password>"
	            	}
	        	}
	    }

* PHP 样例程序:

		<?php
		function is_select($query)
		{
		  switch (mysqlnd_ms_query_is_select($query))
		  {
		    case MYSQLND_MS_QUERY_USE_MASTER:
		      printf("'%s' should be run on the master.<br>\n", $query);
		      break;
		    case MYSQLND_MS_QUERY_USE_SLAVE:
		      printf("'%s' should be run on a slave.<br>\n", $query);
		      break;
		    case MYSQLND_MS_QUERY_USE_LAST_USED:
		      printf("'%s' should be run on the server that has run the previous query.<br>\n", $query);
		      break;
		    default:
		      printf("No suggestion where to run the '%s', fallback to master recommended.<br>\n", $query);
		      break;
		  }
		}
		
		if (!($mysqli = new mysqli("myapp", "<your username>", "<your password>", "<your db>")) || mysqli_connect_errno())
		{
		  die(sprintf("Failed to connect: [%d] %s\n", mysqli_connect_errno(), mysqli_connect_error()));
		}
		$query = "INSERT INTO user(name, num) VALUES ('test', 1)";
		is_select($query);
		
		if (!($res = $mysqli->query($query)))
		{
		  printf("Failed to insert: [%d] %s<br>\n", $mysqli->errno, $mysqli->error);
		}
		
		$query = "SELECT * FROM user";
		is_select($query);
		if (!($res = $mysqli->query($query)))
		{
		  printf("Failed to query: [%d] %s<br>\n", $mysqli->errno, $mysqli->error);
		}
		else
		{
		  for ($i=0; $row = $res->fetch_assoc(); $i++)
		  {
		    $value[$i] = $row;
		  }
		  print_r($value);
		  printf("<br>\n");
		  $res->close();
		}
		
		$query = "/*" . MYSQLND_MS_LAST_USED_SWITCH . "*/SELECT * FROM user limit 1";
		is_select($query);
		if (!($res = $mysqli->query($query)))
		{
		  printf("Failed to query: [%d] %s<br>\n", $mysqli->errno, $mysqli->error);
		}
		else
		{
		  $value = $res->fetch_assoc();
		  print_r($value);
		  printf("<br>\n");
		  $res->close();
		}
		$mysqli->close();
		?>

## 提升从属实例
用户可以把一个从属实例提升为联机读写实例。提升后，主实例上的改动不再复制到该实例上。用户可以对该实例进行读写操作。
1.	选择需要被提升的从属实例，点击“复制”页。
2.	点击页面下方的“提升”， 然后点击确认。
![提升从属实例][4]

## 关于主从复制的常见问题
•	我想更改主实例的性能版本，比如从MS4升到MS5，或者MS6降到MS5。从属实例的版本会随着更新吗？
从属实例的性能版本不会随之改变，但用户可以根据需要另行改变从属实例的性能版本。

•	我想使用主从复制功能，但是我的数据库实例现在是5.5版本，我该怎么做？
主从复制只支持5.6版本。可以先将数据库实例手动升级到5.6版本，然后再使用复制功能。

•	如何将数据库从5.5升级到5.6？
目前我们暂不支持一键升级，替代办法是用户可以通过mysqldump将数据从源5.5实例中导出，创建新的5.6数据库服务器实例，再将数据进行导入，测试兼容性通过后，可将应用迁移至5.6实例。 如果用户的源数据库是生产环境或不能接受任何停机时间，可以在源数据库上手动创建快照，恢复到全新实例后，对恢复后的实例进行迁移升级，以减少对源数据库的影响。


<!--Image references-->

[1]: ./media/mysql-database-read-replica/replica1.jpg
[2]: ./media/mysql-database-read-replica/replica2.jpg
[3]: ./media/mysql-database-read-replica/replica3.jpg
[4]: ./media/mysql-database-read-replica/replica4.jpg
