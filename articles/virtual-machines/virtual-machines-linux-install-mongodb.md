<properties
    pageTitle="使用 Azure CLI 2.0（预览版）在 Linux VM 上安装 MongoDB | Azure"
    description="了解如何使用 Azure CLI 2.0（预览版）在 Linux 虚拟机上安装和配置 MongoDB"
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
    wacn.date="04/10/2017"
    ms.author="iainfou" />

# 如何使用 Azure CLI 2.0（预览版）在 Linux VM 上安装和配置 MongoDB
[MongoDB](http://www.mongodb.org) 是一个流行的开源、高性能 NoSQL 数据库。本文说明如何使用 Resource Manager 部署模型在 Azure 中的 Linux VM 上安装和配置 MongoDB。文中提供了一些示例，详细说明如何执行以下操作：

* [手动安装和配置基本的 MongoDB 实例](#manually-install-and-configure-mongodb-on-a-vm)
* [使用 Resource Manager 模板创建基本的 MongoDB 实例](#create-basic-mongodb-instance-on-centos-using-a-template)
* [使用 Resource Manager 模板创建包含副本集的复杂 MongoDB 分片群集](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template)

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-install-mongodb-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- Azure CLI 2.0（预览版）- 下一代 CLI，适用于本文介绍的资源管理部署模型。

## <a name="manually-install-and-configure-mongodb-on-a-vm"></a> 在 VM 上手动安装和配置 MongoDB
MongoDB 为 Red Hat/CentOS、SUSE、Ubuntu 和 Debian 等 Linux 分发版[提供了安装说明](https://docs.mongodb.com/manual/administration/install-on-linux/)。以下示例使用 `~/.ssh/id_rsa.pub` 中存储的 SSH 密钥创建 `CentOS` VM。若要创建此环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `China North` 位置创建名为 `myResourceGroup` 的资源组：

     az group create --name myResourceGroup --location chinanorth

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。以下示例使用 SSH 公钥身份验证和 `mypublicdns` 的公共 DNS 条目，创建名为 `myVM` 的 VM 和名为 `azureuser` 的用户：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image CentOS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --public-ip-address-dns-name mypublicdns

使用 VM 的公共 DNS 地址登录到 VM。可以使用 [az vm show](https://docs.microsoft.com/cli/azure/vm#show) 查看公共 DNS 地址：

    az vm show -g myResourceGroup -n myVM -d --query [fqdns] -o tsv

使用自己的用户名和公共 DNS 地址通过 SSH 连接到 VM：

    ssh azureuser@mypublicdns.chinanorth.chinacloudapp.cn

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

若要创建此环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。首先，使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `China North` 位置创建一个名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

接下来，使用 [az group deployment create](https://docs.microsoft.com/cli/azure/group/deployment#create) 部署 MongoDB 模板。根据需要定义自己的资源名称和大小，例如 `newStorageAccountName`、`virtualNetworkName` 和 `vmSize`：

    az group deployment create --resource-group myResourceGroup \
      --parameters '{"newStorageAccountName": {"value": "mystorageaccount"},
        "adminUsername": {"value": "azureuser"},
        "adminPassword": {"value": "P@ssw0rd!"},
        "dnsNameForPublicIP": {"value": "mypublicdns"},
        "virtualNetworkName": {"value": "myVnet"},
        "vmSize": {"value": "Standard_DS1_v2"},
        "vmName": {"value": "myVM"},
        "publicIPAddressName": {"value": "myPublicIP"},
        "nicName": {"value": "myNic"}}' \
      --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

使用 VM 的公共 DNS 地址登录到 VM。可以使用 [az vm show](https://docs.microsoft.com/cli/azure/vm#show) 查看公共 DNS 地址：

    az vm show -g myResourceGroup -n myVM -d --query [fqdns] -o tsv

使用自己的用户名和公共 DNS 地址通过 SSH 连接到 VM：

    ssh azureuser@mypublicdns.chinanorth.chinacloudapp.cn

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

若要创建此环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。首先，使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `China North` 位置创建一个名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

接下来，使用 [az group deployment create](https://docs.microsoft.com/cli/azure/group/deployment#create) 部署 MongoDB 模板。根据需要定义自己的资源名称和大小，例如 `mongoAdminUsername`、`sizeOfDataDiskInGB` 和 `configNodeVmSize`：

    az group deployment create --resource-group myResourceGroup \
      --parameters '{"adminUsername": {"value": "azureuser"},
        "adminPassword": {"value": "P@ssw0rd!"},
        "mongoAdminUsername": {"value": "mongoadmin"},
        "mongoAdminPassword": {"value": "P@ssw0rd!"},
        "dnsNamePrefix": {"value": "mypublicdns"},
        "environment": {"value": "AzureCloud"},
        "numDataDisks": {"value": "4"},
        "sizeOfDataDiskInGB": {"value": 20},
        "centOsVersion": {"value": "7.0"},
        "routerNodeVmSize": {"value": "Standard_DS3_v2"},
        "configNodeVmSize": {"value": "Standard_DS3_v2"},
        "replicaNodeVmSize": {"value": "Standard_DS3_v2"},
        "zabbixServerIPAddress": {"value": "Null"}}' \
      --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json \
      --name myMongoDBCluster --no-wait

此部署可能需要花费一个多小时来部署和配置所有 VM 实例。上述命令的末尾使用了 `--no-wait` 标志，该标志可在 Azure 平台接受模板部署后将控制权返回给命令提示符。然后，可以使用 [az group deployment show](https://docs.microsoft.com/cli/azure/group/deployment#show) 查看部署状态。以下示例演示如何查看 `myResourceGroup` 资源组中 `myMongoDBCluster` 部署的状态：

    az group deployment show --resource-group myResourceGroup --name myMongoDBCluster \
        --query [properties.provisioningState] --output tsv

## 后续步骤
在上述示例中，你已在本地从 VM 连接到 MongoDB 实例。如果想要从另一个 VM 或网络连接到 MongoDB 实例，请确保[创建相应的网络安全组规则](/documentation/articles/virtual-machines-linux-nsg-quickstart/)。

有关使用模板创建这些规则的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

Azure Resource Manager 模板使用自定义脚本扩展在 VM 上下载并执行脚本。有关详细信息，请参阅[在 Linux 虚拟机上使用 Azure 自定义脚本扩展](/documentation/articles/virtual-machines-linux-extensions-customscript/)。

<!---HONumber=Mooncake_0320_2017-->