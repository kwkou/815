<properties linkid="" urlDisplayName="" pageTitle="Use Azure Resource Manager and PowerShell to deploy and use MySQL Database on Azure – Microsoft Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, beginner’s guide, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="This article explains how to use Azure PowerShell scripts to quickly set up and use MySQL services." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="10/14/2015"/>

#Use Azure Resource Manager and Windows PowerShell to deploy and use MySQL Database on Azure
Azure Resource Manager is a completely new management framework for Azure services. Now you can use APIs and tools based on Azure Resource Manager to manage MySQL Database on Azure. If you would like to find out more about Resource Manager, read [Using Resource Manager to manage Azure resources](/documentation/articles/azure-preview-portal-using-resource-groups). MySQL Database on Azure supports the use of PowerShell scripts to query, create, manage, and delete MySQL servers, databases, firewall rules, and users. It also supports the use of scripts to change parameters and settings. This article explains how to use PowerShell scripts to quickly set up and use MySQL services. You can also read [Using PowerShell to manage MySQL Database on Azure](/documentation/articles/mysql-database-commandlines) for more information on the create, view, delete, and modify operations.

###Contents
- [Step 1: Install PowerShell](#step1)
- [Step 2: Configure account details](#step2)
- [Step 3: Subscribe to MySQL Database on Azure services](#step3)
- [Step 4: Create resource groups](#step4)
- [Step 5: Create servers](#step5)
- [Step 6: Create server firewall rules](#step6)
- [Step 7: Create databases](#step7)
- [Step 8: Create users](#step8)
- [Step 9: Add user permissions](#step9)

##<a id="step1"></a>Step 1: Install PowerShell
You must install and run PowerShell before you run scripts. You can download and install the latest version of PowerShell by [running the Microsoft web platform installation program](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409). See [How to install and configure PowerShell](http://www.windowsazure.cn/documentation/articles/powershell-install-configure) for more details of this process. The cmdlet that is used to create and manage MySQL Database on Azure databases is located in the Resource Manager module. Starting PowerShell imports the cmdlet from the Azure module by default. If you want to switch to the Resource Manager module, you can do so by using the following commands:

```
Switch-AzureMode -Name AzureResourceManager
```

If you receive a warning that the “Switch-AzureMode cmdlet is out of date and will be deleted in subsequent versions,” you can ignore this message and go to the section below. This command will be superseded in the new version, so we recommend that you update PowerShell to the latest version. For more information, see [Using Windows PowerShell with Resource Manager](http://www.windowsazure.cn/documentation/articles/powershell-azure-resource-manager)

##<a id="step2"></a>Step 2: Configure account details
You must bind the Azure account before running PowerShell for Azure subscriptions. Run the following commands, entering the same email address and password as for the Azure portal on the sign-in page in order to verify your identity.

```
Add-AzureAccount -Environment AzureChinaCloud 
```

If you have multiple SubIDs to choose from, you can make your selection by running the Select-AzureSubscription command.

##<a id="step3"></a>Step 3: Subscribe to MySQL Database on Azure services
Run the following command to subscribe to MySQL services.

```
Register-AzureProvider -ProviderNamespace "Microsoft.MySql"
```

##<a id="step4"></a>Step 4: Create resource groups
If you already have resource groups, you can create a server directly or edit and run the following commands to create a new resource group.

```
New-AzureResourceGroup -Name "resourcegroupChinaEast" -Location "chinaeast"
```

>[AZURE.NOTE]**The default location option is “chinanorth.” For reasons of both performance and security, we strongly recommend that you choose services that are in the same region as your resource group. **

##<a id="step5"></a>Step 5: Create servers
Edit and run the following commands to set information, including your server name, location, version, and other details, in order to complete creating the server.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -Location chinaeast -PropertyObject @{version = '5.5'} 
```

>[AZURE.NOTE]**“-ApiVersion 2015-09-01” specifies the API version and is essential. It is also important to note that running the above commands will finish creating the MySQL server but not the user. You must create user settings and permissions in the subsequent steps, which is a slightly different process than the way that the Azure portal works.**

##<a id="step6"></a>Step 6: Create server firewall rules
Edit and run the following commands to set information, including your firewall rule names and IP whitelist range (start IP address and end IP address), in order to finish creating the firewall rules.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```

##<a id="step7"></a>Step 7: Create databases
Edit and run the following commands to set information, including your database name and character sets, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```

##<a id="step8"></a>Step 8: Create users
Edit and run the following commands to set information, including your username and password, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

##<a id="step9"></a>Step 9: Add user permissions
Edit and run the following commands to assign database read/write permissions to users. Permissions can be either “Read” or “ReadWrite.”

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```

Using the operations described above, you have now finished creating the services, databases, users, and firewall rules. You are ready to begin using MySQL Database on Azure database services. If you need to perform further create, view, delete, or modify operations during the usage process, you can refer to [Using PowerShell to manage MySQL Database on Azure](/documentation/articles/mysql-database-commandlines).



<!--HONumber=81-->