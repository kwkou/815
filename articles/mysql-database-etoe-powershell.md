<properties linkid="" urlDisplayName="" pageTitle="使用PowerShell管理使用MySQL Database on Azure - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,Azure 资源管理器,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="本文主要介绍如何使用PowerShell脚本快速搭建使用MySQL服务。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-etoe-powershell/)
- [English](/documentation/articles/mysql-database-enus-etoe-powershell/)

#利用Azure 资源管理器与 PowerShell 来部署使用MySQL Database on Azure

Azure 资源管理器 (ARM) 是适用于 Azure 服务的全新管理框架。现在可以使用基于 Azure 资源管理器的 API 和工具来管理 MySQL Database on Azure。若要了解有关 Azure 资源管理器的详细信息，请参阅[使用资源组管理 Azure 资源](/documentation/articles/azure-preview-portal-using-resource-groups/)。MySQL Database on Azure支持您通过PowerShell脚本查询创建管理删除MySQL服务器、数据库、防火墙原则、用户等，同时支持您通过脚本更改参数设置。本文主要介绍如何使用PowerShell脚本快速搭建使用MySQL服务。对于更多创建、查看、删除、更改的操作，您可以查看[使用PowerShell管理MySQL Database on Azure](/documentation/articles/mysql-database-commandlines/)。

###目录
- [步骤1：安装Azure PowerShell](#step1)
- [步骤2：配置账户信息](#step2)
- [步骤3：订阅MySQL Database on Azure服务](#step3)
- [步骤4：创建资源组](#step4)
- [步骤5：创建服务器](#step5)
- [步骤6：创建服务器防火墙原则](#step6)
- [步骤7：创建数据库](#step7)
- [步骤8：创建用户](#step8)
- [步骤9：添加用户权限](#step9)

##<a id="step1"></a>步骤1： 安装Azure PowerShell
运行脚本前，您需要安装并运行Azure PowerShell。您可以通过[运行Microsft Web平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)下载并安装最新版本Azure PowerShell 。可参阅[如何安装和配置Azure PowerShell](/documentation/articles/powershell-install-configure/)来了解更多详细步骤。
用于创建和管理MySQL Database on Azure 数据库的cmdlet位于Azure资源管理器模块中。启动Azure PowerShell时，默认情况下将导入Azure模块中的cmdlet。
请先确认Azure PowerShell版本， 可在控制台运行以下命令：
```
(Get-Module Azure).Version 
```
### Azure PowerShell 0.9* 版本：
切换到Azure资源管理器模块，请使用以下命令转换：

```
Switch-AzureMode -Name AzureResourceManager
```
### Azure PowerShell 1.0.0+版本：
此版本最大的改变在于将原来的Switch-AzureMode的指令移除。有关详细信息，请参阅[Azure PowerShell 1.0.0以上版本在中国Azure使用的注意事项](http://blogs.msdn.com/b/azchina/archive/2015/12/18/azure-powershell-1.0.0_e54e0a4e48722c6728572d4efd56_azure_7f4f28758476e86c0f618b4e7998_.aspx)

##<a id="step2"></a>步骤2： 配置账户信息
在针对Azure订阅运行PowerShell之前，必须先将Azure账户绑定。运行以下命令，在登陆页面输入与Azure管理门户相同的电子邮件和密码，进行身份验证。
### Azure PowerShell 0.9* 版本：

```
Add-AzureAccount -Environment AzureChinaCloud 
```
### Azure PowerShell 1.0.0+版本：
```
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
```

若您有多个SubID需要选择，可通过运行 Select-AzureSubscription命令进行选择。

##<a id="step3"></a>步骤3： 订阅MySQL Database on Azure服务
运行以下命令订阅MySQL服务。
### Azure PowerShell 0.9* 版本：
```
Register-AzureProvider -ProviderNamespace "Microsoft.MySql"
```
### Azure PowerShell 1.0.0+版本：
```
Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.MySql"
```
##<a id="step4"></a>步骤4： 创建资源组
如果您已有资源组，可以直接创建服务器，或者编辑运行以下命令，创建新的资源组：
### Azure PowerShell 0.9* 版本：
```
New-AzureResourceGroup -Name "resourcegroupChinaEast" -Location "chinaeast"
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResourceGroup -Name "resourcegroupChinaEast" -Location "chinaeast"
```
>[AZURE.NOTE] ** 注意:Location的默认选项为chinanorth, 处于性能以及安全性考虑，强烈建议您将资源组中的服务选择在同一个地域中。**

##<a id="step5"></a>步骤5： 创建服务器
编辑运行以下命令，定义您的服务器名称、位置、版本等信息来完成服务器创建。
### Azure PowerShell 0.9* 版本：
```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -Location chinaeast -SkuObject @{name='MS4'} -PropertyObject @{version = '5.5'} 
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -Location chinaeast -PropertyObject @{version = '5.5'} 
```
>[AZURE.NOTE] ** 注意:“-ApiVersion 2015-09-01”指定了API的版本，是必要的。另外，运行上述命令可以完成MySQL服务器的创建，但没有用户，须在后续步骤中创建用户设置权限，这一点和使用Azure管理门户创建稍有不同**

##<a id="step6"></a>步骤6： 创建服务器防火墙原则
编辑运行以下命令，定义您的防火墙原则名称、IP白名单范围（起始IP地址，终止IP地址）等信息来完成防火墙原则的创建。
### Azure PowerShell 0.9* 版本：
```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```
##<a id="step7"></a>步骤7： 创建数据库
编辑运行以下命令，定义您的数据库名称、字符集等信息完成数据库创建。
### Azure PowerShell 0.9* 版本：
```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```


##<a id="step8"></a>步骤8： 创建用户
编辑运行以下命令，定义您的用户名、密码等信息完成数据库创建。
### Azure PowerShell 0.9* 版本：

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

##<a id="step9"></a>步骤9： 添加用户权限
编辑运行以下命令，设置数据库读写权限给用户。权限分为"Read"以及"ReadWrite"。
### Azure PowerShell 0.9* 版本：
```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```
### Azure PowerShell 1.0.0+版本：
```
New-AzureRmResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```
通过上述操作，您已经完成了服务器、数据库、用户、防火墙原则等的创建工作，可以开始使用MySQL Database on Azure的数据库服务。在使用过程中，如需更多创建、查看、删除、更改的操作，您可以查看[使用PowerShell管理MySQL Database on Azure](/documentation/articles/mysql-database-commandlines/)。


