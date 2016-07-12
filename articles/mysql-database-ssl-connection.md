<properties linkid="" urlDisplayName="" pageTitle="如何使用SSL访问MySQL Database on Azure- Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池,SSL安全访问，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="
通过SSL加密访问数据库，可以保障您访问的安全性，本文介绍如何下载并配置SSL证书。目前MySQL Database on Azure支持利用公钥在服务器端进行加密验证。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

# SSL安全访问MySQL Database on Azure
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-ssl-connection/)
- [English](/documentation/articles/mysql-database-enus-ssl-connection/)

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

通过SSL加密访问数据库，可以保障您访问的安全性，本文介绍如何下载并配置SSL证书。目前MySQL Database on Azure支持利用公钥在服务器端进行加密验证。

在您创建MySQL Database on Azure实例时，我们强烈建议您将数据库实例与其他Azure服务放在同一区域，即使不用SSL加密，也可保障安全性。


## 步骤 1:下载SSL连接的公钥证书
[点击下载](https://www.wosign.com/root/WS_CA1_NEW.crt)SSL证书至本地。

## 步骤 2：使用SSL来连接MySQL Database on Azure上的服务器

### 利用MySQL客户端进行配置
以MySQL.exe为例，创建连接时需使用--ssl-ca参数来指定公钥证书。关于SSL连接的更多信息，请参考 Using SSL for Secure Connections。

参考样例：

mysql.exe --ssl-ca=WS_CA1_NEW.crt -h mysqlservices.chinacloudapp.cn -u ssltest%test -p

![mysql.exe访问数据库][1]

连接成功后，使用status命令可以查看客户端SSL连接特性。若SSL的参数值为Cipher in use，则成功创建SSL连接；若SSL的参数值为Not in use, 则仍是非SSL连接。

![验证][6]

>[AZURE.NOTE]**需要注意的是MySQL on Azure在代理服务器proxy和用户端之间建立了SSL安全链接，所以服务器中SSL相关的全局变量或者会话变量仍然是DISABLED值，而实际上整个通信过程已被TLSv1加密。**

以MySQL Workbench为例，通过Parameters标签设置访问数据库的Connection String，如下图所示。

![配置connection string][2]

通过SSL标签配置SSL证书.

![配置SSL证书][3]

> **注意** 
> 
> 1.在Use SSL中选择‘If Available’，否则可能会造成配置失败。 在Test Connection过程中可能会提示SSL not enabled，这是一个假预警，点击确认后连接数据库，
>
> ![errormessage][4]
>

> 2.MySQL workbench 6.3.5中，会默认进行SSL加密，且存在一定的兼容性问题，具体解决方法可参见[客户端兼容性常见问题](/documentation/articles/mysql-database-compatibilityinquiry/)

> **提示** 当前证书支持MySQL.exe 5.5.44和5.6.25及其后续版本。
> 
### 利用函数进行配置
以Python为例，下图是一段示例代码，供参考：

![python SSL访问][5]



<!--Image references-->

[1]: ./media/mysql-database-ssl-connection/ssl-001.png
[2]: ./media/mysql-database-ssl-connection/ssl-002.png
[3]: ./media/mysql-database-ssl-connection/ssl-003.png
[4]: ./media/mysql-database-ssl-connection/ssl-004.png
[5]: ./media/mysql-database-ssl-connection/ssl-005.png
[6]: ./media/mysql-database-ssl-connection/ssl-006.png
