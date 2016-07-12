<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-read-replica/)
- [English](/documentation/articles/mysql-database-enus-read-replica/)

# MySQL master-subordinate replication and read-only instances

The **MySQL Database on Azure** option on the Azure portal supports the use of the replicate function to create subordinate instances, also known as slave instances, for a MySQL instance. All changes to the master instance will be replicated to the subordinate instance. You can use this function as a simple way to implement flexible expansion and overcome the access restrictions on an individual database instance, which reduces the running load and increases availability.

  For business intelligence (BI) reports or database warehouse solutions, users generally want to run service report queries on independent read-only instances. A subordinate instance serves this role. The replicate function can also be used to migrate databases between the generation environment and the development environment. You can also use the replicate function to improve the generation environment’s availability to provide disaster tolerance. If an error occurs, you can upgrade read-only instances to replace the faulty master instance and switch the workload to provide availability and continuity of services.

When you create a subordinate instance, Azure first performs a backup of the master instance and then creates a new read-only instance that's based on the backup to serve as the subordinate instance. After this, Azure will continually replicate all changes to the master instance on the subordinate instance.


Notes:

1. **MySQL Database on Azure** supports the creation of subordinate instances only for MySQL 5.6. MySQL 5.5 is not supported.
2. The subordinate instance must use the same version of MySQL as the master instance. **MySQL Database on Azure** does not support replication between different versions of MySQL.
3. **MySQL Database on Azure** currently only supports subordinate instances in the same location (data center) as the master instance.
4. **MySQL Database on Azure** currently supports a maximum of five subordinate instances of the same master instance.
5. In order to ensure that data remains consistent between the master and subordinate instances, subordinate instances are read-only instances. All MySQL links on subordinate instances are read-only links. You cannot create, modify, or delete databases or database accounts on the subordinate instance. You can perform these operations on the master instance, and the system will automatically synchronize them to the subordinate instances.


## Create read-only (subordinate) instances
1.	Select an existing instance, and then click **Replicate**.
2.	Click **Add slave instance** at the bottom, and enter the name of the subordinate server. Confirm that you understand the effect on the charges, and then click **Submit**.
![Add slave instances][1]

Note: **MySQL Database on Azure** performs a backup of the master instance during the process of creating subordinate instances. To prevent a failure during backup, make sure that no queries or changes that take a long time are running on the master instance.

## Monitor the replication status of subordinate instances
After you create the subordinate instance, you can monitor the status of replication between the master instance and subordinate instances in a variety of ways.

Select the master instance, and click **Replicate**. This page shows all of the master instance’s subordinate instances and their statuses.

![Monitor replication][2]

Select a subordinate instance and click **Replicate**. This page shows the replication status of the subordinate instance and the name of the corresponding master instance.
![Replicate][3]

## Implement read/write separation in applications
A Java sample program for read/write separation at the application end is shown below:


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
	
	        // Set up connection to subordinate;
	        conn.setReadOnly(true);
	        
	        // Now, do a query from a subordinate
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

A PHP sample program for read/write separation at the application end is shown below:

* Read-write splitting is achievable with PHP MySQL native driver with master slave plugin(PECL/mysqlnd_ms). The plugin executes read-only statements on the configured MySQL slaves, and all other queries on the MySQL master. Details can be found <a href="http://php.net/manual/en/mysqlnd-ms.rwsplit.php" target="_blank">here</a>.
*  A user-defined read-write splitter can request the built-in logic to send a statement to a specific location, by invoking mysqlnd_ms_is_select().
*  Install PECL/mysqlnd_ms, Details can be found <a href="http://php.net/manual/en/mysqlnd-ms.quickstart.configuration.php" target="_blank">here</a>.
*  Create PECL/mysqlnd_ms plugin config file as follows:

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

* PHP sample program:

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

## Upgrade subordinate instances
You can upgrade a subordinate instance to an online read/write instance. After you upgrade the instance, changes on the master instance will no longer be replicated to this instance. You can perform read/write operations on this instance.
1.	Select the subordinate instance that you need to upgrade, and click **Replicate**.
2.	Click **Upgrade** at the bottom of the page, and then click **Confirm**.
![Upgrade subordinate instances][4]

## Master-subordinate replication FAQs
Q: I want to change the performance version of the master instance. For example, I might want to go up from MS4 to MS5 or go down from MS6 to MS5. Will the version for subordinate instances be updated as well?

A: The performance version for subordinate instances will not change with that of the master instance. However, you can change the performance version of subordinate instances separately.

Q: I would like to use the master-subordinate replication function, but my database instance is on version 5.5. What should I do?

A: Master-subordinate replication only supports version 5.6. You can manually upgrade the database implementation to version 5.6 first, and then use the replicate function.

Q: How do I upgrade the database from version 5.5 to version 5.6?

A: We do not currently support one-key upgrades. The way around this is to export the data from the original version 5.5 instance by using mysqldump, create a new version 5.6 database server instance, and then import the data. After it passes compatibility testing, you can migrate the application to the version 5.6 instance. If your original database is the generation environment or you cannot accept any downtime, you can manually create a snapshot of the original database, restore it to a completely new instance, and then perform the migration and upgrade on the restored instance. This will reduce the impact on the original database.



<!--Image references-->

[1]: ./media/mysql-database-read-replica/replica1-en.jpg
[2]: ./media/mysql-database-read-replica/replica2-en.jpg
[3]: ./media/mysql-database-read-replica/replica3-en.jpg
[4]: ./media/mysql-database-read-replica/replica4-en.jpg

<!---HONumber=Acom_0418_2016_MySql-->
