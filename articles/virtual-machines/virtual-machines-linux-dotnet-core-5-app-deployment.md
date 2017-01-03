<properties
   pageTitle="使用虚拟机扩展自动化应用程序部署 | Azure"
   description="Azure 虚拟机 DotNet Core 教程"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines"
   authors="neilpeterson"
   manager="timlt"
   editor="tysonn"
   tags="azure-service-management"/>  


<tags
   ms.service="virtual-machines-linux"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="infrastructure"
   ms.date="11/21/2016"
   wacn.date="12/30/2016"
   ms.author="nepeters"/>  


# 使用 Azure Resource Manager 模板部署应用程序

确定所有 Azure 基础结构要求并转换成部署模板后，需要解决实际的应用程序部署需求。此处的应用程序部署是指将实际应用程序二进制文件安装到 Azure 资源上。以音乐应用商店示例而言，需要在每个虚拟机上安装并配置 .Net Core、NGINX 和监督程序。需要将音乐应用商店二进制文件安装到虚拟机上，并预先创建音乐应用商店数据库。

本文档详细说明虚拟机扩展如何自动化 Azure 虚拟机的应用程序部署和配置。所有依赖项和独特配置都已突出显示。为了获得最佳体验，请将一个解决方案实例预先部署到 Azure 订阅，然后将它与 Azure Resource Manager 模板配合运行。可通过以下链接找到完整模板 – [Ubuntu 上的音乐应用商店部署](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux)。

## 配置脚本

虚拟机扩展是针对虚拟机执行的专用程序，可提供配置自动化。许多特定的目的（例如防病毒、日志记录配置和 Docker 配置）都有相应的扩展。使用自定义脚本扩展可对虚拟机运行任何脚本。在音乐应用商店示例中，需依赖自定义脚本扩展来配置 Ubuntu 虚拟机以及安装音乐应用商店应用程序。

在详细说明如何在 Azure Resource Manager 模板中声明虚拟机扩展之前，我们先了解所要运行的脚本。此脚本配置用于托管音乐应用商店应用程序的 Ubuntu 虚拟机。在运行时，该脚本将安装全部所需的软件、从源代码管理安装音乐应用商店应用程序，以及准备数据库。

有关如何在 Linux 上托管 .Net Core 应用程序的详细信息，请参阅 [Publish to a Linux Production Environment](https://docs.asp.net/en/latest/publishing/linuxproduction.html)（发布到 Linux 生产环境）。

    #!/bin/bash

    # install dotnet core
    sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ trusty main" > /etc/apt/sources.list.d/dotnetdev.list'
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    sudo apt-get update
    sudo apt-get install -y dotnet-dev-1.0.0-preview2-003121

    # download application
    sudo wget https://raw.github.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/music-app/music-store-azure-demo-pub.tar /
    sudo mkdir /opt/music
    sudo tar -xf music-store-azure-demo-pub.tar -C /opt/music

    # install nginx, update config file
    sudo apt-get install -y nginx
    sudo service nginx start
    sudo touch /etc/nginx/sites-available/default
    sudo wget https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/music-app/nginx-config/default -O /etc/nginx/sites-available/default
    sudo cp /opt/music/nginx-config/default /etc/nginx/sites-available/
    sudo nginx -s reload

    # update and secure music config file
    sed -i "s/<replaceserver>/$1/g" /opt/music/config.json
    sed -i "s/<replaceuser>/$2/g" /opt/music/config.json
    sed -i "s/<replacepass>/$3/g" /opt/music/config.json
    sudo chown $2 /opt/music/config.json
    sudo chmod 0400 /opt/music/config.json

    # config supervisor
    sudo apt-get install -y supervisor
    sudo touch /etc/supervisor/conf.d/music.conf
    sudo wget https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/music-app/supervisor/music.conf -O /etc/supervisor/conf.d/music.conf
    sudo service supervisor stop
    sudo service supervisor start

    # pre-create music store database
    /usr/bin/dotnet /opt/music/MusicStore.dll &

## VM 脚本扩展

通过将扩展资源包含在 Azure Resource Manager 模板中，可在构建时对虚拟机运行 VM 扩展。可以通过使用 Visual Studio 中的“添加新资源”向导或者在模板中插入有效的 JSON 来添加该扩展。脚本扩展资源嵌套在虚拟机资源中，如以下示例中所示。

单击以下链接可查看 Resource Manager 模板中的 JSON 示例 – [VM 脚本扩展](https://github.com/Microsoft/dotnet-core-sample-templates/blob/master/dotnet-core-music-linux/azuredeploy.json#L359)。

请注意，在以下 JSON 中，脚本存储在 GitHub 中。此脚本也可以存储在 Azure Blob 存储中。此外，Azure Resource Manager 模板允许构建脚本执行字符串，使模板参数值可用作脚本执行参数。在本例中，数据是在部署模板时提供的，然后，可在执行脚本时使用这些值。

    {
      "apiVersion": "2015-06-15",
      "type": "extensions",
      "name": "config-app",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
      ],
      "tags": {
        "displayName": "config-app"
      },
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "typeHandlerVersion": "2.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "fileUris": [
            "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
          ]
        },
        "protectedSettings": {
          "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
        }
      }
    }

有关使用自定义脚本扩展的详细信息，请参阅 [Custom script extensions with Resource Manager templates](/documentation/articles/virtual-machines-linux-extensions-customscript/)（使用 Resource Manager 模板自定义脚本扩展）。

## 后续步骤

<hr>

[探索更多 Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates)

<!---HONumber=Mooncake_1114_2016-->