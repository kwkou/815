<properties linkid="" urlDisplayName="" pageTitle="Use Windows PowerShell to manage MySQL Database on Azure – Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, beginner’s guide, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="This article explains how to use Windows PowerShell to do more with MySQL Database on Azure, including create, view, delete, and modify operations." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="10/14/2015"/>

#Use Windows PowerShell to manage MySQL Database on Azure
> [AZURE.SELECTOR]
- [Chinese Version](/documentation/articles/mysql-database-commandlines)
- [English Version](/documentation/articles/mysql-database-enus-commandlines)

This white paper explains how to use PowerShell to do more with MySQL Database on Azure, including create, view, delete, and modify operations. We recommend that you first read [Using Azure Resource Manager and PowerShell to deploy and use MySQL Database on Azure](/documentation/articles/mysql-database-etoe-powershell). This article explains how to download and use PowerShell, and how to use it to quickly create a MySQL Database on Azure data services.

Before you begin, ensure that you have properly prepared PowerShell.

###Contents
- [Understanding Azure Resource Manager templates and resource groups](#gettoknow)
- [Operations: Create](#creat)
- [Operations: View](#view)
- [Operations: Modify](#set)
- [Operations: Delete](#delete)

## <a id="gettoknow"></a>1. Understand Azure Resource Manager templates and resource groups

Most apps that are deployed and run on Azure are built from combinations of different types of cloud resources. Azure Resource Manager templates allow you to centralize the deployment and management of these different resources, so that all you need to do is complete a JavaScript Object Notation (JSON) description of the resources and the associated configuration and deployment parameters.
###1\.1 Understand resource type parameter information in MySQL Database on Azure
Six resource types are currently defined in MySQL Database on Azure JSON files: servers, databases, users, privileges, firewall rules, and backups. Users can perform view, create, modify, and delete operations on these six resource types by using the Get, New, Set, and Remove commands. Download the [JSON template file](http://wacnppe.blob.core.chinacloudapi.cn/marketing-resource/2015-09-01.json) to find out more about API parameter definitions for MySQL Database on Azure services.

## <a id="creat"></a>2. Operations: Create
The New command allows you to create MySQL servers, databases, users, user permissions, backups, and firewall rules.
###2.1 Create servers
Edit and run the following commands to set information, including your server name, location, version, and other details, in order to complete creating the server.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 		
-ResourceGroupName resourcegroupChinaEast -Location chinaeast -PropertyObject @{version = '5.5'} 
```

###2\.2 Create server firewall rules
Edit and run the following commands to set information, including your firewall rule names and IP whitelist range (start IP address and end IP address), in order to finish creating the firewall rules.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```

###2\.3 Create databases
Edit and run the following commands to set information, including your database name and character sets, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```

###2\.4 Create users
Edit and run the following commands to set information, including your username and password, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

###2\.5 Add user permissions
Edit and run the following commands to assign database read/write permissions to users. Permissions can be either “Read” or “ReadWrite.”

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 	
-ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```
###2\.6 Create on-demand backup files
Edit and run the following commands to set backup file names and create on-demand backup files.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{}
```

## <a id="view"></a>3. Operations: View
Use the Get command to view lists such as current MySQL servers, databases, users, user permission, backups, and firewall rules. You can also view detailed parameters and configurations.
###3\.1 View server lists
Edit and run the following commands to view all the current server lists.

```
Get-AzureResource -ResourceType "Microsoft.MySql/servers"  -ApiVersion 2015-01-01 
```
>[AZURE.NOTE]**Unlike other commands, this checks that the “-ApiVersion 2015-01-01” in the server is directed at the Azure Resource Manager API, whereas this is “-ApiVersion 2015-09-01” in all other commands and is directed at the MySQL API. **

###3\.2 View database lists and parameters
Edit and run the following commands to view all database lists for a specific server in the current resource group.

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3\.3 View user lists and parameters
Edit and run the following commands to view all user lists for a specific server in the current resource group.

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/users" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3\.4 View user permission lists and parameters
Edit and run the following commands to view all user permissions for a specific database in the current resource group.

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -Name testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3\.5 View backup lists and parameters
Edit and run the following commands to view all backup files for a specific server in the current resource group.

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3\.6 View firewall rule lists and parameters
Edit and run the following commands to view all firewall rules for a specific server in the current resource group.

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

## <a id="set"></a>4. Operations: Modify
Use the Set command to carry out configuration tasks, such as changing account passwords, modifying permissions, and altering certain server parameters.
###4\.1 Modify account passwords
Edit and run the following commands to change the password for a specific account.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc1234'} -UsePatchSemantics
```

###4\.2 Modify access permission for a particular user
Edit and run the following commands to modify the access permissions for a specific user.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" 		
-ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='Read'} -UsePatchSemantics
```

###4\.3 Allow access for all Azure services
Edit and run the following commands to allow all Azure services to access specific servers.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{allowAzureServices='true'} -UsePatchSemantics
```

###4\.4 Modify the default time for daily backups
Edit and run the following commands to change the default daily backup start time for a specific server. The time can be set from 0 to 24, and means Beijing time UTC+8 in this location.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{dailyBackupTimeInHour='5'} -UsePatchSemantics
```

###4\.5 Turn on slow query logs
Edit and run the following commands to turn on slow query logs.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{slow_query_log='ON'}} -UsePatchSemantics
```

###4\.6 Modify firewall rules
Edit and run the following commands to alter the existing firewall rules.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="1.1.1.1"} -ResourceGroupName resourcegroupChinaEast -UsePatchSemantics
```

###4\.7 Modify parts of the MySQL server settings
Using the wait\_timeout parameter as an example, edit and run the following commands to change the default value for wait\_timeout. See the JSON files for details of other parameters.

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{wait_timeout=70}} -UsePatchSemantics
```
Refer to the JSON file definitions below for details of how to modify other parameters. See [Setting MySQL Database on Azure server parameters](/documentation/articles/mysql-database-advanced-settings) for details of valid parameter ranges:

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
Use the Remove command to delete MySQL servers, databases, users, backups, and firewall rules.
###5\.1 Delete servers
Edit and run the following commands to delete a specific server.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5\.2 Delete server firewall rules
Edit and run the following commands to delete a specific firewall.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5\.3 Delete databases
Edit and run the following commands to delete a specific database.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5\.4 Delete users
Edit and run the following commands to delete a specific user.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5\.5 Delete backup files
Edit and run the following commands to delete a specific backup file.

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

We will continue to update the corresponding PowerShell guides as the range of features and functions in MySQL Database on Azure grows.
<!--HONumber=81-->