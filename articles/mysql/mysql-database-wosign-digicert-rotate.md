<properties linkid="" urlDisplayName="" pageTitle="如何如何替换WoSign根证书" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池,SSL安全访问，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="本文介绍如何将应用的WoSign根证书替换为DigiCert根证书。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

#替换应用的WoSign根证书为DigiCert
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-wosign-digicert-rotate/)

<tags ms.service="mysql" ms.date="3/8/2017" wacn.date="3/8/2017" wacn.lang="cn" />


由于最近WoSign CA证书遭到质疑（Mozilla不再信任2016年10月21日之后签发的WoSign CA证书，背景分析请参见mozilla.org官方文章：[WoSign Issues](https://wiki.mozilla.org/CA:WoSign_Issues)），为了保障客户数据库的安全性，我们决定在2017年4月11日将服务器端的WoSign证书替换为DigiCert证书。当天维护完成之后，客户端将无法利用原有WoSign证书发起SSL连接。如您的应用需启用SSL连接，请参考本文**“客户端根证书替换方案”**部分进行操作。

以下是维护预计的开始时间，用北京时间表示。**我们预计维护将在开始时间后10分钟内完成。**

**中国北部（北京数据中心）：**	4月11日，星期二，11:20AM（周二上午11点20分）

**中国东部（上海数据中心）：**	4月11日，星期二，11:30AM（周二上午11点30分）



> 提示：通过SSL加密访问数据库，可以保障您访问的安全性。我们强烈建议您在数据库通信中使用加密连接。有关如何启用SSL，请参考技术文章：[使用SSL安全访问](https://www.azure.cn/documentation/articles/mysql-database-ssl-connection/)。

##客户端根证书替换方案：

我们提供两种根证书替换方案。您可以根据实际业务情况选择具体实施何种方案。我们贴心地为您提前准备了测试环境，供您验证配置。详情请参见**“验证您的证书配置”**部分。

###方案一：使用WoSign + DigiCert混合证书

> **注意：**采用此方案，您可以随时根据业务情况进行客户端升级，不受我们维护时间的影响。我们建议您在4月11日之前实施此方案。

1. 前往DigiCert官网下载DigiCertGlobalRootCA.cer根证书 <[点此下载](https://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt)>。

2. 前往WoSign官网下载WS_CA1_NEW.cer根证书 <[点此下载](https://www.wosign.com/Root/WS_CA1_NEW.crt)>。

3. 下载并安装OpenSSL <[点此下载](http://slproweb.com/download/Win32OpenSSL_Light-1_1_0e.exe)>。

4. 把步骤1和2中下载的两张证书放到…\OpenSSL-Win32\bin目录下。

5. 使用openssl.exe命令行工具将DigiCertGlobalRootCA.cer转换为pem格式。

    OpenSSL>x509 -inform DEV -in DigiCertGlobalRootCA.cer -out DigiCertGlobalRootCA.pem

6. 使用Windows命令行工具（cmd）将两张证书合并为一张。

    C:\OpenSSL-Win32\bin>type DigiCertGlobalRootCA.pem WS_CA1_NEW.cer > root_certs.pem

7. 使用步骤6生成的root_certs.pem替换您应用中的WoSign证书。


> 以上仅为参考步骤。您也可以利用记事本打开转换成pem的DigiCert证书，将全部证书内容复制到WoSign中。最终生成的多证书格式如下所示：

    -----BEGIN CERTIFICATE-----
    <WoSign证书密钥>
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    <DigiCert证书密钥>
    -----END CERTIFICATE-----

###方案二：使用单张DigiCert证书替换原WoSign证书


> **注意：**采用此方案，您只能在我们完成维护之后进行客户端升级。在此之前使用此方案升级客户端可能导致期间您的应用无法成功发起SSL连接。

1.	前往DigiCert官网下载DigiCertGlobalRootCA.cer根证书 <[点此下载](https://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt)>。

2.	下载并安装OpenSSL <[点此下载](http://slproweb.com/download/Win32OpenSSL_Light-1_1_0e.exe)>。

3.	把步骤1中下载的证书放到…\OpenSSL-Win32\bin目录下。

4.	使用openssl.exe命令行工具将DigiCertGlobalRootCA.cer转换为pem格式。

    OpenSSL>x509 -inform DEV -in DigiCertGlobalRootCA.cer -out DigiCertGlobalRootCA.pem

5.	使用步骤4生成的DigiCertGlobalRootCA.pem替换您应用中的WoSign证书。

##验证您的证书配置：

我们为您提前准备了华东（上海）和华北（北京）两个测试环境，您可以使用这些环境测试新的证书配置。

*华东（上海）主机名称：mysqlservice-sslverify-sha.chinacloudapp.cn*

*华北（北京）主机名称：mysqlservice-sslverify-bjb.chinacloudapp.cn*

*端口：3306*

*用户名：<待测实例的用户名，如实例为’myinstance’，其有用户名’user’，则此处填写’myinstance%user’>*


> **注意：**您无需创建新的实例，只要利用现有实例和账号便可配置新的SSL根证书并连接到我们为您准备的测试环境中，以此验证应用配置的正确性。此测试环境仅供您验证新证书配置，请勿将生产系统部署在该环境中。完成维护后，我们可能会删除这些测试环境。

以mysql.exe（版本：5.7.15）为例，采用混合证书配置之后，SSL连接华东（上海）测试环境的屏幕截图如下所示。

![验证SSL配置][1]

<!--Image references-->

[1]: ./media/mysql-database-wosign-digicert-rotate/validating.png