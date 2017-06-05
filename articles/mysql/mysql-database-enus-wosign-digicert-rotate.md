<properties
    linkid=""
    urlDisplayName=""
    pageTitle="How to replace a WoSign root certificate"
    metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pooling, secure SSL access，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS"
    description="This article explains how to replace an app’s WoSign root certificate with a DigiCert root certificate."
    metaCanonical=""
    services="MySQL"
    documentationCenter="Services"
    authors="v-chenyh"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="mysql_en"
    ms.author="v-chenyh"
    ms.topic="article"
    ms.date="3/8/2017"
    wacn.date="3/8/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-wosign-digicert-rotate/)
- [English](/documentation/articles/mysql-database-enus-wosign-digicert-rotate/)

#<a name="wosigndigicert"></a>Replace an app’s WoSign root certificate with a DigiCert


As the use of WoSign CA certificates has been called into question (Mozilla no longer trusts WoSign CA certificates issued after October 21, 2016; see the official mozilla.org website for a background analysis: [WoSign Issues](https://wiki.mozilla.org/CA:WoSign_Issues)), in order to protect the security of your database, we have decided to replace server-side WoSign certificates with DigiCert certificates from April 11, 2017. After maintenance for that day has been completed, it will no longer be possible to use your original WoSign certificate to initiate SSL links from the client side. If your app requires SSL connections, please refer to the following section of this article: **“Client-side root certificate replacement solutions.”**

The expected maintenance start times are indicated below in Beijing time. **We expect that maintenance will be completed within 10 minutes of the start time.** 

**North China (Beijing Data Center):** Tuesday, April 11, 11:20 AM (11:20 on Tuesday morning)

**East China (Shanghai Data Center):** Tuesday, April 11, 11:20 AM (11:20 on Tuesday morning)

>Tip: Encrypting access to a database with SSL can ensure that your access is secure. We strongly recommend using encrypted connections for database communications. Please refer to the technical article [Use SSL secure access](/documentation/articles/mysql-database-enus-ssl-connection/) for information on how to enable SSL.

##<a name=""></a>Client-side root certificate replacement solutions:

We offer two types of root certificate replacement solution. You can choose which solution to implement based on your actual business situation. For your convenience, we have prepared a test environment. Please feel free to use it to verify your configuration. See the **“Verify your certificate configuration” section for details.**

###<a name="wosign--digicert"></a>Solution 1: Use a WoSign + DigiCert hybrid certificate

>**Note:** This solution allows you to perform client-side upgrades at any time based on your business situation, without being affected by our maintenance periods. We recommend that you implement this solution before April 11.

1. Go to the official DigiCert website and download the DigiCertGlobalRootCA.cer certificate <[Click here to download](https://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt)>.

2. Go to the official WoSign website and download the WS_CA1_NEW.cer certificate <[Click here to download](https://www.wosign.com/Root/WS_CA1_NEW.crt)>.

3. Download and install OpenSSL <[Click here to download](http://slproweb.com/download/Win32OpenSSL_Light-1_1_0e.exe)>.

4. Put the two certificates downloaded in steps 1 and 2 in the ...\OpenSSL-Win32\bin directory.

5. Use the openssl.exe command line tool to convert DigiCertGlobalRootCA.cer to PEM format.

    OpenSSL>x509 -inform DEV -in DigiCertGlobalRootCA.cer -out DigiCertGlobalRootCA.pem

6. Use the Windows command line tool (CMD) to merge the two certificates into one.

    C:\OpenSSL-Win32\bin>type DigiCertGlobalRootCA.pem WS_CA1_NEW.cer > root_certs.pem

7. Use the root_certs.pem file generated in step 6 to replace the WoSign certificate in your app.

>These steps are provided solely for reference. You can also open the DigiCert certificate you converted into PEM format with Notepad and copy the entire contents of the certificate into WoSign. The multiple certificate format that is ultimately generated is shown below:

    -----BEGIN CERTIFICATE-----
    <WoSign证书密钥>
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    <DigiCert证书密钥>
    -----END CERTIFICATE-----

###<a name="digicertwosign"></a>Solution 2: Replace the original WoSign certificate with a single DigiCert certificate

>**Note:** If you use this solution, you will only be able to perform client-side upgrades after we have completed our maintenance. Using this solution for client-side upgrades before then could make it impossible for your apps to successfully initiate SSL links.

1. Go to the official DigiCert website and download the DigiCertGlobalRootCA.cer certificate <[Click here to download](https://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt)>.

2. Download and install OpenSSL <[Click here to download](http://slproweb.com/download/Win32OpenSSL_Light-1_1_0e.exe)>.

3. Put the certificate downloaded in step 1 in the ...\OpenSSL-Win32\bin directory.

4. Use the openssl.exe command line tool to convert DigiCertGlobalRootCA.cer to PEM format.

    OpenSSL>x509 -inform DEV -in DigiCertGlobalRootCA.cer -out DigiCertGlobalRootCA.pem

5. Use the DigiCertGlobalRootCA.pem file generated in step 4 to replace the WoSign certificate in your app.

##<a name=""></a>Verify your certificate configuration:

We have prepared two test environments for you, one for East China (Shanghai) and one for North China (Beijing). You can use these test environments to test new certificate configurations.

*East China (Shanghai) host name: mysqlservice-sslverify-sha.chinacloudapp.cn*

*North China (Beijing) host name: mysqlservice-sslverify-bjb.chinacloudapp.cn*

*Port: 3306*

*Username: <the usernames for instances awaiting testing are such that if the instance “myinstance” has the username “user,” you should enter “myinstance%user” here>*

>**Note:** You do not need to create a new instance. You can simply use the existing instance and account to configure the new SSL root certificate, connect to the test environments that we have prepared for you, and verify that the app is correctly configured. The test environments are provided for the sole purpose of verifying new certificate configurations. Please do not produce system deployments in these environments. We may delete these test environments after the maintenance has been completed.

Taking mysql.exe (version 5.7.15) as an example, once the hybrid certificate configuration is in use, the SSL connection to the East China (Shanghai) test environment will look like the screenshot below.

![Verify the SSL configuration][1]

<!--Image references-->

[1]: ./media/mysql-database-wosign-digicert-rotate/validating.png

<!--HONumber=May17_HO3-->