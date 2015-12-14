<properties linkid="" urlDisplayName="" pageTitle="Use SSL to securely access MySQL Database on Azure – Microsoft Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Using SSL encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="07/27/2015"/>

# Use SSL to securely access MySQL Database on Azure


Using SSL encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side.

When you create a MySQL Database on Azure instance, we strongly recommend that you put the database instance in the same region as other Azure services. This helps ensure their security even if you do not use SSL encryption.


## Step 1: Download the public key certificates for an SSL connection
[Click to download](https://www.wosign.com/root/WS_CA1_NEW.crt) the SSL certificate locally.

## Step 2: Use SSL to connect to a server on MySQL Database on Azure

### Configure by using the MySQL client
Using MySQL.exe as an example, you need to use the --ssl-ca parameter to specify a public key certificate when you create the connection. For more information on SSL connections, refer to “Using SSL for secure connections.”

Reference examples:

mysql.exe --ssl-ca=WS\_CA1\_NEW.crt -h mysqlservices.chinacloudapp.cn -u ssltest%test -p

![mysql.exe database access][1]

Once you have successfully connected, use the status command. If the SSL parameter value is “Cipher in use,” then you have successfully created the SSL connection. If the SSL parameter is “Not in use,” then the connection is still a non-SSL connection.

![Verification][6]

> **Indicates **that the current certificate supports MySQL.exe 5.5.44 and 5.6.25, as well as subsequent versions.

Using MySQL Workbench as an example, use the parameters label to set up the connection string for accessing the database, as shown in the image below.

![Configuring the connection string][2]

Configure the SSL certificate by using SSL labels

![Configuring SSL certificates][3]

> **Make sure that you** select “If Available” under Use SSL; otherwise, this may cause configuration failures. “SSL not enabled” may be shown during the test connection process. This is a false alarm, as the communication process is encrypted once you have clicked “OK” and connected to the database.
>
> ![errormessage][4]
>

### Configure by using functions
Using Python as an example, the image below shows a piece of sample code for your reference.

![python SSL access][5]



<!--Image references-->

[1]: ./media/mysql-database-ssl-connection/ssl-001.png
[2]: ./media/mysql-database-ssl-connection/ssl-002.png
[3]: ./media/mysql-database-ssl-connection/ssl-003.png
[4]: ./media/mysql-database-ssl-connection/ssl-004.png
[5]: ./media/mysql-database-ssl-connection/ssl-005.png
[6]: ./media/mysql-database-ssl-connection/ssl-006.png

<!--HONumber=81-->