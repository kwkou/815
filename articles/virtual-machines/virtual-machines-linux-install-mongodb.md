<properties
    pageTitle="使用 Azure CLI 1.0 在 Linux VM 上安装 MongoDB | Azure"
    description="了解如何使用 Resource Manager 部署模型在 Azure 中的 Linux 虚拟机上安装和配置 MongoDB。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3f55b546-86df-4442-9ef4-8a25fae7b96e"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/14/2017"
    wacn.date="04/06/2017"
    ms.author="iainfou" />  


# 如何使用 Azure CLI 1.0 在 Linux VM 上安装和配置 MongoDB
[MongoDB](http://www.mongodb.org) 是一个流行的开源、高性能 NoSQL 数据库。本文说明如何使用 Resource Manager 部署模型在 Azure 中的 Linux VM 上安装和配置 MongoDB。文中提供了一些示例，详细说明如何执行以下操作：

* [手动安装和配置基本的 MongoDB 实例](#manually-install-and-configure-mongodb-on-a-vm)
* [使用 Resource Manager 模板创建基本的 MongoDB 实例](#create-basic-mongodb-instance-on-centos-using-a-template)
* [使用 Resource Manager 模板创建包含副本集的复杂 MongoDB 分片群集](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template)

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- Azure CLI 1.0 - 用于经典部署模型和资源管理部署模型（本文）的 CLI
- Azure CLI 2.0 - 不支持 Azure 中国区的虚拟机。

## <a name="manually-install-and-configure-mongodb-on-a-vm"></a> 在 VM 上手动安装和配置 MongoDB
MongoDB 为 Red Hat/CentOS、SUSE、Ubuntu 和 Debian 等 Linux 分发版[提供了安装说明](https://docs.mongodb.com/manual/administration/install-on-linux/)。以下示例使用 `~/.ssh/id_rsa.pub` 中存储的 SSH 密钥创建 `CentOS` VM。出现提供存储帐户名称、DNS 名称和管理员凭据的提示时，请输入所需的信息：

    azure vm quick-create --ssh-publickey-file ~/.ssh/id_rsa.pub --image-urn CentOS

使用上述 VM 创建步骤结束时显示的公共 IP 地址登录到 VM：

    ssh azureuser@40.78.23.145

若要为 MongoDB 添加安装源，请按如下所示创建一个 `yum` 存储库文件：

    sudo touch /etc/yum.repos.d/mongodb-org-3.2.repo

打开该 MongoDB 存储库文件进行编辑。添加以下行：

    [mongodb-org-3.2]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc

按如下所示使用 `yum` 安装 MongoDB：

    sudo yum install -y mongodb-org

默认情况下，CentOS 映像中强制实施 SELinux，会阻止你访问 MongoDB。安装策略管理工具并配置 SELinux 可让 MongoDB 在其默认 TCP 端口 27017 上运行，如下所示。

    sudo yum install -y policycoreutils-python
    sudo semanage port -a -t mongod_port_t -p tcp 27017

按如下所示启动 MongoDB 服务：

    sudo service mongod start

通过使用本地 `mongo` 客户端进行连接来验证 MongoDB 安装：

    mongo

现在，通过添加一些数据并执行搜索来测试 MongoDB 实例：

    > db
    test
    > db.foo.insert( { a : 1 } )  
    > db.foo.find()  
    { "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
    > exit

如果需要，可将 MongoDB 配置为在系统重新引导期间自动启动：

    sudo chkconfig mongod on

## <a name="create-basic-mongodb-instance-on-centos-using-a-template"></a> 使用模板在 CentOS 上创建基本的 MongoDB 实例
可以使用 Github 中的以下 Azure 快速入门模板，在单个 CentOS VM 上创建基本的 MongoDB 实例。此模板使用适用于 Linux 的自定义脚本扩展将 `yum` 存储库添加到新建的 CentOS VM，然后安装 MongoDB。

* [CentOS 上的基本 MongoDB 实例](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

以下示例在 `ChinaNorth` 区域中创建名为 `myResourceGroup` 的资源组。按如下所示输入自己的值：

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

    azure group create --name myResourceGroup --location ChinaNorth \
        --template-file /path/to/azuredeploy.json

> [AZURE.NOTE]
创建部署后的几秒内，Azure CLI 会返回到提示符，但安装和配置需要几分钟才能完成。使用 `azure group deployment show myResourceGroup` 检查部署状态，并相应地输入资源组的名称。等到 `ProvisioningState` 显示“成功”，然后尝试通过 SSH 连接到 VM。
> 
> 

完成部署后，通过 SSH 连接到 VM。按以下示例中所示，使用 `azure vm show` 命令获取 VM 的 IP 地址：

    azure vm show --resource-group myResourceGroup --name myLinuxVM

在靠近输出的末尾处显示了 `Public IP address`。使用 VM 的 IP 地址通过 SSH 连接到 VM：

    ssh azureuser@138.91.149.74

如下所示，通过使用本地 `mongo` 客户端进行连接来验证 MongoDB 安装：

    mongo

现在，按如下所示，通过添加一些数据并执行搜索来测试实例：

    > db
    test
    > db.foo.insert( { a : 1 } )  
    > db.foo.find()  
    { "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
    > exit

## <a name="create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template"></a> 使用模板在 CentOS 上创建复杂的 MongoDB 分片群集
可以使用 Github 中的以下 Azure 快速入门模板创建复杂的 MongoDB 分片群集。此模板遵循 [MongoDB 分片群集最佳实践](https://docs.mongodb.com/manual/core/sharded-cluster-components/)来提供冗余和高可用性。该模板将创建两个分片，其中每个副本集包含三个节点。另外，它还会创建一个包含三个节点的配置服务器副本集，以及两个 `mongos` 路由器服务器（用于对各个分片中的应用程序提供一致性）。

* [CentOS 上的 MongoDB 分片群集](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json

> [AZURE.WARNING]
部署这种复杂 MongoDB 分片群集需要 20 个以上的核心（每个区域中一个订阅的默认核心计数通常为 20 个）。请提出 Azure 支持请求，以增加核心计数。
> 
> 

以下示例在 `ChinaNorth` 区域中创建名为 `myResourceGroup` 的资源组。按如下所示输入自己的值：

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

    azure group create --name myResourceGroup --location ChinaNorth \
        --template-file /path/to/azuredeploy.json

> [AZURE.NOTE]
创建部署后的几秒内，Azure CLI 会返回到提示符，但安装和配置可能需要一小时以上才能完成。使用 `azure group deployment show myResourceGroup` 检查部署状态，并相应地调整资源组的名称。等到 `ProvisioningState` 显示“成功”，然后连接到 VM。
> 
> 

## 后续步骤
在上述示例中，你已在本地从 VM 连接到 MongoDB 实例。如果想要从另一个 VM 或网络连接到 MongoDB 实例，请确保[创建相应的网络安全组规则](/documentation/articles/virtual-machines-linux-nsg-quickstart/)。

有关使用模板创建这些规则的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

Azure Resource Manager 模板使用自定义脚本扩展在 VM 上下载并执行脚本。有关详细信息，请参阅[在 Linux 虚拟机上使用 Azure 自定义脚本扩展](/documentation/articles/virtual-machines-linux-extensions-customscript/)。

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: update meta data-->