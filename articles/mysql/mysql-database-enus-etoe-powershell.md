<properties linkid="" urlDisplayName="" pageTitle="Use Azure Resource Manager and PowerShell to deploy and use MySQL Database on Azure – Azure cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, beginner’s guide, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="This article explains how to use Azure PowerShell scripts to quickly set up and use MySQL services." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-etoe-powershell/)
- [English](/documentation/articles/mysql-database-enus-etoe-powershell/)

#Use Azure Resource Manager and Windows PowerShell to deploy and use MySQL Database on Azure

Azure Resource Manager is a new management framework for Azure services. Now you can use APIs and tools based on Azure Resource Manager to manage MySQL Database on Azure. For more information about Azure Resource Manager, see [Use resource groups to manage Azure resources](/documentation/articles/azure-preview-portal-using-resource-groups/). MySQL Database on Azure supports the use of PowerShell scripts to query, create, manage, and delete MySQL servers, databases, firewall rules, and users. It also supports the use of scripts to change parameters and settings. For more information about create, view, remove and modify operations, please see [Use PowerShell to manage MySQL Database on Azure](/documentation/articles/mysql-database-commandlines/).

### Contents
- [Step 1: Install Azure PowerShell](#step1)
- [Step 2: Configure account details](#step2)
- [Step 3: Subscribe to MySQL Database on Azure services](#step3)
- [Step 4: Create resource groups](#step4)
- [Step 5: Create servers](#step5)
- [Step 6: Create server firewall rules](#step6)
- [Step 7: Create databases](#step7)
- [Step 8: Create users](#step8)
- [Step 9: Add user privileges](#step9)

## <a id="step1"></a>Step 1: Install Azure PowerShell
You must install and run Azure PowerShell before you run scripts. You can run the [Microsoft Web platform installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) to download and install the latest version of Azure PowerShell. For more details, see [How to install and configure Azure PowerShell](http://www.windowsazure.cn/documentation/articles/powershell-install-configure/).
The cmdlet that is used to create and manage MySQL Database on Azure databases is located in the Resource Manager module. Starting PowerShell imports the cmdlet from the Azure module by default.
Confirm the Azure PowerShell version first by running the following commands in the console:
```
(Get-Module Azure).Version 
```
### Azure PowerShell version 0.9*:
Switch to the Azure Resource Manager module. Use the following command to switch:

```
Switch-AzureMode -Name AzureResourceManager
```
### Azure PowerShell version 1.0.0+:
In this version, the original Switch-AzureMode instruction has been removed. For details, see [Precautions for using Azure PowerShell version 1.0.0 and above in China](http://blogs.msdn.com/b/azchina/archive/2015/12/18/azure-powershell-1.0.0_e54e0a4e48722c6728572d4efd56_azure_7f4f28758476e86c0f618b4e7998_.aspx)

## <a id="step2"></a>Step 2: Configure account details
You must configure the Azure account before running PowerShell for Azure subscriptions. Run the following commands, entering the same email address and password as for the Azure portal on the sign-in page in order to verify your identity.
### Azure PowerShell version 0.9*:

```
Add-AzureAccount -Environment AzureChinaCloud 
```
### Azure PowerShell version 1.0.0+:
```
Login-AzureRmAccount –EnvironmentName AzureChinaCloud
```

If you have multiple SubIDs to choose from, you can make your selection by running the **Select-AzureSubscription** command.

## <a id="step3"></a>Step 3: Subscribe to MySQL Database on Azure services
Run the following command to subscribe to MySQL services.
### Azure PowerShell version 0.9*:
```
Register-AzureProvider -ProviderNamespace "Microsoft.MySql"
```
### Azure PowerShell version 1.0.0+:
```
Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.MySql"
```
## <a id="step4"></a>Step 4: Create resource groups
If you already have resource groups, you can create a server directly or edit and run the following commands to create a new resource group.

```
New-AzureResourceGroup -Name "resourcegroupChinaEast" -Location "chinaeast"
```

>[AZURE.NOTE] **The default location option is “chinanorth.” For performance and security, choose services that are in the same region as your resource group.**

## <a id="step5"></a>Step 5: Create servers
Edit and run the following commands to set information, including your server name, location, version, and other details, in order to finish creating the server.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -Location chinaeast -SkuObject @{name='MS4'} -PropertyObject @{version = '5.5'} 
```

>[AZURE.NOTE] **“-ApiVersion 2015-09-01” specifies the API version and is essential. It is also important to note that running the above commands will finish creating the MySQL server but not the user. You must create user settings and privileges in the subsequent steps, which is a different process than that used by the Azure portal.**

## <a id="step6"></a>Step 6: Create server firewall rules
Edit and run the following commands to set information, including your firewall rule names and safe IP range (start IP address and end IP address), in order to finish creating the firewall rules.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```

## <a id="step7"></a>Step 7: Create databases
Edit and run the following commands to set information, including your database name and character sets, in order to finish creating the database.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```

## <a id="step8"></a>Step 8: Create users
Edit and run the following commands to set information, including your username and password, in order to finish creating users.

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

## <a id="step9"></a>Step 9: Add user privileges
Edit and run the following commands to assign database read/write privileges to users. Privileges can be either “Read” or “ReadWrite.”

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```

You have now finished creating the services, databases, users, and firewall rules. You are ready to begin using MySQL Database on Azure database services. For more information about create, view, remove and read operations during the course of using database services, see [Use PowerShell to manage MySQL Database on Azure](/documentation/articles/mysql-database-commandlines/).

<!---HONumber=Acom_0218_2016_MySql-->