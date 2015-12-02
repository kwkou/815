<properties linkid="" urlDisplayName="" pageTitle="Using PowerShell to Manage MySQL Database on Azure – Microsoft Azure Cloud" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="This article explains how to use PowerShell to do more with MySQL Database on Azure, including create, view, delete and modify operations." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="10/14/2015"/>

#Using PowerShell to Manage MySQL Database on Azure
This white paper explains how to use PowerShell to do more with MySQL Database on Azure, including create, view, delete and modify operations. We recommend that you first read [Using Azure Resource Manager and PowerShell to Deploy and Use MySQL Database on Azure](/documentation/articles/mysql-database-etoe-powershell). This article explains how to download and use Azure PowerShell, and how to use PowerShell to quickly create a MySQL Database on Azure data services.

Before beginning, please ensure that you have properly prepared Azure PowerShell.

###Contents
- [Understanding Azure Resource Templates and Resource Groups](#gettoknow)
- [Operations: Create](#creat)
- [Operations: View](#view)
- [Operations: Modify](#set)
- [Operations: Delete](#delete)

## <a id="gettoknow"></a>1. Understanding Azure Resource Templates and Resource Groups

Most apps that are deployed and run on Microsoft Azure are built from combinations of different types of cloud resources. Azure Resource Manager templates allow you to centralize the deployment and management of these different resources, so that all you need to do is complete JSON description of the resources and the associated configuration and deployment parameters.
###1.1 Understanding resource type parameter information in MySQL Database on Azure
Six resource types are currently defined in MySQL Database on Azure JSON files: Servers, Databases, Users, Privileges, Firewall Rules, and Backups. Users can perform view, create, modify, and delete operations on these six resource types using the "Get," "New," "Set," and "Remove" commands. You can download the [JSON template file](http://wacnppe.blob.core.chinacloudapi.cn/marketing-resource/2015-09-01.json) to find out more about API parameter definitions for MySQL Database on Azure services.

## <a id="creat"></a>2. Operations: Create
The New command allows you to create MySQL servers, databases, users, user permissions, backups, and firewall rules.
###2.1 Creating servers
Edit and run the following commands to set information including your server name, location, version and other details, in order to complete creating the server.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 		
-ResourceGroupName resourcegroupChinaEast -Location chinaeast -PropertyObject @{version = '5.5'} 
```

###2.2 Creating server firewall rules
Edit and run the following commands to set information including your firewall rule names and IP whitelist range (start IP address, end IP address), in order to finish creating the firewall rules.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```

###2.3 Creating databases
Edit and run the following commands to set information including your database name and character sets, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```

###2.4 Creating users
Edit and run the following commands to set information including your username and password, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

###2.5 Adding user permissions
Edit and run the following commands to assign database read-write permissions to users. Permissions can be either “Read” or “ReadWrite”.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 	
-ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```
###2.6 Creating on-demand backup files
Edit and run the following commands to set backup file names and create on-demand backup files.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{}
```

## <a id="view"></a>3. Operations: View
Use the Get command to view lists such as current MySQL servers, databases, users, user permission, backups and firewall rules. You can also view detailed parameters and configurations.
###3.1 Viewing server lists
Edit and run the following commands to view all the current server lists.

```
Get-AzureResource -ResourceType "Microsoft.MySql/servers"  -ApiVersion 2015-01-01 
```
>[AZURE.NOTE] **Note: Unlike other commands, this checks that the “-ApiVersion 2015-01-01” in the server is directed at the ARM API, whereas this is “-ApiVersion 2015-09-01” in all other commands and is directed at the MySQL API.**

###3.2 Viewing database lists and parameters
Edit and run the following commands to view all database lists for a specific server in the current resource group:

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.3 Viewing user lists and parameters
Edit and run the following commands to view all user lists for a specific server in the current resource group:

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/users" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.4 Viewing user permission lists and parameters
Edit and run the following commands to view all user permissions for a specific database in the current resource group:

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -Name testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.5 View backup lists and parameters
Edit and run the following commands to view all backup files for a specific server in the current resource group:

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.6 Viewing firewall rule lists and parameters
Edit and run the following commands to view all firewall rules for a specific server in the current resource group:

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

## <a id="set"></a>4. Operations: Modify
You can use the Set command to carry out configuration tasks, such as changing account passwords, modifying permissions, and altering certain server parameters.
###4.1 Modifying account passwords
Edit and run the following commands to change the password for a specific account.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc1234'} -UsePatchSemantics
```

###4.2 Modifying access permission for a particular user
Edit and run the following commands to modify the access permissions for a specific user.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" 		
-ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='Read'} -UsePatchSemantics
```

###4.3 Allowing access for all Azure services
Edit and run the following commands to allow all Azure services to access specific servers.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{allowAzureServices='true'} -UsePatchSemantics
```

###4.4 Modifying the default time for daily backups
Edit and run the following commands to change the default daily backup start time for a specific server. The time can be set from 0-24, and means Beijing time UTC+8 in this location.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{dailyBackupTimeInHour='5'} -UsePatchSemantics
```

###4.5 Turning on slow query logs
Edit and run the following commands to turn on slow query logs.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{slow_query_log='ON'}} -UsePatchSemantics
```

###4.6 Modifying firewall rules
Edit and run the following commands to alter the existing firewall rules.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="1.1.1.1"} -ResourceGroupName resourcegroupChinaEast -UsePatchSemantics
```

###4.7 Modifying parts of the MySQL server settings
Taking the wait_timeout parameter as an example, edit and run the following commands to change the default value for wait_timeout; see the JSON files for details of other parameters.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{wait_timeout=70}} -UsePatchSemantics
```
You can refer to the JSON file definitions below for details of how to modify other parameters. See [Setting MySQL Database on Azure Server Parameters](http://www.windowsazure.cn/documentation/articles/mysql-database-advanced-settings) for details of valid parameter ranges:

```
	"options": {
          "type": "object",
          "properties": {
            "div_precision_increment": {
              "type": "integer"
            },
            "event_scheduler": {
              "type": "string"
            },
            "group_concat_max_len": {
              "type": "integer"
            },
            "innodb_adaptive_hash_index": {
              "type": "string"
            },
            "innodb_lock_wait_timeout": {
              "type": "integer"
            },
            "interactive_timeout": {
              "type": "integer"
            },
            "log_bin_trust_function_creators": {
              "type": "string"
            },
            "log_queries_not_using_indexes": {
              "type": "string"
            },
            "long_query_time": {
              "type": "number"
            },
            "max_allowed_packet": {
              "type": "integer"
            },
            "server-id": {
              "type": "integer"
            },
            "slow_query_log": {
              "type": "string"
            },
            "sql_mode": {
              "type": "string"
            },
            "wait_timeout": {
              "type": "integer"
            }
          }
        }
```
	
## <a id="delete"></a>5. Operations: Delete
You can use the Remove command to delete MySQL servers, databases, users, backups and firewall rules.
###5.1 Deleting servers
Edit and run the following commands to delete a specific server.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5.2 Deleting server firewall rules
Edit and run the following commands to delete a specific firewall.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5.3 Deleting databases
Edit and run the following commands to delete a specific database.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5.4 Deleting users
Edit and run the following commands to delete a specific user.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5.5 Deleting backup files
Edit and run the following commands to delete a specific backup file.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

We will continue to update the corresponding PowerShell guides as the range of features and functions in MySQL Database on Azure grows.