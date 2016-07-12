<properties linkid="" urlDisplayName="" pageTitle="使用PowerShell管理MySQL Database on Azure - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,Azure MySQL PowerShell" description="本文介绍如何通过PowerShell实现更多MySQL Database on Azure的查询、创建、修改、删除等操作。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-commandlines/)
- [English](/documentation/articles/mysql-database-enus-commandlines/)

#使用PowerShell管理MySQL Database on Azure

本文主要介绍如何通过PowerShell实现更多MySQL Database on Azure的创建、查看、删除、更改等操作。建议您先阅读[利用Azure 资源管理器与 PowerShell 来部署使用MySQL Database on Azure](/documentation/articles/mysql-database-etoe-powershell/),该文介绍了如何下载使用Azure PowerShell, 如何利用PowerShell来快速创建MySQL Database on Azure数据服务。

在开始之前，请确保已将 Azure PowerShell 准备就绪。

注意：本文主要针对Azure PowerShell 0.9*版本，如果您使用1.0.0+版本，请参考[Azure PowerShell 1.0.0以上版本在中国Azure使用的注意事项](http://blogs.msdn.com/b/azchina/archive/2015/12/18/azure-powershell-1.0.0_e54e0a4e48722c6728572d4efd56_azure_7f4f28758476e86c0f618b4e7998_.aspx)。将下述命令当中AzureResource改为AzureRmResource运行。

###目录
- [了解 Azure 资源模板和资源组](#gettoknow)
- [创建操作](#creat)
- [查看操作](#view)
- [修改操作](#set)
- [删除操作](#delete)

## <a id="gettoknow"></a>1. 了解 Azure 资源模板和资源组

大多数部署和运行在 Azure 中的应用程序是通过不同云资源类型的组合构建的。Azure资源管理器模板使你能够集中部署和管理这些不同的资源，只需对资源和关联的配置及部署参数进行 JSON 描述即可。
###1.1 了解MySQL Database on Azure的资源类型参数信息
目前，MySQL Database on Azure的Json File中定义了六种资源类型： Servers, Databases, Users, Privileges, FirewallRules, Backups。用户可以通过"Get","New","Set","Remove"指令分别对上述六种资源类型进行查看、创建、修改、删除的操作。
您可以通过下载[Json模板文件](http://wacnppe.blob.core.chinacloudapi.cn/marketing-resource/2015-09-01.json)来了解更多MySQL Database on Azure 服务的API参数定义。

## <a id="creat"></a>2. 创建操作
通过New指令可以创建MySQL服务器、数据库、用户、用户权限、备份、防火墙规则等。
###2.1 创建服务器
编辑运行以下命令，定义您的服务器名称、位置、版本等信息来完成服务器创建。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 		
-ResourceGroupName resourcegroupChinaEast -Location chinaeast -SkuObject @{name='MS4'} -PropertyObject @{version = '5.5'} 
```

###2.2 创建服务器防火墙原则
编辑运行以下命令，定义您的防火墙原则名称、IP白名单范围（起始IP地址，终止IP地址）等信息来完成防火墙原则的创建。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="0.0.0.0"; endIpAddress="255.255.255.255"} -ResourceGroupName resourcegroupChinaEast
```

###2.3 创建数据库
编辑运行以下命令，定义您的数据库名称、字符集等信息完成数据库创建。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{collation='utf8_general_ci'; charset='utf8'}
```

###2.4 创建用户
编辑运行以下命令，定义您的用户名、密码等信息完成数据库创建。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc123'}
```

###2.5 添加用户权限
编辑运行以下命令，设置数据库读写权限给用户。权限分为"Read"以及"ReadWrite"。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 	
-ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='ReadWrite'}
```
###2.6 创建按需备份文件
编辑运行以下命令，制定备份文件名称，创建按需备份文件。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{}
```

###2.7 服务器备份恢复 （基于快照的恢复）
编辑运行以下命令，通过制定快照来恢复服务器。
其中，server和region的信息必填，backup不指定的情况下，默认通过最新的快照拷贝进行恢复。

```
New-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testrestore -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -Location ChinaEast -Properties @{creationSource=@{server='testPSH';region='chinaEast' ; backup = 'testpsh1b0a9038-6953-42ad-ac8e-42f73180825b'};version = '5.5'}
```

## <a id="view"></a>3. 查看操作
通过Get指令可以查看当前MySQL服务器、数据库、用户、用户权限、备份、防火墙规则等列表,也可以查看详细参数配置。
###3.1 查看服务器列表
>[AZURE.NOTE] ** 注意:“在Azure管理门户上创建的实例，按照实例所处的区域分别在默认资源组：Default-MySql-ChinaEast以及Default-MySql-ChinaNorth**

编辑运行以下命令，查看当前所有服务器列表

```
Get-AzureResource -ResourceType "Microsoft.MySql/servers"  -ApiVersion 2015-01-01 
```
>[AZURE.NOTE] ** 注意:与其他指令不同，查看服务器中的“-ApiVersion 2015-01-01”，指向ARM的API，其他指令中，均为“-ApiVersion 2015-09-01”，指向MySQL的API。**

###3.2 查看数据库列表及参数
编辑运行以下命令，查看当前资源组内某个服务器的所有数据库列表：

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.3 查看用户列表及参数
编辑运行以下命令，查看当前资源组内某个服务器的所有用户列表：

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/users" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.4 查看用户权限列表及参数
编辑运行以下命令，查看当前资源组内某个数据库的用户权限：

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" -Name testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.5 查看备份列表及参数
编辑运行以下命令，查看当前资源组内某个服务器的所有备份文件：

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###3.6 查看防火墙规则列表及参数
编辑运行以下命令，查看当前资源组内某个服务器的所有防火墙规则：

```
 Get-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -Name testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

## <a id="set"></a>4. 修改操作
通过Set指令可以完成账户密码修改，权限修改，某些服务器参数修改等配置工作。
###4.1 修改账户密码
编辑运行以下命令，修改某个账户密码。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{password='abc1234'} -UsePatchSemantics
```

###4.2 修改某个用户的读写权限
编辑运行以下命令，修改某个用户的读写权限。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/databases/privileges" 		
-ResourceName testPSH/demodb/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{level='Read'} -UsePatchSemantics
```

###4.3 允许所有Azure服务访问
编辑运行以下命令，允许所有Azure服务访问指定服务器。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{allowAzureServices='true'} -UsePatchSemantics
```

###4.4 修改默认每日备份时间
编辑运行以下命令，修改指定服务器的默认日备起始时间(此处为北京时间UTC+8)，0-24可选。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{dailyBackupTimeInHour='5'} -UsePatchSemantics
```

###4.5 开启慢查询日志
编辑运行以下命令，开启慢查询日志。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{slow_query_log='ON'}} -UsePatchSemantics
```

###4.6 修改防火墙原则
编辑运行以下命令，更改已有的防火墙原则。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -PropertyObject @{startIpAddress="1.1.1.1"} -ResourceGroupName resourcegroupChinaEast -UsePatchSemantics
```

###4.7 修改部分MySQL服务器设置
以wait\_timeout参数为例，其他参数详见Json文件，编辑运行以下命令，更改wait_timeout默认值。

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -PropertyObject @{options=@{wait_timeout=70}} -UsePatchSemantics
```
对其他参数的修改，可以参考下面Json文件的定义，参数的有效值范围可参考[定制MySQL Database on Azure服务器参数](/documentation/articles/mysql-database-advanced-settings/)：

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

###4.8 修改MySQL服务器性能版本

```
Set-AzureResource -ResourceType "Microsoft.MySql/servers " -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast -SkuObject @{name="MS4"} -UsePatchSemantics
```

	
## <a id="delete"></a>5. 删除操作
通过Remove指令可以删除MySQL服务器、数据库、用户、备份、防火墙规则等。
###5.1 删除服务器
编辑运行以下命令，删除指定服务器。

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers" -ResourceName testPSH -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5.2 删除服务器防火墙原则
编辑运行以下命令，删除指定防火墙。

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/firewallRules" -ResourceName testPSH/rule1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5.3 删除数据库
编辑运行以下命令，删除指定数据库。

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/databases" -ResourceName testPSH/demodb -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

###5.4 删除用户
编辑运行以下命令，删除指定用户。

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/users" -ResourceName testPSH/admin -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast
```

###5.5 删除备份文件
编辑运行以下命令，删除指定备份文件。

```
Remove-AzureResource -ResourceType "Microsoft.MySql/servers/backups" -ResourceName testPSH/back1 -ApiVersion 2015-09-01 -ResourceGroupName resourcegroupChinaEast 
```

随着MySQL Database on Azure提供的功能增多，我们也会陆续更新相应的PowerShell指南。