<properties linkid="" urlDisplayName="" pageTitle="如何使用SSL访问MySQL Database on Azure- Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池,SSL安全访问，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="
通过SSL加密访问数据库，可以保障您访问的安全性，本文介绍如何下载并配置SSL证书。目前MySQL Database on Azure支持利用公钥在服务器端进行加密验证。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

# SSL安全访问MySQL Database on Azure
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-ssl-connection/)
- [English](/documentation/articles/mysql-database-enus-ssl-connection/)

<tags ms.service="mysql" ms.date="04/12/2017" wacn.date="04/12/2017" wacn.lang="cn" />

通过SSL加密访问数据库，可以保障您访问的安全性，本文介绍如何下载并配置SSL证书。目前MySQL Database on Azure支持利用公钥在服务器端进行加密验证。

在您创建MySQL Database on Azure实例时，我们强烈建议您将数据库实例与其他Azure服务放在同一区域，即使不用SSL加密，也可保障安全性。


## 步骤1：下载证书至本地

前往DigiCert官网下载DigiCertGlobalRootCA.cer根证书 <[点此下载](https://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt)>

## 步骤2：下载并安装OpenSSL
<[点此下载](http://slproweb.com/download/Win32OpenSSL_Light-1_1_0e.exe)>

## 步骤3：移动本地证书文件至OpenSSL目录

把步骤1中下载的证书放到…\OpenSSL-Win32\bin目录下。

## 步骤4：将证书文件转换为pem格式

下载的根证书文件为cer格式，需利用openssl.exe命令行工具执行以下命令来实现文件格式的转换：

	OpenSSL>x509 -inform DEV -in DigiCertGlobalRootCA.cer -out DigiCertGlobalRootCA.pem

## 步骤5：将根证书与应用绑定

最后，将步骤4中生成的pem根证书文件与应用绑定。后文提供了mysql和workbench两种常见客户端的配置方法。


### 使用mysql命令行工具进行SSL连接

以mysql.exe命令行工具为例，创建连接时需使用--ssl-ca参数来指定公钥证书。命令如下：

	mysql.exe --ssl-ca=C:\OpenSSL-Win32\bin\DigiCertGlobalRootCA.pem -h mysql4doc.mysqldb.chinacloudapi.cn -u mysql4doc%admin -p

连接成功后，使用status命令可以查看客户端SSL连接特性。若SSL的参数值为Cipher in use，则表明成功创建SSL连接。

![mysql-ssl-connection](./media/mysql-database-ssl-connection/5-1_mysql-ssl-connection.png)


>[AZURE.NOTE]**需要注意的是MySQL on Azure在代理服务器proxy和用户端之间建立了SSL安全链接，所以服务器中SSL相关的全局变量或者会话变量仍然是DISABLED值，而实际上整个通信过程已被TLSv1加密。**

### 使用MySQL Workbench图形界面工具进行SSL连接

MySQL Workbench是常用的图形化数据库管理工具。通过Setup New Connection或Manage Server Connections对话框的SSL标签页，可以配置SSL连接所需的证书，如下图所示。

![workbench-ssl-connection](./media/mysql-database-ssl-connection/5-2_workbench-ssl-connection.png)

> **注意** 
> 
> 1. 在Use SSL中选择‘If Available’，否则可能会造成配置失败。 在Test Connection过程中可能会提示SSL not enabled，这是一个假预警，点击确认后连接数据库，
>
>
> 2. MySQL workbench 6.3.5中，会默认进行SSL加密，且存在一定的兼容性问题，具体解决方法可参见[客户端兼容性常见问题](/documentation/articles/mysql-database-compatibilityinquiry/)

> **提示** 当前证书支持MySQL.exe 5.5.44和5.6.25及其后续版本。


### 利用函数进行配置

以Python为例，下图是一段示例代码，供参考：

![python SSL访问][5]



<!--Image references-->

[5]: ./media/mysql-database-ssl-connection/ssl-005.png