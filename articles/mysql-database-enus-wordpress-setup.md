<properties linkid="" urlDisplayName="" pageTitle="Use MySQL Database on Azure to configure WordPress websites – Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, Wordpress, website configuration, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="WordPress is a very widely used CMS. This article explains how to use MySQL Database on Azure and Azure Web Apps to install and configure WordPress." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor=""/>

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-wordpress-setup/)
- [English](/documentation/articles/mysql-database-enus-wordpress-setup/)

# Use MySQL Database on Azure to configure WordPress websites

WordPress is a very widely used CMS. This article explains how to use MySQL Database on Azure and Azure Web Apps to install and configure WordPress.

## Step 1: Download the latest version of the WordPress installation package  

Download the latest version of the installation package to your local storage directory from [the official WordPress website](https://wordpress.org/download). You can then extract it to this directory.

## Step 2: Create a MySQL Database on the Azure server  

Use the [Azure portal](https://manage.windowsazure.cn) to create your MySQL server.

![Creating MySQL Servers][1]

> **Ensure** that the allowed services are already set to “Yes” in your “configuration”; otherwise, this may cause the deployment to fail. The image below shows that allowed services are set to “Yes” by default.

![Allowed Services][2]


### Create a new MySQL database for WordPress to use (essential)  

Switch the workspace to “database,” and click “Create” at the bottom of the screen to create the new database.

![Creating New Databases][3]

> [AZURE.NOTE]Users must manually create the MySQL database in the Azure portal. The WordPress deployment will fail if you miss this step.**

## Step 3: Create a new web app  

See [How to create and deploy web apps](/documentation/articles/web-sites-php-web-site-gallery/) for details of this process.

![Creating New Web Apps][4]

### Deploy the WordPress installation files  

Next, deploy all the WordPress installation files you uncompressed in Step 1 to the new web app. At this point, we recommend that you use Git to complete the deployment process. Refer to [Publishing to Azure websites from source control](/documentation/articles/web-sites-publish-source-control/) for the specific method.

> [AZURE.NOTE]Step 3 is chiefly intended for new WordPress users. Some of you have already used Azure Web Apps and successfully deployed the WordPress installation files and begun using them. To keep the content consistent, we recommend that you export the data and then reimport it after you reinstall and deploy WordPress and successfully connect to a new MySQL database. Complete the specific WordPress data import and export procedures by using the WordPress control panel (as shown in the image below).

![Importing and Exporting With WordPress][9]

## Step 4: Install and configure WordPress

### Access and deploy the web app  

In order to access the web app you created earlier, go to the WordPress configuration page and select the required language. The content in the image below will then appear on the screen.

![Configuring WordPress][5]

### Connect and configure the MySQL database  

Enter the configuration details for the database created in Step 2 into the interface shown in the image below.

![Connecting to WordPress][6]

If you see what appears in the image below, WordPress is now connected to MySQL.

![Successfully Connected to WordPress][7]

Click to proceed with the installation, and finish deploying WordPress.


## Step 5: Use WordPress  

Once you have completed the process above, if you access the web app that you created earlier, you will find that the main WordPress interface now works. You are ready to start using WordPress.

![Installing WordPress][8]



<!--Image references-->
[1]: ./media/mysql-database-wordpress-setup/001-en.png
[2]: ./media/mysql-database-wordpress-setup/002-en.png
[3]: ./media/mysql-database-wordpress-setup/003-en.png
[4]: ./media/mysql-database-wordpress-setup/004-en.png
[5]: ./media/mysql-database-wordpress-setup/005-en.png
[6]: ./media/mysql-database-wordpress-setup/006-en.png
[7]: ./media/mysql-database-wordpress-setup/007-en.png
[8]: ./media/mysql-database-wordpress-setup/008-en.png
[9]: ./media/mysql-database-wordpress-setup/009-en.png
<!--HONumber=81-->