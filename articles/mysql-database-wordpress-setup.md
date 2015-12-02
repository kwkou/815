<properties linkid="" urlDisplayName="" pageTitle="How to Use MySQL Database on Azure to Configure WordPress Websites – Microsoft Azure Cloud" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，WordPress, 网站配置,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="WordPress is a very widely used CMS system. This article explains how to use MySQL Database on Azure and Azure Web apps to install and configure WordPress." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor=""/>

<tags ms.service="mysql" ms.date="" wacn.date="05/21/2015"/>

# How to Use MySQL Database on Azure to Configure WordPress Websites

WordPress is a very widely used CMS system. This article explains how to use MySQL Database on Azure and Azure Web apps to install and configure WordPress.

## Step 1: Download the latest version of the WordPress installation package  

You can download the latest version of the installation package to your local storage directory from [the official WordPress website](https://wordpress.org/download). You can then extract it to this directory.

## Step 2: Create a MySQL Database on Azure server  

You can use the [Microsoft Azure management portal](https://manage.windowsazure.cn) to create your MySQL server.

![Creating MySQL Servers][1]

> **Ensure** that the allowed services are already set to “yes” in your “configuration,” otherwise this may cause the deployment to fail. This is shown in the image below. (Set to “yes” by default)

![Allowed Services][2]


### Creating a new MySQL database for WordPress to use (essential)  

Switch the workspace to “database,” and click on the “create” button at the bottom of the screen to create the new database.

![Creating New Databases][3]

>[AZURE.NOTE] **Note:** Users must manually create the MySQL database in the management portal. The WordPress deployment will fail if you miss this step.

## Step 3: Create a new Web app  

See [How to Create and Deploy Web Apps](/documentation/articles/web-sites-php-web-site-gallery) for details of this process.

![Creating New Web Apps][4]

### Deploying the WordPress installation files  

Next, deploy all the WordPress installation files you uncompressed in Step 1 to the new Web app. At this point, we recommend using Git to complete the deployment process. Please refer to [Publishing to Azure Websites From Source Control](/documentation/articles/web-sites-publish-source-control) for the specific method.

>[AZURE.NOTE] **Note:** Step 3 is chiefly intended for new WordPress users. If you already use Azure Web apps and have successfully deployed the WordPress installation files and begun using them, in order to keep the content consistent, we recommend that you export the data and then reimport it after you have reinstalled and deployed WordPress and successfully connected to a new MySQL database. Please complete the specific WordPress data import and export procedures using the WordPress control panel (as shown in the image below).

![Importing and Exporting With WordPress][9]

## Step 4: Install and configure WordPress

### 1. Access and deploy the Web app  

In order to access the Web app you created earlier, go to the WordPress configuration page, and select the required language; the content shown in the image below will then appear on the screen.

![Configuring WordPress][5]

### 2. Connect and configure the MySQL database  

Enter the configuration details for the database created in Step 2 into the interface shown in the image below.

![Connecting to WordPress][6]

If you see what is shown in the image below, this indicates that your WordPress is now connected to MySQL.

![Successfully Connected to WordPress][7]

Click to proceed with the installation, and finish deploying your WordPress.


## Step 5: Using WordPress  

Once you have completed the process above, if you access the Web app you created earlier, you will find that the main WordPress interface is now working. Now you are ready to start your journey with WordPress!

![Installing WordPress][8]



<!--Image references-->
[1]: ./media/mysql-database-wordpress-setup/001.png
[2]: ./media/mysql-database-wordpress-setup/002.png
[3]: ./media/mysql-database-wordpress-setup/003.png
[4]: ./media/mysql-database-wordpress-setup/004.png
[5]: ./media/mysql-database-wordpress-setup/005.png
[6]: ./media/mysql-database-wordpress-setup/006.png
[7]: ./media/mysql-database-wordpress-setup/007.png
[8]: ./media/mysql-database-wordpress-setup/008.png
[9]: ./media/mysql-database-wordpress-setup/009.png