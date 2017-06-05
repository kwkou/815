<properties
    linkid=""
    urlDisplayName=""
    pageTitle="Use Azure Resource Manager templates to deploy MySQL PaaS – Azure"
    metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, Resource Manager, ARM, ARM Template, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS"
    description="This article describes how to use Azure Resource Manager templates to rapidly deploy MySQL PaaS and other popular apps."
    metaCanonical=""
    services="MySQL"
    documentationCenter="Services"
    authors="v-chenyh"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="mysql"
    ms.author="v-chenyh"
    ms.topic="article"
    ms.date="03/14/2017"
    wacn.date="03/14/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-armtemplate-deploymysql/)
- [English](/documentation/articles/mysql-database-enus-armtemplate-deploymysql/)

#<a name="azuremysql-paas"></a>Use Azure Resource Manager templates to deploy MySQL PaaS


This article describes how to use Azure Resource Manager templates to rapidly deploy MySQL PaaS and other popular apps.

##<a name="azure"></a>Understanding Azure Resource Manager templates

A Resource Manager template is a JavaScript Object Notation (JSON) file that defines one or several resources that are to be deployed to resource groups. It also defines the dependencies among the resources being deployed. Using templates allows you to repeatedly deploy resources in a consistent manner. For more information about Resource Manager and Resource Manager templates, see [Azure Resource Manager Overview](https://docs.microsoft.com/zh-cn/azure/azure-resource-manager/resource-group-overview).

##<a name="mysql-paas"></a>Use Resource Manager templates to deploy MySQL PaaS

This section describes how to use PowerShell with Resource Manager templates to deploy MySQL PaaS on Azure. Your template may be a local file, or an external file that is accessed via a URI. If the template resides in a storage account, you can restrict access to the template and provide a Shared Access Signature (SAS) token during the deployment process.

Use the following PowerShell commands to get started quickly with your deployment (this example uses PowerShell 1.0.0 or later):

    New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "chinaeast"

    New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\example.json

These commands create a resource group and deploy the template to the resource group. The template file is a local file. If this operation is successful, you will be able to obtain all the required deployed resources.

The following code is a fairly complete Resource Manager template in JSON format. You can modify this template to meet your specific requirements. You can modify this template to meet your specific requirements.

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "mysqlServerName": {
          "type": "string",
          "metadata": {
            "description": "MySql 服务器的名字. 最大长度为64个字母数字字符. 请通过 @mysqlServerName.mysqldb.chinacloudapi.cn访问MySql数据库. 管理配置MySql数据库请访问 https://manage.windowsazure.cn"
          }
        },
        "mysqlServerSku": {
          "type": "string",
          "defaultValue": "MS2",
          "allowedValues": [
            "MS1", "MS2", "MS3", "MS4", "MS5", "MS6"
          ],
          "metadata": {
            "description": "MySql 服务器的性能版本，价格详情请访问 https://www.azure.cn/pricing/details/mysql/"
          }
        },
        "mysqlServerVersion": {
          "type": "string",
          "allowedValues": [
            "5.6", "5.7", "5.5"
           ],
          "defaultValue": "5.6",
          "metadata": {
            "description": "MySQL服务器版本. "
          }
        },
        "mysqlUserName": {
          "type": "string",
          "metadata": {
            "description": "MySql 服务器登录名。登录名格式为 ‘MySQL服务器名%登录名’，长度小于16个字符。用户名和密码规范请参考： http://dev.mysql.com/doc/refman/5.6/en/user-names.html"
          }
        },
        "mysqlPassword": {
          "type": "securestring",
          "metadata": {
            "description": "MySql 服务器登录密码。长度必须大于8个字符，小于64个字符。用户名和密码规范请参考： http://dev.mysql.com/doc/refman/5.6/en/user-names.html"
          }
        },
        "mysqldatabaseName": {
          "type": "string",
          "metadata": {
            "description": "数据库名字。必须小于64个字符，其它命名规范请参考: http://dev.mysql.com/doc/refman/5.6/en/identifiers.html"
          }
        }
      },

      "resources": [   
        {
          "type": "Microsoft.MySql/servers",
          "name": "[parameters('mysqlServerName')]",
          "apiVersion": "2015-09-01",
          "location": "[resourceGroup().location]",
          "tags": {
            "displayName": "[parameters('mysqlServerName')]"
          },
          "sku": { "name": "[parameters('mysqlServerSku')]" },
          "properties": {
            "version": "[parameters('mysqlServerVersion')]",
            "AllowAzureServices": true
          },
          "resources": [
            {
              "name": "[parameters('mysqlUserName')]",
              "type": "users",
              "apiVersion": "2015-09-01",
              "dependsOn": [
                "[concat('Microsoft.MySql/servers/', parameters('mysqlServerName'))]"
              ],
              "properties": {
                "password": "[parameters('mysqlPassword')]"
              }
            },
            {
              "name": "[parameters('mysqldatabaseName')]",
              "type": "databases",
              "tags": {
                "displayName": "[parameters('mysqldatabaseName')]"
              },
              "apiVersion": "2015-09-01",
              "dependsOn": [
                "[concat('Microsoft.MySql/servers/', parameters('mysqlServerName'))]"
              ],
              "properties": {
               "Charset": "utf8",
               "Collation": "utf8_general_ci"
              },
              "resources": [
                {
                  "name": "[parameters('mysqlUserName')]",
                  "type": "privileges",
                  "apiVersion": "2015-09-01",
                  "dependsOn": [
                    "[concat('Microsoft.MySql/servers/', parameters('mysqlServerName'), '/users/', parameters('mysqlUserName'))]",
                    "[concat('Microsoft.MySql/servers/', parameters('mysqlServerName'), '/databases/', parameters('mysqldatabaseName'))]"
                  ],
                  "properties": {
                    "level": "ReadWrite"
                  }
                }
              ]
            }
          ]
        }
      ]
    }

##<a name="arm-templateazure"></a>Publish an Azure Resource Manager template on the Azure Marketplace

You can publish a customized Resource Manager template on the Azure Marketplace to enable users to implement one-click deployment for all cloud services. For detailed instructions on how to publish an ARM template to the Azure Marketplace, see the [Azure Marketplace Publisher Guide](https://market.azure.cn/Documentation/article/publishguide/).

The Marketplace is a warehouse used by third-party independent software vendors (ISVs) to publish mirror-image software. The Marketplace enables paying Azure users to create new virtual machine instances, and to download and use the third-party images that they need. This function helps companies looking for innovative cloud-based solutions to get in direct contact with Azure ISV partners that develop innovative solutions.

The Marketplace makes it easier to find, obtain, and use apps built and thoroughly tested on the Azure platform, allowing you to be certain that the app can be trusted and runs smoothly on Azure.

The Marketplace brings new business channels and users to ISVs, while also greatly simplifying the promotion, management, and deployment of software. We strongly recommend that you use mirror images to deploy relevant apps. With just a few clicks, you can rapidly obtain system environments or software identical to the mirror image. This means that you don’t need to worry about infrastructure and environment configuration. You can conveniently create ready-to-use runtime environments for every platform.

##<a name=""></a>Star products on the Marketplace

![](./media/mysql-database-armtemplate-deploymysql/fuwang.png)

###<a name="lnmp-w-mysql-paas-10"></a>Servinet LNMP Cluster with MySQL PaaS-1.0

The LNMP provided by Shanghai Servinet relies on the latest Azure Resource Manager architecture, and provides full cluster deployment (default dual-server deployment). All the virtual machines have been added to high-availability groups, and are automatically assigned to different Nginx 1.10.1. The framework is suitable for use with Nginx front-end and back-end separation (active-passive separation).

[More information](https://market.azure.cn/Vhd/Show?vhdId=12005&version=14150)

[Immediate deployment](https://market.azure.cn/VM/Launch?vhdId=12005&version=14150) ([requires sign-in for the Azure Marketplace)](https://market.azure.cn/Sign/Login?url=%2fVhd%2fShow%3fvhdId%3d12005%26version%3d14150)

![](./media/mysql-database-armtemplate-deploymysql/wordpress.png)

###<a name="wordpress-461web--mysql-paas"></a>WordPress-4.6.1 (web app + MySQL PaaS)

WordPress is a very widely used content management system. This app was created with a Resource Manager template. By using this template, you can quickly create web apps and MySQL databases, and deploy WordPress website source code. You can perform subsequent maintenance on websites and databases via the Management Portal for data security, high availability, and more convenient management.

[More information](https://market.azure.cn/Vhd/Show?vhdId=12006&version=14125)

[Immediate deployment](https://market.azure.cn/VM/Launch?vhdId=12006&version=14125) ([requires sign-in for the Azure Marketplace)](https://market.azure.cn/Sign/Login?url=%2fVhd%2fShow%3fvhdId%3d12006%26version%3d14125)

![](./media/mysql-database-armtemplate-deploymysql/zabbix.png)

###<a name="zabbix-222ubuntu-1404-ltsopenlogic-72"></a>Zabbix 2.2.2 (Ubuntu 14.04 LTS/OpenLogic 7.2)

Zabbix (http://www.zabbix.com) is a free, enterprise class, open-source network monitoring tool. It can effectively monitor the status of Windows, Linux, and Unix servers, switches, routers, and other network devices, as well as devices such as printers.

[More information](https://market.azure.cn/Vhd/Show?vhdId=12009&version=14123)

[Immediate deployment](https://market.azure.cn/VM/Launch?vhdId=12009&version=14123) ([requires sign-in for the Azure Marketplace)](https://market.azure.cn/Sign/Login?url=%2fVhd%2fShow%3fvhdId%3d12009%26version%3d14123)

<!--HONumber=May17_HO3-->