<properties
    pageTitle="使用 Azure CLI 1.0 在 Linux 虚拟机上部署 LAMP | Azure"
    description="了解如何在 Azure 中的 Linux VM 上安装 LAMP 堆栈"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="jluk"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="6c12603a-e391-4d3e-acce-442dd7ebb2fe"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="NA"
    ms.topic="article"
    ms.date="2/21/2017"
    wacn.date="04/17/2017"
    ms.author="juluk"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="442b5e509200e191d4defe2ab4743d99f308d4b0"
    ms.lasthandoff="04/06/2017" />

# <a name="deploy-lamp-stack-with-the-azure-cli-10"></a>使用 Azure CLI 1.0 部署 LAMP 堆栈
本文介绍如何在 Azure 上部署 Apache web 服务器、MySQL 和 PHP（LAMP 堆栈）。 需要一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）和[连接到 Azure 帐户](/documentation/articles/xplat-cli-connect/)的 [Azure CLI](/documentation/articles/cli-install-nodejs/)。

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- Azure CLI 1.0 - 适用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0](/documentation/articles/virtual-machines-linux-create-lamp-stack/) - 适用于资源管理部署模型的下一代 CLI

<br/>

    # One command to create a resource group holding a VM with LAMP already on it
    $ azure group create -n uniqueResourceGroup -l chinanorth --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json

* 在现有 VM 上部署 LAMP

        # Two commands: one updates packages, the other installs Apache, MySQL, and PHP
        user@ubuntu$ sudo apt-get update
        user@ubuntu$ sudo apt-get install apache2 mysql-server php5 php5-mysql

## <a name="deploy-lamp-on-new-vm-walkthrough"></a>在新的 VM 上部署 LAMP 的演练
可以首先创建一个包含新 VM 的[资源组](/documentation/articles/resource-group-overview/)：

    $ azure group create uniqueResourceGroup chinanorth
    info:    Executing command group create
    info:    Getting resource group uniqueResourceGroup
    info:    Creating resource group uniqueResourceGroup
    info:    Created resource group uniqueResourceGroup
    data:    Id:                  /subscriptions/########-####-####-####-############/resourceGroups/uniqueResourceGroup
    data:    Name:                uniqueResourceGroup
    data:    Location:            chinanorth
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK

若要创建 VM 本身，可以使用在 [GitHub 上的此处](https://github.com/Azure/azure-quickstart-templates/tree/master/lamp-app)找到的已编写好的 Azure Resource Manager 模板。

    $ azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json uniqueResourceGroup uniqueLampName

你会看到提示输入更多内容的响应：

    info:    Executing command group deployment create
    info:    Supply values for the following parameters
    storageAccountNamePrefix: lampprefix
    location: chinanorth
    adminUsername: someUsername
    adminPassword: somePassword
    mySqlPassword: somePassword
    dnsLabelPrefix: azlamptest
    info:    Initializing template configurations and parameters
    info:    Creating a deployment
    info:    Created template deployment "uniqueLampName"
    info:    Waiting for deployment to complete
    data:    DeploymentName     : uniqueLampName
    data:    ResourceGroupName  : uniqueResourceGroup
    data:    ProvisioningState  : Succeeded
    data:    Timestamp          :
    data:    Mode               : Incremental
    data:    CorrelationId      : d51bbf3c-88f1-4cf3-a8b3-942c6925f381
    data:    TemplateLink       : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json
    data:    ContentVersion     : 1.0.0.0
    data:    DeploymentParameters :
    data:    Name                      Type          Value
    data:    ------------------------  ------------  -----------
    data:    storageAccountNamePrefix  String        lampprefix
    data:    location                  String        chinanorth
    data:    adminUsername             String        someUsername
    data:    adminPassword             SecureString  undefined
    data:    mySqlPassword             SecureString  undefined
    data:    dnsLabelPrefix            String        azlamptest
    data:    ubuntuOSVersion           String        14.04.2-LTS
    info:    group deployment create command OK

现在你已创建已安装 LAMP 的 Linux VM。 可以根据需要跳转到[验证是否已成功安装 LAMP](#verify-lamp-successfully-installed) 来验证此安装。

## <a name="deploy-lamp-on-existing-vm-walkthrough"></a>在现有 VM 上部署 LAMP 的演练
如果需要有关创建 Linux VM 方面的帮助，可以转到[此处了解如何创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。 接下来，需通过 SSH 登录 Linux VM。 如果需要有关创建 SSH 密钥方面的帮助，可以转到[此处了解如何在 Linux/Mac 上创建 SSH 密钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。
如果已有 SSH 密钥，请继续操作，使用 `ssh exampleUsername@exampleDNS` 从命令行通过 SSH 登录 Linux VM。

现在你是在 Linux VM 中操作，我们可以指导你在基于 Debian 的分发版上安装 LAMP 堆栈。 对于其他 Linux 分发版，确切的命令可能会有所不同。

#### <a name="installing-on-debianubuntu"></a>在 Debian/Ubuntu 上安装
需要安装以下程序包：`apache2`、`mysql-server`、`php5`、`php5-mysql`。 可以直接使用这些包来安装，也可以使用 Tasksel 来安装。 下面列出了这两种方法的说明。
在安装之前，需要下载并更新包列表。

    user@ubuntu$ sudo apt-get update

##### <a name="individual-packages"></a>单个包
使用 apt-get：

    user@ubuntu$ sudo apt-get install apache2 mysql-server php5 php5-mysql

##### <a name="using-tasksel"></a>使用 tasksel
此外，你可以下载 Tasksel，它是一个 Debian/Ubuntu 工具，可将多个相关包作为协同“任务”安装到你的系统中。

    user@ubuntu$ sudo apt-get install tasksel
    user@ubuntu$ sudo tasksel install lamp-server

运行上述任一选项以后，系统会提示用户安装这些包以及各种其他的依赖项。 按“y”再按“Enter”即可继续，然后再遵循任何其他的提示为 MySQL 设置一个管理密码。 此时会安装最低要求的 PHP 扩展，这些扩展是通过 MySQL 使用 PHP 所必需的。 

![][1]

运行以下命令即可查看以程序包形式提供的其他 PHP 扩展：

    user@ubuntu$ apt-cache search php5

#### <a name="create-infophp-document"></a>创建 info.php 文档
现在你可以通过在命令行中键入 `apache2 -v`、`mysql -v` 或 `php -v` 来检查 Apache、MySQL 和 PHP 的版本。

如果你想要进行进一步的检测，可以创建在浏览器中查看的快速 PHP 信息页。 通过以下命令使用 Nano 文本编辑器创建一个文件：

    user@ubuntu$ sudo nano /var/www/html/info.php

在 GNU Nano 文本编辑器中，添加以下行：

    <?php
    phpinfo();
    ?>

然后保存并退出文本编辑器。

使用此命令重启 Apache，以便所有新安装都生效。

    user@ubuntu$ sudo service apache2 restart

## <a name="verify-lamp-successfully-installed"></a>验证是否已成功安装 LAMP
现在，用户可以查看所创建的 PHP 信息页，只需打开浏览器并转到 http://youruniqueDNS/info.php 即可。 应如下图所示。

![][2]

可以通过转到 http://youruniqueDNS/ 查看 Apache2 Ubuntu 默认页来检查 Apache 安装。 用户会看到类似下图的内容。

![][3]

恭喜，你刚刚在 Azure VM 上安装了 LAMP 堆栈！

## <a name="next-steps"></a>后续步骤
签出 LAMP 堆栈上的 Ubuntu 文档：

* [https://help.ubuntu.com/community/ApacheMySQLPHP](https://help.ubuntu.com/community/ApacheMySQLPHP)

[1]: ./media/virtual-machines-linux-deploy-lamp-stack/configmysqlpassword-small.png
[2]: ./media/virtual-machines-linux-deploy-lamp-stack/phpsuccesspage.png
[3]: ./media/virtual-machines-linux-deploy-lamp-stack/apachesuccesspage.png