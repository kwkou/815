<properties linkid="" urlDisplayName="" pageTitle="Use SSL to securely access MySQL Database on Azure – Azure cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Using Secure Sockets Layer (SSL) SSL encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-ssl-connection/)
- [English](/documentation/articles/mysql-database-enus-ssl-connection/)

# Use SSL to securely access MySQL Database on Azure


Using Secure Sockets Layer (SSL) encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side.

When you create a MySQL Database on Azure instance, we strongly recommend that you put the database instance in the same region as other Azure services. This helps ensure their security even if you do not use SSL encryption.


## Step 1: Download the public key certificates for an SSL connection
[Click to download ](https://www.wosign.com/root/WS_CA1_NEW.crt)the SSL certificate locally.

## Step 2: Use SSL to connect to a server on MySQL Database on Azure

You can configure either by using the MySQL client or by using functions.

### Configure by using the MySQL client
For example, using MySQL.exe, use the --ssl-ca parameter to specify a public key certificate when you create the connection. For more information about SSL connections, see “Using SSL for secure connections”.

Reference examples:

mysql.exe --ssl-ca=WS\_CA1\_NEW.crt -h mysqlservices.chinacloudapp.cn -u ssltest%test -p

![mysql.exe database access][1]

Once the connection is successful, you can use the status command to view the client-side SSL connection properties. If the SSL parameter value is **Cipher in use**, then you have successfully created the SSL connection. If the SSL parameter is **Not in use**, then the connection is still a non-SSL connection.

![Verification][6]

>[AZURE.NOTE] **MySQL on Azure created an SSL secure connection between the proxy server and the client, so while SSL-related global variables or session variables on the server remain set to DISABLED, the entire communication process has actually already been encrypted with TLSv1.**

Using MySQL Workbench as an example, use the **Parameters** tab to set up the connection string for accessing the database, as shown in the following image.

![Configuring the connection string][2]

Configure the SSL certificate by using the fields in the **SSL** tab.

![Configuring SSL certificates][3]

> **Notes**
> 
> In the **Use SSL** field, select **If available**. Otherwise this may cause the configuration to fail. You may see **SSL not enabled** during the test connection process, but disregard this. Click **OK** to connect to the database.
>
> ![errormessage][4]
>

> MySQL Workbench 6.3.5 uses SSL encryption by default, but involves certain compatibility issues. For specific solutions, see [Common client compatibility issues](/documentation/articles/mysql-database-compatibilityinquiry/).

> **Tip:** The current certificate supports MySQL.exe 5.5.44 and 5.6.25 and subsequent versions.
> 
### Configure by using functions
For example, using Python, you can see from the following sample code how to configure by using functions.

![python SSL access][5]



<!--Image references-->

[1]: ./media/mysql-database-ssl-connection/ssl-001.png
[2]: ./media/mysql-database-ssl-connection/ssl-002.png
[3]: ./media/mysql-database-ssl-connection/ssl-003.png
[4]: ./media/mysql-database-ssl-connection/ssl-004.png
[5]: ./media/mysql-database-ssl-connection/ssl-005.png
[6]: ./media/mysql-database-ssl-connection/ssl-006.png

<!---HONumber=Acom_0218_2016_MySql-->