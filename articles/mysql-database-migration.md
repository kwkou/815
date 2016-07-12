<properties linkid="" urlDisplayName="" pageTitle="如何迁移数据库至MySQL Database on Azure- Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池,应用迁移，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="
通过SSL加密访问数据库，可以保障您访问的安全性，本文介绍如何下载并配置SSL证书。目前MySQL Database on Azure支持利用公钥在服务器端进行加密验证。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-migration/)
- [English](/documentation/articles/mysql-database-enus-migration/)

# 迁移数据库至MySQL Database on Azure
本文概括介绍迁移用用数据库到MySQL Database on Azure的方式和需要考虑的注意事项。

##应用的可迁移性
迁移前，建议您首先评估自己的应用数据库的可迁移性。

MySQL Database on Azure兼容MySQL 5.5 和 MySQL 5.6，所以绝大部分应用可以不用做任何改动可以顺利地运行在MySQL Database on Azure上。当然为了更好地在MySQL Database on Azure上运行您的应用，我们建议应用要有数据库重连机制以保证良好的容错性，避免由于数据库短暂连不上的时候应用死掉，因为即使是高可用的云端的数据库也不可避免有故障切换和服务器维护等会导致短暂数据库连不上的情况出现。另外我们也建议尽量采用连接池和长连接来访问数据库，特别是对性能要求比较高的应用，详细可以参考[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/)。 

另外一个需要注意的地方是MySQL Database on Azure不支持老的MYISAM引擎，可以参考[常见问题](/documentation/articles/mysql-database-serviceinquiry/)中，为什么MySQL Database on Azure不支持MYISAM格式的数据库?在大多数情况下，您可以直接在建表的代码里把MyISAM数据引擎改成InnoDB就可以正常使用。 

##方案一： 基于数据库导入导出的迁移
如果您的系统可以接受较长时间（比如一二个小时）因系统迁移导致的downtime，您可以用比较简单的数据库导出和导入的方式进行数据库的迁移。
具体步骤：

1.登录Azure管理门户，在MySQL　Database on Azure上创建一个新的MySQL服务器并进行必要的配置比如每天的备份时间，具体步骤可以参考https://www.azure.cn/documentation/articles/mysql-database-get-started/#step1 。 


2.通过Azure管理门户在新创建的MySQL服务器上创建要迁移的目标数据库。具体步骤可以参考https://www.azure.cn/documentation/articles/mysql-database-get-started/#step4 。 


3.如果有多个数据库账号需要访问原数据库，您需要通过Azure管理门户在新的数据库服务器上创建对应的账号。 


4.如果数据库比较大（比如超过1GB），我们建议在同一个Azure数据中心准备一台VM这样可以先把数据传输到VM上然后再导入到DB里。 


5.在Azure上完成应用的除了数据库之外的组件的部署（比如website）。 


6.所有准备工作做好后，现在开始迁移。首先建议把应用关停或运行在只读模式（如果支持），这样以避免迁移过程中有新的数据。 


7.从当前数据库服务器导出应用数据库到一个文件。您可以用您熟悉的工具比如mysqldump，workbench，等等。下面是用mysqldump导出数据库的例子: mysqldump --databases <数据库名> --single-transaction --order-by-primary -r <备份文件名> --routines -h<服务器地址> -P<端口号> –u<用户名> -p<密码> 


8.如果数据文件比较大，建议先把数据文件传输到Azure上的一台VM上（应在同一个数据中心），您可以用您熟悉的数据传输工具（比如FTP或AzCopy等），这样可以避免因网络连接中断导致整个数据传输过程失败。如果备份文件很大，可以压缩后上传。 


9.把数据库数据导入到目标数据库中。您可以用您熟悉的工具比如mysql.exe，workbench，等等。下面是用mysql.exe导入数据库的例子 


9.1 在您的客户端通过mysql.exe连接新创建的MySQL服务器 （注意：如果您不是从Azure的VM上导入数据您需要把客户端加入IP白名单中): mysql -h<服务器地址> -P<端口号> –u<用户名> -p<密码> 


9.2 从SQL命令行导入数据：source <备份文件名>; 


10.把新部署的应用指向迁移好的数据库上，并完成剩余的应用迁移的步骤。 


##方案二： 基于数据同步的迁移


上面的数据库迁移方式比较简单，但缺点是会有较长时间的downtime，同时每一个迁移步骤要能够很熟练的完成并能很好地预期时间，否则会对您的业务连续性带来较大的影响。如果您不能接受迁移过程中的downtime， 比如是公司的 Web 应用，或者希望分阶段完成平滑的迁移，建议您用数据库同步的方式把当前的生产数据库同步到Azure上，MySQL Database on Azure提供了这个功能，在同步期间您的应用和当前的数据库可以继续工作不受影响。 


建议的迁移步骤： 

1.同步数据库到MySQL Database on Azure，您可以把运行在MySQL Database on Azure上的数据库服务器配置为从服务器。具体配置和同步数据库的步骤请参考如何配置数据同步复制到MySQL Database on Azure。 



>[AZURE.NOTE] ** 注意：您会需要打开当前的数据库服务器的外部访问，我们强烈建议您配置SSL并只允许外部通过SSL访问。**

2.在Azure上部署新的应用并指向新建的Azure上的数据库。 


提示： 此时由于Azure上的数据库运行在只读模式，应用的功能可能受限。

3.确认Azure上的从数据库已达到同步的状态。您可以根据复制页面上的复制状态和复制滞后来确认同步状态。 
![迁移][1]

4.关停老的应用或者让应用运行在只读模式（如果支持只读模式）。
	
5.停止数据库同步复制。您只需在下面的页面上点击“禁用”然后保存就可以。
![迁移][2]
>[AZURE.NOTE] **注意：这个操作会重启数据库服务器 **

6.启用新的应用。

## 关于数据库迁移的常见问题：
###导入TRIGGER, PROCEDURE, VIEW, FUNCTION, 或EVENT过程中报”Access denied; you need (at least one of) the SUPER privilege(s) for this operation” 错误。
检查报错的语句有否使用DEFINER并使用非当前用户，比如DEFINER=user@host， 如果这样的话MySQL是要求SUPER权限来执行该语句，由于MySQL Database on Azure不提供用户SUPER权限（参考[服务限制](/documentation/articles/mysql-database-operation-limitation/) ），导致运行该语句失败。您只需要把DEFINER从该语句删掉而使用缺省的当前用户就可以了。

###MYSQL Database on Azure 的管理门户只支持对用户设置整个数据库的读写权限，如果我现在的数据库有对用户权限更细化的设置，迁移会成功吗？
没有问题，虽然我们的管理门户 以及PowerShell /REST API在创建用户或数据库时只支持对整个数据库设置读写权限，但您可以用“grant”语句对用户权限进行更细化的设置。

<!--Image references-->

[1]: ./media/mysql-database-migration/migration1.png
[2]: ./media/mysql-database-migration/migration2.png

